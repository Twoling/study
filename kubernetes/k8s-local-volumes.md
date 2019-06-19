# 本地卷
项目地址：https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner/
## Kubernetes本地卷
* kubernetes本地卷在1.14转为GA状态，本地持久卷允许用户以简单且可移植的方式通过标准的PVC接口访问本地存储，PV包含节点关联信息，系统使用这些信息将容器调度到正确的节点上。

### emptyDir
* 临时存储卷，emptyDir的生命周期与Pod的生命周期相同，emptyDir无法脱离Pod对象提供数据存储能力，因此emptyDir通常用于数据缓存或临时存储，也可以基于emptyDir将gitRepo构建与Pod中，从而使它具有一定意义上的持久性。

### hostPath
* hostPath类型是映射node文件系统中的文件或者目录pod里，拥有独立于Pod资源的生命周期，因此具有持久性。

### local volume
* 本地持久存储卷
* 本地卷静态配置程序通过检测和创建主机上每个本地磁盘的PV并在释放时清理磁盘来管理预分配磁盘的PersistentVolume生命周期。它不支持动态配置。

---
## Overview
* 本地持久卷有一个外部静态供应者(provisioner)来简化本地卷的管理工作，本地卷不支持动态配置，它要求管理员在每个节点上预先配置好本地卷，并且判断卷是否是一下类型
	* `Filesystem volumeMode(default) PVs`	: 挂载到发现目录下
	* `Block volumeMode PVs`					: 在发现目录下为节点上的块设备创建符号链接
* 静态配置程序(provisioner)将通过发现目录管理节点上的本地卷，包括为每个卷创建和清理`PersistentVolumes`

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

#### 将一个磁盘的多个目录绑定到发现目录

1. 格式化
```
~]# mkfs.xfs /dev/path/to/disk
~]# DISK_UUID=$(blkid -s UUID -o value /dev/path/to/disk)
~]# mkdir /mnt/$DISK_UUID
~]# ount -t ext4 /dev/path/to/disk /mnt/$DISK_UUID
```
**不要将磁盘挂载到发现目录中**

2. 将分区写入到`/etc/fstab`
```
UUID=`sudo blkid -s UUID -o value /dev/path/to/disk` /mnt/$DISK_UUID xfs defaults 0 0 | sudo tee -a /etc/fstab
```

3. 创建目录挂载到发现目录中
```
for i in $(seq 1 10); do
  mkdir -p /mnt/${DISK_UUID}/vol${i} /mnt/disks/${DISK_UUID}_vol${i}
  mount --bind /mnt/${DISK_UUID}/vol${i} /mnt/disks/${DISK_UUID}_vol${i}
done
```

4. 将挂载信息写入/etc/fstab
```
for i in $(seq 1 10); do
  echo /mnt/${DISK_UUID}/vol${i} /mnt/disks/${DISK_UUID}_vol${i} none bind 0 0 | sudo tee -a /etc/fstab
done
```
**共享一个磁盘的本地PV具有相同的容量，没有容量隔离，可以将一个磁盘分成多个分区来进行容量隔离**

#### 将磁盘分成多个分区

1. 分区

```
parted --script /dev/path/to/disk \
    mklabel gpt \
    mkpart primary 1MiB 1000MiB \
    mkpart primary 1000MiB 2000MiB \
    mkpart primary 2000MiB 3000MiB \
    mkpart primary 3000MiB 4000MiB

parted /dev/path/to/disk print
```

2. 重复上面的挂载步骤将分区挂载至本地卷发现目录


#### 将设备链接到块PV发现目录中
* 如果要直接使用块设备，只需将他们链接到发现目录即可
* 必须使用设备的唯一的设备路径

1. 查看设备的唯一路径

```
~]# ls /dev/disk/by-id
lrwxrwxrwx 1 root root  9 Jun  8 12:05 /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_drive-scsi0 -> ../../sda
lrwxrwxrwx 1 root root 10 Jun  8 12:05 /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_drive-scsi0-part1 -> ../../sda1
lrwxrwxrwx 1 root root 10 Jun  8 12:05 /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_drive-scsi0-part2 -> ../../sda2
lrwxrwxrwx 1 root root  9 Jun  8 12:05 /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_drive-scsi1 -> ../../sdb
```

2. 将块设备链接至发现目录

```
~]# ln -sv /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_drive-scsi1 /mnt/disks/
```

### 删除/删除基础卷流程
1. 停止正在使用此卷的Pod
2. 从节点上移除本地卷(卸载卷、删除挂载信息、移除磁盘) 
3. 删除PVC
4. Privisioner将尝试清理卷，但因为卷不存在会失败
5. 手动删除PV对象



# 部署

1. 部署`StorageClass`

