# Kubernetes
## Overview：
* Kubernetes 是一个跨主机集群的开源的容器调度平台，它可以自动化应用容器的部署、扩展和操作， 提供以容器为中心的基础架构

## 特点：
* 便携性：无论是公有云、私有云、混合云还是多云架构都全面支持
* 可扩展：它是模块化、可插拔、可挂载、可组合的，支持各种形式的扩展
* 自修复：它可以自保持应用状态、可自重启、自复制、自缩放的，通过生命式语法提供了强大的自修复能力

# 各组件：
## Master节点组件：
* master节点组件提供集群控制平面，master组件可以对集群做出全局决策（例如：调度）并且检测和响应集群事件（当集群中的副本数量不满足复制控制器中'replicas'字段时，启动新的容器）

### API-Server：
* `API Server`的核心功能主要是为Kubernetes的各类资源对象(如`node`、`Pod`、`Service`)提供了增、删、改、查遗迹`watch`的`HTTP Rest`接口，`API Server`是集群中各个功能模块之间数据交互和通信的中心

* 特性：
	* 集群管理的API入口
	* 资源配额控制的入口
	* 提供完备的集群安全机制

### etcd
* `etcd`是一个分布式键值存储，旨在可靠、快速的保存和提供给对关键数据的访问，它通过分布式锁、`leader`选举、并行写入障碍实现可靠的分布式协调，`etcd`集群旨在实现高可用性和永久数据存储和检索。
* `etcd`用于 Kubernetes 的后端存储。所有集群数据都存储在此处。

### kube-controller-manager
* `kube-controller-manager`运行控制器，它们是处理集群中常规任务的后台线程。逻辑上，每个控制器是一个单独的进程，但为了降杂性，它们都被编低复译成独立的可执行文件，并在单个进程中运行。
* 控制器：
	* Node Controller：负责节点出现故障时的通知和响应。
	* Replication Controller：负责为集群中的每个复制控制器维护正确的`Pod`数量。
	* Endpoints Controller：负责生成`Endpoints`对象（连接`Service` & `Pods`）。
	* Service Account & Token Controller：为新命名空间创建默认账号和API访问令牌。

### kube-scheduler
* `kube-scheduler`监视没有分配节点建的`Pod`，选择一个节点供他们运行。
 DNS:
* 虽然其他插件并不是必需的，但所有Kubernetes集群都应该具有Cluster DNS，许多示例依赖于它。
* Cluster DNS是一个DNS服务器，和您部署环境中的其他DNS服务器一起工作，为Kubernetes服务提供DNS记录。Kubernetes启动的容器自动将DNS服务器包含在DNS搜索中。

### Network Plugin
* DNS Plugin:
	* coredns
* Network Plugin:
	* flannel
	* clico

## Node节点组件：
* Node节点组件运行在每一台Node上，位置`Pod`运行并为Kubernetes提供运行时环境

### kubelet
* 在集群中的每个node上运行的agent，它确保了容器运行在Pod中.
* `kubelet`是主要节点代理，他检测已分配给其节点的`Pod`（通过`apserver`或通过本地配置文件，提供如下功能：
	* 挂载`Pod`所需要的数据卷(`Volume`)
	* 下载`Pod`的`secrets`
	* 通过`Docker`运行`Pod`的容器
	* 周期性的对容器生命周期进行探测
	* 如果需要，通过创建镜像`Pod`(`Mirror Pod`)将`Pod`的状态报告回系统的其他部分
	* 将节点的状态报告回系统的其他部分

### kube-proxy
* `kube-proxy`通过维护主机上的网络规则并执行连接转发，实现了Kubernetes服务抽象

### docker
* docker用于运行容器

### supervisord
* `supervisord`是一个轻量级的进程监控系统，可以用来保证`kubelet`和`docker`运行。









* RESTful
	* GET
	* POST
	* DELETE
	* PUT

* 资源：对象
* workload:
	* pod
	* ReplicaSet
	* Deployment
	* statefulSet
	* daemonSet
	* Job
	* CronJob

* 服务发现及均衡
	* Service
	* Ingress

* 配置与存储：
	* Volume
	* CSI
	* ConfigMap
	* Secret
	* DownwardAPI

* 集群级资源
	* Namespace
	* Node
	* Role
	* ClusterRole
	* RoleBind
	* ClusterRoleBind

* 元数据型资源
	* HPA
	* PodTemplate
	* LimitRange
创建资源的方法：
	apiserver仅接受JSON格式的资源定义
	yaml格式提供配置清单，apiserver会自动将其转为JSON格式的信息
大部分资源都有5个一级资源组成

* 资源清单格式：
	一级字段：
		apiVersion(group/version):省略group默认为core，核心组
		kind：资源类别
		metadata：元数据
			name:
			namespace:
			labels:
			annotations
		spec：期望字段
		status：状态 current state，本字段有k8s集群维护

* 资源的引用PATH：
	`/api/GROUP/VERSION/namespaces/NAMESPACE/TYPE/NAME`

* Pod资源
	spec.containers <[]object>
	- name <string>
	  image <string>
	  imagePullPolicy <string>
	  	Always：总是去互联网上下载
	  	Never：永远不下载
	  	IfNotPresent：如果本地存在则是用本地，如果没有，则去公用仓库下载
	* 修改镜像中的默认应用：
		command args
---
| __Image EntryPoint__ | __image Cmd__ | __Container command__ | __Container args__ | __Command run__ |
| :------------------: | :-----------: | :-------------------: | :----------------: | :-------------: |
| [/ep-1]              | [foor bar]    |  `<not set>`          | `<not set>`        | [ep-1 foo bar]  |
| [/ep-1]              | [foor bar]    |  [/ep-2]              | `<not set>`        | [ep-2]          |
| [/ep-1]              | [foor bar]    | `<not set>`           | [zoo boo]          | [ep-1 zoo boo]  |
| [/ep-1]              | [foor bar]    |  [/ep-2]              | [zoo boo]          | [ep-2 zoo boo]  |
---


* 获取容器应用日志：
		kubectl logs pod [-c application]

* 标签：
	* key=value
		key: 字母、数字、下划线
		value：可以为空，只能字母或数字开头及结尾

* 标签选择器：
	* 等值关系：
		=、==、!=
	* 集合关系：
		KEY in (VALUE1, VALUE2...)
		KEY notin (VALUE1, VALUE2...)
		KEY
		!KEY
	* 许多资源支持内嵌字段定义其使用的标签选择器
		matchLabels：直接给定简直
		matchExpressions：基于给定的表达式来定义使用标签选择器，{key:"", operator:"OPERATOR", value: [va1, va2,...]}
		操作符：
			In, NotIn: values字段的值必须为非空列表
			Exists, NotExists：values字段的值必须为空列表	



