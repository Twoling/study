# CNI(Container Network Interface)
> CNI最早是由CoreOS发起的容器网络规范，是Kubernetes网络插件的基础，其基本的思想为：Container Runtime在创建时，先创建好Network Namespace， 然后调用CNI插件为这个Network Namespace配置网络，其后在启动容器内的进程。现已加入CNCF，成为CNCF主推的网络模型。

<div align=center>
	<img src="./CNI.png">
</div>

> 这个协议连接了两个组件：容器管理系统和网络插件。它们之间通过 JSON 格式的文件进行通信，实现容器的网络功能。具体的事情都是插件来实现的，包括：创建容器网络空间（network namespace）、把网络接口（interface）放到对应的网络空间、给网络接口分配 IP 等等。

* 使用CNI后，容器的IP分配就变成了以下步骤：
	* kubelet先创建`pause`容器生成`network namespace`
	* 调用`CNI driver`
	* `CNI driver`根据配置调用具体的cni插件
	* cni插件给`pause`容器配置网络
	* `pod`中其他的容器都使用`pause`容器的网络


> 关于网络，`docker`也提出了`CNM`标准，它要解决的问题和`CNI`是重合的，也就是说目前两者是竞争关系。目前`CNM` 只能使用在`docker`中，而`CNI`可以使用在任何容器运行时。`CNM`主要用来实现`docker`自身的网络问题，也就是`docker network`子命令提供的功能。

## 官方网络插件

--------------------------------------------------------------------

> 所有的标准和协议都要有具体的实现，才能够被大家使用。`CNI`也不例外，目前官方在`github`上维护了同名的`CNI`代码库，里面已经有很多可以直接拿来使用的`CNI`插件。
>
> 官方提供的插件目前分成三类：`main`、`meta`和`ipam`

* __main__： 主要的实现了某种特定网络功能的插件
	* loopback：这个插件很简单，负责生成`lo`网卡，并配置上`127.0.0.1/8`地址
	* bridge：和`docker`默认的网络模型很像，把所有的容器连接到虚拟交换机上
	* macvlan：使用`macvlan`技术，从某个物理网卡虚拟出多个虚拟网卡，它们有独立的`ip`和`mac`地址
	* ipvlan：和`macvlan`类似，区别是虚拟网卡有着相同的`mac`地址
	* ptp：通过`veth pair`在容器和主机之间建立通道

* __meta__：本身并不会提供具体的网络功能，它会调用其他插件，或者单纯是为了测试
	* flannel：结合`bridge`插件使用，根据`flannel`分配的网段信息，调用`bridge`插件，保证多主机情况下容器地址不冲突。

* __ipam__：ipam 是分配 IP 地址的插件。
	* host-local：基于本地文件的IP分配和管理，把分配的IP地址保存在文件中
	* dhcp：从已经运行的`DHCP`服务器中获取IP地址

## CNI Plugin
### Overview
> 每个CNI网络插件必须实现为容器管理系统调用的可执行文件（例如rkt或kubernetes）
> CNI插件负责将网络接口插入容器网络名称空间，并在主机上进行任何必要的修改（例如：将veth的另一端连接到桥中）。然后，它应该通过调用适当的`IPAM`插件将IP分配给接口并设置与IP地址管理部分一致的路由

### CNi插件必须支持的操作：

* `ADD`：将容器添加到网络
<!-- 	* 参数：
		* Container ID：由运行时分配的容器的唯一明文标识符。一定不能为空。
		* Network namespace path：这表示要添加的网络命名空间的路径，即/ proc / [pid] / ns / net或其绑定装载/链接。
		* Network configuration：这是一个JSON文档，描述了容器可以加入的网络。该模式如下所述。
		* Extra arguments：这提供了一种替代机制，允许基于每个容器简单地配置CNI插件。
		* Name of the interface inside the container：这是应该分配给在容器内创建的接口的名称（网络命名空间）; 因此，它必须符合Linux对接口名称的标准限制。
	* 结果
		* Interface list：根据插件，这可以包括沙箱（例如，容器或管理程序）接口名称和/或主机接口名称，每个接口的硬件地址以及接口所在的沙箱（如果有）的详细信息。
		* IP configuration assigned to each interface：分配给沙箱和/或主机接口的IPv4和/或IPv6地址，网关和路由。
		* DNS infomation：包含域名服务器，域，搜索域和选项的DNS信息的字典。
 -->
