# Helm 学习
## Architecture
`Helm` 是一个用于管理 `Kubernetes` 工作负载的工具, 其将一系列的声明式文件以固定的格式组织成一个包, 称为 `charts`,  `Helm` 可以执行以下操作
  * 从头创建一个新的 `charts`
  * 将 `charts` 打包成 `charts` 归档格式(tgz)文件
  * 与 `charts` `repository` 交互
  * 将 `charts` 安装或卸载到现有的 `Kubernetes` 集群中
  * 管理使用 `Helm` 安装的 `charts` 的发布周期

对于 `Helm`, 有三个重要概念：
  1. `charts` 是创建 `Kubernetes` 应用程序所必须的一组信息
  2. `config` 包含的配置信息可以合并到 `charts` 中以用来创建应用程序
  3. 结合特定的配置, 每个 `release` 都是一个运行的 `charts` 实例

## 组件
**Helm Client**
`helm client` 是用户可以执行的的命令行客户端，其可以实现以下功能：
  * 本地 `charts` 开发
  * 管理 `charts` 仓库
  * 与 `Tiller Server` 进行交互
    * 发送要安装的 `charts`
    * 查询 `release` 的相关信息
    * 请求更新或者卸载存在的 `release`

**Tiller Server**
`Tiller server` 是一个运行在集群内的服务端程序, 与 `helm` 客户端交互，并与 `Kubernetes API` 连接, `Tiller server` 负责以下事项：
  * 监听来自 `helm` 客户端的请求
  * 结合 `charts` 和 `config` 来构建 `release`
  * 将 `charts` 安装到 `Kubernetes` 中, 然后追踪后续版本
  * 通过与 `Kubernetes` 交互来升级和卸载 `charts`

**简单来说，Helm客户端负责管理charts, Tiller server负责管理release**
