## kubebuilder

### kubebuilder 初体验

```markdown
#  mkdir kubedev   
# cd kubedev      
# go mod init kubedev
go: creating new go.mod: module kubedev
#  kubebuilder init --domain zise.feizhu
Error: failed to initialize project: unable to run pre-scaffold tasks of "base.go.kubebuilder.io/v3": go version 'go1.17.6' is incompatible because 'requires 1.13 <= version < 1.17'. You can skip this check using the --skip-go-version-check flag 
```

这种情况下可以添加 --skip-go-version-check 忽略这个错误，但是还是建议使用官方推荐的版本

```markdown
# kubebuilder init --domain zise.feizhu --skip-go-version-check
Writing scaffold for you to edit...
Get controller runtime:
$ go get sigs.k8s.io/controller-runtime@v0.8.3
Update dependencies:
$ go mod tidy
Next: define a resource with:
$ kubebuilder create api
```

在当前目录下新增以下内容，可见这是个标准的go module工程：

```markdown
.
├── Dockerfile
├── Makefile                                                    # 这里定义了很多脚本命令，例如运行测试，开始执行等
├── PROJECT                                                     # 这里是 kubebuilder 的一些元数据信息
├── config                                                      # config目录下获得启动配置
│   ├── default                                           # 一些默认配置
│   │   ├── kustomization.yaml
│   │   ├── manager_auth_proxy_patch.yaml
│   │   └── manager_config_patch.yaml
│   ├── manager                                           # 部署 crd 所需的 yaml
│   │   ├── controller_manager_config.yaml
│   │   ├── kustomization.yaml
│   │   └── manager.yaml
│   ├── prometheus                                        # 监控指标数据采集配置
│   │   ├── kustomization.yaml
│   │   └── monitor.yaml
│   └── rbac                                              # 部署所需的 rbac 授权 yaml
│       ├── auth_proxy_client_clusterrole.yaml
│       ├── auth_proxy_role.yaml
│       ├── auth_proxy_role_binding.yaml
│       ├── auth_proxy_service.yaml
│       ├── kustomization.yaml
│       ├── leader_election_role.yaml
│       ├── leader_election_role_binding.yaml
│       ├── role_binding.yaml
│       └── service_account.yaml
├── go.mod                                                      # 一个新的Go模块，与我们的项目相匹配，具有基本的依赖项
├── go.sum
├── hack                                                        # 脚本
│   └── boilerplate.go.txt
└── main.go

6 directories, 24 files
```

#### 创建API(CRD和Controller)

```markdown
# kubebuilder create api --group apps --version v1 --kind Application
Create Resource [y/n]               # 是否创建资源
y
Create Controller [y/n]             # 是否创建控制器
y
```

执行之后我们可以发现项目结构发生了一些变化

```markdown
├── api
│   └── v1
│       ├── application_types.go                            # crd定义
│       ├── groupversion_info.go                            # 主要存放schema和我们的gvk的定义
│       └── zz_generated.deepcopy.go                        # 序列化和反序列化我们的crd资源。这个一般不需要关注
├── bin
│   └── controller-gen
├── config
│   ├── crd                                                      # 自动生成的 crd 文件，不用修改这里，只需要修改了 v1 中的 go 文件之后执行 make generate 即可
│   │   ├── kustomization.yaml
│   │   ├── kustomizeconfig.yaml
│   │   └── patches
│   │       ├── cainjection_in_applications.yaml
│   │       └── webhook_in_applications.yaml
│   ├── rbac
│   │   ├── application_editor_role.yaml
│   │   ├── application_viewer_role.yaml
│   │   ├── auth_proxy_client_clusterrole.yaml
│   │   ├── auth_proxy_role.yaml
│   │   ├── auth_proxy_role_binding.yaml
│   │   ├── auth_proxy_service.yaml
│   │   ├── kustomization.yaml
│   │   ├── leader_election_role.yaml
│   │   ├── leader_election_role_binding.yaml
│   │   ├── role_binding.yaml
│   │   └── service_account.yaml
│   └── samples                                            # 这里是 crd 示例文件，可以用来部署到集群当中
│       └── apps_v1_application.yaml
├── controllers
│   ├── application_controller.go                          # 自定义控制器主要实现cr的reconcile方法，通过拿到cr资源信息，我们真正业务逻辑存放的地方
│   └── suite_test.go                                      # 测试组件
    
13 directories, 37 files
```

