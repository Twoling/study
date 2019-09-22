# RabbieMQ

RabbitMQ是实现了高级消息队列协议（AMQP）的开源消息代理软件（亦称面向消息的中间件）。RabbitMQ服务器是用Erlang语言编写的，而聚类和故障转移是构建在开放电信平台框架上的。所有主要的编程语言均有与代理接口通讯的客户端库

## 工作机制
* 生产者: 消息的创建者，负责创建消息并推动到消息系统
* 消费者: 消息的接收方，用于处理数据和确认消息
* 代理: RabbitMQ本身，用于扮演 "快递" 的角色，本身不生产消息，只是扮演 "快递" 的角色

### 消息发送
应用程序与MQ Server之前建立TCP连接，通过MQ的认证后，应用程序与MQ之间就会创建一条AMQP信道
信道是创建在TCP之前的一个虚拟连接，AMQP命令都是通过信道发送出去的，每个信道都会有一个卫衣的ID，不论是发送消息，订阅队列或者介绍消息都是通过信道完成的

[AMQP连接](!./images/rabbit_channel.png)

### 为什么直接通过TCP发送命令
> 对于操作系统来说创建和销毁TCP会话是非常昂贵的开销，假设高峰期每秒有成千上万条连接，每个连接都要创建一条TCP会话，这就造成了TCP连接的巨大浪费，而且操作系统每秒能创建的TCP也是有限的，因此很快就会遇到系统瓶颈。
>
> 如果我们每个请求都使用一条TCP连接，既满足了性能的需要，又能确保每个连接的私密性，这就是引入信道概念的原因。

## 什么事消息队列
消息队列(Message Queue) 是一种应用间的通信方式，应用将消息发送至消息系统后立即返回，由消息系统来确保消息的可靠传递，消息生产者将消息发布至消息系统中，消息消费者从消息系统中消费消息，而不需要了解消息生成者是谁。

## 名词
* ConnectionFactory: 连接管理器，应用程序与Rabbitmq之间建立连接的管理器
* Channel: 信道，消息推送使用的通道
* Exchange: 交换器，用于接受，分配消息
* Queue: 队列，用于存储生产者的生产的消息
* RoutingKey: 用于吧生产者的数据分配到交换器上
* BindingKey: 用于吧交换器的消息绑定到队列上


## 主要特性
* 可伸缩性: 集群服务
* 消息持久化: 从内存持久化消息到硬盘，再从硬盘加载到内存

## 消息持久化
默认情况下，RabbitMQ重启后会导致消息丢失，要开启持久化，需要在将消息发送至MQ的时候显式开启持久化，但要保证RabbitMQ的消息可以从奔溃中恢复，必须满足以下3个条件
* 投递消息时 durable 设置为 True

* 投递模式 deliveryMode 设置为2(持久)

* 消息已经到达持久化交换器上

* 消息已经到达持久化的队列

#### 持久化工作原理
RabbitMQ会将持久化的消息写入磁盘上的持久化日志文件，等消息被消费之后，RabbitMQ会把这条消息标识为等待垃圾回收

#### 持久化的缺点
消息持久化所带来的性能损耗显而易见，磁盘的写入远比内存消耗要高，因此开启了持久化后，RabbitMQ的吞吐量将大大降低，即使使用SSD磁盘，当千万条以上的数据写入磁盘的时候，性能上的损耗也是非常高的

## 虚拟主机
每个RabbitMQ都可以创建很多个vhost，每个虚拟主机都拥有自己的队列、交换器、和绑定，拥有自己的权限机制

vhost特性
1. RabbitMQ 默认的vhost是"/"开箱即用

2. 多个vhost是隔离的，多个vhost之间无法通讯，实现了多层分离

3. 创建用户的时候必须指定vhost


## 安装

1. 安装erlang
```
# 下载erlang
~]# wget https://packages.erlang-solutions.com/erlang/rpm/centos/7/x86_64/esl-erlang_22.0.7-1~centos~7_amd64.rpm

# 安装
~]# yum -y install ./esl-erlang_22.0.7-1_centos_7_amd64.rpm
```

2. 安装RabbitMQ

