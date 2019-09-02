### Kubernetes启用ipvs
* __注：教程环境为kubernetes 1.12__

--------

* 安装依赖

```
yum -y install ipset ipvsadm
```

* 启用`ipvs`模块

```
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
```

* 开机自动加载
```shell
cat >> /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/sh 
#

modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
```

* 检查加载的模块
ipvsadm
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

	* 重启相关服务或者删除`kube-proxy`的相关pod，pod自动重建之后就会创建相关`ipvs`规则。

* 初始化时指定使用`ipvs`

	* 创建 `kube-proxy` 配置文件
	```yaml
	apiVersion: kubeproxy.config.k8s.io/v1alpha1
	kind: KubeProxyConfiguration
	mode: ipvs
	```

	* 正常初始化集群
	```
	kubeadm --config kube-proxy.yaml ...
	```

### 安装`ipvsadm`查看生成的规则
```
yum -y install ipvsadm

ipvsadm -ln
```
