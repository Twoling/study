# zookeeper
---

## Overview
zookeeper 是一种用于分布式应用程序的分布式开源协调服务

## ZAB 协议
`ZAB` 协议 `(Zookeeper Atomic Broadcast Protocol 原子广播)` 是专门为 `zookeeper` 设计的支持崩溃恢复的原子广播协议，在 `zookeeper` 中，主要依赖ZAB协议来实现分布式数据一致性，基于该协议，`zookeeper `实现了一种基于主备的系统架构来保证集群中的各个副本之间的数据一致性

`ZAB` 协议运行过程中，所有客户端更新都发完 `Leader`， `Leader` 写入本地日志后再复制到所有的 `Follower` 节点， 一旦 `Leader` 节点故障无法工作，`ZAB` 协议能够自动从 `Follower` 节点中重新选择出一个合适的替代者，这个过程被称为选主。

### 服务器状态
* `Looking`: 不确定 `Leader` 状态，该状态下的服务器认为当前集群中没有 `Leader` 会发起 `Leader` 选举

* `Following`: 跟随者状态，表明当前服务器角色是 `Follower`，并且它知道 `Leader` 是谁

* `Leading`: 领导者状态，表明当前服务器角色是 `Leader`， 它会维持与 `Follower` 间的心跳

* `Observing`: 观察者状态，表明当前服务器是 `Observer`，与 `Follower` 唯一的不同时 `Follower` 节点不参与选举，也不参与集群写操作时的投票

### 每个服务器的角色
zookeeper集群是一个基于主从复制的高可用集群，集群中的每个服务器都可以承担一下三种角色的一中
* `Leader`: 一个zookeeper集群同一时间只会有一个实际工作的`Leader`，所有写操作必须要通过`Leader`完成再有`Leader`将写操作广播给其他服务器。
* `Follower`: 一个zookeeper集群可能同时存在多个`Follower`，它会响应 `Leader` 的心跳，`Follower` 可以直接处理并返回客户端的读请求，同时会将客户端的写请求转发给 `Leader` 处理，并且负责在 `Leader` 处理写请求时对请求进行投票
* `Observer`: 角色与 `Follower` 类似，但无投票权

### 写 Leader

1. 客户端向 `Leader` 发起写请求

2. `Leader` 将写请求以 `Proposal` 的形式发送给所有的 `Follower` 并等待 ACK

3. `Follower` 收到 `Proposal` 后返回 ACK

4. `Leader` 得到超过半数的 ACK 后向所有的 `Follower` 与 `Observer` 发送 `commit`

5. `Leader` 将处理结果返回给客户端

#### 需要注意的点
* `Leader` 并不需要得到 `Observer` 的ACK

* `Leader` 并不需要得到所有 `Follower` 的 ACK，只要收到半数的 `ACK`	 即可，同时 `Leader` 本身对自己有一个 ACK

* `Observer` 虽然无投票权，但仍须同步 `Leader` 的数据从而在处理读请求时可以返回尽可能新的数据

### 写 Follower/Observer

* `Follower`/`Observer`均可接受写请求，但不能直接处理，会将其转发给 `Leader` 节点，后续的操作都有 `Leader` 节点进行

### 读操作
* `Leader`/`Follower`/`Observer`都可直接处理读请求，从本地内存中读取数据并返回给客户端即可


### 选举过程
> 每个节点启动的时候动是 `LOOKING` 状态
> `Leader` 节点运行后会周期性的向 `Follower` 发送心跳信息，如果一个 `Follower` 未收到 `Leader` 节点的心跳信息，`Follower` 节点的状态会从 `Following` 转变为 `Looking`

#### 投票节点都处于 LOOKING 状态
1. 初始化投票

2. 更新选票

3. 根据选票确定角色

#### Follower 节点重启选举
1. 投票给自己

2. 发现已有 `Leader` 后成为 `Follower` 

#### Leader 节点重启选举

1. `Follower` 发起新投票

2. 广播更新选票

3. 选出新 `Leader`

4. 旧 `Leader` 恢复后发起选举

5. 旧 `Leader` 成为 `Follower`























