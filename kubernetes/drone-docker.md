# Drone 
以Docker的方式运行Drone(Kubernetes集成方案尚不成熟，各种问题还有待官方解决)

[Drone介绍](./drone.md)

---
## 目录:
* [安装](#安装)
  * [生成密钥](#生成密钥)
  * [认证配置](#创建OAuth2应用)
  * [配置](#配置)
* [插件使用](#插件)
  * [常用插件列表](#常用插件列表)
  * [docker插件](#docker插件)
  * [DingTalk](#DingTalk插件)
  * [Wechat](#Wechat插件)
  * [Volume Cache](#Volume-Cache插件)

## 安装
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