# 资源创建流程
* 所有的 `Kubernetes` 组件均使用 `watch` 机制来跟踪检查 `API Server` 上相关内容的变动

## Pod

![pod-create](./pod-workflow.png)

1. 用户通过 `kubectl` 或其他 `API` 客户端提交 `Pod Spec` 给 `API Server`

2. `API Server` 尝试着将 `Pod` 对象的相关信息存入到 `etcd`, 待写入操作执行完成，`API Server` 即会返回确认信息至客户端。

3. `kube-scheduler`（调度器）通过 `watch` 机制发现 `API Server` 创建了新的Pod，但为绑定到任何工作节点

4. `kube-scheduler` 通过一些预选、优选算法为 `Pod` 对象挑选一个工作节点并将结果信息更新至 `API Server`

5. 调度信息由 `API Server` 更新至 `etcd` 存储系统，而且 `API Server` 也开始反映此 `Pod` 对象的调度结果

6. `Pod` 被调度到的目标工作节点上的 `kubelet` 尝试在当前节点上调用 `Docker` 启动容器，并将容器的结果状态返回至 `API Server`

7. `API Server` 将 `Pod` 状态信息存入 `etcd` 系统中

8. 在 `etcd` 确认写入操作完成后，`API Server` 将确认信息发送至相关的 `kubelet`

9. `kublet` 还会通过 `container runtime` 获取 `pod` 的状态，然后更新到 `apiserver` 中，最后由 `apiserver` 将信息写入到 `etcd` 中

### Pod生命周期的重要行为
* 初始化容器

* 生命周期钩子函数
> 钩子处理器的实现方式有 Exec 和 HTTP
>
> Exec: 在钩子事件触发时直接在当前容器中运行由用户指定的命令
>
> HTTP: 在当前容器中向某个URL发起HTTP请求
  * `postStart`: 容器创建完成之后立即运行的钩子处理器（handler），不过 `Kubernetes` 无法确保它一定会于容器中的 `ENTRYPOINT` 之前运行
  * `preStop`: 容器终止之前运行的钩子处理器，它以同步方式调用，因此，在其完成之前会阻塞删除容器操作的调用

* 容器探测
  * ExecAction：在容器中执行一个命令，根据其返回值判断健康状态

  * TCPSocketAction：通过与容器的某个TCP端口建立连接来判断健康状态

  * HTTPGetAction：通过向容器指定的端口指定的路径发起HTTP GET请求进行判断，2xx与3xx为健康，否则为失败

  **任何一种探测方式都存在三种结果 Success(成功) Failure(失败) 和 Unknown(未知)，只有 Success结果表示成功通过检测**

  * 存活性探测：判断容器是否处于存活，Running 状态，一旦检测未通过，kubelet将杀死容器并根据其restartPolicy决定是否将其重启，未定义存货检测的容器默认为success

  * 就绪性探测：用于检测容器是否就绪可以对外提供服务，未通过检测的容器意味着为就绪，无法对外提供服务，端点控制器（如service）会将其IP地址从Endpoints对象中移除，直至检测通过，重新加入。

* 容器的重启策略
  * Always： 但凡Pod对象终止就将其重启

  * OnFailure：尽在Pod对象出现错误时才将其重启

  * Never：从不重启



















## Deployment



## ReplicaSet
