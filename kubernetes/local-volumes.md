# 本地卷
---

## Requirements
* 本地卷插件期望路径稳定，包括重新启动以及添加或删除磁盘的时间
* 静态配置程序仅发现挂载点，对于基于目录的本地卷，必须将它们绑定到发现目录中。

## 术语：
* discovery directory： 主机上的目录，provisioner可以从中发现filesystem 或者 block PV
* provisioner：  local-volume-provisioner程序
* local PV： kubernetes本地持久卷
* filesystem local PV： 具有文件系统的本地pv卷
* block local PV： 块模式的本地卷

## 步骤

### 创建 discovery directoroy
```
~]# mkdir -p /mnt/disks/
```
注意:
* 此目录可以为任意位置
* 此目录在provisioner配置中的`hostDir`字段中配置，并且只能为一个存储类配置
* 如果要配置多个本地存储类，请为每个存储类创建一个目录


### 在发现目录中准备和设置本地卷
准备好发现目录后，可以设置`provisioner`发现的本地卷
#### 使用整个磁盘作为filesystem PV
1. 格式化安装
```
$ sudo mkfs.xfs /dev/path/to/disk
$ DISK_UUID=$(blkid -s UUID -o value /dev/path/to/disk) 
$ sudo mkdir /mnt/disks/$DISK_UUID
$ sudo mount -t xfs /dev/path/to/disk /mnt/disks/$DISK_UUID
```

2. 写入 /etc/fstab
```
~]# echo "UUID=$(blkid -s UUID -o value /dev/path/to/disk) /mnt/disks/$DISK_UUID xfs defaults 0 0" | tee -a /etc/fstab 
```
注意：
* 您还可以根据需要调整文件系统和挂载选项
* 最好在fstab条目和挂载点的目录名中使用UUID
* 如果需要IO隔离，最好使用整个磁盘

#### 通过多个文件系统PV共享磁盘文件系统

#### 将设备链接到块PV发现目录中
#### 将磁盘分成多个分区

### 删除/删除基础卷
