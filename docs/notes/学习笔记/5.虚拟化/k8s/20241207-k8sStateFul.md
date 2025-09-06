---
title: K8s中StateFul类型资源
icon: streamline-freehand-color:notes-book
order: 5
author: bugcode
date: 2024-12-7
category:
  - K8S
tag:
  - k8s
  - 场景
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 14:16:48
permalink: /learning-notes/virtualization/k8s/k8sStateFul/
---



## K8s中StateFul类型资源

### 介绍

RC、Deployment、DaemonSet都是面向无状态的服务，它们所管理的Pod的IP、名字，启停顺序等都是随机的，而StatefulSet是什么？顾名思义，有状态的集合，管理所有有状态的服务，比如MySQL、MongoDB集群等。

StatefulSet本质上是Deployment的一种变体，在v1.9版本中已成为GA版本，它为了解决有状态服务的问题，它所管理的Pod拥有固定的Pod名称，启停顺序，在StatefulSet中，**Pod名字称为网络标识**(hostname)，还必须要用到共享存储。

在Deployment中，与之对应的服务是service，而在StatefulSet中与之对应的headless service，headless service，即无头服务，与service的区别就是它没有Cluster IP，解析它的名称时将返回该Headless Service对应的全部Pod的Endpoint列表。

每个由`StatefulSet`创建出来的`Pod`都拥有一个序号（从0开始）和一个固定的网络标识。你还可以在`YAML`定义中添加**VolumeClaimTemplate**来声明Pod存储使用的`PVC`。当`StatefulSet`部署`Pod`时，会从编号0到最终编号逐一部署每个`Pod`，只有前一个`Pod`部署完成并处于运行状态后，下一个`Pod`才会开始部署。

除此之外，StatefulSet在Headless Service的基础上又为StatefulSet控制的每个Pod副本创建了一个DNS域名，这个域名的格式为：
`$(podname).(headless server name)`
全称为:`$(podname).(headless server name).namespace.svc.cluster.local`

> 所以Stateful类型资源 的核心就是，过某种方式记录这些状态，然后在 Pod 被重新创建时，能够为新 Pod 恢复这些状态。

### 特点

1. Pod一致性：包含次序（启动、停止次序）、网络一致性。此一致性与Pod相关，与被调度到哪个node节点无关；
2. 稳定的次序：对于N个副本的StatefulSet，每个Pod都在[0，N)的范围内分配一个数字序号，且是唯一的；
3. 稳定的网络：Pod的hostname模式为(statefulset名称)−(序号)；
4. 稳定的存储：通过VolumeClaimTemplate为每个Pod创建一个PV。删除、减少副本，不会删除相关的卷。



### 组成部分

1. Headless Service：用来定义Pod网络标识( DNS domain)；
2. volumeClaimTemplates ：存储卷申请模板，创建PVC，指定pvc名称大小，将自动创建pvc，且pvc必须由存储类供应；
3. StatefulSet ：定义具体应用，名为Nginx，有三个Pod副本，并为每个Pod定义了一个域名部署statefulset。

### HeadlessService

想要维护应用的拓扑状态，必须保证能用固定的网络标识访问到固定的`Pod`实例，`Kubernetes`是通过`Headless Service`给每个`Endpoint（Pod）`添加固定网络标识的，所以接下来我们花些时间了解下`Headless Service`。

**`Service`是在逻辑抽象层上定义了一组`Pod`，为他们提供一个统一的固定IP和访问这组`Pod`的负载均衡策略**。

对于 `ClusterIP` 模式的 `Service` 来说，它的 A 记录的格式是:
**`serviceName.namespace.svc.cluster.local`**，当你访问这条 A 记录的时候，它解析到的就是该 Service 的 VIP 地址。

对于指定了 clusterIP=None 的 Headless Service来说，它的A记录的格式跟上面一样，但是访问记录后返回的是Pod的IP地址集合。Pod 也会被分配对应的 DNS A 记录，格式为：**`podName.serviceName.namesapce.svc.cluster.local`**

普通的`Service`都有`ClusterIP`，它其实就是一个虚拟`IP`，会把请求转发到该`Service`所代理的某一个`Pod`上。

### 为什么需要 headless service 无头服务？

在用Deployment时，每一个Pod名称是没有顺序的，是随机字符串，因此是Pod名称是无序的，但是在statefulset中要求必须是有序 ，每一个pod不能被随意取代，pod重建后pod名称还是一样的。而pod IP是变化的，所以是以Pod名称来识别。pod名称是pod唯一性的标识符，必须持久稳定有效。这时候要用到无头服务，它可以给每个Pod一个唯一的名称 。



