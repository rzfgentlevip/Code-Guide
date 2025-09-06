---
title: K8s Service和Ingress使用
icon: streamline-freehand-color:notes-book
order: 4
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - K8S
tag:
  - k8s
  - docker
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 14:16:48
permalink: /learning-notes/virtualization/k8s/k8s-ingressAndService/
---

# K8s service和ingress

# 1、Service介绍

在kubernetes中,pod是应用程序的载体,我们可以通过pod的ip来访问应用程序,但是pod的ip地址不是固定的,如果pod挂掉或者重启后,其ip会发生变化,也就意味着每次pod重启后都需要手工更换ip去访问应用程序。

为了解决这个问题,kubernetes提供了Service资源,Service会对提供同一个服务的多个pod进行聚合,并且提供一个统一的入口地址,一个固定的ip地址。通过访问Service的入口地址就能访问到后面的pod服务,即使pod发生重启或者删除,用户也不需更换service ip。对用户来说service ip是不变的,用户无法感知pod ip地址。

Service在很多情况下只是一个概念,真正起作用的其实是kube-proxy服务进程,每个Node节点上都运行着一个kube-proxy服务进程。当创建Service的时候会通过api-server向etcd写入创建的service的信息,而kube-proxy会基于监听的机制发现这种Service的变动,然后**它会将最新的Service信息转换成对应的访问规则。**


service资源定义:

```yaml
kind: Service  # 定义资源类型
apiVersion: v1  # 定义资源版本
metadata: # 元数据
  name: service # 资源名称
  namespace: dev # 命名空间
spec: # 定义资源的描述信息
  selector: # 标签选择器,用于确定当前service代理哪些pod,只支持精准匹配,基于标签选择固定的pod
			# 所以即使pod挂掉,只要重启后的标签不变,也就意味着仍然在一组service内,可以对外提供服务
    app: nginx
  type: # Service类型,指定service的访问方式,如果定义NodePort,就要定义他的端口
		# ClusterIP,NotePort,LoaderBalancer,ExternalName四种类型
  clusterIP:  # 虚拟服务的ip地址,定义ip,不写的话默认自动生成,必须在service网段之内
  sessionAffinity: # session亲和性,支持ClientIP、None两个选项
  ports: # 端口信息
    - protocol: TCP 
      port: 3017  # service端口
      targetPort: 5003 # pod端口
      nodePort: 31122 # 主机端口,默认是30000-32700
      protocal: TCP     #反向代理的网络类型
```

端口说明:

- targetPort:pod端口
- port:service资源类型的端口，一般情况下,都是将port设置成和targetPort一样的值。 
- clusterIP:集群ip,虚拟的ip地址,集群内部使用。

service默认支持的协议是TCP协议

## 1.1、Service配置多端口

也可以为一个service公开多个端口信息:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

pod中的端口也可以指定一个名字,然后在service资源中通过端口名字指定

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```

# 2、kube-proxy工作模式

工作模式可以参考本篇介绍:[kube-proxy工作模式](https://codinglab.online/ContainerCloud/k8sTheory/act_one_k8s%E4%B8%AD%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E5%8E%9F%E7%90%86.md.html)

# 3、Service类型

Service资源清单:

```yaml
kind: Service  # 资源类型
apiVersion: v1  # 资源版本
metadata: # 元数据
  name: service # 资源名称
  namespace: dev # 命名空间
spec: # 描述
  selector: # 标签选择器,用于确定当前service代理哪些pod,只支持精准匹配
    app: nginx
  type: # Service类型,指定service的访问方式,如果定义NodePort,就要定义他的端口
  clusterIP:  # 虚拟服务的ip地址
  sessionAffinity: # session亲和性,支持ClientIP、None两个选项
  ports: # 端口信息
    - protocol: TCP 
      port: 3017  # service端口
      targetPort: 5003 # pod端口
      nodePort: 31122 # 主机端口,默认是30000-32700
      protocal: TCP     #反向代理的网络类型
