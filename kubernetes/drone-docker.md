# Drone 
以Docker的方式运行Drone(Kubernetes集成方案尚不成熟，各种问题还有待官方解决)

[Drone介绍](./drone.md)

---
## 目录:
* [安装 Server端](#安装Server)
  * [生成密钥](#生成密钥)
  * [认证配置](#创建OAuth2应用)
  * [配置](#配置)
* [安装 Agent端](#安装Agent)
  * [配置](#配置Agent)
* [插件使用](#插件)
  * [常用插件列表](#常用插件列表)
  * [docker插件](#docker插件)
  * [DingTalk](#DingTalk插件)
  * [Wechat](#Wechat插件)
  * [Volume Cache](#Volume-Cache插件)

## 安装Server
### 生成密钥
生成密钥用于客户端连接RPC连接使用
```bash
~]# openssl rand -hex 16
b244ea22eee4f53163c0727e46431d50
```

### 创建OAuth2应用
在 `gitea` 上创建 `OAuth2` 应用
1. 
![drone](./images/drone-docker-step-1.png)

2. 
![drone](./images/drone-docker-step-2.png)

3. 
![drone](./images/drone-docker-step-3.png)

注: 这里的重定向 `URL` 要后面的 `/login` 格式为固定，前面的域名可以按照需求自定义

4. 
![drone](./images/drone-docker-step-4.png)

### 配置
按照上面的得到的值替换相关变量的值

```bash
docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_AGENTS_ENABLED=true \
  --env=DRONE_GITEA_SERVER=${GITEA_SERVER_ADDR_OR_Domain} \
  --env=DRONE_GITEA_CLIENT_ID=896d8f26-ddf8-410c-aff0-20bfa49e6f3a \
  --env=DRONE_GITEA_CLIENT_SECRET=KhfiCDzFEb_1LcHcBmHnZLD0gHKWTJTo6HxZ7m-iuCc= \
  --env=DRONE_RPC_SECRET=b244ea22eee4f53163c0727e46431d50 \
  --env=DRONE_SERVER_HOST=${DRONE_SERVER_HOST} \
  --env=DRONE_SERVER_PROTO=http \
  --publish=80:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:1

```
注: 以上 `DRONE_GITEA_SERVER` `DRONE_SERVER_HOST` `DRONE_SERVER_PROTO` 等变量的值，根据自身环境来进行替换

## 安装Agent
Drone agent端称为 `Runner`，`Runner` 的种类有3种，分别是
* Docker Runner
* Exec Runner
* SSH Runner
本文主要讲解 `Docker Runner` 的安装与使用

### 配置Agent
安装 `Docker Runner` 非常简单，将官方提供的 `Docker image` 拉取到本地，再将 `Drone Server` 的相关信息以环境变量的方式注入到 `Runner` 中即可

```bash
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e DRONE_RPC_PROTO=http \
  -e DRONE_RPC_HOST=${DRONE_SERVER_HOST} \
  -e DRONE_RPC_SECRET=b244ea22eee4f53163c0727e46431d50 \
  -e DRONE_RUNNER_CAPACITY=2 \
  -e DRONE_RUNNER_NAME=drone \
  -p 3000:3000 \
  --restart always \
  --name runner \
  drone/agent:1
```
注: 以上 `DRONE_RPC_SECRET` `DRONE_RPC_HOST` `DRONE_RPC_PROTO` 等变量的值，要根据上面 `Drone Server` 的配置信息来替换成相应的值

## 插件
### 常用插件列表
* docker
* DingTalk
* Wechat
* Volume-Cache

### docker插件
* dockerfile内容
```
FROM busybox:latest

RUN test -d /data/ || mkdir /data 

RUN ln -s /etc/hostname /data/hostname.html

COPY . /data/

CMD ["httpd","-h","/data/","-f"]
```

* `.dockerignore` 文件忽略一些文件
```
.drone.yml
README.md
Dockerfile
```

* `.drone.yml` 文件示例:
```yaml
kind: pipeline
type: docker
name: build

steps:
- name: docker
  image: plugins/docker
  settings:
    username:
      from_secret: docker_username
    password:
      from_secret: docker_password
    # 默认是dockerhub，可以改成私有仓库地址
    registry: registry.server.com  
    # 仓库标识加镜像名
    repo: myapp/myapp
    # 根据BUILD ID来作为tag
    tag: ${CI_BUILD_NUMBER}
    dockerfile: Dockerfile
```
参考: http://plugins.drone.io/drone-plugins/drone-docker/

* 提交测试
![docker-plugin](./images/docker-plugin-build.png)

### DingTalk插件
* 在钉钉中添加自定义机器人

* 将机器人 `token` 保存为`Drone Secret`

* `.drone.yml` 文件内容
```yaml
kind: pipeline
type: docker
name: build

steps:
- name: docker
  image: plugins/docker
  settings:
    username:
      from_secret: docker_username
    password:
      from_secret: docker_password
    # 默认是dockerhub，可以改成私有仓库地址
    registry: registry.server.com  
    # 仓库标识加镜像名
    repo: myapp/myapp
    # 根据BUILD ID来作为tag
    tag: ${CI_BUILD_NUMBER}
    dockerfile: Dockerfile

# 钉钉通知
- name: dingtalk
  image: lddsb/drone-dingtalk-message
  settings:
    token: 
      from_secret: ding_token
    type: markdown
    message_color: true
    message_pic: true
    sha_link: true
  when:
    status:
    - success
    - failure

```

* 测试

![dingTalk](./images/dingtalk-plugin-notify.png)

参考: http://plugins.drone.io/lddsb/drone-dingtalk-message/

### Wechat插件
* 在企业微信中添加自定义应用

* 将企业`corpid`与应用`secret`保存为`Drone Secret`

* `.drone.yml` 文件内容
```yaml
kind: pipeline
type: docker
name: build

steps:
- name: docker
  image: plugins/docker
  settings:
    username:
      from_secret: docker_username
    password:
      from_secret: docker_password
    # 默认是dockerhub，可以改成私有仓库地址
    registry: registry.server.com  
    # 仓库标识加镜像名
    repo: myapp/myapp
    # 根据BUILD ID来作为tag
    tag: ${CI_BUILD_NUMBER}
    dockerfile: Dockerfile

- name: wechat
  image: lizheming/drone-wechat
  settings:
    corpid: 
      from_secret: corpid
    corp_secret:
      from_secret: corp_secret
    agent_id: 
      from_secret: agent_id
    to_user: "@all"
    to_party: 1
    to_tag: ${DRONE_REPO_NAME}
    msg_url: ${DRONE_BUILD_LINK}
    safe: 1
    btn_txt: more
    title: ${DRONE_REPO_NAME}
    message: >
      {%if success %}
        build {{build.number}} succeeded. Good job.
      {% else %}
        build {{build.number}} failed. Fix me please.
      {% endif %}
  when:
    status:
    - success
    - failure

```

* 测试

![wechat](./images/wechat-plugin-notify.png)

参考: http://plugins.drone.io/lizheming/drone-wechat/
