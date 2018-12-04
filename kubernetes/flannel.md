### Kubernetes Cluster中的网络

* Node Netowrk：承载Kubernetes集群中各个物理节点通信的网络
* Service Network： 由Kubernetes集群中的Services所组成的网络
* Pod Netowrk：Pod网络，承载集群中各个Pod相互通信的网络





### Flannel网络

> Flannel是CoreOS团队针对Kubernetes设计的一个覆盖网络(Overlay Netowrk)，用于帮助kubernetes集群中的每个节点分配一个完整的且与集群其他节点不冲突的子网，使不同主机上的docker容器具有全局唯一的虚拟IP地址




```flow
st=>start: Start
e=>end: End
op=>operation: My Operation
cond=>condition: Yes or No?

st->op->cond
cond(yes)->e
cond(no)->op
```