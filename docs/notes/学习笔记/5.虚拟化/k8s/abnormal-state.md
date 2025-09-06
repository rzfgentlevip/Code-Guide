---
title: Pod异常状态排错
icon: streamline-freehand-color:notes-book
order: 2
author: bugcode
date: 2020-01-01T00:00:00.000Z
category:
  - K8S
  - DOCKER
  - JAVA
tag:
  - DOCKER
  - 云原生
  - K8S
sticky: false
star: true
footer: 云原生
copyright: bugcode
createTime: 2025/09/04 14:16:48
permalink: /learning-notes/virtualization/k8s/abnormal-state/
---

<!-- TOC -->

- [2、Pod异常状态排错](#2pod异常状态排错)
  - [一、常用命令](#一常用命令)
  - [二、Pod状态](#二pod状态)
  - [三、pod遇到的问题](#三pod遇到的问题)
    - [1、pod一直处于Pending状态](#1pod一直处于pending状态)
    - [2、Pod 一直处于 ContainerCreating 或 Waiting 状 态](#2pod-一直处于-containercreating-或-waiting-状-态)
    - [3、Pod 处于 CrashLoopBackOff 状态](#3pod-处于-crashloopbackoff-状态)
    - [4、Pod 一直处于 Terminating 状态](#4pod-一直处于-terminating-状态)
    - [5、Pod 一直处于 Unknown 状态](#5pod-一直处于-unknown-状态)
    - [6、Pod 一直处于 Error 状态](#6pod-一直处于-error-状态)
    - [7、Pod 一直处于 ImagePullBackOff 状态](#7pod-一直处于-imagepullbackoff-状态)
    - [8、Pod 健康检查失败](#8pod-健康检查失败)

<!-- /TOC -->

# 2、Pod异常状态排错

## 一、常用命令

首先列出Pod排查过程中我这边的常用命令：

- 查看Pod状态：kubectl get pod podname -o wide
- 查看Pod的yaml配置：kubectl get pods podname -o yaml
- 查看pod事件：kubectl describe pods podname
- 查看容器日志：kubectl logs podsname -c container-name

## 二、Pod状态

- Error：Pod 启动过程中发生错误
- NodeLost : Pod 所在节点失联
- Unkown : Pod 所在节点失联或其它未知异常
- Waiting : Pod 等待启动
- Pending : Pod 等待被调度
- ContainerCreating : Pod 容器正在被创建
- Terminating : Pod 正在被销毁
- CrashLoopBackOff ： 容器退出，kubelet 正在将它重启
- InvalidImageName ： 无法解析镜像名称
- ImageInspectError ： 无法校验镜像
- ErrImageNeverPull ： 策略禁止拉取镜像
- ImagePullBackOff ： 正在重试拉取
- RegistryUnavailable ： 连接不到镜像中心
- ErrImagePull ： 通用的拉取镜像出错
- CreateContainerConfigError ： 不能创建 kubelet 使用的容器配置
- CreateContainerError ： 创建容器失败
- RunContainerError ： 启动容器失败
- PreStartHookError : 执行 preStart hook 报错
- PostStartHookError ： 执行 postStart hook 报错
- ContainersNotInitialized ： 容器没有初始化完毕
- ContainersNotReady ： 容器没有准备完毕
- ContainerCreating ：容器创建中
- PodInitializing ：pod 初始化中
- DockerDaemonNotReady ：docker还没有完全启动
- NetworkPluginNotReady ： 网络插件还没有完全启动

## 三、pod遇到的问题

### 1、pod一直处于Pending状态

Pending 状态说明 Pod 还没有被调度到某个节点上，需要看下 Pod 事件进一步判断原因，比如:

```
$ kubectl describe pod tikv-0 
     . ... 
       Events: 
        Type     Reason   Age     From   Message 
         ----    ------   ----    ----    ------- 
        Warning FailedScheduling 3m (x106 over 33m) default-scheduler 0/4 nodes are available: 1 node(s) had no available volume zone, 2 Insufficient cpu, 3 Insufficient memory.
```

下面是我遇到的一些原因：

- **节点资源不够**：节点资源不够有以下几种情况:
    - CPU负载过高
    - 剩余可被分配的内存不足
    - 剩余可用GPU数量不足

如何判断某个 Node 资源是否足够？ 通过下面的命令查看node资源情况，关注以下信息：

```java
kubectl describe node nodename
```

- Allocatable : 表示此节点能够申请的资源总和
- Allocated resources : 表示此节点已分配的资源 (Allocatable 减去节点上所有 Pod 总 的 Request)

前者与后者相减，可得出剩余可申请的资源。如果这个值小于 Pod 的 request，就不满足 Pod 的 资源要求，Scheduler 在 Predicates (预选) 阶段就会剔除掉这个 Node，也就不会调度上去。

- **不满足 nodeSelector 与 affinity**

如果 Pod 包含 nodeSelector 指定了节点需要包含的 label，调度器将只会考虑将 Pod 调度到 包含这些 label 的Node 上，如果没有 Node 有这些 label 或者有这些 label 的 Node 其它 条件不满足也将会无法调度。参考官方文档
[https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector)
如果 Pod 包含 affinity（亲和性）的配置，调度器根据调度算法也可能算出没有满足条件的 Node，从而无法调度。affinity 有以下几类:

- nodeAffinity: 节点亲和性，可以看成是增强版的 nodeSelector，用于限制 Pod 只允许 被调度到某一部分 Node。
- podAffinity: Pod 亲和性，用于将一些有关联的 Pod 调度到同一个地方，同一个地方可以 是指同一个节点或同一个可用区的节点等。
- podAntiAffinity: Pod 反亲和性，用于避免将某一类 Pod 调度到同一个地方避免单点故 障，比如将集群 DNS 服务的 Pod 副本都调度到不同节点，避免一个节点挂了造成整个集群 DNS 解析失败，使得业务中断。

**Node 存在 Pod 没有容忍的污点**
如果节点上存在污点 (Taints)，而 Pod 没有响应的容忍 (Tolerations)，Pod 也将不会调度上 去。通过 describe node 可以看下 Node 有哪些 Taints:

```
$ kubectl describe nodes host1 
    ... 
    Taints: special=true:NoSchedule
    ...
```

污点既可以是手动添加也可以是被自动添加，下面可以看一下。
**手动添加的污点：**
通过类似以下方式可以给节点添加污点:

```
$ kubectl taint node host1 special=true:NoSchedule 
  node "host1" tainted
```

另外，有些场景下希望新加的节点默认不调度 Pod，直到调整完节点上某些配置才允许调度，就给新加 的节点都加上node.kubernetes.io/unschedulable 这个污点。
**自动添加的污点**
如果节点运行状态不正常，污点也可以被自动添加，从 v1.12 开始， TaintNodesByCondition 特性进入 Beta 默认开启，controller manager 会检查 Node 的 Condition，如果命中条件 就自动为 Node 加上相应的污点，这些 Condition 与 Taints 的对应关系如下:

```
Conditon Value Taints 
 -------- ----- ------
 OutOfDisk True node.kubernetes.io/out-of-disk
 Ready False node.kubernetes.io/not-ready
 Ready Unknown node.kubernetes.io/unreachable
 MemoryPressure True node.kubernetes.io/memory-pressure
 PIDPressure True node.kubernetes.io/pid-pressure
 DiskPressure True node.kubernetes.io/disk-pressure
 NetworkUnavailable True node.kubernetes.io/network-unavailable
```

解释下上面各种条件的意思:

- OutOfDisk 为 True 表示节点磁盘空间不够了
- Ready 为 False 表示节点不健康
- Ready 为 Unknown 表示节点失联，在 node-monitor-grace-period 这么长的时间内没有 上报状态 controller-manager 就会将 Node 状态置为 Unknown (默认 40s)
- MemoryPressure 为 True 表示节点内存压力大，实际可用内存很少
- PIDPressure 为 True 表示节点上运行了太多进程，PID 数量不够用了
- DiskPressure 为 True 表示节点上的磁盘可用空间太少了
- NetworkUnavailable 为 True 表示节点上的网络没有正确配置，无法跟其它 Pod 正常通 信

另外，在云环境下，比如腾讯云 TKE，添加新节点会先给这个 Node 加上
node.cloudprovider.kubernetes.io/uninitialized 的污点，等 Node 初始化成功后才自动移 除这个污点，避免 Pod 被调度到没初始化好的 Node 上。
**低版本 kube-scheduler 的 bug**
可能是低版本 kube-scheduler 的 bug, 可以升级下调度器版本。
**kube-scheduler 没有正常运行**
检查 maser 上的 kube-scheduler 是否运行正常，异常的话可以尝试重启临时恢复。
**驱逐后其它可用节点与当前节点有状态应用不在同一个可用区**

有时候服务部署成功运行过，但在某个时候节点突然挂了，此时就会触发驱逐，创建新的副本调度到其 它节点上，对于已经挂载了磁盘的 Pod，它通常需要被调度到跟当前节点和磁盘在同一个可用区，如果 集群中同一个可用区的节点不满足调度条件，即使其它可用区节点各种条件都满足，但不跟当前节点在 同一个可用区，也是不会调度的。为什么需要限制挂载了磁盘的 Pod 不能漂移到其它可用区的节点？ 试想一下，云上的磁盘虽然可以被动态挂载到不同机器，但也只是相对同一个[数据中心](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E4%B8%AD%E5%BF%83&spm=1001.2101.3001.7020)，通常不允许跨数据中心挂载磁盘设备，因为网络时延会极大的降低 IO 速率。

### 2、Pod 一直处于 ContainerCreating 或 Waiting 状 态

- **Pod 配置错误**
    - 检查是否打包了正确的镜像
    - 检查配置了正确的容器参数
- **挂载 Volume 失败**
    - Volume 挂载失败也分许多种情况，先列下我这里目前已知的。

**Pod 漂移没有正常解挂之前的磁盘**

> 在云尝试托管的 [K8S](https://so.csdn.net/so/search?q=K8S&spm=1001.2101.3001.7020) 服务环境下，默认挂载的 Volume 一般是块存储类型的云硬盘，如果某个节点 挂了，kubelet 无法正常运行或与 apiserver 通信，到达时间阀值后会触发驱逐，自动在其它节点 上启动相同的副本 (Pod 漂移)，但是由于被驱逐的 Node 无法正常运行并不知道自己被驱逐了，也 就没有正常执行解挂，cloud-controller-manager 也在等解挂成功后再调用云厂商的接口将磁盘 真正从节点上解挂，通常会等到一个时间阀值后 cloud-controller-manager 会强制解挂云盘， 然后再将其挂载到 Pod 最新所在节点上，这种情况下 ContainerCreating 的时间相对长一点，但 一般最终是可以启动成功的，除非云厂商的 cloud-controller-manager 逻辑有 bug。

**磁盘爆满**
启动 Pod 会调 CRI 接口创建容器，容器运行时创建容器时通常会在数据目录下为新建的容器创建一 些目录和文件，如果数据目录所在的磁盘空间满了就会创建失败并报错:

```
Events：
    Type Reason Age From
Message
     ---- ------ ---- ----
 Warning FailedCreatePodSandBox 2m (x4307 over 16h) kubelet, 10.179.80.31 (combined from similar events): Failed create pod sandbox: rpc error: code = Unknown desc = failed to create a sandbox for pod "apigateway-6dc48bf8b6-l8xrw": Error response from daemon: mkdir /var/lib/docker/aufs/mnt/1f09d6c1c9f24e8daaea5bf33a4230de7dbc758e3b22785e8ee21e3e3d921214 no space left on device

```

处理方法: [https://blog.csdn.net/qq_31055683/article/details/126974456](https://blog.csdn.net/qq_31055683/article/details/126974456)
**节点内存碎片化**
如果节点上内存碎片化严重，缺少大页内存，会导致即使总的剩余内存较多，但还是会申请内存失败，处理方法请看：[https://blog.csdn.net/qq_31055683/article/details/12697487](https://blog.csdn.net/qq_31055683/article/details/126974873)
**limit 设置太小或者单位不对**
如果 limit 设置过小以至于不足以成功运行 Sandbox 也会造成这种状态，常见的是因为 memory limit 单位设置不对造成的 limit 过小，比如误将 memory 的 limit 单位像 request 一样设 置为小 **m** ，这个单位在 memory 不适用，会被 k8s 识别成 byte， 应该用 **Mi** 或**M**
_举个例子: 如果 memory limit 设为 1024m 表示限制 1.024 Byte，这么小的内存， pause 容器一起来就会被 cgroup-oom kill 掉，导致 pod 状态一直处于 ContainerCreating。_
这种情况通常会报下面的 event:

```
Pod sandbox changed, it will be killed and re-created。
```

kubelet 报错:

```
to start sandbox container for pod ... Error response from daemon: OCI runtime create failed: container_linux.go:348: starting container process caused "process_linux.go:301: running exec setns process for init caused \"signal: killed\"": unknown
```

**拉取镜像失败**
镜像拉取失败也分很多情况，这里列举下:

- 配置了错误的镜像
- Kubelet 无法访问镜像仓库（比如默认 pause 镜像在 gcr.io 上，国内环境访问需要特殊处 理）
- 拉取私有镜像的 imagePullSecret 没有配置或配置有误
- 镜像太大，拉取超时（可以适当调整 kubelet 的 —image-pull-progress-deadline 和 —runtime-request-timeout 选项）

**CNI 网络错误**
如果发生 CNI 网络错误通常需要检查下网络插件的配置和运行状态，如果没有正确配置或正常运行通 常表现为:

- 无法配置 Pod 网络
- 无法分配 Pod IP

**controller-manager 异常**
查看 master 上 kube-controller-manager 状态，异常的话尝试重启。
**安装 docker 没删干净旧版本**

如果节点上本身有 docker 或者没删干净，然后又安装 docker，比如在 centos 上用 yum 安装:

```
yum install -y docker
```

这样可能会导致 dockerd 创建容器一直不成功，从而 Pod 状态一直 ContainerCreating，查看 event 报错:

```
Type     Reason        Age       From
Message
      ----    ------       ----       ----
      Warning FailedCreatePodSandBox 18m (x3583 over 83m) kubelet, 192.168.4.5 (combined from similar events): Failed create pod sandbox: rpc error: code = Unknown desc = failed to start sandbox container for pod "nginx-7db9fccd9b-2j6dh": Error response from daemon: ttrpc: client shutting down: read unix @->@/containerd- shim/moby/de2bfeefc999af42783115acca62745e6798981dff75f4148fae8c086668f667/shim.sock
 read: connection reset by peer: unknown
 Normal SandboxChanged 3m12s (x4420 over 83m) kubelet, 192.168.4.5
 Pod sandbox changed, it will be killed and re-created
```

可能是因为重复安装 docker 版本不一致导致一些组件之间不兼容，从而导致 dockerd 无法正常创 建容器。

- **存在同名容器**

如果节点上已有同名容器，创建 sandbox 就会失败，event:

```
Warning FailedCreatePodSandBox 2m kubelet, 10.205.8.91 Failed create pod sandbox: rpc error: code = Unknown desc = failed to create a sandbox for pod "lomp-ext-d8c8b8c46-4v8tl": operation timeout: context deadline exceeded
Warning FailedCreatePodSandBox 3s (x12 over 2m) kubelet, 10.205.8.91 Failed create pod sandbox: rpc error: code = Unknown desc = failed to create a sandbox for pod "lomp-ext-d8c8b8c46-4v8tl": Error response from daemon: Conflict. The container name "/k8s_POD_lomp-ext-d8c8b8c46- 4v8tl_default_65046a06-f795-11e9-9bb6-b67fb7a70bad_0" is already in use by container "30aa3f5847e0ce89e9d411e76783ba14accba7eb7743e605a10a9a862a72c1e2". You have to remove (or rename) that container to be able to reuse that name.

```

关于什么情况下会产生同名容器，这个有待研究。

### 3、Pod 处于 CrashLoopBackOff 状态

Pod 如果处于 CrashLoopBackOff 状态说明之前是启动了，只是又异常退出了，只要 Pod 的 restartPolicy不是 Never 就可能被重启拉起，此时 Pod 的 RestartCounts 通常是大于 0 的，可以先看下容器进程的退出状态码来缩小问题范围

- **容器进程主动退出：**


---

如果是容器进程主动退出，退出状态码一般在 0-128 之间，除了可能是业务程序 BUG，还有其它许 多可能原因

- **系统OOM**


---

如果发生系统 OOM，可以看到 Pod 中容器退出状态码是 137，表示被 SIGKILL 信号杀死，同时 内核会报错: Out of memory: Kill process … 。大概率是节点上部署了其它非 K8S 管理的进 程消耗了比较多的内存，或者 kubelet 的 --kube-reserved 和 --system-reserved 配的 比较小，没有预留足够的空间给其它非容器进程，节点上所有 Pod 的实际内存占用总量不会超过 /sys/fs/cgroup/memory/kubepods 这里 cgroup 的限制，这个限制等于 capacity - "kube- reserved" - "system-reserved" ，如果预留空间设置合理，节点上其它非容器进程（kubelet, dockerd, kube-proxy, sshd 等) 内存占用没有超过 kubelet 配置的预留空间是不会发生系统 OOM 的，可以根据实际需求做合理的调整。

- **系统OOM**


---

如果是 cgrou OOM 杀掉的进程，从 Pod 事件的下 Reason 可以看到是 OOMKilled ，说明 容器实际占用的内存超过 limit 了，同时内核日志会报: ``。 可以根据需求调整下 limit。

- **节点内存碎片化**


---

如果节点上内存碎片化严重，缺少大页内存，会导致即使总的剩余内存较多，但还是会申请内存失败，

- **健康检查失败**


---

### 4、Pod 一直处于 Terminating 状态

- **磁盘爆满**


---

如果 docker 的数据目录所在磁盘被写满，docker 无法正常运行，无法进行删除和创建操作，所以 kubelet 调用 docker 删除容器没反应，看 event 类似这样：

```
Normal Killing 39s (x735 over 15h) kubelet, 10.179.80.31 Killing container with id docker://apigateway:Need to kill Pod
```

- **存在 “i” 文件属性**


---

如果容器的镜像本身或者容器启动后写入的文件存在 “i” 文件属性，此文件就无法被修改删除，而删 除 Pod 时会清理容器目录，但里面包含有不可删除的文件，就一直删不了，Pod 状态也将一直保持 Terminating，kubelet 报错:

```
Sep 27 14:37:21 VM_0_7_centos kubelet[14109]: E0927 14:37:21.922965 14109 remote_runtime RemoveContainer "19d837c77a3c294052a99ff9347c520bc8acb7b8b9a9dc9fab281fc09df38257" failed: rpc error: code = Unknown desc = failed to remove container "19d837c77a3c294052a99ff9347c520bc8acb7b8b9a9dc9fab281fc09df38257": Error response 19d837c77a3c294052a99ff9347c520bc8acb7b8b9a9dc9fab281fc09df38257: driver "overlay2" filesystem: remove /data/docker/overlay2/b1aea29c590aa9abda79f7cf3976422073fb3652757f0391db88534027546868 operation not permitted
Sep 27 14:37:21 VM_0_7_centos kubelet[14109]: E0927 14:37:21.923027 14109 kuberuntime_gc to remove container "19d837c77a3c294052a99ff9347c520bc8acb7b8b9a9dc9fab281fc09df38257" Unknown desc = failed to remove container "19d837c77a3c294052a99ff9347c520bc8acb7b8b9a9dc9fab281fc09df38257": Error response 19d837c77a3c294052a99ff9347c520bc8acb7b8b9a9dc9fab281fc09df38257: driver "overlay2" filesystem: remove /data/docker/overlay2/b1aea29c590aa9abda79f7cf3976422073fb3652757f0391db88534027546868 operation not permitted
```

通过 **man chattr** 查看 “i” 文件属性描述:

```
A file with the 'i' attribute cannot be modified: it cannot be deleted or renamed, no link can be created to this file and no data can be written to the file. Only the superuser or a process possessing the CAP_LINUX_IMMUTABLE capability can set or clear this attribute.
```

彻底解决当然是不要在容器镜像中或启动后的容器设置 “i” 文件属性，临时恢复方法： 复制 kubelet 日志报错提示的文件路径，然后执行 chattr -i 文件名 :

```
chattr -i /data/docker/overlay2/b1aea29c590aa9abda79f7cf3976422073fb3652757f0391db88534027
```

执行完后等待 kubelet 自动重试，Pod 就可以被自动删除了。

- **存在 Finalizers**


---

k8s 资源的 metadata 里如果存在 finalizers ，那么该资源一般是由某程序创建的，并且在其 创建的资源的 metadata 里的 finalizers 加了一个它的标识，这意味着这个资源被删除时需要 由创建资源的程序来做删除前的清理，清理完了它需要将标识从该资源的 finalizers 中移除，然 后才会最终彻底删除资源。比如 Rancher 创建的一些资源就会写入 finalizers 标识。
处理建议： kubectl edit 手动编辑资源定义，删掉 finalizers ，这时再看下资源，就会发现 已经删掉了

### 5、Pod 一直处于 Unknown 状态

通常是节点失联，没有上报状态给 apiserver，到达阀值后 controller-manager 认为节点失联 并将其状态置为 Unknown
可能原因:

- 节点高负载导致无法上报
- 节点宕机
- 节点被关机
- 网络不通

### 6、Pod 一直处于 Error 状态

通常处于 Error 状态说明 Pod 启动过程中发生了错误。常见的原因包括：

- 依赖的 ConfigMap、Secret 或者 PV 等不存在
- 请求的资源超过了管理员设置的限制，比如超过了 LimitRange 等
- 违反集群的安全策略，比如违反了 PodSecurityPolicy 等
- 容器无权操作集群内的资源，比如开启 RBAC 后，需要为 ServiceAccount 配置角色绑定

### 7、Pod 一直处于 ImagePullBackOff 状态

- **http 类型 registry，地址未加入到 insecure- registry**


---

dockerd 默认从 https 类型的 registry 拉取镜像，如果使用 https 类型的 registry，则 必须将它添加到 insecure-registry 参数中，然后重启或 reload dockerd 生效。

- **https 自签发类型 resitry，没有给节点添加 ca 证书**


---

如果 registry 是 https 类型，但证书是自签发的，dockerd 会校验 registry 的证书，校验 成功才能正常使用镜像仓库，要想校验成功就需要将 registry 的 ca 证书放置到
/etc/docker/certs.d/<registry:port>/ca.crt 位置

- **私有镜像仓库认证失败**


---

如果 registry 需要认证，但是 Pod 没有配置 imagePullSecret，配置的 Secret 不存在或者 有误都会认证失败。

- **镜像文件损坏**


---

如果 push 的镜像文件损坏了，下载下来也用不了，需要重新 push 镜像文件。

- **镜像拉取超时**


---

如果节点上新起的 Pod 太多就会有许多可能会造成容器镜像下载排队，如果前面有许多大镜像需要下 载很长时间，后面排队的 Pod 就会报拉取超时。
kubelet 默认串行下载镜像:

```
--serialize-image-pulls Pull images one at a time. We recommend *not* changing the default value on nodes that run docker daemon with version < 1.9 or an Aufs storage backend. Issue #10959 has more details. (default true)
```

也可以开启并行下载并控制并发:

```
--registry-qps int32 If > 0, limit registry pull QPS to this value. If 0, unlimited. (default 5)
--registry-burst int32 Maximum size of a bursty pulls, temporarily allows pulls to burst to this number, while still not exceeding registry-qps. Only used if --registry-qps > 0 (default 10)

```

- **镜像不存在**


---

kubelet日志：

```
PullImage "imroc/test:v0.2" from image service failed: rpc error: code = Unknown desc = Error response from daemon: manifest for imroc/test:v0.2 not found
```

### 8、Pod 健康检查失败

- Kubernetes 健康检查包含就绪检查(readinessProbe)和存活检查(livenessProbe)
- pod 如果就绪检查失败会将此 pod ip 从 service 中摘除，通过 service 访问，流量将 不会被转发给就绪检查失败的 pod
- pod 如果存活检查失败，kubelet 将会杀死容器并尝试重启

健康检查失败的可能原因有多种，除了业务程序BUG导致不能响应健康检查导致 unhealthy，还能有 有其它原因，下面我们来逐个排查。

- **健康检查配置不合理**


---

initialDelaySeconds 太短，容器启动慢，导致容器还没完全启动就开始探测，如果 successThreshold 是默认值 1，检查失败一次就会被 kill，然后 pod 一直这样被 kill 重 启。

- **节点负载过高**


---

cpu 占用高（比如跑满）会导致进程无法正常发包收包，通常会 timeout，导致 kubelet 认为 pod 不健康

- **容器进程被木马进程杀死**


---

- **容器内进程端口监听挂掉**


---

使用 netstat -tunlp 检查端口监听是否还在，如果不在了，抓包可以看到会直接 reset 掉健 康检查探测的连接:

```
20:15:17.890996 IP 172.16.2.1.38074 > 172.16.2.23.8888: Flags [S], seq 96880261, win 14600, options [mss 1424,nop,nop,sackOK,nop,wscale 7], length 0
20:15:17.891021 IP 172.16.2.23.8888 > 172.16.2.1.38074: Flags [R.], seq 0, ack 96880262, win 0, length 0
20:15:17.906744 IP 10.0.0.16.54132 > 172.16.2.23.8888: Flags [S], seq 1207014342, win 14600, options [mss 1424,nop,nop,sackOK,nop,wscale 7], length 0
20:15:17.906766 IP 172.16.2.23.8888 > 10.0.0.16.54132: Flags [R.], seq 0, ack 1207014343, win 0, length 0
```

连接异常，从而健康检查失败。发生这种情况的原因可能在一个节点上启动了多个使用 hostNetwork 监听相同宿主机端口的 Pod，只会有一个 Pod 监听成功，但监听失败的 Pod 的 业务逻辑允许了监听失败，并没有退出，Pod 又配了健康检查，kubelet 就会给 Pod 发送健康检查 探测报文，但 Pod 由于没有监听所以就会健康检查失败。

- **SYN backlog 设置过小**


---

SYN backlog 大小即 SYN 队列大小，如果短时间内新建连接比较多，而 SYN backlog 设置太 小，就会导致新建连接失败，通过 netstat -s | grep TCPBacklogDrop可以看到有多少是因为 backlog 满了导致丢弃的新连接。
如果确认是 backlog 满了导致的丢包，建议调高 backlog 的值，内核参数为 net.ipv4.tcp_max_syn_backlog 。

- **容器进程主动退出**


---

容器进程如果是自己主动退出(不是被外界中断杀死)，退出状态码一般在 0-128 之间，根据约定，正 常退出时状态码为 0，1-127 说明是程序发生异常，主动退出了，比如检测到启动的参数和条件不满 足要求，或者运行过程中发生 panic 但没有捕获处理导致程序退出。除了可能是业务程序 BUG，还 有其它许多可能原因，这里我们一一列举下。

- **DNS 无法解析**
  可能程序依赖 集群 DNS 服务，比如启动时连接数据库，数据库使用 service 名称或外部域名都需 要 DNS 解析，如果解析失败程序将报错并主动退出。解析失败的可能原因:1、集群网络有问题，Pod 连不上集群 DNS 服务
  2、集群 DNS 服务挂了，无法响应解析请求
  3、Service 或域名地址配置有误，本身是无法解析的地址
- **程序配置有误**
  1、配置文件格式错误，程序启动解析配置失败报错退出
  2、配置内容不符合规范，比如配置中某个字段是必选但没有填写，配置校验不通过，程序报错主动 退出

参考：
[https://blog.csdn.net/qq_31055683/article/details/126970831](https://blog.csdn.net/qq_31055683/article/details/126970831)