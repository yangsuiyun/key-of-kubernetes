# 安装过程
版本： KubeSphere 3.2.1

1. 安装k8s-install中的两个yaml，其中需要根据默认storageclass修改第二个yaml中的storageclass

2. 查看执行过程

   ```shell
   kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
   
   ```

3. 登录
admin/P@88w0rd
