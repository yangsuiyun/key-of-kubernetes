kubeproxy是service的透明代理兼负载均衡，核心功能是将service的访问转发到后端到pod实例上。

第二代kubeproxy相当于iptables的自动修改器，实现流量借助iptables转发。当集群规模增大后，iptables会增大，性能会下降。第三代kubeproxy，通过控制IPVS而非iptables，实现其功能。IPVS相对iptabe的优势：

* 更好的扩展性和性能
* 支持更复杂的算法
* 支持服务的健康检查和连接重试
* 动态修改ipset集合（相比iptable，ipset配置更高效）