### 为什么需要volumeClaimTemplate？

对于有状态的副本集都会用到持久存储，对于分布式系统来讲，它的最大特点是数据是不一样的，所以各个节点不能使用同一存储卷，每个节点有自已的专用存储，但是如果在Deployment中的Pod template里定义的存储卷，是所有副本集共用一个存储卷，数据是相同的，因为是基于模板来的 ，而statefulset中每个Pod都要自已的专有存储卷，所以statefulset的存储卷就不能再用Pod模板来创建了，于是statefulSet使用volumeClaimTemplate，称为卷申请模板，它会为每个Pod生成不同的pvc，并绑定pv，从而实现各pod有专用存储。这就是为什么要用volumeClaimTemplate的原因。

## Stateful资源案例

使用的`Service`和`Deployment`的定义如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  type: NodePort #创建NodePort类型Service时会先创建一个ClusterIp类型的Service
  selector:
    app: go-app
  ports:
    - name: http
      protocol: TCP
      nodePort: 30080
      port: 80
      targetPort: 3000

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-go-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: go-app
  template:
    metadata:
      labels:
        app: go-app
    spec:
      containers:
        - name: go-app-container
          image: kevinyan001/kube-go-app
          ports:
            - containerPort: 3000
```

在Kubernetes里创建好上述资源后，可以进入其中一个`Pod`查看Service的A记录

```text
kubectl exec -it my-go-app-69d6844c5c-gkb6z  -- /bin/sh
/app # nslookup app-service.default.svc.cluster.local
Server:     10.96.0.10 
Address:    10.96.0.10:53

Name:   app-service.default.svc.cluster.local
Address: 10.108.26.155
```

如果想让DNS通过刚才的`Service`名直接解析出`Pod`名对应的IP是不可以的：

```text
/app # nslookup my-go-app-69d6844c5c-gkb6z.app-service.default.svc.cluster.local
Server:     10.96.0.10
Address:    10.96.0.10:53

** server can't find my-go-app-69d6844c5c-gkb6z.app-service.default.svc.cluster.local: NXDOMAIN
```

因为`Service`有`ClusterIp`，直接被DNS解析了，那怎么才能让DNS通过`Service`解析`Pod`的IP呢？所以就有了`Headless Service`。

>  创建`Headless Service`跟创建普通`Service`时唯一的不同就是在`YAML`定义里指定`spec:clusterIP: None`，也就是不需要`ClusterIP`的`Service`。
>

下面我创建一个`Headless Service`代理上面例子中的那两个应用`Pod`实例，它的`YAML`定义如下

```yaml
# headless-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: app-headless-svc
spec:
  clusterIP: None # <-- Don't forget!!
  selector:
    app: go-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
```

> **创建Service的命令**
>  ➜ kubectl apply -f headless-service.yaml service/app-headless-svc created
>

`Headless Service`创建完后，我们再来看一下这个`Service`在DNS里对应的A记录

>  还是在刚才进入的那个Pod里，记住Service的DNS记录的格式是 **serviceName.namespace.svc.cluster.local**
>

```yaml
/app # nslookup app-headless-svc.default.svc.cluster.local
Server:     10.96.0.10
Address:    10.96.0.10:53

Name:   app-headless-svc.default.svc.cluster.local
Address: 10.1.0.38
Name:   app-headless-svc.default.svc.cluster.local
Address: 10.1.0.39
```

DNS查询会返回`HeadlessService`代理的两个`Endpoint (Pod)`对应的IP，这样客户端就能通过`Headless Service`拿到每个`EndPoint`的 IP，如果有需要可以自己在客户端做些负载均衡策略。**`Headless Service`还有一个重要用处**（也是使用`StatefulSet`时需要`Headless Service`的真正原因）**，它会为代理的每一个`StatefulSet`创建出来的`Endpoint`也就是`Pod`添加DNS域名解析，这样`Pod`之间就可以相互访问。**

> **划重点**:
>
> 1. 这个分配给`Pod`的DNS域名就是`Pod`的固定唯一网络标识，即使发生重建和调度DNS域名也不会改变。
> 2. Deployment创建的`Pod`的名字是随机的，所以`HeadlessService`不会为`Deployment`创建出来的Pod单独添加域名解析。

我们把上面的例子稍作修改，新增一个`StatefulSet`对象用它创建`Pod`来验证一下。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-headless-svc
spec:
  clusterIP: None # <-- Don't forget!!
  selector:
    app: stat-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000

---
apiVersion: apps/v1
kind: StatefulSet # <-- claim stateful set
metadata:
  name: stat-go-app
spec:
  serviceName: app-headless-svc # <-- Set headless service name
  replicas: 2
  selector:
    matchLabels:
      app: stat-app
  template:
    metadata:
      labels:
        app: stat-app
    spec:
      containers:
        - name: go-app-container
          image: kevinyan001/kube-go-app
          resources:
            limits:
              memory: "64Mi"
              cpu: "50m"
          ports:
            - containerPort: 3000
```