```

对一些应用的某些部分,你可能希望将其公开于某外部 IP 地址, 也就是可以从集群外部访问的某个地址。Kubernetes Service 类型允许指定你所需要的 Service 类型。

type字段可选的类型:

- ClusterIP:默认值,它是Kubernetes系统自动分配的虚拟IP,只能在集群内部访问,容器内部的其他pod可以访问此ip,外部服务不能通过ClusterIp访问服务,但是可以使用 [Ingress](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/)或者 [Gateway API](https://gateway-api.sigs.k8s.io/) 向公共互联网公开服务。
- NodePort:将Service通过指定的Node上的端口暴露给外部,通过此方法,就可以在集群外部访问服务。通过每个节点上的 IP 和静态端口(`NodePort`)公开 Service。
  为了让 Service 可通过节点端口访问,Kubernetes 会为 Service 配置集群 IP 地址,
  相当于你请求了 `type: ClusterIP` 的服务。
- LoadBalancer:使用外接负载均衡器完成到服务的负载分发,注意此模式需要外部云环境支持,使用云平台的负载均衡器向外部公开 Service。Kubernetes 不直接提供负载均衡组件; 你必须提供一个,或者将你的 Kubernetes 集群与某个云平台集成。
- ExternalName: 把集群外部的服务引入集群内部,直接使用

> 服务 API 中的 `type` 字段被设计为层层递进的形式 - 每层都建立在前一层的基础上。
> 但是,这种层层递进的形式有一个例外。
> 你可以在定义 `LoadBalancer` 服务时[禁止负载均衡器分配 `NodePort`](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#load-balancer-nodeport-allocation)

# 4、如何使用Service

使用service资源,首先要创建一组pod并且指定标签,注意要为pod设置`app=nginx-pod`的标签,deployment.yaml文件内容:

```yaml
apiVersion: apps/v1
kind: Deployment      
metadata:
  name: pc-deployment
  namespace: dev
spec: 
  replicas: 3 #创建3个副本
  selector: # 匹配标签
    matchLabels:
      app: nginx-pod  
  template:
    metadata:
      labels:
        app: nginx-pod # pod模板标签
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
        ports:
        - containerPort: 80
```

此时可以查看pod创建情况:

```yaml
kubectl create -f deployment.yaml

```

查看pod详细信息:

```yaml
kubectl get pods -n dev -o wide --show-labels

# 为了方便后面的测试,修改下三台nginx的index.html页面(三台修改的IP地址不一致)
# kubectl exec -it pc-deployment-66cb59b984-8p84h -n dev /bin/sh
# echo "10.244.1.39" > /usr/share/nginx/html/index.html

#修改完毕之后,访问测试
[root@k8s-master01 ~]# curl 10.244.1.39
10.244.1.39
[root@k8s-master01 ~]# curl 10.244.2.33
10.244.2.33
[root@k8s-master01 ~]# curl 10.244.1.40
10.244.1.40
```

## 4.1、ClusterIP类型的Service

ClusterIp是k8s默认的service类型。如果不指定service的ip地址,k8s会默认分配一个ip地址给service资源。

- 此默认 Service 类型从你的集群中为此预留的 IP 地址池中分配一个 IP 地址。
- 其他几种 Service 类型在 `ClusterIP` 类型的基础上进行构建。
- 如果你定义的 Service 将 `.spec.clusterIP` 设置为 `"None"`,则 Kubernetes不会为其分配 IP 地址。

创建service-clusterip.yaml文件

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-clusterip
  namespace: dev
spec:
  selector:
    app: nginx-pod
  clusterIP: 10.97.97.97 # service的ip地址,如果不写,默认会生成一个
  type: ClusterIP
  ports:
  - port: 80  # Service端口       
    targetPort: 80 # pod端口,内部代理的pod 端口
```

创建service资源:

```yaml
# 创建service
kubectl create -f service-clusterip.yaml
service/service-clusterip created

# 查看service
kubectl get svc -n dev -o wide
NAME                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service-clusterip   ClusterIP   10.97.97.97   <none>        80/TCP    13s   app=nginx-pod

# 查看service的详细信息
# 在这里有一个Endpoints列表,里面就是当前service可以负载到的服务入口
[root@k8s-master01 ~]# kubectl describe svc service-clusterip -n dev
Name:              service-clusterip
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx-pod
Type:              ClusterIP
IP:                10.97.97.97
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.39:80,10.244.1.40:80,10.244.2.33:80
Session Affinity:  None
Events:            <none>

# 查看ipvs的映射规则
[root@k8s-master01 ~]# ipvsadm -Ln
TCP  10.97.97.97:80 rr
  -> 10.244.1.39:80               Masq    1      0          0
  -> 10.244.1.40:80               Masq    1      0          0
  -> 10.244.2.33:80               Masq    1      0          0

# 访问10.97.97.97:80观察效果
[root@k8s-master01 ~]# curl 10.97.97.97:80
10.244.2.33
```

