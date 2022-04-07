![image-20220326214601522](../../pics/istio-archi.png)

# 组件

* Pilot：xDS服务器，为数据面代理提供各种配置
* Citadel：为数据面签发证书
* Galley：Admission Webhook，校验Istio API配置
* Sidecar-Injector：Admission Webhook，自动注入sidecar

# 注入sidecar

1. 自动注入
2. 手动注入