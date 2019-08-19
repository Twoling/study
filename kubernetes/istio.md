# Istio

## 架构
## 数据平面
有一组以 `sidecar` 方式部署的智能代理(`Envoy`)组成，这些代理可以调节和控制微服务及 `Mixer` 之间所有的网络通信

### `Envoy`
* 基本术语
	* Host: 能够进行网络通信的实体

	* Downstream: 下游，主机连接到 `Envoy` ，发送请求或获得响应

	* Upstream: 上游 主机获取来自 `Envoy` 的连接请求和响应

	* Cluster: 集群是 `Envoy` 连接到的一组逻辑上相似的上游主机， `Envoy` 通过服务发现发现集群中的成员

	* Mesh: 一组相互协调以提供一致网络拓扑的主机，`Envoy Mesh` 是指一组 `Envoy` 代理，它们构成了由不同不同服务于应用程序组成的分布式消息的传递基础

	* Runtime configuration:  `Envoy` 支持运行时配置，更改配置后立即影响影响运算，而无需重启或手动更改主配置

	* Listener: `Envoy` 支持在单个进程中创建若干个监听器，每个监听器绑定在一个可连接的网络位置上（例如，socket，unix域套接字）

	* Filter: 过滤器，过滤流量以实现高级流量控制，现官方支持的filter包括：`Listener filters` `Network filters` `HTTP filters` `Dubbo filters`

	* Route: HTTP路由规则，根据给定的规则，来决定路由到个 `Cluster`


* xDS: xDS是一个关键概念，它是一类发现服务的统称，其包括如下几类
		* CDS: Cluster Discovery Service
		* EDS: Endpoint Discovery Service
		* SDS: Service Discovery Service
		* RDS: Route Discovery Service
		* LDS: Listener Discovery Service

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

#### 配置
`Envoy 的配置主要有监听器(Listener)和集群(Cluster)组成`
* 监听器实例配置
```
listeners:
- address:
		socket_address:
			address: 0.0.0.0
			port_value: 80
	filter_chains:
	- filters:
		- name: envoy.http_connection_manager
			config:
				codec_type: auto
				stat_prefix: ingress_http
				route_config:
					name: local_route
					virtual_hosts:
					- name: backend
						domains:
						- "*"
						routes:
						- match:
								prefix: "/service/1"
							route:
								cluster: service1
						- match:
								prefix: "/service/2"
							route:
								cluster: service2
```

## 控制平面
负责管理和配置代理的路由流量，此外控制平面配置 `Mixer` 以实施策略和收集遥测数据。

### `Mixer`




### `Pilot`
> `Istio` 流量管理的核心组件是 `Pilot`， 它管理和配置部署在特定 `Istio` 服务网格中的所有 `Envoy` 代理实例，它允许您指定在 `Envoy` 代理之间使用什么样的路由流量规则，并配置故障恢复功能，如超时、重试、熔断器。它还维护了网格中所有服务的规范模型，并使用这个模型通过发现服务让 `Envoy` 了解网格中的其他实例。
>
> 每个 `Envoy` 实例都会维护负载均衡信息，这些信息来自 `Pilot` 以及对负载均衡池中其他实例的定期健康检查，从而允许其在目标实例之间只能分配流量，同时遵循其指定的路由规则
>
> `Pilot` 负责管理通过 `Istio` 服务网格发布的 `Envoy` 实例的生命周期

![Pilot](./images/PilotAdapters.svg)

> 如上图所示，在网格中 `Pilot` 维护了一个服务的规则表示并独立于底层平台. `Pilot` 中的特定于平台的适配器负责适当地填充这个规范模型，例如，在 `Pilot` 中的 `Kubernetes` 适配器实现了必要的控制器，来观察 `Kubernetes API` 服务器，用于更改 `Pod` 的注册信息、入口资源以及存储流量管理规则的第三方资源。 这些数据被转换为规范表示，然后根据规范表示生成特定的 `Envoy` 的配置
>
> `Pilot` 公开了用于服务发现、负载均衡池和路由表的动态更新API
>
> 运维人员可以通过 `Pilot` 的 `Rules API` 指定高级流量管理规则，这些规则被翻译成低级配置，并通过 `Discovery API` 分发到 `Envoy` 实例

##### 请求路由
> 如 `Pilot` 所述，特定个网格中服务的规范表示有 `Pilot` 维护。 服务的 `Istio` 模型和在底层平台(`Kubernetes`、 `Mesos`、 `Cloud Foundry`) 中的表达无关，特定平台的适配器负责从各自平台中获取元数据的各种字段，然后对服务模型进行填充
>
>

### `Galley`





#### `Citadel`
