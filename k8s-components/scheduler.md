## 调度器原理

k8s原生的调度器原理如下图，informer不断从api server中拉取pod和node的信息，对没有绑定node的pod会放到调度队列中，pod和node信息则放在缓存中。调度器的调度循环中：

* 第一步是拉取一个要处理的pod，
* 在Pridicate阶段过滤不符合的Node；
* 在Priority阶段给每个合适node打分，
* 根据得分选择Node；
* 当没有合适Node，则对优先级低pod进行抢占
* 绑定pod到Node（Pod对象添加Node的信息），提到缓存中

<img src="../pics/调度原理.png" alt="image-20220316162837894" style="zoom:50%;" />

新版调度器增加下面的scheduler framework，增加了扩展点，支持用户以插件的方式进行扩展。

![img](../pics/scheduling-framework-extensions.png)

## 算法

Predicate算法

![image-20220316163550339](../pics/predicate算法.png)

Priority算法

![image-20220316163719053](../pics/Priority算法.png)

## 多调度器

* 添加自定义调度器
* 通过scheduler profile对调度器进行不同的配置，实现多调度器的特性
