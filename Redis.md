# Redis
#### Redis安装：
1. 下载Redis：
	* wget http://download.redis.io/releases/redis-4.0.2.tar.gz
2. 解压并安装：
	* tar xf redis-4.0.2.tar.gz
	* cd redis-4.0.2
	* make
3. 启动Redis
	* src/redis-server
#### Redis主从复制：

##### 环境：

---
| 主    | 172.18.54.3 |
| :-----| -----------:|
| 从    |   172.18.54.123 |
| 从    |   172.18.54.125 |

`所有主机redis均安装在/opt/目录下`

---
##### 配置主从：
##### 主：
1. 编辑master配置文件，修改一下参数：
	配置文件为redis.conf
```
bind 0.0.0.0
#绑定监听地址
protected-mode no
#关闭保护模式，如果不关闭必须设置redis认证密码，否则无法远程连接redis
daemonize yes
#是否为守护模式运行；
logfile "/var/log/redis_6379.log"
#日志文件位置，如果为空，前台运行日志输出到标准输出，后台运行输出到/dev/null
min-slaves-to-write 1
#根据主从数量修改，从节点少于此处指定的值，主节点不可写；
requirepass 123 
#设置redis认证
```

2. 启动Redis
`src/redis-server  redis.conf`

---
*默认Redis监听端口：6379*
*可通过`ss -tanl`查看端口是否监听或通过`redis-cli` 去连接redis检测redis运行状态。*

---

##### 从：
1. 编辑配置文件
```
bind 0.0.0.0
#绑定监听地址
protected-mode no
#关闭保护模式，如果不关闭必须设置redis认证密码，否则无法远程连接redis
daemonize yes
#是否为守护模式运行；
logfile "/var/log/redis_6379.log"
#日志文件位置，如果为空，前台运行日志输出到标准输出，后台运行输出到/dev/null
slaveof 172.18.54.3 6379
#指明master的地址及端口
masterauth 123
#指定连接master时的认证密码；
```

2. 启动Redis
` src/redis-cli  redis.conf`

##### 从master上查看主从状态：
```
~]# redis-cli
127.0.0.1:6379> AUTH 123
OK
127.0.0.1:6666> INFO REPLICATION
# Replication
role:master
connected_slaves:2
min_slaves_good_slaves:2
slave0:ip=172.18.54.125,port=6379,state=online,offset=9044235,lag=0
slave1:ip=172.18.54.123,port=6379,state=online,offset=9044094,lag=1
master_replid:1b7f4cf531d6185ef810b61cee9537a150cc064b
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:9044376
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:8972142
repl_backlog_histlen:72235


```

##### 测试主从复制：
1. 在主服务器上设置一个Key
```
~]# redis-cli
127.0.0.1:6379> SET tom 100

```

2. 在各从服务器上测试：
```
~]# redis-cli 
127.0.0.1:6379> GET tom
"100"

```

### Redis Sentinel：
* Redis-Sentinel是Redis官方推荐的高可用解决方案，它能监控多个Redis集群。当使用sentinel做redis主从进群的高可用方案时，master宕机时，sentinel会自动从集群中的slave中选出一个提升其为新的master，Redis-sentinel是一个单独运行的程序，所以说其并不会受redis宕机的影响。
* 主要功能：
	* 定期监控redis是否按照预期良好运行；
	* 如果发现某个redis节点运行出现状况，能够通知其他进程；
	* 能够自动进行failover；
	
---
* Sentinel集群：显然，如果只是用单个sentinel进程来监控整个集群，那sentinel也会存在单点问题；因此，sentinel也支持集群。


#### 配置Redis-Sentinel
1. 编辑Sentinel的配置文件：
```
bind 0.0.0.0
#配置sentinel监听地址
port 26379
#配置sentinel监听端口(默认26379)
sentinel monitor mymaster 172.18.54.3 6379 1
#配置需要监控的redis集群信息；
#格式: sentinel monitor <master-name> <ip> <redis-port> <quorum>
#quorum：当网络发生阻塞或暂时性不可达时，可能会发生误判认为master宕机了；而quorum则定义了只有指定数量的sentinel节点认为master宕机了，才能真正认为master不可用了；(sentinel集群中的各个节点也能互相通信，通过gossip协议。)	
sentinel auth-pass 123
#设置master认证密码；
#格式：sentinel auth-pass <master-name> <password>



```
