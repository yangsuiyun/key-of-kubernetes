kubernetes作用是容器编排平台，具备调度、自愈、水平伸缩

![image-20220316083052382](./pics/k8s arch.png)

## 核心概念

* pod

  最小调度单元，提供容器运行环境，定义容器执行方式

* Volume

  Pod可访问的文件目录；支持多种存储抽象

* Deployment

  管理Pod部署的副本、部署方案版本

* Service

  提供Pod对外访问地址

* Namespace

  集群内资源隔离的机制



Yaml文件格式：

![image-20220211105319191](./pics/image-20220211105319191.png)