#### 构建和部署CRD

kubebuilder提供的Makefile将构建和部署工作大幅度简化，执行以下命令会将最新构建的CRD部署在kubernetes上

```markdown
#  make install  
......
customresourcedefinition.apiextensions.k8s.io/applications.apps.zise.feizhu created
```

#### 编译和运行controller

以体验基本流程为主，不深入研究源码，所以对代码仅做少量修改，用于验证是否能生效
main.go

```markdown
if err = (&controllers.ApplicationReconciler{
		Client: mgr.GetClient(),
		Log:    ctrl.Log.WithName("controllers").WithName("Application"),                   // add this line
		Scheme: mgr.GetScheme(),
	}).SetupWithManager(mgr); err != nil {
		setupLog.Error(err, "unable to create controller", "controller", "Application")
		os.Exit(1)
	}
```

./controllers/application_controller.go

```markdown
type ApplicationReconciler struct {
	client.Client
	Log    logr.Logger
	Scheme *runtime.Scheme
}
func (r *ApplicationReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	_ = log.FromContext(ctx)
	_ = r.Log.WithValues("application",req.NamespacedName)
	// your logic here
	r.Log.Info("1. %v",req)  				// 打印入参
	r.Log.Info("2. %s",debug.Stack())		// 打印堆栈

	return ctrl.Result{}, nil
}
...
```

执行以下命令，会编译并启动刚才修改的controller：

```markdown
# make run
/Users/zisefeizhu/linkun/goproject/kubedev/bin/controller-gen "crd:trivialVersions=true,preserveUnknownFields=false" rbac:roleName=manager-role webhook paths="./..." output:crd:artifacts:config=config/crd/bases
main.go:35:2: package kubedev/controllers imports github.com/go-logr/logr from implicitly required module; to add missing requirements, run:
        go get github.com/go-logr/logr@v0.3.0
Error: not all generators ran successfully
run `controller-gen crd:trivialVersions=true,preserveUnknownFields=false rbac:roleName=manager-role webhook paths=./... output:crd:artifacts:config=config/crd/bases -w` to see all available markers, or `controller-gen crd:trivialVersions=true,preserveUnknownFields=false rbac:roleName=manager-role webhook paths=./... output:crd:artifacts:config=config/crd/bases -h` for usage
make: *** [manifests] Error 1
#  go get github.com/go-logr/logr@v0.3.0
# make run
```

#### 创建Application资源的实例

现在kubernetes已经部署了Application类型的CRD，而且对应的controller也已正在运行中，可以尝试创建Application类型的实例了(相当于有了pod的定义后，才可以创建pod)；

kubebuilder已经自动创建了一个类型的部署文件 ./config/samples/apps_v1_application.yaml ，内容如下，很简单，接下来就用这个文件来创建Application实例：

