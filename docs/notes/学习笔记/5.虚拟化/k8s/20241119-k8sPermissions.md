---
title: K8s中权限管理
icon: streamline-freehand-color:notes-book
order: 3
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
permalink: /learning-notes/virtualization/k8s/k8sPermissions/
---
<!-- TOC -->

- [K8s权限管理](#k8s权限管理)
- [k8s授权简介](#k8s授权简介)
  - [RBAC 基础](#rbac-基础)
  - [k8s中的RBAC](#k8s中的rbac)
    - [核心元素](#核心元素)
- [用户分类](#用户分类)
  - [普通用户](#普通用户)
  - [ServiceAccount(服务账户)](#serviceaccount服务账户)
- [K8s角色\&角色绑定(以ServiceAccount展开讲解)](#k8s角色角色绑定以serviceaccount展开讲解)
  - [授权介绍](#授权介绍)
- [案例](#案例)
  - [Role:角色](#role角色)
  - [ClusterRole:集群角色](#clusterrole集群角色)
  - [角色绑定](#角色绑定)
- [案例演示](#案例演示)
  - [User](#user)
  - [Group](#group)
  - [ServiceAccount](#serviceaccount)
  - [五、总结](#五总结)

<!-- /TOC -->

# K8s权限管理

# k8s授权简介

Kubernetes 主要通过 API Server 对外提供服务,Kubernetes 对于访问 API 的用户提供了相应的安全控制:认证和授权。认证解决用户是谁的问题,授权解决用户能做什么的问题。只有通过合理的权限控制,才能够保证整个集群系统的安全可靠。

下图是 API 访问需要经过的三个步骤,它们分别是:认证、授权和准入。

![准入鉴权](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/20241207155718.png)

我们核心关注在鉴权部分,Kubernetes 提供了以下 4 种鉴权模式:

- Node:一种特殊的授权模块,基于 Node 上运行的 Pod 为 Kubelet 授权
- ABAC:基于属性的访问控制
- RBAC:基于角色的访问控制
- Webhook:HTTP 请求回调,通过一个 WEB 应用鉴定是否有权限进行某项操作

kubernetes 集群相关所有的交互都通过apiserver来完成,对于这样集中式管理的系统来说,权限管理尤其重要,**在1.5版的时候引入了RBAC(Role Base Access Control)的权限控制机制**。

API Server目前支持以下几种授权策略:

- **AlwaysDeny**:表示拒绝所有请求,一般用于测试。
- **AlwaysAllow**:允许接收所有请求。
  如果集群不需要授权流程,则可以采用该策略,这也是Kubernetes的默认配置。
- **ABAC**(Attribute-Based Access Control):基于属性的访问控制。
  表示使用用户配置的授权规则对用户请求进行匹配和控制。
- Webhook:通过调用外部REST服务对用户进行授权。
- **RBAC**:Role-Based Access Control,基于角色的访问控制(**本章讲解**)。
- **Node**:是一种专用模式,用于对kubelet发出的请求进行访问控制。

## RBAC 基础

RBAC就是一个权限控制模型,这个模型是经过时间沉淀之后,相当通用、成熟且被大众接受认可的一个模型。我的理解是RBAC和数学公式是一个道理,数学题可以套用数学公式,而权限系统也可以套用RBAC权限模型。

RBAC(Role-Based Access Control)权限模型的概念,即:基于角色的权限控制。通过角色关联用户,角色关联权限的方式间接赋予用户权限。

按照小白(我)的逻辑呢,权限嘛,只要给用户分配权限就好咯,何必多此一举,中间加一个角色,把权限倒个手。

其实之所以在中间加一层角色,是为了增加安全性和效率,而且后续扩展上也会提升不少。

打个比方,比如多个用户拥有相同的权限,在分配的时候就要分别为这几个用户指定相同的权限,修改时也要为这几个用户的权限进行一一修改。有了角色后,只需要为该角色制定好权限后,将相同权限的用户都指定为同一个角色即可,便于权限管理。对于批量的用户权限调整,只需调整用户关联的角色权限,无需对每一个用户都进行权限调整,既大幅提升权限调整的效率,又降低了漏调权限的概率。

## k8s中的RBAC

RBAC 鉴权机制使用 rbac.authorization.k8s.io API 组来驱动鉴权决定, 允许你通过 Kubernetes API 动态配置策略。

要启用 RBAC,在启动 API 服务器时将 --authorization-mode 参数设置为一个逗号分隔的列表并确保其中包含 RBAC。

### 核心元素

掌握K8s 中的RBAC,首先我们要了解会涉及到的基本概念和元素

- Rule:规则,一组属于不同 API Group 的操作集合;
- Role:角色,用于定义一组对 Kubernetes API 对象操作的一组规则,范围限定在 namespace;
- ClusterRole:集群角色,该角色不受 namespace 的限制;
- Subject:对象,也就是规则作用的对象;
- RoleBinding:将角色和对象进行绑定,范围限定在 namespace;
- ClusterRoleBinding:将集群角色和对象进行绑定,不受 namespace 限制;

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/20241207161030.png)

# 用户分类

> K8s的用户分两种,一种是普通用户,一种是ServiceAccount(服务账户)。

## 普通用户

- **普通用户**是假定被外部或独立服务管理的。管理员分配私钥。平时常用的**kubectl命令都是普通用户执行**的。
- 如果是用户需求权限,则将Role与User(或Group)绑定(这需要创建User/Group),是给**用户使用**的。

## ServiceAccount(服务账户)

- **ServiceAccount**(服务帐户)是由Kubernetes API管理的用户。它们绑定到特定的命名空间,并由API服务器自动创建或通过API调用手动创建。服务帐户与存储为Secrets的一组证书相关联,这些凭据被挂载到pod中,以便集群进程与Kubernetes API通信。(登录dashboard时我们使用的就是ServiceAccount)
- 如果是程序需求权限,将Role与ServiceAccount指定(这需要创建ServiceAccount并且在deployment中指定ServiceAccount),是给**程序使用**的。

k8s创建两套独立的账号系统,原因如下:

1. User账号给用户用,Service Account是给Pod里的进程使用的,面向的对象不同 
2. User账号是全局性的,Service Account则属于某个具体的Namespace 
3. User账号是与后端的用户数据库同步的,创建一个新用户通常要走一套复杂的业务流程才能实现,Service Account的创建则需要极轻量级的实现方式,集群管理员可以很容易地为某些特定任务创建一个Service Account 
4. 对于一个复杂的系统来说,多个组件通常拥有各种账号的配置信息,**Service Account是Namespace隔离的**,可以针对组件进行一对一的定义,同时具备很好的“便携性”

> 相当于Role是一个类,用作权限申明,User/Group/ServiceAccount将成为类的实例。


# K8s角色&角色绑定(以ServiceAccount展开讲解)

RBAC有四个资源对象,分别是**Role、ClusterRole、RoleBinding、ClusterRoleBinding**

## 授权介绍

在RABC API中,通过如下的步骤进行授权:

1. **定义角色**:在定义角色时会**指定此角色对于资源的访问控制的规则**。
2. **绑定角色**:**将主体与角色进行绑定,对用户进行访问授权**。

**角色分类**

- Role:授权特定命名空间的访问权限
- ClusterRole:授权所有命名空间的访问权限

**角色绑定**

- RoleBinding:将角色绑定到主体(即subject)
- ClusterRoleBinding:将集群角色绑定到主体

**主体(subject)**

- User:用户
- Group:用户组
- ServiceAccount:服务账号

> 角色(Role和ClusterRole)
>
>Role是权限的定义,在kubernetes中角色分为两种一种是Role针对特定的命名空间,一种是ClusterRole在整个集群范围内都生效。

# 案例

## Role:角色

一组权限的集合,在一个**命名空间**中,可以用其来定义一个角色,只能对命名空间内的资源进行授权。如果是集群级别的资源,则需要使用ClusterRole。例如:定义一个角色用来读取Pod的权限

```yaml
# 给默认命名空间下创建一个角色
cat >Role-001.yaml<<EOF
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-role
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
EOF
kubectl apply -f Role-001.yaml
kubectl get role -n default
# 查看创建的角色拥有的权限
kubectl describe role pod-role -n default

[root@centos1 k8saccountrole]# kubectl describe role pod-role -n default
Name:         pod-role
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"rbac.authorization.k8s.io/v1","kind":"Role","metadata":{"annotations":{},"name":"pod-role","namespace":"default"},"rules":[...
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [get watch list] 拥有的权限
```

rules中的参数说明:

1、apiGroups:支持的API组列表,例如:"apiVersion: batch/v1"等

2、resources:支持的资源对象列表,例如pods、deplayments、jobs等

3、resourceNames: 指定resource的名称

3、verbs:对资源对象的操作方法列表。

## ClusterRole:集群角色

具有和角色一致的命名空间资源的管理能力,还可用于以下特殊元素的授权

1、集群范围的资源,例如Node

2、非资源型的路径,例如:/healthz

3、包含全部命名空间的资源,例如Pods

例如:定义一个集群角色可让用户访问任意secrets

```yaml
cat >ClusterRole-001.yaml<<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-clusterrole
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
EOF
kubectl apply -f ClusterRole-001.yaml
kubectl get clusterrole pod-clusterrole
kubectl describe clusterrole pod-clusterrole
```

**相关参数**

1、Role、ClsuterRole **Verbs**可配置参数

> "get", "list", "watch", "create", "update", "patch", "delete", "exec"

2、Role、ClsuterRole **Resource**可配置参数

> "services", "endpoints", "pods","secrets","configmaps","crontabs","deployments","jobs","nodes","rolebindings","clusterroles","daemonsets","replicasets","statefulsets","horizontalpodautoscalers","replicationcontrollers","cronjobs"

3、Role、ClsuterRole **APIGroup**可配置参数

> "","apps", "autoscaling", "batch"

## 角色绑定

RoleBinding:角色绑定,ClusterRoleBinding:集群角色绑定

定义好了角色也就是一个**权限的集合**,然后创建了一个ServiceAccount也就是一个服务账号,然后将这两个东西绑定起来,就是授权的过程了。

角色绑定和集群角色绑定用于把一个角色绑定在一个目标上,可以是User,Group,Service Account,使用RoleBinding为某个命名空间授权,使用ClusterRoleBinding为集群范围内授权。

**Role角色绑定ServiceAccount**

```yaml
cat >RoleBinding-001.yaml<<EOF
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rb
  namespace: default
subjects:
- kind: ServiceAccount
  name: zhangsan
  namespace: default
roleRef:
  kind: Role
  name: pod-role
  apiGroup: rbac.authorization.k8s.io
EOF
$ kubectl apply -f RoleBinding-001.yaml
$ kubectl get RoleBinding
$ kubectl describe RoleBinding rb
```

RoleBinding也可以引用ClusterRole,对属于同一命名空间内的ClusterRole定义的资源主体进行授权,

```yaml
cat >ClusterRoleBinding-001.yaml<<EOF
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: crb
subjects:
- kind: ServiceAccount
  name: mark
  namespace: default
roleRef:
  kind: ClusterRole
  name: pod-clusterrole
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mark
  namespace: default
EOF
$ kubectl apply -f ClusterRoleBinding-001.yaml
$ kubectl get ClusterRoleBinding crb
$ kubectl describe ClusterRoleBinding crb
```

上面只是说到了绑定ServiceAccount,其实还有两种类型:User,Group,绑定类似,把subjects.kind。

尽管无法通过 API 调用来添加给kubernetes增加普通用户,Kubernetes 仍然认为能够提供由集群的证书 机构签名的合法证书的用户是通过身份认证的用户。基于这样的配置,Kubernetes 使用证书中的 'subject' 的**通用名称(Common Name)字段(例如,"/CN=devuser")来 确定用户名**, Kubernetes使用证书中的 'subject' 的**单位名称 (Organization Name) 字段(例如,"/O=system:masters")来确定用户组**。接下来,基于角色访问控制(RBAC)子系统会确定用户是否有权针对 某资源执行特定的操作。

Kubernetes 有着以下几个内建的用于特殊目的的组:

- system:unauthenticated :未能通过任何一个授权插件检验的账号,即未通过认证测 试的用户所属的组 。
- system :authenticated :认证成功后的用户自动加入的一个组,用于快捷引用所有正常通过认证的用户账号。
- system : serviceaccounts :当前系统上的所有 Service Account 对象。
- system :serviceaccounts :<namespace＞:特定命名空间内所有的 Service Account 对象。

# 案例演示

## User


> 普通用户并不是通过k8s来创建和维护,是通过创建证书和切换上下文环境的方式来创建和切换用户。

**a、创建证书**

```yaml
#　创建私钥
$ openssl genrsa -out devuser.key 2048
生成devuser.key文件
 
#　用此私钥创建一个csr(证书签名请求)文件
$ openssl req -new -key devuser.key -subj "/CN=devuser" -out devuser.csr
生成devuser.csr文件
 
#　拿着私钥和请求文件生成证书
$ openssl x509 -req -in devuser.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out devuser.crt -days 365
Signature ok
subject=/CN=devuser
Getting CA Private Key

生成devuser.crt文件
```

**b、生成账号**

```yaml
kubectl config set-credentials devuser --client-certificate=./devuser.crt --client-key=./devuser.key --embed-certs=true
User "devuser" set.
```

**c、设置上下文参数**

```yaml
# # 设置上下文, 默认会保存在 $HOME/.kube/config
kubectl config set-context devuser@kubernetes --cluster=kubernetes --user=devuser --namespace=dev
Context "devuser@kubernetes" created.
# 查看
$ kubectl config get-contexts
*:表示默认的上下文
```

**d、设置 默认上下文**

```yaml
$ kubectl config use-context devuser@kubernetes
Switched to context "devuser@kubernetes".
# 查看
$ kubectl config get-contexts

CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         devuser@kubernetes            kubernetes   devuser            dev
          kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
[root@centos1 .kube]# kubectl config use-context devuser@kubernetes
Switched to context "devuser@kubernetes".
[root@centos1 .kube]# kubectl get nodes
Error from server (Forbidden): nodes is forbidden: User "devuser" cannot list resource "nodes" in API group "" at the cluster scope

#重新切换会默认的上下文:
kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
#上下文信息在config里面保存,切换上下文只需要指定user即可

#切回管理员上下文
$ kubectl config use-context kubernetes-admin@kubernetes
$ kubectl get nodes
```

发现使用我们创建的用户查询是失败的,是因为账号还没授权,接下来就是对账号进行授权。这里需要先把用切回来,要不然就无法进行下一步授权了。

**2、对用户授权**

```yaml
cat >devuser-role-bind<<EOF
kind: Role  # 角色
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: dev
  name: devuser-role
rules:
- apiGroups: [""] # ""代表核心api组
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
kind: RoleBinding # 角色绑定
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: devuser-rolebinding
  namespace: dev
subjects:
- kind: User
  name: devuser   # 目标用户
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: devuser-role  # 角色信息
  apiGroup: rbac.authorization.k8s.io
EOF
```

创建角色以及绑定角色

```yaml
//创建角色以及绑定
[root@centos1 k8saccountrole]# kubectl apply -f devuser-role-bind
role.rbac.authorization.k8s.io/devuser-role created
rolebinding.rbac.authorization.k8s.io/devuser-rolebinding created
//切换上下文
[root@centos1 k8saccountrole]# kubectl config use-context devuser@kubernetes
Switched to context "devuser@kubernetes".

# 不带命名空间,这里默认dev,也只能查看dev上面限制的命名空间的pods资源,
# 从而也验证了role是针对命名空间的权限限制
[root@centos1 k8saccountrole]# kubectl get pods 
#查看其它命名空间的资源 devuser用户无法查看其他命名空间的pods
//可以查看dev命名空间下的pods
[root@centos1 k8saccountrole]# kubectl get pods -n dev
No resources found in dev namespace.

[root@centos1 k8saccountrole]# kubectl get pods -n flink
Error from server (Forbidden): pods is forbidden: User "devuser" cannot list resource "pods" in API group "" in the namespace "flink"
[root@centos1 k8saccountrole]# kubectl get pods -n default
Error from server (Forbidden): pods is forbidden: User "devuser" cannot list resource "pods" in API group "" in the namespace "default"
[root@centos1 k8saccountrole]#
[root@centos1 k8saccountrole]#
[root@centos1 k8saccountrole]#
[root@centos1 k8saccountrole]# kubectl get pods -n kube-system
Error from server (Forbidden): pods is forbidden: User "devuser" cannot list resource "pods" in API group "" in the namespace "kube-system"
[root@centos1 k8saccountrole]# kubectl get nodes
Error from server (Forbidden): nodes is forbidden: User "devuser" cannot list resource "nodes" in API group "" at the cluster scope
[root@centos1 k8saccountrole]#
```

可以看到,用devuser,已经可以管理dev命名空间下的pod资源了,也只能管理dev命名空间下的pod资源,无法管理dev以外的资源类型,验证ok。ClusterRoleBinding绑定类似.

## Group

> 因为跟user类型,这里就不过多文字介绍,直接上命令和配置

1、创建K8S 用户和用户组

```yaml
#　创建私钥$ openssl genrsa -out devgroupuser.key 2048

#　用此私钥创建一个csr(证书签名请求)文件$ openssl req -new -key devgroupuser.key -subj "/O=devgroup/CN=devgroupuser" -out devgroupuser.csr

#　拿着私钥和请求文件生成证书$ openssl x509 -req -in devgroupuser.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out devgroupuser.crt -days 365

# 生成账号$ kubectl config set-credentials devgroupuser --client-certificate=./devgroupuser.crt --client-key=./devgroupuser.key --embed-certs=true# 设置上下文参数$ kubectl config set-context devgroupuser@kubernetes --cluster=kubernetes --user=devgroupuser --namespace=dev

# 查看$ kubectl config get-contexts
```


2、对组授权

```yaml
$ cat >devgroup-role-bind.yaml<<EOFkind: Role  # 角色
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: dev
  name: devgroup-role
rules:
- apiGroups: [""] # ""代表核心api组
  resources: ["services","pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
kind: RoleBinding # 角色绑定
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: devgroup-rolebinding
  namespace: dev
subjects:
- kind: Group
  name: devgroup   # 目标用户组
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: devgroup-role  # 角色信息
  apiGroup: rbac.authorization.k8s.io
EOF

```

执行并验证(命名空间默认dev,而不是系统的default)

```yaml
$ kubectl apply -f devgroup-role-bind.yaml
#切用户$ kubectl config use-context devgroupuser@kubernetes
# 查看$ kubectl config get-contexts
$ kubectl get pods
$ kubectl get svc
$ kubectl get nodes
$ kubectl get jobs
```


从上图实验看,只能管理dev命名空间下的pods和services了,验证ok。

切回管理员用户

```yaml
$ kubectl config use-context kubernetes-admin@kubernetes
```

## ServiceAccount

已经很详细的介绍了ServiceAccount,这里也是直接上配置和操作命令。

```yaml
#创建角色和绑定角色
cat >RoleBinding-ServiceAccount-001.yaml<<EOF
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: role001
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: rb001
  namespace: default
subjects:
- kind: ServiceAccount
  name: lisi
  namespace: default
roleRef:
  kind: Role
  name: role001
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: lisi
  namespace: default
EOF
```

执行:

```yaml
$ kubectl delete -f RoleBinding-ServiceAccount-001.yaml
```

**为ServiceAccount生成Token**

```yaml
cat >ClusterRoleBinding-ServiceAccount-token-001.yaml<<EOF
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: sa001
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa001
  namespace: kube-system
EOF
```

获取tocken:

```yaml
$ kubectl -n kube-system get secret|grep sa001
$ kubectl -n kube-system describe secret sa001-token-c2klg
# 也可以使用 jsonpath 的方式直接获取 token 的值,如:
$ kubectl -n kube-system get secret sa001-token-c2klg -o jsonpath={.data.token}|base64 -d
```

> 【注意】yaml 输出里的那个 token 值是进行 base64 编码后的结果

默认的Token

> 每次创建了新的namespace下都会生成一个默认的token,名为default-token-xxxx。default就相当于该namespace下的一个用户。

```yaml
$ kubectl -n dev get secret
$ kubectl -n dev describe secret default-token-67wgz
# 也可以使用 jsonpath 的方式直接获取 token 的值,如:
$ kubectl -n dev get secret default-token-67wgz -o jsonpath={.data.token}|base64 -d
```

可以使用下面的命令给该用户分配该namespace的管理员权限

> kubectl create rolebinding $ROLEBINDING_NAME --clusterrole=admin --serviceaccount=$NAMESPACE:default --namespace=$NAMESPACE

- $ROLEBINDING_NAME必须是该namespace下的唯一的
- admin表示用户该namespace的管理员权限,关于使用clusterrole进行更细粒度的权限控制请参考RBAC——基于角色的访问控制。
- 我们给默认的serviceaccount default分配admin权限,这样就不要再创建新的serviceaccount,当然你也可以自己创建新的serviceaccount,然后给它admin权限


## 五、总结

> RoleBinding 和 ClusterRoleBinding:角色绑定和集群角色绑定,简单来说就是把声明的 Subject 和我们的 Role 进行绑定的过程(给某个用户绑定上操作的权限),二者的区别也是作用范围的区别:RoleBinding 只会影响到当前namespace 下面的资源操作权限,而 ClusterRoleBinding 会影响到所有的namespace。

- **Rule**:规则,规则是一组属于不同 API Group 资源上的一组操作的集合
- **Role 和 ClusterRole**:角色和集群角色,这两个对象都包含上面的 Rules 元素,二者的区别在于,在 Role 中,定义的规则只适用于单个命名空间,也就是和namespace 关联的,而 ClusterRole 是集群范围内的,因此定义的规则不受命名空间的约束。另外 Role 和 ClusterRole 在Kubernetes中都被定义为集群内部的 API 资源,和我们前面学习过的 Pod、ConfigMap 这些类似,都是我们集群的资源对象,所以同样的可以使用我们前面的kubectl相关的命令来进行操作
- **Subject**:主题,对应在集群中尝试操作的对象,集群中定义了3种类型的主题资源:
    1. **User**:用户,这是有外部独立服务进行管理的,管理员进行私钥的分配,用户可以使用 KeyStone或者 Goolge 帐号,甚至一个用户名和密码的文件列表也可以。对于用户的管理集群内部没有一个关联的资源对象,所以用户不能通过集群内部的 API 来进行管理
    2. **Group**:组,这是用来关联多个账户的,集群中有一些默认创建的组,比如cluster-admin
    3. **ServiceAccount**:服务帐号,通过Kubernetes API 来管理的一些用户帐号,和namespace 进行关联的,适用于集群内部运行的应用程序,需要通过 API 来完成权限认证,所以在集群内部进行权限操作,我们都需要使用到 ServiceAccount,这也是我们这节课的重点