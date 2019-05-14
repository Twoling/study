# Prometheus
---

## 数据采集方式
### pull方式
* 由Prometheus通过HTTP协议主动去拉取，因为使用HTTP协议，因此只要应用系统提供HTTP接口，就可以很便捷的接入到监控系统中来

### push方式
* 对于一些定时任务这种短时间内的指标采集，如果采用pull的方式，可能会出现任务已经结束了，Prometheus还没来得及采集，这种问题Prometheus也提供了通过push的方式让客户端主动推送Metrics数据，此功能通过Push Gateway来实现，需要额外搭建Push Gateway并且需要在Prometheus上建立相应的job取Push Gateway上取采集数据

## Prometheus Architecture
![Prometheus](./architecture.png)

## 组件
### Prometheus server
* 主要负责数据采集和存储，并提供PromQL查询语言的支持

### client libraries
* 客户端库，如go、java、python、ruby等

### push gateway
* 支持临时性job主动推送数据的中间网关

### exporters
* 支持其他数据源的指标数据导入Prometheus

### alertmanager
* 报警管理

### 其他辅助性工具

## 数据模型
Prometheus从根本上是将所有监控数据以时间序列的形式存储在内置的时间序列存储中(TSDB),属于同一指标名称，同一标签集合的、有时间戳标记的数据流.除了存储的时间序列，Prometheus 还可以根据查询请求产生临时的、衍生的时间序列作为返回结果。

### Metric names and labels
* 每一条时间序列由指标名称(Metrics Name)以及一组标签(键值对)唯一标识，其中指标的名称可以反映被监控样本的含义，指标名称只能由ASCII字符、数字、下划线以及冒号组成（例如一般特征http_requests_total-接收到的HTTP请求的总数），同时必须匹配正则表达式：`[a-zA-Z_:][a-zA-Z0-9_:]*`,`__` 作为前缀的标签，是系统保留的关键字，只能在系统内部使用.

> [info] 注意
> 冒号用来表示用户自定义的记录规则，不能在exporter中或监控对象直接暴露的指标中使用冒号来定义指标名称。

### Samples
在时间序列中的每一个点称为一个样本（sample），样本由以下三部分组成：
* 指标（metric）：指标名称和描述当前样本特征的 labelsets；
* 时间戳（timestamp）：一个精确到毫秒的时间戳；
* 样本值（value）： 一个 folat64 的浮点型数据表示当前样本的值。

### Notation
通过如下表达方式表示指定指标名称和指定标签集合的时间序列：
```
	<metric name>{<label name>=<label value>, ...}
```
* 例如，指标名称为`api_http_requests_total`标签为`method="POST"`和`handler="/messages"`的时间序列可以表示为：
```
	api_http_requests_total{method="POST", handler="/messages"}
```

## Prometheus的四种指标类型
### Counter
* 计数器，代表一个指标的递增计数器，他的值只能增加或者重启启动时重置为0，例如，可以使用计数器统计服务的请求次数，已完成的任务数或错误数量
* 不要使用 Counter 暴露可能减少的值，例如进程数

### Gauge
* 常规数值，可变大、可变小，通常用于记录键控值，如温度、内存、网络等

### Histogram
* 柱状图，用于统计一些数据的分布情况，用于计算在一定范围内的分布情况，同时还提供了度量指标值的总和

### Summary
* Summary与Histogram类似，主要用于计算在一定时间窗口范围内度量指标对象的总数以及所有度量指标值的总和
---

## 记录规则和报警规则
### 记录规则
* 记录规则允许我们把一些经常需要使用并且查询时计算量很大的查询表达式，预先计算并保存到一个新的时序。查询这个新的时序比从原始的一个或多个时序实时计算快得多，并且还能够避免重复计算。在一些特殊场景下这甚至是必须的，比如仪表盘里展示的各类定时刷新的数据，数据种类多且需要计算非常快

### 报警规则
* 报警规则会在条件满足时发送警告到告警管理器（Alertmanager）。



# Prometheus-operator
Prometheus是通过pull方式去拉取target的数据，但k8s中所有服务以容器的方式运行，每一个容器的销毁与创建势必会导致target目标失效，不可能通过人工去维持监控目标。
Protmetheus-operator相当于一个控制器，通过一组用户自定义CRD来实现对不断变化的资源自动化完成对Prometheus Server自身以及配置的自动化管理工作

## Prometheus-operator 自定义资源(CRD)
### Prometheus
* 定义了所期望的Prometheus Deployment，Operator保证正在运行的Prometheus的Deployment始终匹配正在运行的资源。

### ServiceMonitor
* 以声明式方式定义如何监控Service，Operator根据ServiceMonitor的定义的目标来自动生成Prometheus的指标收刮配置(scrape/pull)，可以动态更新Prometheus Server的target列表，并让Prometheus Server去reload配置(prometheus有对应的reload的http接口`/-/reload`)，该资源主要通过Selector拉取匹配特定Labels的Service的Endpoins来进行监控对象的发现。

### PrometheusRule
* 定义了Prometheus所需的规则文件，该定义包含Prometheus的报警规则和记录规则可以被Prometheus实例加载

### Alertmanager
* 定义了所需的AlertManager的Deployment，Operator保证正在运行的资源始终匹配Deployment的资源定义
