---
title: 1、Docker核心原理
icon: icon-park:notes
order: 1
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - DOCKER
tag:
  - docker
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 14:30:11
permalink: /learning-notes/blvv1jq1/
---

<!-- TOC -->

- [Docker](#docker)
  - [Docker架构](#docker架构)
  - [Docker组成](#docker组成)
    - [Docker守护进程](#docker守护进程)
    - [Docker客户端](#docker客户端)
    - [Docker内部](#docker内部)
  - [Docker底层技术](#docker底层技术)
    - [**命名空间「Namespaces」**](#命名空间namespaces)
    - [**mnt namespace**](#mnt-namespace)
    - [**net namespace**](#net-namespace)
    - [**uts namespace**](#uts-namespace)
    - [**ipc namespace**](#ipc-namespace)
    - [**user namespace**](#user-namespace)
    - [**资源配额「cgroups」**](#资源配额cgroups)
  - [**Docker 四种网络模式**](#docker-四种网络模式)
    - [桥接模式（Bridge Network）](#桥接模式bridge-network)
    - [Host 网络](#host-网络)
    - [None 网络](#none-网络)
    - [container 模式](#container-模式)
    - [Docker网络驱动程序](#docker网络驱动程序)
  - [常用Docker命令](#常用docker命令)
    - [启动建立容器](#启动建立容器)
    - [操作容器](#操作容器)
      - [**进入容器操作**：docker exec](#进入容器操作docker-exec)
      - [**暂停容器**：docker pause/unpause](#暂停容器docker-pauseunpause)
      - [**重启正在运行的容器**：Docker restart](#重启正在运行的容器docker-restart)
      - [**查看日志输出：Docker logs**](#查看日志输出docker-logs)
      - [**获取容器或者镜像详细信息：Docker inspect**](#获取容器或者镜像详细信息docker-inspect)
      - [**查看镜像构建过程：docker history**](#查看镜像构建过程docker-history)
      - [**终止正在运行的容器：Docker kill**](#终止正在运行的容器docker-kill)
      - [**删除镜像：Docker rm/docker rmi**](#删除镜像docker-rmdocker-rmi)

<!-- /TOC -->


# Docker

## Docker架构

Docker 使用客户端-服务器 (C/S) 架构模式。Docker 客户端会与 Docker 守护进程进行通信。Docker  守护进程会处理复杂繁重的任务，例如建立、运行、发布你的 Docker 容器。Docker  客户端和守护进程可以运行在同一个系统上，当然你也可以使用 Docker 客户端去连接一个远程的 Docker 守护进程。Docker  客户端和守护进程之间通过 socket 或者 RESTful API 进行通信。

## Docker组成

Docker 有两个主要的部件：

- Docker: 开源的容器虚拟化平台。
- Docker Hub: 用于分享、管理 Docker 容器的 Docker SaaS 平台。

### Docker守护进程

Docker 守护进程运行在一台主机上。用户并不直接和守护进程进行交互，而是通过 Docker 客户端间接和其通信。

### Docker客户端

Docker 客户端，实际上是 docker 的二进制程序，是主要的用户与 Docker 交互方式。它接收用户指令并且与背后的 Docker 守护进程通信，如此来回往复。

### Docker内部

要理解 Docker 内部构建，需要理解以下三种部件：

- Docker 镜像：Docker images，静态的。
- Docker 仓库 ：Docker registeries
- Docker 容器 ：Docker containers，运行起来的镜像。

**Docker镜像**

Docker 镜像是 Docker 容器运行时的只读模板，每一个镜像由一系列的层 (layers) 组成。Docker 使用 UnionFS  来将这些层联合到单独的镜像中。UnionFS  允许独立文件系统中的文件和文件夹(称之为分支)被透明覆盖，形成一个单独连贯的文件系统。正因为有了这些层的存在，Docker  是如此的轻量。当你改变了一个 Docker  镜像，比如升级到某个程序到新的版本，一个新的层会被创建。因此，不用替换整个原先的镜像或者重新建立(在使用虚拟机的时候你可能会这么做)，只是一个新的层被添加或升级了。现在你不用重新发布整个镜像，只需要升级，层使得分发 Docker 镜像变得简单和快速。

**Docker仓库**

Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。同样的，Docker 仓库也有公有和私有的概念。公有的 Docker 仓库名字是 Docker Hub。Docker Hub 提供了庞大的镜像集合供使用。这些镜像可以是自己创建，或者在别人的镜像基础上创建。Docker  仓库是 Docker 的分发部分。

**Docker容器**

Docker 容器和文件夹很类似，一个Docker容器包含了所有的某个应用运行所需要的环境。每一个 Docker 容器都是从 Docker  镜像创建的。Docker 容器可以运行、开始、停止、移动和删除。每一个 Docker 容器都是独立和安全的应用平台，Docker 容器是  Docker 的运行部分。

## Docker底层技术

### **命名空间「Namespaces」**

Docker 充分利用了一项称为`namespaces`的技术来提供隔离的工作空间，我们称之为 *container(容器)*。当你运行一个容器的时候，Docker 为该容器创建了一个命名空间集合。

这样提供了一个隔离层，每一个应用在它们自己的命名空间中运行而且不会访问到命名空间之外。

一些 Docker 使用到的命名空间有：

- **`pid`命名空间：** 使用在进程隔离 (PID: Process ID)。
- **`net`命名空间：** 使用在管理网络接口 (NET: Networking)。
- **`ipc`命名空间：** 使用在管理进程间通信资源 (IPC: InterProcess Communication)。
- **`mnt`命名空间：** 使用在管理挂载点 (MNT: Mount)。
- **`uts`命名空间：** 使用在隔离内核和版本标识  (UTS: Unix Timesharing System)。

不同用户的进程就是通过 pid namespace 隔离开的，且不同 namespace 中可以有相同 PID。具有以下特征:

- 每个 namespace 中的 pid 是有自己的 pid=1 的进程(类似 /sbin/init 进程)
- 每个 namespace 中的进程只能影响自己的同一个 namespace 或子 namespace 中的进程
- 因为 /proc 包含正在运行的进程，因此在 container 中的 pseudo-filesystem 的 /proc 目录只能看到自己 namespace 中的进程
- 因为 namespace 允许嵌套，父 namespace 可以影响子 namespace 的进程，所以子 namespace 的进程可以在父 namespace 中看到，但是具有不同的 pid

### **mnt namespace**

类似 chroot，将一个进程放到一个特定的目录执行。mnt namespace 允许不同 namespace 的进程看到的文件结构不同，这样每个  namespace 中的进程所看到的文件目录就被隔离开了。同 chroot 不同，每个 namespace 中的 container 在  /proc/mounts 的信息只包含所在 namespace 的 mount point。

> `chroot`（change root）命令在Linux/[Unix系统](https://so.csdn.net/so/search?q=Unix系统&spm=1001.2101.3001.7020)中用于更改当前运行进程及其子进程的根目录。这个命令可以创建一个封闭的文件系统环境，该环境中的进程无法访问到此环境之外的文件。这种被封闭的环境被称为“chroot监狱”。

### **net namespace**

网络隔离是通过 net namespace 实现的， 每个 net namespace 有独立的 network devices, IP addresses, IP routing tables, /proc/net 目录。这样每个 container 的网络就能隔离开来。 docker 默认采用  veth 的方式将 container 中的虚拟网卡同 host 上的一个 docker bridge 连接在一起。

### **uts namespace**

UTS ("UNIX Time-sharing System") namespace 允许每个 container 拥有独立的 hostname 和 domain name, 使其在网络上可以被视作一个独立的节点而非 Host 上的一个进程。

### **ipc namespace**

container 中进程交互还是采用 Linux 常见的进程间交互方法 (interprocess communication - IPC), 包括常见的信号量、[消息队列](https://zhida.zhihu.com/search?content_id=190498388&content_type=Article&match_order=1&q=消息队列&zhida_source=entity)

和共享内存。然而同 VM 不同，container 的进程间交互实际上还是 host 上具有相同 pid namespace 中的进程间交互，因此需要在 IPC  资源申请时加入 namespace 信息 - 每个 IPC 资源有一个唯一的 32bit ID。

### **user namespace**

每个 container 可以有不同的 user 和 group id, 也就是说可以以 container 内部的用户在 container 内部执行程序而非 Host 上的用户。

有了以上 6 种 namespace 从进程、网络、IPC、文件系统、UTS 和用户角度的隔离，一个 container  就可以对外展现出一个独立计算机的能力，并且不同 container 从 OS 层面实现了隔离。 然而不同 namespace  之间资源还是相互竞争的，仍然需要类似 ulimit 来管理每个 container 所能使用的资源 - cgroup。

### **资源配额「cgroups」**

cgroups 实现了对资源的配额和度量。 cgroups 的使用非常简单，提供类似文件的接口，在 /cgroup 目录下新建一个文件夹即可新建一个  group，在此文件夹中新建 task 文件，并将 pid 写入该文件，即可实现对该进程的资源控制。具体的资源配置选项可以在该文件夹中新建子  subsystem ，{子系统前缀}.{资源项} 是典型的配置方法， 如 memory.usageinbytes 就定义了该 group 在  subsystem memory 中的一个内存限制选项。 另外，cgroups 中的 subsystem 可以随意组合，一个 subsystem 可以在不同的 group 中，也可以一个 group 包含多个 subsystem - 也就是说一个 subsystem。

- memory 

- - 内存	相关的限制

- cpu 

- - 在 cgroup 中，并不能像硬件虚拟化方案一样能够定义 CPU 能力，但是能够定义 CPU 轮转的优先级，因此具有较高 CPU  优先级的进程会更可能得到 CPU 运算。 通过将参数写入 cpu.shares，即可定义改 cgroup 的 CPU 优先级 -  这里是一个相对权重，而非绝对值

- blkio 

- - block IO 相关的统计和限制，byte/operation 统计和限制 (IOPS 等)，读写速度限制等，但是这里主要统计的都是同步 IO

- devices 

- - 设备权限限制

## **Docker 四种网络模式**

| 模式名称  | 简介                                                         | 备注                                                         |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| bridge    | 容器拥有独属于自己的虚拟网卡和和虚拟IP等网络资源，它们分别通过docker0虚拟网卡与宿主机的eth0网卡交互，进而和外界网络交互 | **bridge模式：--net=bridge 桥接模式（默认设置，自己创建也使用bridge 模式）** |
| host      | 容器没有自己的任何独立的网络资源(比如：容器的IP、网卡和端口)，完全和宿主机共享网络空间 | 弊端：同一个端口只能同时被一个容器服务绑定,**host模式：--net=host 和宿主即共享网络** |
| none      | 该模式关闭了容器的网络功能，仅有独自的网络空间（一个空架子），并且该模式不会给容器分配任何网络资源，包括虚拟网卡、路由、防火墙、IP、网关、端口等 | 只有一个容器和独立网络空间没有任何网络资源,**none模式：--net=none 不配置网络** |
| container | 它是bridge和host模式的合体，优先以bridge方式启动启动第一个容器，后面的所有容器启动时，均指定网络模式为container，它们均共享第一个容器的网络资源，除了网络资源，其他资源，容器彼此之间依然是相互隔离的 | 第一个以bridge方式启动的容器服务挂掉，后面依赖它的容器，都暂停服务,**container模式：--net=container:NAME_or_ID 容器网络连通!(很少用，局限性很大！)** |
| 自定义    | 该模式也更为灵活，可以通过-d 指定自定义的网络模式的类型，可以是bridge或者overlay，其中overlay功能更为强大，可以指定多个subnet子网网段。 | 该模式，在容器之间可以使用别名相互通信，这一点很nice（重要） |

查看docker网络模式：

```shell
docker network ls

NETWORK ID     NAME      DRIVER    SCOPE
8ddb7e9846c6   bridge    bridge    local
48e785b7efb3   host      host      local
7e07c5b5ae34   none      null      local
```

查看所有网桥：

```shell
1. `brctl show`：此命令用于显示所有网桥的信息，包括网桥名称、物理接口、虚拟接口等。

2. `ip link show`：此命令显示系统中的网络接口信息。可以通过查找带有”br”前缀的接口来定位网桥。例如，eth0是物理接口，br0是通过brctl命令创建的网桥接口。

3. `/sys/class/net`目录：此目录包含了系统中所有的网络接口，也包括网桥接口。可以使用`ls /sys/class/net`命令列出所有的网络接口，再进入相应的接口目录查看相关的信息。

4. `ifconfig`：此命令显示和配置网络接口的信息。可以使用`ifconfig -a`命令显示所有的网络接口，包括网桥接口。
```



### 桥接模式（Bridge Network）

Docker的bridge网络模式是Docker的默认网络模式。当Docker进程启动时，它会在主机上创建一个名为docker0的虚拟网桥,处于七层网络模型的数据链路层，后续每当我们创建一个新的docker容器，在不指定容器网络模式的情况下，docker会通过 docker0 与主机的网络连接，docker0 相当于网桥。这个虚拟网桥的工作方式类似于物理交换机，使得主机上的所有容器都通过交换机连接在一个二层网络中。
![桥接模式](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20250307142848200.png)

> 使用 bridge 模式新创建的容器，容器内部都会有一个虚拟网卡，名为 eth0，容器之间可以通过容器内部的IP相互通信。
>
> docker run -d -name tomcat01 --net=bridge -p 8085:80 tomcat:latest
>
> - -net=bridge 可省略 ，-p 指定端口映射
> - 网桥默认 IP 范围是一般都是 172.17.x.x

在这种模式下，Docker会为每个新创建的容器分配独立的Network Namespace和IP段等，同时文件系统、进程等也是隔离的。容器内部会有一个虚拟网卡，名为eth0，容器之间可以通过这个虚拟网卡和内部的IP地址进行通信。另外，从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。

然而，处于桥接模式的容器和宿主机网络不在同一个网段，容器一般使用172.16.0.xx/24这种网段。因此，容器不能直接和宿主机以外的网络进行通信，而必须要经过NAT转换。同时，容器需要在宿主机上竞争端口，完成端口映射的配置后，从外部到容器内的网络访问tcp流量将会通过DNAT从宿主机端口转发到容器内对应的端口上。此外，容器对于宿主机以外是不可见的，从容器发出的网络请求会通过SNAT从已对接的虚拟网桥（如宿主机的docker0）上统一发出。

### Host 网络 

Docker的host网络模式是另一种网络模式，与bridge模式不同，它将容器直接融入到主机的网络栈中，使得容器直接使用主机的网络接口和IP地址。在这种模式下，容器不会获得一个独立的Network Namespace，而是和宿主机共用一个Network Namespace。因此，容器内部的服务可以使用宿主机的网络地址和端口，无需进行NAT转换，网络性能较好。

使用host网络模式的一个典型场景是需要容器与宿主机共享网络资源或者容器需要快速访问宿主机网络服务的场景。例如，如果需要在容器内部运行一些网络相关的应用，如网络监控、日志收集等，这些应用需要直接访问宿主机的网络接口和IP地址，此时就可以使用host网络模式。
![host模式](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20250307143100325.png)

需要注意的是，由于容器与宿主机共用一个网络栈，因此容器的网络隔离性较差，可能存在安全隐患。如果需要在不同主机上运行容器并实现跨主机通信，则不能使用host网络模式。

总的来说，Docker的host网络模式提供了一种将容器与宿主机网络栈直接融合的方式，使得容器可以直接使用宿主机的网络接口和IP地址，适用于一些需要快速访问宿主机网络服务的场景。但是需要注意的是，该模式下容器的网络隔离性较差，需要谨慎使用

### None 网络

Docker的none网络模式是Docker提供的一种特殊网络模式，它将容器与宿主机隔离开来，不提供任何网络能力。在这种模式下，容器内部没有网卡、IP地址、路由等信息，只有一个回环网络（loopback）接口。这意味着容器不能访问外部网络，也不能被外部网络访问。
none网络模式通常用于一些特殊场景，比如需要在容器内部运行一些独立的、与网络无关的应用程序，或者需要进行一些网络调试。由于容器与外部网络完全隔离，这种模式可以增加容器的安全性。

### container 模式

Docker的container网络模式是指新创建的容器和已经存在的一个容器共享一个Network Namespace，而不是和宿主机共享。这意味着新创建的容器不会创建自己的网卡、配置自己的IP地址，而是和一个已存在的容器共享IP地址、端口范围等网络资源。同时，这两个容器的进程可以通过lo网卡设备通信。然而，这两个容器在其他方面，如文件系统、进程列表等，仍然是隔离的。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20250307143340077.png)

使用container网络模式的一个典型场景是，当需要多个容器之间共享网络配置时，可以使用该模式。例如，可以使用该模式创建一个nginx容器，并指定其网络模型为container模式，和另一个已经存在的容器共享网络命名空间。这样，nginx容器就可以直接使用另一个容器的IP地址和端口，无需进行额外的网络配置。

### Docker网络驱动程序

Docker使用Linux内核的一些特性来实现其网络功能，而这些功能是通过不同的网络驱动程序来实现的。Docker支持多种网络驱动程序，每种驱动程序都有其特定的用途和场景。

- bridge（桥接）：这是Docker的默认网络驱动程序。它会在宿主机上创建一个虚拟网桥（通常是docker0），并将容器连接到这个网桥上。容器之间以及容器与宿主机之间可以通过这个网桥进行通信。bridge模式适用于单个宿主机上的容器互联场景。
- host：host网络驱动程序将容器直接融入主机的网络栈中，容器将共享主机的网络接口和IP地址。这意味着容器内部的服务可以直接使用主机的网络地址和端口，无需进行NAT转换。host模式适用于需要容器与宿主机共享网络资源的场景，但需要注意安全性和隔离性问题。
- overlay：overlay网络驱动程序用于创建跨多个Docker守护进程的分布式网络。它通过内置的DNS服务实现容器之间的跨主机通信。overlay模式适用于需要构建分布式应用程序的场景，可以让容器在不同宿主机之间进行通信。
- macvlan：macvlan网络驱动程序允许容器使用宿主机的物理网络接口，并为其分配一个MAC地址。这样，容器可以像虚拟机一样直接连接到物理网络上，并与其他设备通信。macvlan模式适用于需要容器直接访问物理网络的场景。
- ipvlan：ipvlan是另一种类似于macvlan的网络驱动程序，但它基于IP地址而不是MAC地址来分配网络。ipvlan模式提供了更好的扩展性和灵活性，适用于不同的网络场景。
- none：none网络驱动程序不提供任何网络功能，容器将处于完全隔离的状态。它通常用于一些特殊场景，如运行与网络无关的应用程序或进行网络调试。

## 常用Docker命令

### 启动建立容器

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
  docker run：运行容器的命令。
  [OPTIONS]：可选参数，用于配置容器的各种选项，如端口映射、容器名称等。
  IMAGE：要运行的镜像名称或ID。
  [COMMAND] [ARG...]：可选的命令和参数，用于在容器内执行特定的命令。

具体应用：
docker run [可选参数] image(镜像名):版本号（默认是最新版本，想要指定版本需要加上版本号！）
 
可选参数：
--name="name" 容器名字，通过一个镜像可以创建多个容器实例，命名可以区分不同的容器
-d  以后台的方式运行 （使用-d命令常见的坑：比如我们想启动以下Nginx服务，只让他在后台运行，但是没有前台的服务可以交互，那么系统就会默认把这个服务关掉！不提供服务！）
-it 使用交互方式运行，进入容器查看内容
-p  指定容器的端口
    有四种方式
        -p ip:主机端口:容器端口
        -p 主机端口:容器内部端口(常用，主机端口映射某个容器内部的端口号，访问主机的端口，即可访问容器)
        -p 容器端口
         容器端口
-P(大写)  随机指定端口
--rm  容器关闭后就被删除掉，一般用于测试！

如果是已经在运行的容器，可以用下面的命令进入：

docker exec   #进入容器后开启一个新的终端，可以在里面做一些操作（常用！）如果使用exit退出，容器也不会停止。
docker attach #进入容器正在执行的终端，不会开启新的进程！

以前台方式启动容器，启动后控制台会卡住
sudo docker run --name="client" dw2020/cluster-client-app:1.2.0.0 

启动并且进入容器终端：
sudo docker exec -it dw2020/cluster-client-app:1.2.0.0 -- /bin/bash
```

### 操作容器

![操作容器](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20250307151032553.png)

#### **进入容器操作**：docker exec

```shell
docker exec [OPTIONS] CONTAINER_ID COMMAND [ARG...]
• docker exec：在容器内执行命令的命令。
• [OPTIONS]：可选参数，用于配置执行过程，如进入交互模式等。
• CONTAINER：要执行命令的容器名称或ID。
• COMMAND [ARG...]：要在容器内执行的命令及其参数。

sudo docker exec -it  container_id /bin/bash

进入容器并且新开一个终端，在容器内部执行命令，使用exec命令退出并不会杀掉容器，只是退出终端；

docker attach 进入容器正在执行的终端，不会启动新的进程。如果使用exit退出，容器会停止运行！
```

#### **暂停容器**：docker pause/unpause

```shell
docker pause CONTAINER [CONTAINER...]

docker pause 命令允许用户暂停一个或多个容器中的所有进程。这个命令非常有用，特别是当您需要临时暂停容器活动进行系统维护或资源管理时。暂停的容器不会被终止，但其进程将被挂起，直到容器被恢复。

docker unpause CONTAINER [CONTAINER...] 
- 恢复被暂停的容器

sudo docker pause/unpause container_id
```

**停止容器：Docker start**

```shell
`docker start [OPTIONS] CONTAINER [CONTAINER...]`
• docker start：启动容器的命令。
• [OPTIONS]：可选参数，用于配置启动过程，如守护模式等。
• CONTAINER [CONTAINER...]：要启动的容器名称或ID。

docker start 和docker stop配合使用使用docker stop停止容器，这样会导致容器退出，然后docker start会重新启动这个容器

sudo docker start/stop container_id
```

需要和docker pause区分的是：

- docker pause是暂停容器，容器运行状态保存
- docker stop是暂停退出容器，重新启动后是一个新建的容器

#### **重启正在运行的容器**：Docker restart

```shell
`docker restart [OPTIONS] CONTAINER [CONTAINER...]`
• docker restart：重启容器的命令。
• [OPTIONS]：可选参数，用于配置重启过程，如超时时间等。
• CONTAINER [CONTAINER...]：要重启的容器名称或ID。

sudo docker restart container_id
```

#### **查看日志输出：Docker logs**

```shell
`docker logs [OPTIONS] CONTAINER`
• docker logs：查看容器日志的命令。
• [OPTIONS]：可选参数，用于配置输出结果，如时间戳等。
• CONTAINER：要查看日志的容器名称或ID。

sudo docker logs container_id
```

#### **获取容器或者镜像详细信息：Docker inspect**

```shell
`docker inspect [OPTIONS] NAME|ID [NAME|ID...]`
• docker inspect：获取详细信息的命令。
• [OPTIONS]：可选参数，用于配置输出结果的格式等。
• NAME|ID [NAME|ID...]：要获取信息的容器或镜像的名称或ID。

sudo docker inspect container_id
```

#### **查看镜像构建过程：docker history**

```shell
docker history 镜像ID
```

#### **终止正在运行的容器：Docker kill**

```shell
`docker kill [OPTIONS] CONTAINER [CONTAINER...]`
• docker kill：终止容器的命令。
• [OPTIONS]：可选参数，用于配置终止过程，如信号等。
• CONTAINER [CONTAINER...]：要终止的容器名称或ID。
```

#### **删除镜像：Docker rm/docker rmi**

```shell
`docker rm [OPTIONS] CONTAINER [CONTAINER...]   docker rmi [OPTIONS] IMAGE [IMAGE...]`
• docker rm：删除容器的命令。
• docker rmi：删除镜像的命令。
• [OPTIONS]：可选参数，用于配置删除过程，如强制删除等。
• CONTAINER [CONTAINER...]：要删除的容器名称或ID。
• IMAGE [IMAGE...]：要删除的镜像名称或ID。
```

