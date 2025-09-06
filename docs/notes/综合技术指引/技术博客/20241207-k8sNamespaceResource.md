---
title: K8s对命名空间进行资源配额
icon: material-icon-theme:terraform
order: 5
author: bugcode
date: 2024-12-7
category:
  - K8S
tag:
  - 面试
  - k8s
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 10:31:15
permalink: /compre-guide/technical-blog/k8sNamespaceResource/
---


# K8s对命名空间进行资源配额

## 为什么要给K8S中不同命名空间进行资源配额

在k8s集群中为了能够使系统正常稳定运行，通常会限制Pod的资源使用情况，在k8s集群中如果有一个程序出现异常，并占用大量的系统资源。如果未对该Pod进行资源限制的话，可能会影响其他的Pod。

## K8S中常见的资源管理方式

- 计算资源管理(Compute Resources)：通常用来配置管理某个pod所申请的资源

- 资源的配置范围管理(LimitRange)：可以对集群内Request和Limits的配置做一个全局的统一的限制，相当于批量设置了某一个范围内(某个命名空间)的Pod的资源使用限制。

- 资源的配额管理(Resource Quotas)：可以为每一个命名空间(namespace)提供一个总体的资源使用限制，通过它可以限制命名空间中某个类型的对象的总数目上限，也可以设置命名空间中Pod可以使用到的计算资源的总上限。资源的配额管理有效解决了多用户或多个团队公用一个k8s集群时资源有效分配的问题。

## 资源配额及使用场景

### 使用场景

当多个团队共用一个拥有固定节点数目的集群时，通常会基于不同团队创建不同的命名空间，然后将节点总资源分配给不同的命名空间，这样就在命名空间层面将节点的资源进行隔离，pod申请资源不会相互影响。

### 计算资源配额

