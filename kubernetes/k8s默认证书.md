# Kubernetes的证书链
* 默认证书链解析

|CA|签发的证书|
|:--:|--:|
|/etc/kubernetes/pki/ca.crt|apiserver.crt、 apiserver-kubelet-client.crt|
|/etc/kubernetes/pki/front-proxy-ca.crt|front-proxy-client.crt|
|/etc/kubernetes/pki/etcd/ca.crt|healthcheck-client.crt、 peer.crt、 server.crt|

## CA证书
### /etc/kubernetes/pki/ca.crt
#### 作用
* Kubernetes集群CA证书

#### 签发的证书
* apiserver.crt
	* 颁发者：kubernetes
	* SAN域名:
		* kubernetes.default.svc.cluster.local
		* kubernetes.default.svc
		* kubernetes.default
		* kubernetes
		* `<IPs>`
	* CN：kube-apiserver

* apiserver-kubelet-client.crt
	* 颁发者：kubernetes
	* SAN域名:
		* null
	* 组织：system:masters
	* CN：kube-apiserver-kubelet-client

### /etc/kubernetes/pki/front-proxy-ca.crt
#### 作用
* apiserver用于验证kube-proxy请求

#### 签发的证书
* front-proxy-client.crt
	* 颁发者：front-proxy-ca
	* SAN域名:
		* null
	* 组织：null
	* CN：front-proxy-client

### /etc/kubernetes/pki/etcd/ca.crt
#### 作用
* etcd集群用到的CA证书

#### 签发的证书
* healthcheck-client.crt
	* 颁发者：etcd-ca
	* SAN域名:
		* null
	* 组织：system:masters
	* CN：kube-etcd-healthcheck-client

* peer.crt
	* 颁发者：etcd-ca
	* SAN域名:
		* localhost
		* `<IPs>`
	* 组织：system:masters
	* CN：默认为当前主机名

* server.crt
	* 颁发者：etcd-ca
	* SAN域名:
		* localhost
		* `<IPs>`
	* 组织：null
	* CN：默认为当前主机名