**Endpoint**

Endpoint是kubernetes中的一个资源对象,存储在etcd中,用来记录一个service对应的所有pod的访问地址,它是根据service配置文件中selector描述产生的。

一个Service由一组Pod组成,这些Pod通过Endpoints暴露出来,**Endpoints是实现实际服务的端点集合**。换句话说,**service和pod之间的联系是通过endpoints实现的**。

查看endpoints列表:

```yaml
kubectl get endpoints -n dev -o wide 
```

**负载分发策略**

对Service的访问被分发到了后端的Pod上去,目前kubernetes提供了两种负载分发策略:

- 如果不定义,默认使用kube-proxy的策略,比如随机、轮询
- 基于客户端地址的会话保持模式,即来自同一个客户端发起的所有请求都会转发到固定的一个Pod上,此模式可以使在spec中添加`sessionAffinity:ClientIP`选项

```yaml
# 查看ipvs的映射规则【rr 轮询】
[root@k8s-master01 ~]# ipvsadm -Ln
TCP  10.97.97.97:80 rr
  -> 10.244.1.39:80               Masq    1      0          0
  -> 10.244.1.40:80               Masq    1      0          0
  -> 10.244.2.33:80               Masq    1      0          0

# 循环访问测试
[root@k8s-master01 ~]# while true;do curl 10.97.97.97:80; sleep 5; done;
10.244.1.40
10.244.1.39
10.244.2.33
10.244.1.40
10.244.1.39
10.244.2.33

# 修改分发策略----sessionAffinity: ClientIP

# 查看ipvs规则【persistent 代表持久】
[root@k8s-master01 ~]# ipvsadm -Ln
TCP  10.97.97.97:80 rr persistent 10800
  -> 10.244.1.39:80               Masq    1      0          0
  -> 10.244.1.40:80               Masq    1      0          0
  -> 10.244.2.33:80               Masq    1      0          0

# 循环访问测试
[root@k8s-master01 ~]# while true;do curl 10.97.97.97; sleep 5; done;
10.244.2.33
10.244.2.33
10.244.2.33
  
# 删除service
[root@k8s-master01 ~]# kubectl delete -f service-clusterip.yaml
service "service-clusterip" deleted
```

## 4.2、HeadLiness类型的Service

在某些场景中,开发人员可能不想使用Service提供的负载均衡功能,而希望自己来控制负载均衡策略,针对这种情况,kubernetes提供了HeadLiness Service,这类Service不会分配Cluster IP,如果想要访问service,只能通过service的域名进行查询。

创建service-headliness.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-headliness
  namespace: dev
spec:
  selector:
    app: nginx-pod
  clusterIP: None # 将clusterIP设置为None,即可创建headliness Service
  type: ClusterIP
  ports:
  - port: 80    
    targetPort: 80
```

创建service资源:

```yaml
# 创建service
[root@k8s-master01 ~]# kubectl create -f service-headliness.yaml
service/service-headliness created

# 获取service, 发现CLUSTER-IP未分配
[root@k8s-master01 ~]# kubectl get svc service-headliness -n dev -o wide
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service-headliness   ClusterIP   None         <none>        80/TCP    11s   app=nginx-pod

# 查看service详情
[root@k8s-master01 ~]# kubectl describe svc service-headliness  -n dev
Name:              service-headliness
Namespace:         dev
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx-pod
Type:              ClusterIP
IP:                None
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.39:80,10.244.1.40:80,10.244.2.33:80
Session Affinity:  None
Events:            <none>

# 查看域名的解析情况
[root@k8s-master01 ~]# kubectl exec -it pc-deployment-66cb59b984-8p84h -n dev /bin/sh
/ # cat /etc/resolv.conf
nameserver 10.96.0.10
search dev.svc.cluster.local svc.cluster.local cluster.local