用户可以对给定命名空间下的可被请求的 [计算资源](https://kubernetes.io/zh-cn/docs/concepts/configuration/manage-resources-containers/) 总量进行限制。

配额机制所支持的资源类型：

| 资源名称           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `limits.cpu`       | 所有非终止状态的 Pod，其 CPU 限额总量不能超过该值。          |
| `limits.memory`    | 所有非终止状态的 Pod，其内存限额总量不能超过该值。           |
| `requests.cpu`     | 所有非终止状态的 Pod，其 CPU 需求总量不能超过该值。          |
| `requests.memory`  | 所有非终止状态的 Pod，其内存需求总量不能超过该值。           |
| `hugepages-<size>` | 对于所有非终止状态的 Pod，针对指定尺寸的巨页请求总数不能超过此值。 |
| `cpu`              | 与 `requests.cpu` 相同。                                     |
| `memory`           | 与 `requests.memory` 相同。                                  |

### 存储资源配额

用户可以对给定命名空间下的[存储资源](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/) 总量进行限制。

此外，还可以根据相关的存储类（Storage Class）来限制存储资源的消耗。

| 资源名称                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `requests.storage`                                           | 所有 PVC，存储资源的需求总量不能超过该值。                   |
| `persistentvolumeclaims`                                     | 在该命名空间中所允许的 [PVC](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) 总量。 |
| `<storage-class-name>.storageclass.storage.k8s.io/requests.storage` | 在所有与 `<storage-class-name>` 相关的持久卷申领中，存储请求的总和不能超过该值。 |
| `<storage-class-name>.storageclass.storage.k8s.io/persistentvolumeclaims` | 在与 storage-class-name 相关的所有持久卷申领中，命名空间中可以存在的[持久卷申领](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)总数。 |

### 对象数量配额

可以使用以下语法为 Kubernetes API 中“一种特定资源类型的总数”设置配额：

- `count/<resource>.<group>`：用于非核心（core）组的资源
- `count/<resource>`：用于核心组的资源

这是用户可能希望利用对象计数配额来管理的一组资源示例：

- `count/persistentvolumeclaims`
- `count/services`
- `count/secrets`
- `count/configmaps`
- `count/replicationcontrollers`
- `count/deployments.apps`
- `count/replicasets.apps`
- `count/statefulsets.apps`
- `count/jobs.batch`
- `count/cronjobs.batch`

还有另一种语法仅用于为某些资源设置相同类型的配额。

支持以下类型：

| 资源名称                 | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `configmaps`             | 在该命名空间中允许存在的 ConfigMap 总数上限。                |
| `persistentvolumeclaims` | 在该命名空间中允许存在的 [PVC](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) 的总数上限。 |
| `pods`                   | 在该命名空间中允许存在的非终止状态的 Pod 总数上限。Pod 终止状态等价于 Pod 的 `.status.phase in (Failed, Succeeded)` 为真。 |
| `replicationcontrollers` | 在该命名空间中允许存在的 ReplicationController 总数上限。    |
| `resourcequotas`         | 在该命名空间中允许存在的 ResourceQuota 总数上限。            |
| `services`               | 在该命名空间中允许存在的 Service 总数上限。                  |
| `services.loadbalancers` | 在该命名空间中允许存在的 LoadBalancer 类型的 Service 总数上限。 |
| `services.nodeports`     | 在该命名空间中允许存在的 NodePort 或 LoadBalancer 类型的 Service 的 NodePort 总数上限。 |
| `secrets`                | 在该命名空间中允许存在的 Secret 总数上限。                   |

## 案例：对命名空间配置资源

K8S中提供了两种在命名空间级别对资源限制的的方式：ResourceQuota 和LimitRange。

其中ResourceQuota 是针对namespace做的资源限制，而LimitRange是针对namespace中的每个组件做的资源限制，如pod等。
即：

对namespace中容器、pod等使用总和限制

- ResourceQuota

对namespace中容器、pod等使用单独限制：

- LimitRange

### ResourceQuota限制的资源类型

```java
计算资源，包括cpu和memory
cpu, limits.cpu, requests.cpu
memory, limits.memory, requests.memory
 
资源名称              描述
limits.cpu          所有非终止状态的 Pod，其 CPU 限额总量不能超过该值。
limits.memory       所有非终止状态的 Pod，其内存限额总量不能超过该值。
requests.cpu        所有非终止状态的 Pod，其 CPU 需求总量不能超过该值。
requests.memory     所有非终止状态的 Pod，其内存需求总量不能超过该值。
hugepages-<size>    对于所有非终止状态的 Pod，针对指定尺寸的巨页请求总数不能超过此值。
cpu                 与 requests.cpu 相同。
memory              与 requests.memory 相同。
 
----------------------------------------------------------------
存储资源，包括存储资源的总量以及指定storage class的总量
requests.storage：存储资源总量，如500Gi
persistentvolumeclaims：pvc的个数
.storageclass.storage.k8s.io/requests.storage
.storageclass.storage.k8s.io/persistentvolumeclaims
例如，如果一个操作人员针对 `gold` 存储类型与 `bronze` 存储类型设置配额， 操作人员可以定义如下配额：
 
*   `gold.storageclass.storage.k8s.io/requests.storage: 500Gi`
*   `bronze.storageclass.storage.k8s.io/requests.storage: 100Gi`
在 Kubernetes 1.8 版本中，本地临时存储的配额支持已经是 Alpha 功能：
 
requests.ephemeral-storage    在命名空间的所有 Pod 中，本地临时存储请求的总和不能超过此值。 
limits.ephemeral-storage      在命名空间的所有 Pod 中，本地临时存储限制值的总和不能超过此值。
ephemeral-storage             与 requests.ephemeral-storage相同。 
----------------------------------------------------------------
对象数，即可创建的对象的个数
pods, replicationcontrollers, configmaps, secrets
resourcequotas, persistentvolumeclaims
services, services.loadbalancers, services.nodeports
 
资源名称                      描述
Configmaps                   在该命名空间中，能存在的 ConfigMap 的总数上限。
Persistentvolumeclaims       在该命名空间中，能存在的持久卷的总数上限。
Pods                         在该命名空间中，能存在的非终止状态的 Pod 的总数上限。Pod 终止状态等价于 Pod 的 status.phase 状态值为 Failed 或者 Succeed is true。
Replicationcontrollers       在该命名空间中，能存在的 RC 的总数上限。
Resourcequotas               在该命名空间中，能存在的资源配额项（ResourceQuota）的总数上限。
Services                     在该命名空间中，能存在的 service 的总数上限。
services.loadbalancers       在该命名空间中，能存在的负载均衡（LoadBalancer）的总数上限。
services.nodeports           在该命名空间中，能存在的 NodePort 的总数上限。
Secrets                      在该命名空间中，能存在的 Secret 的总数上限。
```

**案例**

```java
 
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-test
  namespace: test
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 2Gi
    limits.cpu: "4"
    limits.memory: 4Gi
    requests.nvidia.com/gpu: 4
    pods: "3"
    services: "6"
```

#### 使用条件--工作原理

    在 API Server 启动时配置准入控制 --admission-control=ResourceQuota
    
    在 namespace 中创建一个 ResourceQuota 对象，并且每个 Namespace 最多只能有一个 ResourceQuota 对象
    
    创建容器时必须配置计算资源请求或限制
    
    用户超额后禁止创建新的资源

------------------------------------------------------------------------------------------------------------------------

    在使用前需确认apiserver的配置文件中的KUBE_ADMISSION_CONTROL是否有ResourceQuota，如果没有需要添加并重启apiserver。
    
    Quota依赖于资源管理器，可以使用资源对象limits或者在创建资源对象是为pod设置资源限制（resources），如果不设置，资源对象无法创建。
    
    当该namespace中的任意个额度达到预设Quota时，将无法创建资源对象。

### LimitRange

默认情况下，Kubernetes 中所有容器都没有任何 CPU 和内存限制，这就可能导致某个容器应用内存泄露耗尽资源影响其它应用的情况。LimitRange 用来给 Namespace 增加一个资源限制，包括最小、最大和默认资源。

#### 为什么使用 LimitRange

使用LimitRange对象，我们可以：

1. 限制namespace中每个Pod或容器的最小与最大计算资源

2. 限制namespace中每个Pod或容器计算资源request、limit之间的比例

3. 限制namespace中每个存储卷声明（PersistentVolumeClaim）可使用的最小与最大存储空间

4. 设置namespace中容器默认计算资源的request、limit，并在运行时自动注入到容器中

如果创建或更新对象（Pod、容器、PersistentVolumeClaim）对资源的请求与LimitRange相冲突，apiserver会返回HTTP状态码403，以及相应的错误提示信息；

如果namespace中定义了LimitRange 来限定CPU与内存等计算资源的使用，则用户创建Pod、容器时，必须指定CPU或内存的request与limit，否则将被系统拒绝；

当namespace总的limit小于其中Pod、容器的limit之和时，将发生资源争夺，Pod或者容器将不能创建，但不影响已经创建的Pod或容器。

#### LimitRange资源使用说明

管理员在一个命名空间内创建一个 LimitRange 对象。用户在命名空间内创建 Pod ，Container 和 PersistentVolumeClaim 等资源。LimitRanger 准入控制器对所有没有设置计算资源需求的 Pod 和 Container 设置默认值与限制值， 并跟踪其使用量以保证没有超出命名空间中存在的任意 LimitRange 对象中的最小、最大资源使用量以及使用量比值。
若创建或更新资源（Pod、 Container、PersistentVolumeClaim）违反了 LimitRange 的约束， 向 API 服务器的请求会失败，并返回 HTTP 状态码 403 FORBIDDEN 与描述哪一项约束被违反的消息。
若命名空间中的 LimitRange 启用了对 cpu 和 memory 的限制， 用户必须指定这些值的需求使用量与限制使用量。否则，系统将会拒绝创建 Pod。
LimitRange 的验证仅在 Pod 准入阶段进行，不对正在运行的 Pod 进行验证。

示例：

    apiVersion: v1
    kind: LimitRange
    metadata:
      name: lr-test
    spec:
      limits:
      - type: Container       #资源类型
        max:
          cpu: "1"            #限定最大CPU
          memory: "1Gi"       #限定最大内存
        min:
          cpu: "100m"         #限定最小CPU
          memory: "100Mi"     #限定最小内存
        default:
          cpu: "900m"         #默认CPU限定
          memory: "800Mi"     #默认内存限定
        defaultRequest:
          cpu: "200m"         #默认CPU请求
          memory: "200Mi"     #默认内存请求
        maxLimitRequestRatio:
          cpu: 2              #限定CPU limit/request比值最大为2  
          memory: 1.5         #限定内存limit/request比值最大为1.5
      - type: Pod
        max:
          cpu: "2"            #限定Pod最大CPU
          memory: "2Gi"       #限定Pod最大内存
      - type: PersistentVolumeClaim
        max:
          storage: 2Gi        #限定PVC最大的requests.storage
        min:
          storage: 1Gi        #限定PVC最小的requests.storage

## 总结

学习LimitRange和ResourceQuota。我们需要始终记住：

- ResourceQuota 是针对namespace做的资源限制，而LimitRange是针对namespace中的每个组件做的资源限制