```markdown
apiVersion: apps.zise.feizhu/v1
kind: Application
metadata:
  name: application-sample
spec:
  # Add fields here
  foo: bar
执行 apply, 去controller所在控制台，可以看到新增和修改的操作都有日志输出，新增的日志都在里面，代码调用栈一目了然
make run            
/Users/zisefeizhu/linkun/goproject/kubedev/bin/controller-gen "crd:trivialVersions=true,preserveUnknownFields=false" rbac:roleName=manager-role webhook paths="./..." output:crd:artifacts:config=config/crd/bases
/Users/zisefeizhu/linkun/goproject/kubedev/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."
go fmt ./...
go vet ./...
go run ./main.go
I0210 15:26:27.993087   31662 request.go:655] Throttling request took 1.031351708s, request: GET:https://192.168.101.45:6443/apis/storage.k8s.io/v1beta1?timeout=32s
2022-02-10T15:26:29.262+0800    INFO    controller-runtime.metrics      metrics server is starting to listen    {"addr": ":8080"}
2022-02-10T15:26:29.263+0800    INFO    setup   starting manager
2022-02-10T15:26:29.263+0800    INFO    controller-runtime.manager      starting metrics server {"path": "/metrics"}
2022-02-10T15:26:29.263+0800    INFO    controller-runtime.manager.controller.application       Starting EventSource    {"reconciler group": "apps.zise.feizhu", "reconciler kind": "Application", "source": "kind source: /, Kind="}
2022-02-10T15:26:29.368+0800    INFO    controller-runtime.manager.controller.application       Starting Controller     {"reconciler group": "apps.zise.feizhu", "reconciler kind": "Application"}
2022-02-10T15:26:29.368+0800    INFO    controller-runtime.manager.controller.application       Starting workers        {"reconciler group": "apps.zise.feizhu", "reconciler kind": "Application", "worker count": 1}
2022-02-10T15:26:36.742+0800    INFO    controllers.Application 1. default/application-sample
2022-02-10T15:26:36.743+0800    INFO    controllers.Application 2. goroutine 420 [running]:
runtime/debug.Stack()
/Users/zisefeizhu/go/go1.17.6/src/runtime/debug/stack.go:24 +0x88
kubedev/controllers.(*ApplicationReconciler).Reconcile(0x140007443c0, {0x10399f3d8, 0x140006c5680}, {{{0x140003ac869, 0x7}, {0x1400071cc30, 0x12}}})
/Users/zisefeizhu/linkun/goproject/kubedev/controllers/application_controller.go:58 +0x184
sigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).reconcileHandler(0x1400055e140, {0x10399f330, 0x14000632600}, {0x10383dfc0, 0x14000443f20})
/Users/zisefeizhu/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.8.3/pkg/internal/controller/controller.go:298 +0x2ac
sigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).processNextWorkItem(0x1400055e140, {0x10399f330, 0x14000632600})
/Users/zisefeizhu/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.8.3/pkg/internal/controller/controller.go:253 +0x1d8
sigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Start.func1.2({0x10399f330, 0x14000632600})
/Users/zisefeizhu/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.8.3/pkg/internal/controller/controller.go:216 +0x44
k8s.io/apimachinery/pkg/util/wait.JitterUntilWithContext.func1()
/Users/zisefeizhu/go/pkg/mod/k8s.io/apimachinery@v0.20.2/pkg/util/wait/wait.go:185 +0x38
k8s.io/apimachinery/pkg/util/wait.BackoffUntil.func1(0x14000025748)
/Users/zisefeizhu/go/pkg/mod/k8s.io/apimachinery@v0.20.2/pkg/util/wait/wait.go:155 +0x68
k8s.io/apimachinery/pkg/util/wait.BackoffUntil(0x1400052ff48, {0x10397a2e0, 0x140005585a0}, 0x1, 0x14000026300)
/Users/zisefeizhu/go/pkg/mod/k8s.io/apimachinery@v0.20.2/pkg/util/wait/wait.go:156 +0x94
k8s.io/apimachinery/pkg/util/wait.JitterUntil(0x14000025748, 0x3b9aca00, 0x0, 0x1, 0x14000026300)
/Users/zisefeizhu/go/pkg/mod/k8s.io/apimachinery@v0.20.2/pkg/util/wait/wait.go:133 +0x88
k8s.io/apimachinery/pkg/util/wait.JitterUntilWithContext({0x10399f330, 0x14000632600}, 0x14000720140, 0x3b9aca00, 0x0, 0x1)
/Users/zisefeizhu/go/pkg/mod/k8s.io/apimachinery@v0.20.2/pkg/util/wait/wait.go:185 +0x88
k8s.io/apimachinery/pkg/util/wait.UntilWithContext({0x10399f330, 0x14000632600}, 0x14000720140, 0x3b9aca00)
/Users/zisefeizhu/go/pkg/mod/k8s.io/apimachinery@v0.20.2/pkg/util/wait/wait.go:99 +0x50
created by sigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Start.func1
/Users/zisefeizhu/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.8.3/pkg/internal/controller/controller.go:213 +0x308
```