[root@k8s-master01 ~]# dig @10.96.0.10 service-headliness.dev.svc.cluster.local
service-headliness.dev.svc.cluster.local. 30 IN A 10.244.1.40
service-headliness.dev.svc.cluster.local. 30 IN A 10.244.1.39
service-headliness.dev.svc.cluster.local. 30 IN A 10.244.2.33
```

## 4.3、NodePort类型的Service

在之前的样例中,创建的Service的ip地址只有集群内部才可以访问,如果希望将Service暴露给集群外部使用,那么就要使用到另外一种类型的Service,称为NodePort类型。NodePort的工作原理其实就是**将service的端口映射到Node的一个端口上**,然后就可以通过`NodeIp:NodePort`来访问service了。

如果将 `type` 字段设置为 `NodePort`,则 Kubernetes 控制平面将在
`--service-node-port-range` 标志所指定的范围内分配端口(默认值:30000-32767)。
每个节点将该端口(每个节点上的相同端口号)上的流量代理到 Service。你的 Service 在其 `.spec.ports[*].nodePort` 字段中报告已分配的端口。

创建service-nodeport.yaml:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-nodeport
  namespace: dev
spec:
  selector:
    app: nginx-pod
  type: NodePort # service类型
  ports:
  - port: 80
    nodePort: 30002 # 指定绑定的node的端口(默认的取值范围是:30000-32767), 如果不指定,会默认分配
    targetPort: 80
```

修改绑定node端口的范围:

```yaml
#指定绑定的node端口可以修改默认值
[root@k8s-m-01 ~]# vim /etc/kubernetes/manifests/kube-apiserver.yaml
--service-node-port-range=0-1000
[root@k8s-m-01 ~]# kubectl apply -f /etc/kubernetes/manifests/kube-apiserver.yaml
```

创建service对象

```yaml
# 创建service
[root@k8s-master01 ~]# kubectl create -f service-nodeport.yaml
service/service-nodeport created

# 查看service
[root@k8s-master01 ~]# kubectl get svc -n dev -o wide
NAME               TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)       SELECTOR
service-nodeport   NodePort   10.105.64.191   <none>        80:30002/TCP  app=nginx-pod

# 接下来可以通过电脑主机的浏览器去访问集群中任意一个nodeip的30002端口,即可访问到pod
```

## 4.4、LoadBalancer类型的Service

弹性IP，LoadBalancer和NodePort很相似,目的都是向外部暴露一个端口,区别在于LoadBalancer会在**集群的外部**再来做一个负载均衡设备,而这个设备需要外部环境支持的,外部服务发送到这个设备上的请求,会被设备负载之后转发到集群中。

必须要在公有云上创建集群弹性ip

![Untitled](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%207.png)

在使用支持外部负载均衡器的云平台时,如果将 `type` 设置为 `"LoadBalancer"`,
则平台会为 Service 提供负载均衡器。
负载均衡器的实际创建过程是异步进行的,关于所制备的负载均衡器的信息将会通过 Service 的
`status.loadBalancer` 字段公开出来。
例如:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 192.0.2.127
```

来自外部负载均衡器的流量将被直接重定向到后端各个 Pod 上,云平台决定如何进行负载平衡。

要实现 `type: LoadBalancer` 的服务,Kubernetes 通常首先进行与请求 `type: NodePort`
服务类似的更改。cloud-controller-manager 组件随后配置外部负载均衡器,以将流量转发到所分配的节点端口。

你可以将负载均衡 Service 配置为[忽略](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#load-balancer-nodeport-allocation)分配节点端口,前提是云平台实现支持这点。

某些云平台允许你设置 `loadBalancerIP`。这时,平台将使用用户指定的 `loadBalancerIP`
来创建负载均衡器。如果没有设置 `loadBalancerIP` 字段,平台将会给负载均衡器分配一个临时 IP。如果设置了 `loadBalancerIP`,但云平台并不支持这一特性,所设置的 `loadBalancerIP` 值将会被忽略。

## 4.5、ExternalName类型的Service

ExternalName类型的Service用于引入集群外部的服务,它通过`externalName`属性指定外部一个服务的地址,然后在集群内部访问此service就可以访问到外部的服务了。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-externalname
  namespace: dev
spec:
  type: ExternalName # service类型
  externalName: www.baidu.com  #改成ip地址也可以
#类型为 ExternalName 的 Service 将 Service 映射到 DNS 名称,而不是典型的选择算符,
#例如 my-service 或者 cassandra。你可以使用 spec.externalName 参数指定这些服务。
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

创建service资源:

```yaml
# 创建service
[root@k8s-master01 ~]# kubectl  create -f service-externalname.yaml
service/service-externalname created

