# 项目资料

* 代码库：https://github.com/istio/envoy.git
* 插件库：https://github.com/istio/proxy.git

# 原理



![](/Users/cloud/Knowledge/22CloudNative/K8s learning/pics/Envoy概念与Istio映射.jpeg)

Envoy 概念与 Istio 和 Kubernetes 的映射如上图，每个 sidecar 都有多个监听器生成。每个 sidecar 都有一个监听器，它被绑定到 `0.0.0.0:15006`。这是 IP Tables 将所有入站流量发送到 Pod 的地址。第二个监听器被绑定到 `0.0.0.0:15001`，这是所有从 Pod 中出站的流量地址。

## 启动

![image-20220315111750180](/Users/cloud/Knowledge/22CloudNative/K8s learning/pics/Envoy-启动.png)

![image-20220315111837780](/Users/cloud/Knowledge/22CloudNative/K8s learning/pics/Envoy-启动说明.png)

## 流量拦截

![image-20220315111937544](/Users/cloud/Knowledge/22CloudNative/K8s learning/pics/流量拦截.png)

## xDS

![image-20220315112406683](/Users/cloud/Knowledge/22CloudNative/K8s learning/pics/Envoy-xds.png)

查看xDS配置

* istioctl pc listener $podname  #LDS
* istioctl pc route $podname  #RDS
* istioctl pc cluster $podname  #CDS
* istioctl pc endpoint $podname  #EDS
* istioctl pc secret $podname  #SDS

## 网络处理

![image-20220315114800091](/Users/cloud/Knowledge/22CloudNative/K8s learning/pics/Envoy网络处理.png)

## 过滤器

![image-20220315114842162](/Users/cloud/Knowledge/22CloudNative/K8s learning/pics/Envoy-filter.png)

![image-20220315114920593](/Users/cloud/Knowledge/22CloudNative/K8s learning/pics/Envoy-filter2.png)

## HTTP请求流程

![image-20220315115111610](/Users/cloud/Knowledge/22CloudNative/K8s learning/pics/Envoy-HTTP.png)

![image-20220315115143996](/Users/cloud/Knowledge/22CloudNative/K8s learning/pics/Envoy-HTTP2.png)