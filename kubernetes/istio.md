# Istio

## 架构
### 数据平面
有一组以 `sidecar` 方式部署的智能代理(`Envoy`)组成，这些代理可以调节和控制微服务及 `Mixer` 之间所有的网络通信

#### `Envoy`
* 支持的网络通信协议
	* HTTP / 1.1
	* HTTP/2
	* gRPC
	* TCP
* 相关功能：
	* 服务发现：从 `Pilot` 得到服务发现信息
	* 过滤
	* 负载均衡
	* 健康检查
	* 执行路由规则(Rule): 规则来自 `Pilot`，包括路由和目的地策略
	* 加密和认证: TLS certs 来自 Istio-Auth
* 数据
	* Metrics
	* Logging
	* Distribution Trace: 目前支持 Zipkin

`Envoy并不是一个Web服务器`

`Client` ---->  `Listener`  ---->   `Filter`   ---->   `Route`   ---->   `Cluster`

##### 配置
`Envoy 的配置主要有监听器(Listener)和集群(Cluster)组成`

### 控制平面
负责管理和配置代理的路由流量，此外控制平面配置 `Mixer` 以实施策略和收集遥测数据。

#### `Mixer`




#### `Pilot`
> `Istio` 流量管理的核心组件是 `Pilot`， 它管理和配置部署在特定 `Istio` 服务网格中的所有 `Envoy` 代理实例，它允许您指定在 `Envoy` 代理之间使用什么样的路由流量规则，并配置故障恢复功能，如超时、重试、熔断器。它还维护了网格中所有服务的规范模型，并使用这个模型通过发现服务让 `Envoy` 了解网格中的其他实例。
>
> 每个 `Envoy` 实例都会维护负载均衡信息，这些信息来自 `Pilot` 以及对负载均衡池中其他实例的定期健康检查，从而允许其在目标实例之间只能分配流量，同时遵循其指定的路由规则
>
> `Pilot` 负责管理通过 `Istio` 服务网格发布的 `Envoy` 实例的生命周期

![Pilot](./PilotAdapters.svg)

> 如上图所示，在网格中 `Pilot` 维护了一个服务的规则表示并独立于底层平台. `Pilot` 中的特定于平台的适配器负责适当地填充这个规范模型，例如，在 `Pilot` 中的 `Kubernetes` 适配器实现了必要的控制器，来观察 `Kubernetes API` 服务器，用于更改 `Pod` 的注册信息、入口资源以及存储流量管理规则的第三方资源。 这些数据被转换为规范表示，然后根据规范表示生成特定的 `Envoy` 的配置
>
> `Pilot` 公开了用于服务发现、负载均衡池和路由表的动态更新API
>
> 运维人员可以通过 `Pilot` 的 `Rules API` 指定高级流量管理规则，这些规则被翻译成低级配置，并通过 `Discovery API` 分发到 `Envoy` 实例

##### 请求路由
> 如 `Pilot` 所述，特定个网格中服务的规范表示有 `Pilot` 维护。 服务的 `Istio` 模型和在底层平台(`Kubernetes`、 `Mesos`、 `Cloud Foundry`) 中的表达无关，特定平台的适配器负责从各自平台中获取元数据的各种字段，然后对服务模型进行填充
>
> 

#### `Galley`





#### `Citadel`
