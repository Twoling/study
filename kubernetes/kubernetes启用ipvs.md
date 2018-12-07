* Kubernetes启用ipvs
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
	