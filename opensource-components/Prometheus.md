![Prometheus architecture](../pics/architecture.png)

数据采集方式：

*  push 的方式，就是通过 pushgateway 进行数据采集，然后数据线到 pushgateway，然后 Prometheus 再通过 pull 的方式去 pushgateway 去拉数据。
* 标准的 pull 模式，它是直接通过拉模式去对应的数据的任务上面去拉取数据。
*  Prometheus on Prometheus，就是可以通过另一个 Prometheus 来去同步数据到这个 Prometheus。
*  通过不同类型的exporter收集不同类型对象的数据