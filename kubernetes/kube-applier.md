# kube-applier 使用

## Overview
* `kube-applier`简单来说应该是一款部署工具，可以自托管在 `k8s` 平台，将部署的工作简化，其从 `git` 仓库拉取 `k8s` 各种工作负载定义到本地，并将定义应用到集群

## 工作原理
* 完整的 `kuba-appler` 有一个主容器 `kube-appler` 与一个 `sidecar` 容器 `git-sync` 容器组成
* `git-sync` 容器 :
	* 类似于 `jenkins Poll SCM` 的作用，轮询 `git` 仓库的变化，并同步到本地

* `kube-applier`容器：
	* 定期将本地 `git` 版本库中所有以 `.json` 于 `.yaml` 结尾的文件应用到集群中

|项目地址|https://github.com/box/kube-applier|
|:--:|:--:|


## 部署清单文件
* `kube-applier` 镜像需要手动构建
### 构建环境
|Go (1.7+)|
|:--|
|Docker (17.05+)|
|Kubernetes cluster|

#### clone 项目到本地
```shell
go get github.com/box/kube-applier
```

#### 构建镜像
```shell
cd $GOPATH/src/github.com/box/kube-applier
make container
```

### RBAC文件

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-applier
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kube-applier
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kube-applier
  namespace: kube-system
```

### deployment文件

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: kube-applier
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: kube-applier
    spec:
      serviceAccountName: kube-applier
      containers:
        - name: kube-applier
          command:
            - /kube-applier
          env:
            # 仓库路径，此路径下的所有以.yaml或.json结尾的文件都会被应用
            - name: "REPO_PATH"
              value: "/k8s/resources/apps/official"
           # 端口设定
            - name: "LISTEN_PORT"
              value: "2020"
          image: registry.iwgame.com/base/kube-applier:v0.2.0
          ports:
            - containerPort: 2020
          volumeMounts:
            # 挂载git仓库路径
            - name: git-repo
              mountPath: /k8s
        - name: git-sync
          command:
            - /git-sync
          env:
            # git仓库地址
            - name: "GIT_SYNC_REPO"
              value: "http://github.com/twoling/test-cd/yaml-templates.git"
           # 同步到本地哪个目录下
            - name: "GIT_SYNC_DEST"
              value: "resources"
          image: gcr.io/google_containers/git-sync:v3.1.1
          ports:
            - containerPort: 2020
          volumeMounts:
           # 挂载卷
            - name: git-repo
              mountPath: /root/git/
      volumes:
        # 卷定义，用于多容器见共享
        - name: git-repo
          emptyDir: {}
```
### deployment和ingress文件

```yaml
---
  apiVersion: "v1"
  kind: "Service"
  metadata:
    name: "kube-applier"
    namespace: "kube-system"
  spec:
    ports:
      - name: "service"
        port: 80
        targetPort: 2020
    selector:
      app: "kube-applier"
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kube-applier
  namespace: kube-system
  annotations:
    # use the shared ingress-nginx
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: kube-applier.kube.k8s.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kube-applier
          servicePort: 80
```
