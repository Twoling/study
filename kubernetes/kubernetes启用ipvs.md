### Kubernetes启用ipvs
** 注：教程环境为kubernetes 1.12 **
--------

* 安装依赖

```
yum -y install ipset
```

* 启用`ipvs`模块

```
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
```

* 检查加载的模块

```
lsmod | grep -E 'ip_vs|nf_conntrack_ipv4'
```

### 启用ipvs
---

* 现有集群启用`ipvs`

	* 修改`/etc/sysconfig/kubelet`添加一下参数
	```
	KUBE_PROXY_MODE=ipvs
	```

	* 修改`kube-proxy`的`configmap`
	```
	# 修改kube-proxy的'mode'的值为 ipvs
	kubectl edit configmap kube-proxy -n kube-system
	...
	apiVersion: v1
	data:
	...
		ipvs:
	      excludeCIDRs: null
	      minSyncPeriod: 0s
	      scheduler: ""
	      syncPeriod: 30s
	    kind: KubeProxyConfiguration
	    metricsBindAddress: 127.0.0.1:10249
	    mode: "ipvs"				--->  修改为"ipvs"
	...
	```

* 初始化时指定使用`ipvs`

	* 修改`/etc/sysconfig/kubelet`添加一下参数
	```
	KUBE_PROXY_MODE=ipvs
	```

	* 正常初始化集群
	```
	kubeadm init .....
	```