## Nginx容器配置重载方案
* 此方案只适用于 `Kubernetes` 环境下的 `Nginx` 容器配置文件重载

#### Overview
* 此方案使用一个 `sidecar` 容器检测 `configmap` 挂摘目录的变更来发现配置变更，通过脚本来向 `Nginx` 容器发送 `HUP` 信号让 `Nginx` 重载配置文件
* 检测与信号发送都是由 `sidecar` 容器中的脚本控制
* `sidecar` 容器通过 `busybox` 构建

#### 要求
* 配置要以 `configmap` 形式提供
* 所有配置均放在一个 `configmap`
* 主容器与 `sidecar` 容器共享进程名称空间

#### 构建 `sidecar` 容器
* Dockerfile

```Dockerfile
FROM busybox:latest

COPY entrypoint.sh /

CMD ["/entrypoint.sh"]
```

* entrypoint 脚本

```bash
#!/bin/sh
#
# Author: LYL
# Date: 2019-08-07
# Version: 0.1
set -e
CHECK_INTERVAL=${CHECK_INTERVAL:-1}
PROG='nginx: master process'
NUM=0

if [ -z ${GIT_REPO_DIR} ];then
        echo "GIT_REPO_DIR value is empty"
        exit 127
fi

if [ ! -d ${GIT_REPO_DIR} ];then
        echo "$GIT_REPO_DIR no such file or directory."
        exit 127
fi

while true;do
        NGX_PID=$(pgrep "${PROG}")
        if [ -z $NGX_PID ];then
        	continue
        else
        	echo "Nginx Pid: ${NGX_PID}"
        	break
        fi
done

while true;do
		DIR=$(cd $GIT_REPO_DIR && find ./ -regex '\./\.\.[0-9]\+_.*[0-9]$')
        INIT_VERSION=$(basename $DIR)
        echo "Now Config Dir: ${INIT_VERSION}"

        while true;do
                DIR=$(cd $GIT_REPO_DIR && find ./ -regex '\./\.\.[0-9]\+_.*[0-9]$')
                NEW_VERSION=$(basename $DIR)
                if [ $INIT_VERSION == $NEW_VERSION ];then
                        sleep $CHECK_INTERVAL
                        continue
                else
          				echo "New Config Dir: ${NEW_VERSION}"
                        let RUN=$NUM+1
                        NUM=$RUN
                        kill -HUP $NGX_PID
                        echo "Reload Run: ${RUN}"
                        break
                fi
        done
done
set +e

```

* 构建镜像

```bash
docker build -t nginx-config-reload:v0.0.1 .
```

#### 示例

* `ConfigMap` 定义

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-vhosts-config
  namespace: default
data:
  myapp.iwgame.com.conf: |
    server {
            listen  80;
            server_name  myapp.iwgame.com;
            root /www/myapp.iwgame.com/;
            #error_page 404 400 403 500 = /404.html;
            index index.html index.php index.htm;
            location / {
                return 401;
            }
    }
```

* 定义ngnix容器

```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx-svc
  namespace: default
spec:
  ports:
  - name: web
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx-deploy
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deploy
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
    # 共享名称空间
      shareProcessNamespace: true
      containers:
      - image: nginx:stable
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        # 挂摘configmap存储卷
        - name: nginx-vhosts-config
          mountPath: /etc/nginx/conf.d/
        resources:
          limits:
            cpu: "1000m"
            memory: "1024Mi"
          requests:
            cpu: "300m"
            memory: "128Mi"
        livenessProbe:
          tcpSocket:
            port: 80
        readinessProbe:
          tcpSocket:
            port: 80
      - image: nginx-config-reload:v0.0.1
        name: config-reload
        env:
          # 定义configmap挂载路径
          - name: "GIT_REPO_DIR"
            value: "/git/configs/"
        volumeMounts:
        # 挂载configmap
        - name: nginx-vhosts-config
          mountPath: /git/configs/
        imagePullPolicy: IfNotPresent
        securityContext:
          capabilities:
            add:
            - SYS_PTRACE
      volumes:
      # 定义存储卷
      - name: nginx-vhosts-config
        configMap:
          name: nginx-vhosts-config
```

* 部署

```bash
kubectl apply -f nginx-configmap.yaml
kubectl apply -f nginx-deployment.yaml
```

* 查看部署

```bash
kubectl get pods -o wide
```

```
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE              NOMINATED NODE   READINESS GATES
nginx-deploy-849dc489d9-jmbzf   2/2     Running   0          4m7s    20.1.3.251   k-n2.test.com   <none>           <none>
nginx-deploy-849dc489d9-mcd6h   2/2     Running   0          4m29s   20.1.3.250   k-n2.test.com   <none>           <none>
nginx-deploy-849dc489d9-xjk5h   2/2     Running   0          4m19s   20.1.5.77    k-n1.test.com   <none>           <none
```

* 测试访问

```bash
# 随便找一个容器测试页面
curl 20.1.3.251
```

会返回以下`401`页面，因为上面的`configmap`定义所有请求均返回`401`
```
<html>
<head><title>401 Authorization Required</title></head>
<body>
<center><h1>401 Authorization Required</h1></center>
<hr><center>nginx/1.16.0</center>
</body>
</html>
```

* 修改`configMap`测试自动重载

```bash
# 将401修改为403测试
kubectl edit configmaps  nginx-vhosts-config
configmap/nginx-vhosts-config edited
```

* 等待一段时间再次测试访问，这个时间取决于`configmap`注入容器变更间隔

```bash
curl 20.1.3.251
```
可以看到此时返回的结果为`403`，此配置重载操作是有`sidecar`容器完成，不需要运维人员手动进入容器执行`nginx -s reload`
```
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.16.0</center>
</body>
</html>
```

##### 默认`sidecar`容器的检测间隔为1秒，可以通过环境变量`CHECK_INTERVAL`指定检测间隔