#### 删除实例并停止controller

```markdown
不再需要Application实例的时候，执行以下命令即可删除：
# ctl delete -f config/samples/
不再需要controller的时候，去它的控制台使用Ctrl+c中断即可
```

不过实际生产环境中controller一般都会运行在kubernetes环境内，像上面这种运行在kubernetes之外的方式就不合适了，来试试将其做成docker镜像然后在kubernetes环境运行

#### 将controller制作成docker镜像

执行以下命令构建docker镜像并推送到aliyun，镜像名为

```markdown
# docker login --username=zisefeizhu registry.cn-shenzhen.aliyuncs.com
Password: 
Login Succeeded
# docker build -t registry.cn-shenzhen.aliyuncs.com/zisefeizhu-xm/application:v001 .
# docker push registry.cn-shenzhen.aliyuncs.com/zisefeizhu-xm/application:v001
```

#### 在kubernetes环境部署controller

```markdown
# make deploy IMG=registry.cn-shenzhen.aliyuncs.com/zisefeizhu-xm/application:v001
/Users/zisefeizhu/linkun/goproject/kubedev/bin/controller-gen "crd:trivialVersions=true,preserveUnknownFields=false" rbac:roleName=manager-role webhook paths="./..." output:crd:artifacts:config=config/crd/bases
cd config/manager && /Users/zisefeizhu/linkun/goproject/kubedev/bin/kustomize edit set image controller=registry.cn-shenzhen.aliyuncs.com/zisefeizhu-xm/application:v001
/Users/zisefeizhu/linkun/goproject/kubedev/bin/kustomize build config/default | kubectl apply -f -
namespace/kubedev-system created
customresourcedefinition.apiextensions.k8s.io/applications.apps.zise.feizhu configured
serviceaccount/kubedev-controller-manager created
role.rbac.authorization.k8s.io/kubedev-leader-election-role created
clusterrole.rbac.authorization.k8s.io/kubedev-manager-role created
clusterrole.rbac.authorization.k8s.io/kubedev-metrics-reader created
clusterrole.rbac.authorization.k8s.io/kubedev-proxy-role created
rolebinding.rbac.authorization.k8s.io/kubedev-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/kubedev-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/kubedev-proxy-rolebinding created
configmap/kubedev-manager-config created
service/kubedev-controller-manager-metrics-service created
deployment.apps/kubedev-controller-manager created

# kubectl get pods -n kubedev-system -w
NAME                                          READY   STATUS              RESTARTS   AGE
kubedev-controller-manager-776899d98c-9xdfg   2/2     Running   0          9s
```

> 将kube-rbac-proxy 改为阿里云镜像地址

#### 卸载和清理

把前面创建的资源和CRD全部清理掉，可以执行以下命令

```markdown
# make uninstall
```

#### 简单实现 Controller

##### 定义 CR

api/v1/application_types.go

```markdown
	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	// Product 该应用所属的产品
	Product string `json:"product,omitempty"`
}
```

修改之后执行一下 make manifests generate 可以发现已经生成了相关的字段，并且代码中的字段注释也就是 yaml 文件中的注释

```markdown
# make manifests generate
/Users/zisefeizhu/linkun/goproject/kubedev/bin/controller-gen "crd:trivialVersions=true,preserveUnknownFields=false" rbac:roleName=manager-role webhook paths="./..." output:crd:artifacts:config=config/crd/bases
/Users/zisefeizhu/linkun/goproject/kubedev/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."
```

config/crd/bases/apps.lailin.xyz_applications.yaml

```markdown
			properties:
              product:
                description: Product 该应用所属的产品
                type: string
```

##### 实现 controller

kubebuilder 已经实现了 Operator 所需的大部分逻辑，所以只需要在 Reconcile 中实现业务逻辑就行了

./controllers/application_controller.go