# 域名解析
[root@k8s-master01 ~]# dig @10.96.0.10 service-externalname.dev.svc.cluster.local
service-externalname.dev.svc.cluster.local. 30 IN CNAME www.baidu.com.
www.baidu.com.          30      IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       30      IN      A       39.156.66.18
www.a.shifen.com.       30      IN      A       39.156.66.14
```

# 5、Ingress

> 参考:[Ingress说明](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/)

[Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.29/#ingress-v1-networking-k8s-io)提供从集群外部到集群内[服务](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/)的HTTP 和 HTTPS 路由。流量路由由 Ingress 资源所定义的规则来控制。典型的路由示意图如下图

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%208.png)

通过配置,Ingress 可为 Service 提供外部可访问的 URL、对其流量作负载均衡、 终止 SSL/TLS,以及基于名称的虚拟托管等能力。

## 5.1、Ingress介绍

Service对集群之外暴露服务的主要方式有两种:NotePort和LoadBalancer,但是这两种方式,都有一定的缺点:

- NodePort方式的缺点是会占用很多集群机器的端口,那么当集群服务变多的时候,这个缺点就愈发明显,每增加一个服务,就需要在node上分配一个端口。
- LB方式的缺点是每个service需要一个LB,浪费、麻烦,并且需要kubernetes之外设备的支持

基于这种现状,kubernetes提供了Ingress资源对象,Ingress只需要一个NodePort或者一个LB就可以满足暴露多个Service的需求。工作机制大致如下图表示:

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%209.png)

实际上,Ingress相当于一个**7层的负载均衡器**,是kubernetes对反向代理的一个抽象,它的工作原理类似于Nginx,可以理解成在**Ingress里建立诸多映射规则,Ingress Controller通过监听这些配置规则并转化成Nginx的反向代理配置 , 然后对外部提供服务**。在这里有两个核心概念:

- ingress:kubernetes中的一个对象,作用是**定义请求如何转发到service的规则**
- ingress controller:具体实现反向代理及负载均衡的程序,对ingress定义的规则进行解析,根据配置的规则来实现请求转发,实现方式有很多,比如Nginx, Contour, Haproxy等等。

Ingress(以Nginx为例)的工作原理如下:

1. 用户编写Ingress规则,**说明哪个域名对应kubernetes集群中的哪个Service**
2. Ingress控制器动态感知Ingress服务规则的变化,然后生成一段对应的Nginx反向代理配置
3. Ingress控制器会将生成的Nginx配置写入到一个运行着的Nginx服务中,并动态更新
4. 到此为止,其实真正在工作的就是一个Nginx了,内部配置了用户定义的请求转发规则

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%2010.png)

Ingress包含以下几个配置规则:

- 可选的 `host`。在此示例中,未指定 `host`,因此该规则基于所指定 IP 地址来匹配所有入站 HTTP 流量。如果提供了 `host`(例如 `foo.bar.com`),则 `rules` 适用于所指定的主机。
- 路径列表(例如 `/testpath`)。每个路径都有一个由 `service.name` 和 `service.port.name`
  或 `service.port.number` 确定的关联后端。主机和路径都必须与入站请求的内容相匹配,负载均衡器才会将流量引导到所引用的 Service,
- `backend`(后端)是 [Service 文档](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/)中所述的 Service 和端口名称的组合,
  或者是通过 [CRD](https://kubernetes.io/zh-cn/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)方式来实现的[自定义资源后端](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/#resource-backend)。对于发往 Ingress 的 HTTP(和 HTTPS)请求,如果与规则中的主机和路径匹配,则会被发送到所列出的后端。

## 5.2、Ingress的使用

首先创建一下两个pod服务

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/Untitled%2011.png)

创建tomcat-nginx.yaml deployment部署

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
        ports:
        - containerPort: 80

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tomcat-pod
  template:
    metadata:
      labels:
        app: tomcat-pod
    spec:
      containers:
      - name: tomcat
        image: tomcat:8.5-jre10-slim
        ports:
        - containerPort: 8080
```

