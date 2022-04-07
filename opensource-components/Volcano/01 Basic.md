## 应用场景

Volcano是用于替代k8s自带调度器，提供支持机器学习、批量job、hpc所需的资源调度模式、更复杂多样的调度算法。

<img src="../pics/volcano-usage.png" alt="img" style="zoom: 67%;" />



遇到如下情形，两个TFJob均需要4卡资源，默认调度器是逐个pod进行调度会出现下面的情况。两个任务都要等待对方释放资源才能运行，造成死锁。

![image-20220328161101488](../pics/调度问题.png)

对分布式训练任务，通常一个任务包含一个ps和多个worker，他们之间需要数据交换。两个任务同时调度时，原生调度器缺乏亲和性调度能力，可能调度的情形如下三种情况。（每一列表示一台机器上有三个pod调度能力）

![image-20220328161329620](../pics/调度问题2-亲和性.png)

## 架构

<img src="../pics/volcano-archi.png" alt="img" style="zoom: 50%;" />

Volcano由scheduler、controllermanager、admission和vcctl组成:

- Scheduler Volcano scheduler通过一系列的action和plugin调度Job，并为它找到一个最适合的节点。与Kubernetes default-scheduler相比，Volcano与众不同的 地方是它支持针对Job的多种调度算法。
- Controllermanager Volcano controllermanager管理CRD资源的生命周期。它主要由**Queue ControllerManager**、 **PodGroupControllerManager**、 **VCJob ControllerManager**构成。
- Admission Volcano admission负责对CRD API资源进行校验。
- Vcctl Volcano vcctl是Volcano的命令行客户端工具。