```
# Only create this for K8s 1.9+
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-local-storage
  namespace: storage-class
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
# Supported policies: Delete, Retain
reclaimPolicy: Delete

```

2. 部署`Provisioner`简化管理

```
---
# Source: provisioner/templates/provisioner.yaml
 
apiVersion: v1
kind: ConfigMap
metadata:
  name: local-provisioner-config 
  namespace: storage-class 
data:
  storageClassMap: |     
    managed-local-storage:							# 这里填写上面创建的存储类的名称
       hostDir: /local-volumes/						# 发现目录
       mountDir:  /local-volumes/					# 与发现目录保持一致
       blockCleanerCommand:
         - "/scripts/shred.sh"
         - "2"
       volumeMode: Filesystem
       fsType: xfs
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: local-volume-provisioner
  namespace: storage-class
  labels:
    app: local-volume-provisioner
spec:
  selector:
    matchLabels:
      app: local-volume-provisioner 
  template:
    metadata:
      labels:
        app: local-volume-provisioner
    spec:
      serviceAccountName: local-storage-admin
      containers:
        - image: "registry.iwgame.com/external_storage/local-volume-provisioner:v2.1.0"
          imagePullPolicy: "Always"
          name: provisioner 
          securityContext:
            privileged: true
          env:
          - name: MY_NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          volumeMounts:
            - mountPath: /etc/provisioner/config 
              name: provisioner-config
              readOnly: true             
            - mountPath:  /local-volumes/									# 容器中的挂载目录，最好与外部目录保持一致
              name: fast-disks	
              mountPropagation: "HostToContainer" 
      volumes:
        - name: provisioner-config
          configMap:
            name: local-provisioner-config         
        - name: fast-disks
          hostPath:
            path: /local-volumes/											# 发现目录

---
# Source: provisioner/templates/provisioner-service-account.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: local-storage-admin
  namespace: storage-class

---
# Source: provisioner/templates/provisioner-cluster-role-binding.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: local-storage-provisioner-pv-binding
  namespace: storage-class
subjects:
- kind: ServiceAccount
  name: local-storage-admin
  namespace: storage-class
roleRef:
  kind: ClusterRole
  name: system:persistent-volume-provisioner
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: local-storage-provisioner-node-clusterrole
  namespace: storage-class
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: local-storage-provisioner-node-binding
  namespace: storage-class
subjects:
- kind: ServiceAccount
  name: local-storage-admin
  namespace: storage-class
roleRef:
  kind: ClusterRole
  name: local-storage-provisioner-node-clusterrole
  apiGroup: rbac.authorization.k8s.io

``` 

3. 等待所有节点的`provisioner`都正常运行后，使用`kubectl get pv`就可以看到由`provisioner`创建的本地卷

```
~]# kubectl get pv
local-pv-13ce11dd                          1014Mi     RWO            Delete           Available                                                              managed-local-storage            7m41s
local-pv-270e7c6e                          5110Mi     RWO            Delete           Available                                                              managed-local-storage            7m42s
local-pv-3d27e350                          10230Mi    RWO            Delete           Available                                                              managed-local-storage            7m42s
local-pv-57327184                          10230Mi    RWO            Delete           Available                                                              managed-local-storage            7m42s
local-pv-609ff9fb                          5110Mi     RWO            Delete           Available                                                              managed-local-storage            7m42s
local-pv-6a3b1955                          5110Mi     RWO            Delete           Available                                                              managed-local-storage            7m42s
local-pv-6ef1f51e                          19Gi       RWO            Delete           Available                                                              managed-local-storage            7m42s
local-pv-890b0157                          24Gi       RWO            Delete           Available                                                              managed-local-storage            7m42s
local-pv-901f0a1                           10230Mi    RWO            Delete           Available                                                              managed-local-storage            7m42s
local-pv-c3659d0                           14Gi       RWO            Delete           Available                                                              managed-local-storage            7m42s
local-pv-d8106721                          1014Mi     RWO            Delete           Available                                                              managed-local-storage            7m42s
local-pv-df424c6b                          24Gi       RWO            Delete           Available                                                              managed-local-storage            7m42s
```

4. 创建本地持久卷申请

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: example-local-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: managed-local-storage
```

注: **此申请会被挂起，直到有Pod被调度才会去申请Bound到本地卷上**

#### 示例

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: example-local-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: managed-local-storage
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myapp-with-pv
  name: myapp-with-pv
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-with-pv
  template:
    metadata:
      labels:
        app: myapp-with-pv
    spec:
      containers:
      - image: registry.iwgame.com/base/myapp:v1
        name: myapp
        volumeMounts:
        - mountPath: /mnt/
          name: local-pv
      volumes:
      - name: local-pv
        persistentVolumeClaim:
          claimName: example-local-claim
          readOnly: true

``` 