这个`YAML`文件，和我们在前面用到的`Deployment`的唯一区别，就是多了一个`spec.serviceName` 字段。

**`StatefulSet`给它所管理的所有 `Pod` 名字，进行了编号，编号规则是：StatefulSet名-序号。这些编号都是从 0 开始累加，与 StatefulSet 的每个 Pod 实例一一对应，绝不重复**。

```text
➜ kubectl get pod                                        
NAME            READY   STATUS              RESTARTS   AGE
stat-go-app-0   1/1     Running             0          9s
stat-go-app-1   0/1     ContainerCreating   0          1s
```

我们可以进入`stat-go-app-0`这个`Pod`查看一下这两个`Pod`的DNS记录

> **提示**： Headless Service给Pod添加的DNS的格式为**podName.serviceName.namesapce.svc.cluster.local**
>

```text
/app # nslookup stat-go-app-0.app-headless-svc.default.svc.cluster.local
Server:     10.96.0.10
Address:    10.96.0.10:53

Name:   stat-go-app-0.app-headless-svc.default.svc.cluster.local
Address: 10.1.0.46

/app # nslookup stat-go-app-1.app-headless-svc.default.svc.cluster.local
Server:     10.96.0.10
Address:    10.96.0.10:53

Name:   stat-go-app-1.app-headless-svc.default.svc.cluster.local
Address: 10.1.0.47
```

于是乎这样就保证了`Pod`之间能够相互通信，如果要用`StatefulSet`编排一个有主从关系的应用，就可以通过DNS域名访问的方式保证相互之间的通信，即使出现`Pod`重新调度它在内部的DNS域名也不会改变。

### 保持Pod的编排顺序

通过上面名字叫`stat-go-app`的`StatefulSet`控制器创建`Pod`的过程我们能发现，**`StatefulSet`它所管理的所有 `Pod` ，名称的命名规则是：StatefulSet名-序号。序号都是从 0 开始累加，与 StatefulSet 的每个 Pod 实例一一对应，绝不重复**。

所以上面我们通过`kubectl get pod` 命令看到了两个名字分别为`stat-go-app-0`和`stat-go-app-1`的Pod实例。

更重要的是，这些`Pod`的创建，也是严格按照名称的编号顺序进行的。比如，在`stat-go-app-0`进入到 Running 状态、并且细分状态（Conditions）成为 Ready 之前，`stat-go-app-1`会一直处于 Pending 等待状态。

`StatefulSet`会一直记录着这个拓扑状态，即使发生调谐，重新调度`Pod`也是严格遵守这个顺序，编号在前面的`Pod`创建完成并且进入Ready运行状态后，下一个编号的`Pod`才会开始创建。

### 保持Pod固定唯一网络标识

理解了`Headless Service`真正的用途后，关于`Kubernetes`内部如何让`Pod`固定唯一网络标识这个问题的答案就是：`Headless Service`为代理的每一个`StatefulSet`创建出来的`Pod`添加DNS域名解析。所以在用`StatefulSet`编排实例之间有主从关系这样的有状态应用时，`Pod`相互之间就能以**podName.serviceName.namesapce.svc.cluster.local** 这个域名格式进行通信，这样就不用在担心`Pod`被重新调度到其他的节点上后IP的变化。

### 保持实例的存储状态

在介绍`Pod`使用的数据卷的时候，我曾提到过，要在一个`Pod`里声明 `Volume`，只需要在`Pod`定义里加上`spec.volumes` 字段即可。然后，你就可以在这个字段里定义一个具体类型的`Volume`了，比如：`hostPath`类型。

```yaml
......
spec:
  volumes:
  - name: app-volume  
    hostPath:
      # 在宿主机上的目录位置
      path: /data
  containers:
  - image: mysql:5.7
    name: mysql
    ports:
      - containerPort: 3306
    volumeMounts:
    - mountPath: /usr/local/mysql/data
      name: app-volume
......
```

但是这种声明使用数据卷的方式，对于每个`Pod`实例都绑定了存储数据的数据存储类应用是不适用的。由于`hostPath`类型的Volume是基于宿主机目录的，如果一旦`Pod`发生重新调度，去了其他节点，就没有办法在新节点上把`Pod`的存储数据恢复回来了。

