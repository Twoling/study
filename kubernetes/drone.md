# Drone简单使用
## Overviews
drone 是一个基于 golang 研发的 CI/CD 解决方案，相比于其他方案，比如 jenkins，其好处是更轻量，pipeline 中的每个 step 都是通过启动一个容器来进行，各step之间通过卷挂载来共享数据，可以通过很灵活的为每个 step 指定相应的容器，支持的各种插件也是以容器方式提供

## 认证管理
Drone 自身不负责用户管理，完全依赖外部版本控制系统，通过 token 或者 OAuth2 来进行用户鉴权，目前官方支持的版本管理系统
* GitHub
* Bitbucket Cloud
* Bitbucket Server
* GitLab
* Gitea
* Gogs

## 运行方式
Drone 本身并不负责运行任何构建任务，所有任务都是通过其他容器来运行。

### 在Kubernetes中运行方式
当构建任务被触发后，Drone 会在 Kubernetes 中创建一个 Job，该 Job 会创建一个 Pod，此 Pod 为一个 Drone 的任务控制器，后续的所有的构建任务都由此控制器控制，每次构建时， `drone/controller` 都会创建一个新的名称空间，每个步骤都会创建一个 `pod`，在`Pod` 中运行构建任务或者命令，通过 `volume` 进行数据共享