创建service资源

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: dev
spec:
  selector:
    app: nginx-pod
  clusterIP: None
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
  namespace: dev
spec:
  selector:
    app: tomcat-pod
  clusterIP: None
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
```

创建部署:

```yaml
# 创建
[root@k8s-master01 ~]# kubectl create -f tomcat-nginx.yaml

# 查看
[root@k8s-master01 ~]# kubectl get svc -n dev
NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
nginx-service    ClusterIP   None         <none>        80/TCP     48s
tomcat-service   ClusterIP   None         <none>        8080/TCP   48s
```

## 5.3、Http代理

创建ingress-http.yaml文件

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-http
  namespace: dev
spec:
  rules:
  - host: nginx.rzf.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-service
          servicePort: 80
  - host: tomcat.rzf.com
    http:
      paths:
      - path: /
        backend:
          serviceName: tomcat-service
          servicePort: 8080
```

创建资源

```yaml
# 创建
[root@k8s-master01 ~]# kubectl create -f ingress-http.yaml
ingress.extensions/ingress-http created

# 查看
[root@k8s-master01 ~]# kubectl get ing ingress-http -n dev
NAME           HOSTS                                  ADDRESS   PORTS   AGE
ingress-http   nginx.itheima.com,tomcat.itheima.com             80      22s

[root@k8s-m-01 ~]# kubectl get svc -n ingress-nginx
...
post(80:32240)/TCP,443:31335/TCP

# 查看详情
[root@k8s-master01 ~]# kubectl describe ing ingress-http  -n dev
...
Rules:
Host                Path  Backends
----                ----  --------
nginx.itheima.com   / nginx-service:80 (10.244.1.96:80,10.244.1.97:80,10.244.2.112:80)
tomcat.itheima.com  / tomcat-service:8080(10.244.1.94:8080,10.244.1.95:8080,10.244.2.111:8080)
...

# 接下来,在本地电脑上配置host文件,解析上面的两个域名到192.168.109.100(master)上
# 然后,就可以分别访问tomcat.itheima.com:32240  和  nginx.itheima.com:32240 查看效果了
```

## 5.3、Https代理

创建证书

```yaml
# 生成证书
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/C=CN/ST=BJ/L=BJ/O=nginx/CN=itheima.com"

# 创建密钥
kubectl create secret tls tls-secret --key tls.key --cert tls.crt
```

创建ingress-https.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-https
  namespace: dev
spec:
  tls:
    - hosts:
      - nginx.itheima.com
      - tomcat.itheima.com
      secretName: tls-secret # 指定秘钥
  rules:
  - host: nginx.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-service
          servicePort: 80
  - host: tomcat.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: tomcat-service
          servicePort: 8080
```

创建资源

```yaml
# 创建
[root@k8s-master01 ~]# kubectl create -f ingress-https.yaml
ingress.extensions/ingress-https created

# 查看
[root@k8s-master01 ~]# kubectl get ing ingress-https -n dev
NAME            HOSTS                                  ADDRESS         PORTS     AGE
ingress-https   nginx.itheima.com,tomcat.itheima.com   10.104.184.38   80, 443   2m42s

# 查看详情
[root@k8s-master01 ~]# kubectl describe ing ingress-https -n dev
...
TLS:
  tls-secret terminates nginx.itheima.com,tomcat.itheima.com
Rules:
Host              Path Backends
----              ---- --------
nginx.itheima.com  /  nginx-service:80 (10.244.1.97:80,10.244.1.98:80,10.244.2.119:80)
tomcat.itheima.com /  tomcat-service:8080(10.244.1.99:8080,10.244.2.117:8080,10.244.2.120:8080)
...

# 下面可以通过浏览器访问https://nginx.itheima.com:31335 和 https://tomcat.itheima.com:31335来查看了
```

参考资料:

官方学习地址:[k8s官方学习文档](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/)