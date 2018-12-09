# Service

> Kubernetes中的`Pod`是有生命周期的，他们可以被创建，也可以被销毁，然而`Pod`一旦被销毁，就代表着其生命永远结束，在通过`ReplicaSet`能够动态创建和销毁`Pod`（例如：需要进行扩容、滚动升级等）。 
> 每个`Pod`在创建时都会拥有自己的IP地址，在`Pod`不断的创建和销毁中，这些IP并不是一直保持不变的，因此会导致一个问题，在Kubernetes集群中，如果一组`Pod`（成为backend）为其他`Pod`（成为frontend）提供服务，那么当这些提供服务的`Pod`发生变化时，frontend如何发现并连接这组`Pod`呢？
>
> Service是一组Pod的服务抽象，相当于一组Pod的`LB`，负责将请求分发给对应的Pod，通常通过`Label Selector`选择关联的后端`Pod`， Service会为这个`LB`提供一个IP，一般称为`ClusterIP`
>
> Endpoints：kubernetes会根据`service`关联到的所有`Pod IP`和`Service`定义的`targetPort`组成一个`endpoints`，若`service`定义中没有`selector`字段，`service`被创建时，`endpoints`不会自动被创建，值为`none`
>
> iptalbes代理模式：这种模式下，[kube-proxy](./kube-proxy.md)会监视kubernetes master对`Service`对象和`Endpoint`的对象添加和移除，对每个`Service`都会设置`iptables`规则，从而捕获到达该`Service`的`ClusterIP`（虚拟IP）和端口的请求，进而将请求重定向到`Service`的一组backend的某个`Pod`上
>
> 任何到达`Service`的`IP:PORT`的请求，都会被代理到一个合适的backend，不需要客户端知道关于Kubernetes、`Service`或`Pod`的任何信息，如果初始选择的`Pod`没有响应，`iptables`代理能够自动重试另一个`Pod`，不过需要配置`readiness probes`

![services-iptables-overview](./services-iptables-overview.svg)
----------------------------

## Service Types：
* ClusterIP：使用集群内部IP暴露服务，选择此值服务只能在集群内部访问
* NodePort：通过`Node`上的IP和静态端口暴露服务，`NodePort`会路由到`ClusterIP`服务，这个`ClusterIP`会自动创建，通过请求`<NodeIP>:<NodePort>`，可以从集群外部访问一个`NodePort`服务。
* LoadBalancer：使用云提供商的负载局衡器，可以向外部暴露服务。外部的负载均衡器可以路由到`NodePort`服务和`ClusterIP`服务。
* ExternlName：通过返回`CNAME`和它的值，可以将服务映射到`externalName`字段的内容（例如，`foo.bar.example.com`）。 没有任何类型代理被创建，这只有`Kubernetes 1.7`或更高版本的`kube-dns`才支持

### 创建Service时指定IP地址：
* 在创建`Service`时，可以通过`spec.clusterIP`来指定自己在集群中的IP地址，要注意，指定的IP地址必须合法，并且在`service-cluster-ip-range` CIDR的范围之内。

### 服务发现：
* Kubernetes支持两种服务发现形式：
	* 环境变量
		* 当`Pod`运行在`Node`上，`kubelet`会为每个活跃的`Service`添加一组环境变量，简单的 `{SVCNAME}_SERVICE_HOST`和`{SVCNAME}_SERVICE_PORT`变量，这里`Service`的名称需大写，横线被转换成下划线。 它同时支持`Docker links`兼容变量。
	* DNS
		* DNS服务是集群中的可选插件，这个DNS服务通过`Kubernetes API`监视着每个新的`Service`的创建并且在DNS上生成相应的记录，如果在整个集群中启用了DNS服务，那么集群中的所有`Pod`都应该能够自动对`Service`进行名称解析。
		* Kubernetes也支持对端口名称的`DNS SRV（Service）`记录。 如果名称为`"my-service.my-ns"`的`Service`有一个名为`"http"`的`TCP`端口，可以对`"_http._tcp.my-service.my-ns"`执行`DNS SRV`查询，得到`"http"`的端口号。
		```
		# service_name: myapp-svc
		# namespace: default
		# dns search: cluster.local
		~]# dig -t srv _http._tcp.myapp.default.svc.cluster.local @10.96.0.10
		; <<>> DiG 9.9.4-RedHat-9.9.4-72.el7 <<>> -t srv _http._tcp.myapp-svc.default.svc.cluster.local @10.96.0.10
		;; global options: +cmd
		;; Got answer:
		;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41367
		;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

		;; OPT PSEUDOSECTION:
		; EDNS: version: 0, flags:; udp: 4096
		;; QUESTION SECTION:
		;_http._tcp.myapp-svc.default.svc.cluster.local.	IN SRV

		;; ANSWER SECTION:
		_http._tcp.myapp-svc.default.svc.cluster.local.	5 IN SRV 0 100 8080 myapp-svc.default.svc.cluster.local.

		;; ADDITIONAL SECTION:
		myapp-svc.default.svc.cluster.local. 5 IN A	10.106.26.196

		;; Query time: 1 msec
		;; SERVER: 10.96.0.10#53(10.96.0.10)
		;; WHEN: Sat Dec 08 23:42:49 EST 2018
		;; MSG SIZE  rcvd: 227
		```





