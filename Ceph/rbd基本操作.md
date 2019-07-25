## Ceph RBD基本操作

### 创建块设备

* 命令格式：`rbd create --size {megabytes} {pool-name}/{image-name}`
```
~]# rbd create --size 10240 foo

# 如果省略pool-name则创建的块设备默认在名为rbd的pool中，可以在创建时指定pool的名称
~]# rbd create --size  10240 mytest/myrbd

# pool需要提前创建好
```

### 查看块设备镜像
* 命令格式：`rbd info {pool-name}/{image-name}`
```
~]# rbd info mytest/myrbd
```

### 将块设备映射到内核
* 命令格式：`rbd map {image-name} --pool {pool-name} 或 rbd map {image-name}/{pool-name}`
```
~]# rbd map mytest/myrbd 
# 可能会有以下报错
rbd: sysfs write failed
RBD image feature set mismatch. You can disable features unsupported by the kernel with "rbd feature disable mytest/myrbd object-map fast-diff deep-flatten".
In some cases useful info is found in syslog - try "dmesg | tail".
rbd: map failed: (6) No such device or address

# 禁用当前内核不支持的features重新映射即可
~]# rbd feature disable mytest/myrbd object-map fast-diff deep-flatten

~]# rbd map mytest/myrbd
/dev/rbd0
```

### 格式化挂载设备
* 此操作与常规磁盘格式化一致
```
~]# mkfs.xfs /dev/rbd0

~]# mount /dev/rbd0 /mnt/rbd0
```

### 取消块设备映射
```
1. 取消挂载
~]# umount /mnt/rbd0

2. 取消块设备映射
~]# rbd unmap mytest/myrbd 
```

### 创建快照
命令格式： rbd snap create {pool-name}/{rbd-name}@{snap-name}
```
~]# rbd snap create  mytest/myrbd@myrbd_snap0
```

#### 列出创建的快照
```
~]# rbd snap list mytest/myrbd

# 或者使用 rbd ls {pool-name} -l 列出
~]# rbd ls mytest -l
NAME               SIZE PARENT FMT PROT LOCK 
myrbd             10GiB          2      excl 
myrbd@myrbd_snap0 10GiB          2         

```
#### 查看快照详细信息
```
~]# rbd info mytest/myrbd@myrbd_snap0
rbd image 'myrbd':
	size 10GiB in 2560 objects
	order 22 (4MiB objects)
	block_name_prefix: rbd_data.10cd6b8b4567
	format: 2
	features: layering, exclusive-lock
	flags: 
	create_timestamp: Mon Jul  1 15:32:28 2019
	protected: False

```

### 克隆快照
> 快照必须处于被保护状态才能被克隆
* 执行命令`rbd snap protect {pool-name}/{image-name}@{snapshot-name}`将快照置成保护状态：

```
~]# rbd snap protect mytest/myrdb@myrdb_snap0
```