既然在`Pod`宿主机上的数据卷不适用，那么只能让`Pod`去使用`Kubernetes`的集群存储资源了。集群持久数据卷资源的配置和使用是通过`PV`和`PVC`完成的，我们先来了解一下这两个概念。

### PV和PVC

持久卷（PersistentVolume，PV）是集群中的一块存储，可以由管理员事先供应，或者 使用存储类（Storage Class）来动态供应。 **持久卷是集群资源，就像节点也是集群资源一样，它们拥有独立于任何使用`PV`的`Pod`的生命周期**。

作为一个应用开发者，我可能对分布式存储项目（比如 Ceph、GFS、HDFS 等）一窍不通，自然不会编写它们对应的 Volume 定义文件，这不仅超越了开发者的知识储备，还会有暴露公司基础设施敏感信息（秘钥、管理员密码等信息）的风险。所以`Kubernetes`后来又引入了持久卷申领（PersistentVolumeClaim，PVC）。

`PVC`表达的是`Pod`对存储的请求。概念上与`Pod`类似。 `Pod`会耗用节点资源，而`PVC`申领会耗用`PV`资源。有了`PVC`后，在需要使用持久卷的`Pod`的定义里只需要声明使用这个`PVC`即可，这为使用者隐去了很多关于存储的信息，举个例子来说就是，我可以在完全不知道远程存储的空间名、服务器地址、AccessKey之类的信息时直接把远程存储挂载到`Pod`的容器里。比如像下面这样：

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pv-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
    - name: pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-storage
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pv-claim
```

可以看到，在这个`Pod` 的`Volumes` 定义中，我们只需要声明它的类型是 `persistentVolumeClaim`，然后指定 `PVC` 的名字，完全不用关心持久卷本身的定义。

`PVC`创建出来后需要和`PV`完成绑定才能使用，不过对于使用者来说我们可以先不用关心这个细节。

>  可以用编程领域的**接口**和**实现**的关系来理解`PVC`和`PV`的关系。
>

### StatefulSet的PVC模板

关于StatefulSet、Pod、PVC和PV之间的关系可以用下面这张图表示

![image-20241207131641151](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241207131641151.png)

在`StatefulSet`的定义里我们可以额外添加了一个`spec.volumeClaimTemplates`字段。它跟 `Pod`模板（`spec.template`字段）的作用类似。

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.1
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
```

> **Note**: StatefulSet和Deployment里都有Pod模板，他是控制器创建Pod实例的依据；
>

也就是说，凡是被这个`StatefulSet`管理的`Pod`，都会声明一个对应的`PVC`；而这个`PVC`的定义，就来自于`volumeClaimTemplates`这个模板字段。更重要的是，这个 `PVC` 的名字，会被分配一个与这个`Pod`完全一致的编号。

`StatefulSet`创建的这些`PVC`，都以**"PVC名-StatefulSet名-序号"**这个格式命名的。

对于上面这个`StatefulSet`来说，它创建出来的`Pod`和`PVC`的名称如下：

```text
Pod： web-0, web-1
PVC： www-web-0, www-web-1
```

假如发生重新调度`web-0`这个`Pod`被重新创建调度到了其他节点，在这个新的`Pod`对象的定义里，由于`volumeClaimTemplates`的存在，它声明使用的`PVC`的名字，还是叫作：**www-web-0**。所以，在这个新的`web-0`被创建出来之后，`Kubernetes`会为它查找绑定名叫`www-web-0`的`PVC`。**由于`PVC`的生命周期是独立于使用它的`Pod`的，这样新`Pod`就接管了以前旧`Pod`留下的数据**。

## 总结

`StatefulSet`就像是一种特殊的`Deployment`，它使用`Kubernetes`里的两个标准功能：`Headless Service` 和 `PVC`，实现了对的拓扑状态和存储状态的维护。

`StatefulSet`通过`Headless Service` ， 为它管控的每个`Pod`创建了一个固定保持不变的DNS域名，来作为`Pod`在集群内的网络标识。加上为`Pod`进行编号并严格按照编号顺序进行`Pod`调度，这些机制保证了`StatefulSet`对维护应用拓扑状态的支持。

而借由`StatefulSet`定义文件中的`volumeClaimTemplates`声明`Pod`使用的`PVC`，它创建出来的`PVC`会以名称编号这些约定与它创建出来的`Pod`进行绑定，借由`PVC`独立于`Pod`的生命周期和两者之间的绑定机制的帮助，`StatefulSet`完成了应用存储状态的维护。

