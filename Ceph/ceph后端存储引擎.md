# Ceph 后端存储引擎

## FileStore
### 读流程
1. ceph 客户端通过计算得出主 OSD，直接给主 OSD 发送消息

2. 主 OSD 受到消息后，调用 FileStore 直接读取处在底层文件系统中主 pg 里的内容然后返回给客户端

### 写
1. 客户端发送数据至主 OSD

2. 主 OSD 要进行写操作预处理，完成后发送写消息至其他从 OSD，让他们对副本 pg 进行更改

3. 从 OSD 通过FileJournal完成写操作到Journal中后发送消息告诉主 OSD 

4. 当主 OSD 收到所有的从 OSD 完成写操作的消息后，会通过FileJournal完成自身的写操作到Journal中。完成后会通知客户端，已经完成了写操作

5. 主 OSD，从 OSD 的线程开始工作调用Filestore将Journal中的数据写入到底层文件系统中。

## BlueStore
BlueStore是在底层裸设备上建立存储系统，内建了 `RocksDB k/v` 数据库用于管理内部元数据，一个小型的内部接口组件 `BlueFS` 实现了类似文件系统的接口，以便提供足够功能让 `RocksDB` 来存储它的数据并向 `BlueStore` 共享相同的裸设备


### 多设备
`BlueStore` 可以组合慢速和快速设备，类似 `FileStore`， 更多的使用快速设备，在 `FileStore` 中，日志设备只用于写，通常位于较快的 `SSD`。

`BlueStore` 最多可以管理3种存储设备
1. 最简单的情况，`BlueStore` 使用单个存储设备，