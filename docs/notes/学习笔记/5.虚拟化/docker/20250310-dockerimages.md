---
title: 2、Docker镜像原理
icon: icon-park:notes
order: 2
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
permalink: /learning-notes/fei6996x/
---
<!-- TOC -->

- [这是文章的标题](#这是文章的标题)
- [你可以自定义封面图片](#你可以自定义封面图片)
- [cover: /assets/images/cover1.jpg](#cover-assetsimagescover1jpg)
- [这是页面的图标](#这是页面的图标)
- [这是侧边栏的顺序](#这是侧边栏的顺序)
- [设置作者](#设置作者)
- [设置写作时间](#设置写作时间)
- [一个页面可以有多个分类](#一个页面可以有多个分类)
- [一个页面可以有多个标签](#一个页面可以有多个标签)
- [此页面会在文章列表置顶](#此页面会在文章列表置顶)
- [此页面会出现在星标文章中](#此页面会出现在星标文章中)
- [你可以自定义页脚](#你可以自定义页脚)
- [你可以自定义版权信息](#你可以自定义版权信息)
- [Docker镜像原理](#docker镜像原理)
    - [镜像原理](#镜像原理)
    - [镜像操作](#镜像操作)
    - [文件系统](#文件系统)
        - [rootfs](#rootfs)
        - [镜像层（Layer）](#镜像层layer)

<!-- /TOC -->

# Docker镜像原理

## 镜像原理

1. 镜像分层

分层： Docker镜像采用分层的方式构建，每一个镜像都由一组镜像组合而成。每一个镜像层都可以被需要的镜像所引用，实现了镜像之间共享镜像层的效果。这样的分层设计在镜像的上传与下载过程当中有效的减少了镜像传输的大小，在传输过程当中本地或注册中心只需要存在一份底层的基础镜像层即可，真正被保存和下载的内容是用户构建的镜像层。而在构建过程中镜像层通常会被缓存以缩短构建过程。

![镜像分层](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20250310094730314.png)

2. 写时复制：底层镜像层在多个容器间共享，每个容器启动时不需要复制一份镜像文件，而是将所有 需要的镜像层以只读的方式挂载到一个挂载点，在只读层上再覆盖一层读写层。在容器运行过程中 产生的新文件将会写入到读写层，被修改过的底层文件会被复制到读写层并且进行修改，而老文件 则被隐藏。

3. 联合挂载：docker采用联合挂载技术，在同一个挂载点同时挂载多个文件系统，从而使得容器的根 目录看上去包含了各个镜像层的所有文件。如下图：

![联合挂载](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20250310094844065.png)

> 这样在容器根目录中，可以看到不同层镜像的文件夹，不同层镜像共同组成了容器的运行时；

**查看镜像详细信息**

```shell
docker inspect crpi-lny1qfykofsn5sx6.cn-shanghai.personal.cr.aliyuncs.com/dw-bugcode/cluster-client-app:v1.0.0

[
    {
        "Id": "sha256:e85e4755c1471d11c0fbb5cd63361cbc63ac67a23ec4f66adcbc132dacd524d3",
        "RepoTags": [
            "crpi-lny1qfykofsn5sx6.cn-shanghai.personal.cr.aliyuncs.com/dw-bugcode/cluster-client-app:v1.0.0",
            "cluster-client-app:v1.0.0"
        ],
        "RepoDigests": [
            "crpi-lny1qfykofsn5sx6.cn-shanghai.personal.cr.aliyuncs.com/dw-bugcode/cluster-client-app@sha256:84c50427d84c7f3e3ac1a5f6f9f819b9a73b9052ca9bd0570507b70600121b8c"
        ],
        "Parent": "",
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2024-11-06T11:03:56.209747903+08:00",
        "Container": "",
        "ContainerConfig": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": null,
            "Cmd": null,
            "Image": "",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "DockerVersion": "",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/openjdk-8/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "JAVA_HOME=/usr/local/openjdk-8",
                "LANG=C.UTF-8",
                "JAVA_VERSION=8u342"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "tail -f"
            ],
            "ArgsEscaped": true,
            "Image": "",
            "Volumes": null,
            "WorkingDir": "/home/bugcode/app/",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "maintainer": "bugcode.online"
            }
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 526049613,
        "VirtualSize": 526049613,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/33cbcf63cd1a14b14fb0926e7808bae8af70d53184f51feefc1a18acd53b639f/diff:/var/lib/docker/overlay2/d1f0717ee217668537f7a76532f22860dd4486e710e95b67afa25f7351ce4581/diff:/var/lib/docker/overlay2/ca98196d933996067cac3ea8223a7a85d7e7659aef05c8a7633ff3344eff1bef/diff:/var/lib/docker/overlay2/3f9a4e45f1b59cebe1cb3bca9431fecac55d69c37dcf31fab570c6e889fdd1e5/diff:/var/lib/docker/overlay2/0690d7947ec8b1367d7bb7ac4c0ace035e9f3d483e93d6491a6f4038f0219d07/diff:/var/lib/docker/overlay2/17b923e35b3a99d2b8cb60c80af2d2b6542b9472f2a5b5943b89db0b9c43c2b8/diff:/var/lib/docker/overlay2/bca24d99823d763bdc5937364dbd55ae9599c95034c7be8c2656b36c4973d0a1/diff",
                "MergedDir": "/var/lib/docker/overlay2/cudi2c2as1vg4pf58crp9o51r/merged",
                "UpperDir": "/var/lib/docker/overlay2/cudi2c2as1vg4pf58crp9o51r/diff",
                "WorkDir": "/var/lib/docker/overlay2/cudi2c2as1vg4pf58crp9o51r/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:9c742cd6c7a5752ee36be8ecb14be45c0885e10e6dd34f26a9ae3eb096c5d492",
                "sha256:03127cdb479b0f1eb8a9b0df8e8d72ead24979728d3c84ff645611b9d8790f94",
                "sha256:293d5db30c9fcf33b65fa033e427fdd118464f9ea0c2a343a478a6e89c29140e",
                "sha256:9b55156abf262eac3e6bd3ae60e7277ab4f9c69543650d7ecefc8c26ee889873",
                "sha256:b626401ef603dd383fc3a43cf474186827db1875591bfc84b178177ca010015b",
                "sha256:53a0b163e9955ffb80569ef37e13fbf5d1074ddd67bc5ad09d7bd874b800396a",
                "sha256:6b5aaff4425423d122ebe4f1514a1994ae60954fc8a2299787df0ddb1a12f6b9",
                "sha256:e5568b42bec844a8ae9cddfde73beccf75a5114ff74ed9872ab9675f7e7afd8b"
            ]
        },
        "Metadata": {
            "LastTagTime": "2024-11-06T11:08:16.592141694+08:00"
        }
    }
]
```

- LowerDir：被引用的镜像层，该层所有内容均为只读

- UpperDir：容器启动之后，创建的读写层
- Merged：容器启动后，会将LowerDir的所有条目的所有文件连同UpperDir的内容，一起挂载到Merged，从而形成一个完成的根目录

**内容寻址**

根据镜像层内容计算校验和，生成一个内容哈希值，并使用该值来充当镜像层ID、索引 镜像层。内容寻址提高了镜像的安全性，在pull、push和load、save操作后检测数据的完整性。另 外基于内容哈希来索引镜像层，对于来自不同构建的镜像层，只要拥有相同的内容哈希值，就能被 不同的镜像所引用。

**Docker镜像概念**

1. registry：注册中心，用来保存docker镜像，其中包括镜像的层次结构和关于镜像的元数据。

2. repository：仓库，即由具有某个功能的Docker镜像的所有迭代版本构成的镜像组。

3. manifest：Docker镜像元数据文件，在pull、push、save和load中作为镜像结构和基础信息的描 述文件。在镜像被pull或者load到Docker宿主机时，manifest被转化为本地的镜像配置文件config 
4.  image：镜像，用来存储一组相关的元数据信息，主要包括镜像的架构（如amd64）、镜像默认配 置信息、构建镜像的容器配置信息、包含所有镜像层的rootfs。

5. layer：镜像层，是docker用来管理镜像的中间概念，镜像是由镜像层组成的，单个镜像层可以被 多个镜像和容器共享。

6. dockerfile：是一个镜像制作过程的定义，文档包含了镜像制作的所有命令和完整操作流程

## 镜像操作

保存镜像到宿主机归档文件

```shell
docker save -o client/client.tar crpi-lny1qfykofsn5sx6.cn-shanghai.personal.cr.aliyuncs.com/dw-bugcode/cluster-client-app:v1.0.0
```

解压归档的镜像文件，可以看到每一层镜像内容：

![镜像内容](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20250310100823975.png)

## 文件系统

容器的文件系统是什么样子的？

因为容器的文件系统是经过 Mount Namespace 的，因此容器的文件系统和宿主机的文件系统时隔离的，也就意味着容器内的进程可以对容器的文件系统进行任何的读写操作从而不影响宿主机的文件系统，

容器是有很多可读层组成，每一层都产生了一个文件目录视图，在镜像的最上层有一个挂载点，所有的镜像层目录统一挂载到最上层镜像的一个挂载点，运行时的容器就产生了一个完整的目录；

### rootfs

Mount Namespace 会修改容器进程对文件系统挂载点的认知，而这个挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统，就是所谓的“**容器镜像**”。它还有一个更为专业的名字，叫作：**rootfs**（根文件系统）。

**rootfs 只是一个操作系统所包含的文件、配置和目录，并不包括操作系统内核**。在 Linux 操作系统中，这两部分是分开存放的，操作系统只有在开机启动时才会加载指定版本的内核镜像。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20250310103147293.png)

所以说，rootfs 只包括了操作系统的“躯壳”，并没有包括操作系统的“灵魂”。**实际上，同一台机器上的所有容器，都共享宿主机操作系统的内核**。

> 这也是容器相比于虚拟机的主要缺陷之一：毕竟后者不仅有模拟出来的硬件机器充当沙盒，而且每个沙盒里还运行着一个完整的 Guest OS 给应用随便折腾。

不过，正是**由于 rootfs 的存在，容器才有了一个被反复宣传至今的重要特性：一致性**。由于 rootfs 里打包的不只是应用，而是**整个操作系统的文件和目录**，也就意味着，应用以及它运行所需要的所有依赖，都被封装在了一起。

### 镜像层（Layer）

Docker 在镜像的设计中，引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs。

**通过引入层（layer）的概念，实现了 rootfs 的复用**。不必每次都重新创建一个 rootfs，而是基于某一层进行修改即可。

Docker 镜像层用到了一种叫做**联合文件系统（Union File System）**的能力。Union File System 也叫 UnionFS，最主要的功能是将多个不同位置的目录联合挂载（union mount）到同一个目录下。

> 例如将目录 A 和目录 B 挂载到目录 C 下面，这样目录 C 下就包含目录 A 和目录 B 的所有文件。
>
> 由于看不到目录 A 和 目标 B 的存在，因此就好像 C 目录就包含这么多文件一样

Docker 镜像分为多个层，然后使用 UFS 将这多个层挂载到一个目录下面，这样这个目录就包含了完整的文件了。

> UnionFS 在不同系统有各自的实现，所以 Docker 的不同发行版使用的也不一样，可以通过 docker info 查看。常见有 aufs（ubuntu 常用）、overlay2（centos 常用）

就像下图这样：union mount 在最上层，提供了统一的视图，用户看起来好像整个系统只有一层一样，实际上下面包含了很多层。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20250310103457970.png)

镜像只包含了静态文件，但是容器会产生实时数据，所以容器的 rootfs 在镜像的基础上增加了**可读写层和 Init 层**。

> 即容器 rootfs 包括：只读层（镜像 rootfs）+ init 层（容器启动时初始化修改的部分数据） + 可读写层（容器中产生的实时数据）。

**只读层（镜像 rootfs）**

它是这个容器的 rootfs 最下面的几层，即**镜像中的所有层的总和**，它们的挂载方式都是只读的（ro+wh，即 readonly+whiteout）

**可读写层（容器中产生的实时数据）**

它是这个容器的 rootfs 最上面的一层，它的挂载方式为：rw，即 read write。在没有写入文件之前，这个目录是空的。

而一旦在容器里做了写操作，你修改产生的内容就会以增量的方式出现在这个层中，删除操作实现比较特殊（类似于标记删除）。

AUFS 的 whiteout 的实现是通过在上层的可写的目录下建立对应的 whiteout 隐藏文件来实现的。

为了实现删除操作，aufs（UnionFS 的一种实现） 会在可读写层创建一个 whiteout 文件，把只读层里的文件“遮挡”起来。

> 比如，你要删除只读层里一个名叫 foo 的文件，那么这个删除操作实际上是在可读写层创建了一个名叫.wh.foo 的文件。这样，当这两个层被联合挂载之后，foo 文件就会被.wh.foo 文件“遮挡”起来，“消失”了。

**init 层（容器启动时初始化修改的部分数据）**

它是一个以“-init”结尾的层，夹在只读层和读写层之间，Init 层是 Docker 项目单独生成的一个内部层，专门用来存放 /etc/hosts、/etc/resolv.conf 等信息。

**为什么需要 init 层？**

比如 hostname 这样的数据，原本是属于镜像层的一部分，要修改的话只能在可读写层进行修改，但是又不想在 docker commit 的时候把这些信息提交上去，所以使用 init 层来保存这些修改。

> 可以理解为提交代码的时候一般也不会把各种配置信息一起提交上去。
>
> docker commit 只会提交 只读层和可读写层。

最后一个问题：**docker 镜像又是怎么实现的**？通过引入 layer 概念进行分层，借助 联合文件系统（Union File System）进行叠加，最终构成了完整的镜像。

> 这里只是镜像的主要内容，具体怎么把这些内容打包成 image 格式就是 OCI 规范了