# HDFS (Hadoop Distributed File System)
> HDFS(Hadoop分布式文件系统)是Apache Hadoop项目的一个子项目，被设计成运行在通用硬件上的分布式文件系统，HDFS有着高容错性的特点，并且设计用来部署在低廉的硬件上，而且提供高吞吐量来访问应用程序的数据，适合那些有着超大数据集的应用程序，HDFS放宽了POSIX的要求，这样可以实现流的形式访问文件系统中的数据。


* MapReduce是Hadoop的核心，HDFS是提供这些能力的基础


[HDFS Architecture](./hdfsarchitecture.png)

## HDFS架构主要由一下几个重要组件所组成
### Name Node
* Name Node(NN)是HDFS的核心组件，管理整个HDFS集群的读写等操作，整个HDFS集群只有一个NN，因此，Name Nodes是一个单点，

### Secondary Name Node
### Backup Name Node
### Data Node

* HDFS读操作：
	* 当客户端想要读取一个文件的时候，客户端需要先于Name Node节点进行交互，因为NN节点是存储着整个集群元数据的节点，NN节点为数据分配存储具体的DN和位置，客户端需要通过NN找到它所需要的数据存放的DN节点，然后直接于DN节点中进行读操作，考虑到安全和授权的目的，NN会给客户端提供token，这个token需要发送给DN节点进行认证，认证通过后，才可以读取文件。

	* 这个过程中，NN节点会检查客户端是否有足够的权限去访问这组数据，如果拥有权限，NN会将文件的存储路径返回给客户端，于此同时，NN也会将用于权限检查的token返回给客户端

	* DN节点在检查token之后，允许客户端读取特定的block，一个客户端打开一个输入流开始从DN读取数据。
	* 如果在读取数据期间DN节点突然下线，这个客户端会继续访问NN，此时NN会给出这个数据的副本节点位置

* HDFS写操作：
	* 读取文件需要访问NN节点，写文件也是如此，客户端先于NN节点交互，NN节点为客户端提供可以写入数据的DN节点的地址。
	* 一个文件同时只能由一个写来保障数据一致性，不允许多个线程同时写。