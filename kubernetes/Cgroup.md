# Cgroup: 控制群组
> 在一个系统中运行的层级制进程组，通过 `Cgroup`， 系统管理员在分配、排序、拒绝、管理和监控系统资源等方面，可以进行精细化控制。硬件资源可以在应用程序和用户间智能分配，从而增加整体效率。

## Cgroup 默认层级
> 默认情况下，`systemd` 会自动创建 `slice`、`scope` 和 `service` 单位的层级，来为 `cgroup` 树提供统一结构.
> 系统中运行的所有进程，都是 `systemd init` 的子进程
> 在资源管控方面， `systemd` 提供了三种 `unit` 类型

* Service
> 一个或一组进程，由 `systemd` 依据 `unit` 配置文件启动。`service` 对指定进程进行封装，这样进程可以作为一个整体被启动或终止。`service` 参照以下方式命名

```shell
name.service
```

* Scope
> 一组外部创建的进程。由任意进程通过 `fork()` 函数启动和终止，然后在运行时，由 `systemd` 注册的进程，`scope` 会将其封装。例如：用户会话、 容器和虚拟机被认为是 `scope`。`scope` 的命名方式如下：

```shell
name.scope
```

* Slice
> 一组按层级排列的单位。slice 并不包含进程，但会组建一个层级，并将 scope 和 service 都放置其中。真正的进程包含在 scope 或 service 中。在这一被划分层级的树中，每一个 slice 单位的名字对应通向层级中一个位置的路径。小横线（"-"）起分离路径组件的作用。例如，如果一个 slice 的名字是：

```shell
parent-name.slice
```

**name 时 parent 的子 slice，以此类推**

#### 默认 slice
* -.slice: 根 slice;
* system.slice: 所有系统 Service 默认位置；
* user.slice: 所有用户会话的默认位置；
* machine.slice: 所有虚拟机和 Linux 容器的默认位置；

### 资源控制器
> 资源管理器 (cgroup subsystem)，代表一种单一资源，如 CPU 时间或内存， Linux Kernel 提供一系列资源控制器，由 `systemd`  自动挂载，可以通过一下命令来查看目前已挂载的资源控制器列表

```shell
~]# mount | grep cgroup
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_prio,net_cls)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
```

#### 可用的管理器

|控制器|作用|
|:--|:--|
|`blkio`|设置对块设备的输入/输出访问的限制|
|`cpu`|使用CPU调度程序让 cgroup 的 tasks 可以访问 CPU， 它与 `cpuacct` 管理器一起 mount 在一起|
|`cpuacct`|自动生成 cgroup 中 tasks 占用 CPU 资源报告|
|`cpuset`|给 cgroup 中的 tasks 分配独立的 CPU 和内存节点|
|`devices`|允许或禁止 cgroup 中的 tasks 存取设备|
|`freezer`|暂停或恢复 cgroup 中的 tasks|
|`memory`|对 cgroup 中 tasks 的可能内存做出限制，并且自动生成 tasks 内存占用报告|
|`net_cls`|使用等级识别符（classid）标记网络数据包，这让 Linux 流量控制器（tc 指令）可以识别来自特定 cgroup 任务的数据包|
|`perf_event`|允许使用 perf 工具来监控 cgroup|
|`hugetlb`|允许使用大篇幅的虚拟内存页，并且给这些内存页强制设定可用资源量|

<!--
* `blkio`: 设置对块设备的输入/输出访问的限制；
* `cpu`: 使用CPU调度程序让 cgroup 的 tasks 可以访问 CPU， 它与 `cpuacct` 管理器一起 mount 在一起；
* `cpuacct`: 自动生成 cgroup 中 tasks 占用 CPU 资源报告；
* `cpuset`: 给 cgroup 中的 tasks 分配独立的 CPU 和内存节点；
* `devices`: 允许或禁止 cgroup 中的 tasks 存取设备；
* `freezer`: 暂停或恢复 cgroup 中的 tasks；
* `memory`: 对 cgroup 中 tasks 的可能内存做出限制，并且自动生成 tasks 内存占用报告；
* `net_cls`: 使用等级识别符（classid）标记网络数据包，这让 Linux 流量控制器（tc 指令）可以识别来自特定 cgroup 任务的数据包；
* `perf_event`: 允许使用 perf 工具来监控 cgroup；
* `hugetlb`: 允许使用大篇幅的虚拟内存页，并且给这些内存页强制设定可用资源量。
-->

<!--
### 控制器参数
待补充。。。
-->






