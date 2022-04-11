Controller Manager内部包含Replication Controller、Node Controller 、 ResourceQuota Controller 、 Namespace Controller 、ServiceAccount Controller 、 Token Controller 、 Service Controller 、Endpoint Controller、Deployment Controller、Router Controller、Volume Controller等各种资源对象的控制器。

## Resource Quota

![image-20220409182650235](../pics/resource-quota.png)

用户在创建资源时，admission controller会先根据Resource Quota控制的数据校验用户资源是否超出限制。