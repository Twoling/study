# Service

> Kubernetes中的`Pod`是有声明周期的，他们可以被创建，也可以被销毁，然而`Pod`一旦被销毁，就代表着其生命永远结束，在通过`ReplicaSet`能够动态创建和销毁`Pod`（例如：需要进行扩容、滚动升级等）。
>
> 每个`Pod`在创建时都会拥有自己的IP地址，在`Pod`不断的创建和销毁中，这些IP并不是一直保持不变的，因此会导致一个问题，在Kubernetes集群中，如果一组`Pod`（成为backend）为其他`Pod`（成为frontend）提供服务，那么当这些提供服务的`Pod`发生变化时，frontend如何发现并连接这组`Pod`呢？

* Service是一组Pod的服务抽象，相当于一组Pod的`LB`，负责将请求分发给对应的Pod，通常通过`Label Selector`选择关联的后端`Pod`， Service会为这个`LB`提供一个IP，一般称为`ClusterIP`

* Endpoints：kubernetes会根据`service`关联到的所有`Pod IP`和`Service`定义的`targetPort`组成一个`endpoints`，若`service`定义中没有`selector`字段，`service`被创建时，`endpoints`不会自动被创建，值为`none`

## Service Types：
* ClusterIP
	* iptalbes代理模式：这种模式下，[kube-proxy](./kube-proxy.md)会监视kubernetes master对`Service`对象和`Endpoint`的对象添加和移除，对每个`Service`都会设置`iptables`规则，从而捕获到达该`Service`的`ClusterIP`（虚拟IP）和端口的请求，进而将请求重定向到`Service`的一组backend的某个`Pod`上

	* 任何到达`Service`的`IP:PORT`的请求，都会被代理到一个合适的backend，不需要客户端知道关于Kubernetes、`Service`或`Pod`的任何信息，如果初始选择的`Pod`没有响应，`iptables`代理能够自动重试另一个`Pod`，不过需要配置`readiness probes`

![services-iptables-overview](./services-iptables-overview.svg)

* NodePort
* LoadBalancer
* ExternlName