```
# 下载预编译二进制程序包
~]# wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.18/rabbitmq-server-generic-unix-3.7.18.tar.xz

# 解压并启动
~]# tar xf rabbitmq-server-generic-unix-3.7.18.tar.xz -C /opt/
~]# cd /opt/rabbitmq_server-3.7.18 
~]# sbin/rabbitmq-server  -detached  # -detached 后台启动

```

3. 开启web管理插件
```
~]# sbin/rabbitmq-plugins enable rabbitmq_management
```

4. 添加用户
```
~]# sbin/rabbitmqctl add_user test test
```

5. 将新添加的用户设置为管理员
```
~]# sbin/rabbitmqctl set_user_tags test administrator
```

6. 通过浏览器登陆web管理界面
```
http://IP-ADDR:15672
```

## 集群命令

1. 加入集群
```
# 停止应用(此步骤在要加入的节点上操作)
~]# sbin/rabbitmqctl stop_app

# 拷贝cookie文件(在现有的集群节点上操作)
~]# scp ~/.erlang.cookie root@cluster_node:/root/

# 新节点加入集群(此步骤在要加入的节点上操作)
~]# sbin/rabbitmqctl join --ram rabbit@node1

注： 集群节点之间主机名需要能解析
```

2. 查看集群状态
```
~]# sbin/rabbitmqctl cluster_status
Cluster status of node rabbit@drone ...
[{nodes,[{disc,[rabbit@loki]},{ram,[rabbit@drone]}]},
 {running_nodes,[rabbit@loki,rabbit@drone]},
 {cluster_name,<<"rabbit@loki">>},
 {partitions,[]},
 {alarms,[{rabbit@loki,[]},{rabbit@drone,[]}]}]
```


3. 退出集群
```
~]# sbin/rabbitmqctl stop_app
~]# sbin/rabbitmqctl reset
~]# sbin/rabbitmqctl start_app

# 再次查看状态
~]# sbin/rabbitmqctl cluster_status
Cluster status of node rabbit@drone ...
[{nodes,[{disc,[rabbit@drone]}]},
 {running_nodes,[rabbit@drone]},
 {cluster_name,<<"rabbit@drone">>},
 {partitions,[]},
 {alarms,[{rabbit@drone,[]}]}]
```

4. 强制移除集群中的节点
```
~]# sbin/rabbitmqctl forget_cluster_node
```

5. 改变节点类型
```
# 改变节点类型必须先停止节点
# 将RAM节点改变成disc节点
~]# sbin/rabbitmqctl stop_app
~]# sbin/rabbitmqctl change_cluster_node_type disc
~]# sbin/rabbitmqctl start_app

# 将disc节点改变为RAM节点，要改变的disc节点不能为集群中最后一个disc节点
~]# sbin/rabbitmqctl stop_app
~]# sbin/rabbitmqctl change_cluster_node_type ram
~]# sbin/rabbitmqctl start_app

```

6. 在集群中开启镜像功能
```
# 对指定的vhost开启ha mode
~]# sbin/rabbitmqctl set_policy --vhost test  ha-all "^" '{"ha-mode": "all"}'
Setting policy "ha-all" for pattern "^" to "{"ha-mode": "all"}" with priority "0" for vhost "test" ...

# 查看策略
~]# sbin/rabbitmqctl list_policies --vhost test
Listing policies for vhost "test" ...
vhost	name	pattern	apply-to	definition	priority
test	ha-all	^	all	{"ha-mode":"all"}	0
```


## 压测工具
#### RabbitMQ Performance Testing Tool
* 介绍: https://www.rabbitmq.com/java-tools.html

* 下载: https://github.com/rabbitmq/rabbitmq-perf-test/releases

#### 使用方法
需要有java环境才能使用

```
~]# bin/runjava  com.rabbitmq.perf.PerfTest --uri amqp://test:test@127.0.0.1:5672  -x10 -y10 -e"testex" -t"fanout" -u"testque" -k"rk01"
```

参数
* -x: 生产者数量
* -y: 消费者数量
* -e: exchange名称
* -t: exchange类型
* -u: queue名称
* -k: RoutingKey名称