* `DEL`：从容器中删除网路
<!-- 	* 参数：
		* Container ID：同上
		* Network NameSpace path：同上
		* Network configuration：同上
		* Extra arguments：同上
		* Name of the interface inside the container：同上
	* 删除时所有参数传递应与添加时的参数相同。
	* 删除操作应释放所配置网络中提供给`Container ID`所有的资源
 -->

<!-- > 每个`CNI`网络插件必须实现为容器管理系统调用的可执行文件(例如rkt或kubernetes)
>
> 网络插件只做两件事：把容器加入到网络以及把容器从网络中删除
>
> 调用插件传递数据的两种方式：环境变量和标准输入
>
> 一般插件需要三种类型的数据：容器相关的信息、比如`namespace`的文件、容器id等，网络配置信息，包括网段、网关、DNS以及插件额外的信息等，还有就是`CNI`本身的信息，比如CNI插件的位置、添加网络还是删除网络
>  -->

### Kubernetes Cluster中的网络

* Node Netowrk：承载Kubernetes集群中各个物理节点通信的网络
* Service Network： 由Kubernetes集群中的Services所组成的网络
* Pod Netowrk：Pod网络，承载集群中各个Pod相互通信的网络

#### Service Network
* Service是一组Pod的服务抽象，相当于一组Pod的`LB`，负责将请求分发给对应的Pod，Service会为这个LB提供一个IP，一般称为`ClusterIP`

* Service Types：
	* ClusterIP
	* NodePort
	* LoadBalancer
	* ExternlName


#### Pod Netowrk:
* 同一Pod内不同容器之间通信
	* 在同一Pod中，所有容器共享同一个网络名称空间，共享同一个TCP/IP协议栈，各容器之间可以直接使用`localhost`地址直接访问彼此的端口，这和传统的一组服务运行在一台机器上的环境是一致的，所以此种场景下的应用不需要对网络做出特别的修改就可以移植；这种方式简单、安全、高效，也能减少将已存在的应用从物理机或者虚拟机迁移至容器下运行的那那度。

* 同一主机的两个Pod之间通信
	* 每个Node节点上都有一个由`CNI`网络插件划分的在集群中不冲突、用于给Pod分配地址的网段，所以同一节点上的所有Pod都处于一个网段中，可以直接通信。同一节点上的所有Pod都桥接至`cni0`桥上，默认网关为`cni0`桥的地址

* 不同主机的两个Pod之间通信
	* `Flannel`通过etcd服务维护了一张节点间的路由表，里面详细记录了各节点的子网网段，源主机的`Flanneld`服务将原本的数据内容UDP封装后根据`etcd`中维护的路由表投递到目标节点的`flanneld`服务，数据包到达后被解包，然后直接进入目标节点的

	* 源主机的`Flanneld`服务将原本的数据内容UDP封装后根据`etcd`中维护的路由表投递到目标节点的`flanneld`服务



### Flannel网络

> Flannel是CoreOS团队针对Kubernetes设计的一个覆盖网络(Overlay Netowrk)，用于帮助kubernetes集群中的每个节点分配一个完整的且与集群其他节点不冲突的子网，使不同主机上的docker容器具有全局唯一的虚拟IP地址
>
> Flannel实质上是一种`覆盖网络(overlay network)`, 也就是将TCP数据包装载另一种网络包里面金星路由转发和通信，目前已经支持UDP、VxLAN、AWS VPC和GCE路由等数据转发方式。
>默认的节点间数据通信方式是UDP转发。

#### 工作原理：

> Flannel通过etcd服务维护了一张节点间的路由表，里面详细记录了各节点的子网网段，
> 源主机的`Flanneld`服务将原本的数据内容UDP封装后根据`etcd`中维护的路由表投递到目标节点的`flanneld`服务，数据到达后被解包，然后直接进入目标节点的`falnnel0`虚拟网卡，然后被转发到目标主机的`docker0`虚拟网卡，最后由`docker0`路由到达目标容器。

