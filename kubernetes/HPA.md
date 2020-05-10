# HPA (Horizontal Pod Autoscaler)
Pod水平收缩

> Horizontal Pod Autoscaler 会根据观察到的 CPU 使用率（或自定义指标(hpa v2)）自动对 `Deployment`、`replication controller`、 `replicaset` 或 `stateful set` 进行扩缩容，但是不适用无法缩放的对象，例如 `DaemonSet`

> HPA 被实现为 Kubernetes API 资源和控制器，资源确定控制器的行为，控制器会定期调整 `Deployment`或`Replica set` 中的副本数量，以使观察到 CPU 平均使用率达到用户所指定的目标


---

### 如何工作
![hpa](./images/horizontal-pod-autoscaler.svg)

> HPA 实现为一个循环控制器，循环周期由 `--horizontal-pod-autoscaler-sync-period` 参数来控制（默认 15 秒），每个周期内，控制器会根据每个 `HorizotalPodAutoscaler` 定义的指标查询资源利用率，控制器从资源指标API或自定义资源指标API获取指标
* 对于每个Pod的资源指标（类似CPU），控制器会从资源指标API获取每个`HorizotalPodAutoscaler` 所定义的Pod的资源使用，如果设置了目标资源利用率，控制器获取每个Pod中的容器资源使用情况，并计算资源使用率，如果使用原始值，将直接使用原始数据（不计算百分比），然后，控制器根据平均的资源使用率或原始值计算出缩放的比例，进而计算出目标副本数量。

> 通常情况下，控制器将从一系列聚合API（metrics.k8s.io custom.metrics.k8s.io external.metrics.io）中获取指标数据，`metrics.k8s.io` API 通常由 metrics-server 提供。

### 算法细节

```
<!-- TargetNumOfPods = ceil(sum(CurrentPodsCPUUtilization) / Target) -->

期望副本数 = ceil[当前副本数 * ( 当前指标 / 期望指标 )]
<!-- desiredReplicas = ceil[currentReplicas * ( currentMetricValue / desiredMetricValue )] -->
```

```
例如：
    当前指标： `200m`
    目标值：   `100m`
    计算：  `200.0 / 100.0 == 2.0`
    结果： 副本数翻倍

    当前指标： `50m`
    目标值：   `100m`
    计算：  `50.0 / 100.0 == 0.5`
    结果： 副本数减半
```

> 如果计算出来的缩放比例接近`1.0`（根据 `--horizontal-pod-autoscaler-tolerance` 参数配置的容忍值， 默认为 0.1）将放弃本次缩放
> 如果 HorizontalPodAutoscaler 指定的是 `targetAverageValue` 或 `targetAverageUtilization`，那么将会把指定pod的平均值作为 `currentMetricValue`
