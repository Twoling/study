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