```markdown
func (r *ApplicationReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	_ = log.FromContext(ctx)
	_ = r.Log.WithValues("application", req.NamespacedName)
	// your logic here
	r.Log.Info("app changed", "ns", req.Namespace)
	//r.Log.Info(fmt.Sprintf("1. %v", req))           // 打印入参
	//r.Log.Info(fmt.Sprintf("2. %s", debug.Stack())) // 打印堆栈

	return ctrl.Result{}, nil
}
```

逻辑修改好之后，先执行 `make install` 安装 CRD，然后执行 `make run`运行 controller

```markdown
make run
/Users/zisefeizhu/linkun/goproject/kubedev/bin/controller-gen "crd:trivialVersions=true,preserveUnknownFields=false" rbac:roleName=manager-role webhook paths="./..." output:crd:artifacts:config=config/crd/bases
/Users/zisefeizhu/linkun/goproject/kubedev/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."
go fmt ./...
go vet ./...
go run ./main.go
I0210 16:23:51.408872   35770 request.go:655] Throttling request took 1.029677459s, request: GET:https://192.168.101.45:6443/apis/storage.k8s.io/v1beta1?timeout=32s
2022-02-10T16:23:52.683+0800    INFO    controller-runtime.metrics      metrics server is starting to listen    {"addr": ":8080"}
2022-02-10T16:23:52.684+0800    INFO    setup   starting manager
2022-02-10T16:23:52.684+0800    INFO    controller-runtime.manager      starting metrics server {"path": "/metrics"}
2022-02-10T16:23:52.684+0800    INFO    controller-runtime.manager.controller.application       Starting EventSource    {"reconciler group": "apps.zise.feizhu", "reconciler kind": "Application", "source": "kind source: /, Kind="}
2022-02-10T16:23:52.786+0800    INFO    controller-runtime.manager.controller.application       Starting Controller     {"reconciler group": "apps.zise.feizhu", "reconciler kind": "Application"}
2022-02-10T16:23:52.786+0800    INFO    controller-runtime.manager.controller.application       Starting workers        {"reconciler group": "apps.zise.feizhu", "reconciler kind": "Application", "worker count": 1}
```

部署一个测试的 `crd kubectl apply -f config/samples/apps_v1_application.yaml`

```markdown
apiVersion: apps.lailin.xyz/v1
kind: Application
metadata:
  name: application-sample
spec:
  # Add fields here
  product: test
```

然后可以看到之前写的日志逻辑已经触发

```markdown
022-02-10T16:25:27.946+0800    INFO    controllers.Application app changed     {"ns": "default"}
```

### Kubebuilder 注释

在生成的代码当中我们可以看到很多 //+kubebuilder:xxx 开头的注释，对 Go 比较熟悉的同学应该知道这些注释是给对应的代码生成器服务的，在 Go 中有一个比较常用的套路就是利用 go gennerate生成对应的 go 代码。

kubebuilder 使用 controller-gen（https://github.com/kubernetes-sigs/controller-tools/blob/master/cmd/controller-gen/main.go）生成代码和对应的 yaml 文件，这其中主要包含 CRD 生成、验证、处理还有 WebHook 的 RBAC 的生成功能，下面我简单介绍一下，完整版可以看 kubebuilder 的官方文档（https://master.book.kubebuilder.io/quick-start.html）

> CRD 生成
>
> > //+kubebuilder:subresource:status 开启 status 子资源，添加这个注释之后就可以对 status进行更新操作了
> >
> > [//+groupName=nodes.lailin.xyz](https://+groupname%3Dnodes.lailin.xyz/) 指定 groupname
> >
> > //+kubebuilder:printcolumn 为 kubectl get xxx 添加一列，这个挺有用的
> >
> > ……
>
> CRD 验证，利用这个功能，只需要添加一些注释，就给可以完成大部分需要校验的功能
>
> > //+kubebuilder:default:= 给字段设置默认值
> >
> > //+kubebuilder:validation:Pattern:=string 使用正则验证字段
> >
> > ……
>
> Webhook
>
> > //+kubebuilder:webhook 用于指定 webhook 如何生成，例如我们可以指定只监听 Update 事件的 webhook
>
> RBAC 用于生成 rbac 的权限
>
> > //+kubebuilder:rbac