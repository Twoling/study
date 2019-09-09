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

### 选举
每个节点启动的时候动是 `LOOKING` 状态
`Leader` 节点运行后会周期性的向 `Follower` 发送心跳信息，如果一个 `Follower` 未收到 `Leader` 节点的心跳信息，`Follower` 节点的状态会从 `Following` 转变为 `Looking`
