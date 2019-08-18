# 使用Envoy做Kubernetes的Ingress控制器

## Envoy 简述
Envoy 是专为大型现代架构设计的7层代理和通信总线，由 Lyft 开源的，使用 C++ 编写，目前是 CNCF 旗下的开源项目，代码托管在 GitHub 上，也是 Istio Service Mesh 中默认的数据平面。

###　特性
* 进程外架构
* 现代 C++11 代码库
* L3/L4 filter 架构
* HTTP L7 filtter 架构
* HTTP/2 支持
* HTTP L7 路由
* gRPC支持
* MongoDB L7支持
* DynamoDB L7 支持
* 服务发现
* 健康检查
* 高级负载均衡
* 前端/边缘代理支持
* 最佳的可观测性
* 动态配置

**具体细节可以到官网了解: https://www.envoyproxy.io/docs/envoy/latest/**

## contour
Envoy 官方并未提供作为 kubernetes ingress controller 运行的解决方案，而是由一个叫 heptio 的公司(现属于Vmware) 开发的一个名叫 `contour` 的解决方案。

github地址： https://github.com/heptio/contour

### contour架构
* Envoy: 提供高性能的反向代理
* Contour: 用于管理 Envoy 并为其提供配置

> 在 Pod 初始化期间， Contour 作为 initcontainer 运行，并将引导程序写入临时卷，该卷被传递给 Envoy 容器，是 Envoy 将 sidecar 容器 contour 视为管理服务器
>
> 初始化完成后， Envoy 容器启动，检索 contour 写入的引导程序配置，并开始轮询 contour 以进行配置
>
> Contour 是 Kubernetes API 的客户端，监视 Ingress、 Service、和 Endpoints 对象的变动，并将这些变动构建成JSON字符串，以此实现Envoy的配置动态更新，Envoy通过xDS机制来轮询contour做服务发现。

## 部署
1. clone项目到本地
```shell
git clone https://github.com/heptio/contour
```

2. 创建名称空间及自定义资源
默认情况下，contour要求svc为的type为LoadBalance类型才能正常工作，如果本地部署的化无法提供 `LoadBalance` 服务，所以选择 `hostnetwork` 方式工作
```shell
cd contour/examples/ds-hostnet

kubectl apply -f 01-common.yaml
namespace/heptio-contour created
serviceaccount/contour created
customresourcedefinition.apiextensions.k8s.io/ingressroutes.contour.heptio.com created
customresourcedefinition.apiextensions.k8s.io/tlscertificatedelegations.contour.heptio.com created
```

3. 创建RBAC规则
```shell
kubectl apply -f 02-rbac.yaml 
clusterrolebinding.rbac.authorization.k8s.io/contour created
clusterrole.rbac.authorization.k8s.io/contour created
```

4. 部署DaemonSet
* 修改监听端口
```shell
vim 02-contour.yaml
# 修改一下参数，将端口改为80
args: ["serve", "--incluster", "--envoy-service-http-port", "80", "--envoy-service-https-port", "443"]

```

* 提交部署
```shell
kubectl apply -f 02-contour.yaml
daemonset.apps/contour created
```

5. 部署contour service
```shell
kubectl apply -f 02-service.yaml
service/contour created
```

6. 查看部署情况
```shell
kubectl apply -f heptio-contour
NAME            READY   STATUS    RESTARTS   AGE
contour-8mjk9   2/2     Running   0          78m
```

## 测试
### 部署测试应用
* 创建deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myapp-dep
  name: myapp-dep
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-dep
  template:
    metadata:
      labels:
        app: myapp-dep
    spec:
      containers:
      - image: busybox:latest
        name: myapp-dep
        command: ["httpd","-f","-h","/etc/"]
```

```shell
kubectl apply -f myapp-dep.yaml
deployment.apps/myapp-dep created
```

* 创建svc
```yaml

apiVersion: v1
kind: Service
metadata:
  labels:
    app: myapp-svc
  name: myapp-svc
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: myapp-dep
  type: ClusterIP
```
```shell
kubectl apply -f myapp-dep-svc.yaml
service/myapp-svc created
```

### 创建Ingress
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.envoy.com
    http:
      paths:
      - path:
        backend:
          serviceName: myapp-svc
          servicePort: http
```
```shell
kubectl apply -f myapp-dep-ingress.yaml
ingress.networking.k8s.io/myapp-ingress created
```

### 绑定host测试
因为 `ingress controller` 使用与Node节点使用同一个网络名称空间，所以将解析指定到Node节点的IP地址即可

* 绑定host
```
192.168.21.201 myapp.envoy.com
```

* 测试
```
# myapp容器将/etc/作为http站点根目录，所以可以通过访问/etc/下的文件来验证

~]# curl myapp.envoy.com/hostname

myapp-dep-7bb5f65d4d-7ppkx
```
