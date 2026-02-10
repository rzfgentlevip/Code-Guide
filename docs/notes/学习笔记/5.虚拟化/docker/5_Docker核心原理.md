---
title: 5、Docker核心原理
icon: vscode-icons:folder-type-docker-opened
order: 5
author: bugcode
date: 2024-01-01T00:00:00.000Z
footer: 云原生
copyright: bugcode
createTime: 2025/09/04 14:29:58
permalink: /learning-notes/virtualization/docker/Docker核心原理/
---


## 一、**Docker 整体架构概述**

### **现代 Docker 架构图**

```shell
┌─────────────────────────────────────────────────────┐
│                    Docker Client                     │
│             (docker CLI / Docker Desktop)            │
└──────────────────────┬───────────────────────────────┘
                       │ REST API over UNIX Socket
┌──────────────────────▼───────────────────────────────┐
│                 Docker Daemon (dockerd)              │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐       │
│  │   Images   │ │  Build     │ │ Networking │       │
│  │  Registry  │ │  Engine    │ │ & Storage  │       │
│  │  Manager   │ │            │ │  Manager   │       │
│  └────────────┘ └────────────┘ └────────────┘       │
└──────────────────────┬───────────────────────────────┘
                       │ gRPC over UNIX Socket
┌──────────────────────▼───────────────────────────────┐
│                    containerd                        │
│  ┌─────────────────────────────────────────────────┐│
│  │   Container Supervision  │  Image Distribution  ││
│  │   Snapshot Management   │  Network Namespaces   ││
│  └─────────────────────────────────────────────────┘│
└──────────────────────┬───────────────────────────────┘
                       │ OCI Runtime Specification
┌──────────────────────▼───────────────────────────────┐
│                      runc                            │
│           (OCI-compliant runtime)                    │
└──────────────────────┬───────────────────────────────┘
                       │ System Calls
┌──────────────────────▼───────────────────────────────┐
│                 Linux Kernel                          │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐       │
│  │ Namespaces │ │  Cgroups   │ │   UnionFS   │       │
│  │   (隔离)    │ │  (资源控制) │ │  (分层存储)  │       │
│  └────────────┘ └────────────┘ └────────────┘       │
└─────────────────────────────────────────────────────┘
```

## 二、**核心技术原理**

### **1. Linux 命名空间 (Namespaces) - 隔离**

**Docker 使用 6 种命名空间实现隔离：**

```shell
// 内核中的命名空间实现
// 每个命名空间都有自己的视图

1. PID 命名空间 (Process Isolation)
   - 每个容器有自己的 PID 1 进程
   - 容器内看不到宿主机的进程
   
2. NET 命名空间 (Network Isolation)
   - 每个容器有自己的网络栈
   - 独立的 IP、端口、路由表
   
3. MNT 命名空间 (Filesystem Isolation)
   - 每个容器有自己的根文件系统视图
   - 挂载点隔离
   
4. IPC 命名空间 (Inter-Process Communication)
   - 信号量、消息队列、共享内存隔离
   
5. UTS 命名空间 (Hostname Isolation)
   - 独立的主机名和域名
   
6. USER 命名空间 (User Isolation)
   - 用户和用户组 ID 映射
   - root 在容器内 ≠ root 在宿主机
```

**示例：查看容器的命名空间**

```bash
# 运行一个容器
docker run -d --name test nginx:alpine

# 获取容器 PID
PID=$(docker inspect test --format '{{.State.Pid}}')

# 查看容器的命名空间
ls -la /proc/$PID/ns/
# 输出：
# lrwxrwxrwx 1 root root 0 Mar 10 10:00 cgroup -> cgroup:[4026531835]
# lrwxrwxrwx 1 root root 0 Mar 10 10:00 ipc -> ipc:[4026532449]
# lrwxrwxrwx 1 root root 0 Mar 10 10:00 mnt -> mnt:[4026532447]
# lrwxrwxrwx 1 root root 0 Mar 10 10:00 net -> net:[4026532452]
# lrwxrwxrwx 1 root root 0 Mar 10 10:00 pid -> pid:[4026532450]
# lrwxrwxrwx 1 root root 0 Mar 10 10:00 user -> user:[4026531837]
# lrwxrwxrwx 1 root root 0 Mar 10 10:00 uts -> uts:[4026532448]
```

### **2. Control Groups (cgroups) - 资源控制**

**cgroups 限制容器资源使用：**

```bash
# Docker 通过 cgroups 控制
1. CPU 限制
2. 内存限制
3. 磁盘 I/O
4. 网络带宽
5. 设备访问

# 查看容器的 cgroup
docker run -d --name cgroup-test \
  --cpus="0.5" \          # 限制使用 0.5 个 CPU
  --memory="100m" \       # 限制内存 100MB
  --memory-swap="200m" \  # 限制交换内存
  nginx:alpine

# 查看 cgroup 配置
cat /sys/fs/cgroup/cpu/docker/<container-id>/cpu.cfs_quota_us
# 输出：50000  (表示 0.5 个 CPU)
```

**cgroups 层次结构：**

```text
/sys/fs/cgroup/
├── cpu
│   └── docker
│       └── <container-id>
│           ├── cpu.shares      # CPU 权重
│           ├── cpu.cfs_quota_us # CPU 限制
│           └── cpu.stat        # CPU 统计
├── memory
│   └── docker
│       └── <container-id>
│           ├── memory.limit_in_bytes      # 内存限制
│           ├── memory.usage_in_bytes      # 内存使用
│           └── memory.stat                # 内存统计
└── blkio
    └── docker
        └── <container-id>
            ├── blkio.throttle.read_bps_device  # 读限制
            └── blkio.throttle.write_bps_device # 写限制
```

### **3. Union File System (UnionFS) - 分层存储**

**Docker 镜像的分层结构：**

```bash
# Docker 镜像就像洋葱，一层一层叠加
Base Image (Alpine)      # 层1: 基础系统
    ↓
ADD apk packages         # 层2: 安装软件
    ↓
COPY app files           # 层3: 应用文件
    ↓
RUN configuration        # 层4: 配置
    ↓
Container (读写层)        # 最上层: 容器运行时修改

# 查看镜像分层
docker history nginx:alpine
# 输出：
# IMAGE          CREATED        CREATED BY                                      SIZE
# e784f4560448   2 weeks ago    /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
# <missing>      2 weeks ago    /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B
# <missing>      2 weeks ago    /bin/sh -c #(nop)  EXPOSE 80                    0B
# <missing>      2 weeks ago    /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B
# <missing>      2 weeks ago    /bin/sh -c #(nop) COPY file:09b214baf42b6330…   4.61kB
# <missing>      2 weeks ago    /bin/sh -c #(nop) COPY file:6fba68fe3229c52e…   1.04kB
```

**UnionFS 工作原理：**

```c
// 联合挂载：多个只读层 + 一个可写层
// 读取文件：从上往下查找，找到第一个就返回
// 写入文件：复制到可写层（Copy-on-Write）

// 支持的 UnionFS 实现：
1. overlay2 (默认，性能最好)
2. aufs (早期版本)
3. devicemapper
4. btrfs
5. zfs

// 查看 Docker 使用的存储驱动
docker info | grep "Storage Driver"
# 输出：Storage Driver: overlay2
```

## 三、**容器创建详细流程**

### **`docker run` 命令的完整执行流程**

```bash
# 当我们执行：
docker run -d --name web -p 80:80 nginx:alpine

# Docker 内部执行以下步骤：
```

### **步骤1：客户端解析命令**

```go
// Docker Client 解析命令行参数
// docker/cli/command/commands.go
func runRun(dockerCli command.Cli, flags *pflag.FlagSet, opts *runOptions, args []string) error {
    // 解析参数
    containerName := opts.name        // "web"
    imageName := args[0]              // "nginx:alpine"
    publish := opts.publish           // "80:80"
    detach := opts.detach             // true
    
    // 调用 Docker API
    return dockerCli.Client().ContainerCreate(...)
}
```

### **步骤2：Docker Daemon 处理请求**

```go
// dockerd 接收 API 请求
// moby/daemon/create.go
func (daemon *Daemon) containerCreate(params types.ContainerCreateConfig) (*container.CreateResponse, error) {
    // 1. 检查镜像是否存在
    img, err := daemon.GetImage(params.Config.Image)
    if err != nil {
        // 从 registry 拉取镜像
        err = daemon.pullImageWithReference(params.Config.Image)
    }
    
    // 2. 创建容器配置
    hostConfig := &container.HostConfig{
        PortBindings: nat.PortMap{
            "80/tcp": []nat.PortBinding{{HostIP: "0.0.0.0", HostPort: "80"}},
        },
    }
    
    // 3. 创建容器
    return daemon.create(params, hostConfig)
}
```

### **步骤3：containerd 创建容器**

```go
// containerd 处理容器创建
// containerd/container.go
func (c *container) Create(ctx context.Context, opts ...NewContainerOpts) error {
    // 1. 准备容器配置
    spec, err := c.Spec(ctx)
    
    // 2. 创建快照（容器文件系统）
    mounts, err := c.createSnapshot(ctx, id, spec)
    
    // 3. 创建容器运行时任务
    task, err := c.NewTask(ctx, cio.NewCreator(cio.WithStdio))
    
    // 4. 启动任务（容器进程）
    return task.Start(ctx)
}
```

### **步骤4：runc 创建容器进程**

```go
// runc 创建容器进程
// runc/libcontainer/process_linux.go
func (p *initProcess) start() error {
    // 1. 创建管道用于通信
    parentPipe, childPipe, err := utils.NewSockPair("init")
    
    // 2. fork 子进程
    cmd := p.cmd
    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID |
                   syscall.CLONE_NEWNS | syscall.CLONE_NEWNET |
                   syscall.CLONE_NEWIPC | syscall.CLONE_NEWUSER,
    }
    
    // 3. 子进程执行 init
    if err := cmd.Start(); err != nil {
        return err
    }
    
    // 4. 配置命名空间、cgroups
    if err := p.sendConfig(); err != nil {
        return err
    }
    
    // 5. 等待容器进程启动
    return p.waitForChild()
}
```

### **步骤5：Linux 内核执行**

```c
// 内核层面发生了什么
// kernel/fork.c
SYSCALL_DEFINE0(fork)
    → do_fork()
        → copy_process()
            → copy_namespaces()  // 创建命名空间
            → cgroup_can_fork()  // 设置 cgroup
        → wake_up_new_task()     // 唤醒新进程

// 容器进程启动后
execve("/docker-entrypoint.sh")  // 执行容器入口点
    → nginx -g "daemon off;"     // 启动 Nginx
```

## 四、**网络工作原理**

### **Docker 网络模型**

```bash
# Docker 提供多种网络模式
1. bridge (默认)    # 网桥模式，容器有自己的网络栈
2. host            # 共享宿主机网络
3. none            # 无网络
4. container:<name> # 共享其他容器的网络
5. overlay         # 跨主机网络（Swarm）
6. macvlan         # MACVLAN 模式
```

### **Bridge 网络实现原理**

```bash
# 创建 bridge 网络时
docker network create mynet

# Docker 实际上执行：
1. 创建 Linux 网桥
   ip link add docker0 type bridge
   ip addr add 172.17.0.1/16 dev docker0
   ip link set docker0 up

2. 创建 veth pair（虚拟以太网设备对）
   ip link add veth1 type veth peer name veth2

3. 一端连接到容器
   ip link set veth2 netns <容器PID>
   ip netns exec <容器PID> ip link set veth2 name eth0
   ip netns exec <容器PID> ip addr add 172.17.0.2/16 dev eth0
   ip netns exec <容器PID> ip link set eth0 up

4. 一端连接到网桥
   ip link set veth1 master docker0
   ip link set veth1 up

5. 设置 iptables NAT
   iptables -t nat -A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
```

### **端口映射原理**

```bash
# docker run -p 80:80
# 实际实现：
1. 创建 DNAT 规则
   iptables -t nat -A DOCKER ! -i docker0 -p tcp --dport 80 -j DNAT --to-destination 172.17.0.2:80

2. 创建 ACCEPT 规则
   iptables -A FORWARD -d 172.17.0.2/32 ! -i docker0 -o docker0 -p tcp --dport 80 -j ACCEPT

# 数据包流向：
外部请求 → 宿主机80端口 → iptables DNAT → 容器172.17.0.2:80
```

## 五、**存储工作原理**

### **镜像存储**

```bash
# 镜像存储在 /var/lib/docker
/var/lib/docker/
├── image/           # 镜像元数据
│   └── overlay2/
│       ├── layerdb/ # 层数据库
│       └── imagedb/ # 镜像数据库
├── overlay2/        # 实际文件存储（overlay2驱动）
│   ├── l/          # 硬链接目录
│   ├── diff/       # 每层差异内容
│   └── merged/     # 合并后的视图
└── containers/     # 容器运行时数据
```

### **Volume 实现原理**

```bash
# docker run -v /host/path:/container/path
# 实现方式：

# 1. bind mount（绑定挂载）
mount --bind /host/path /var/lib/docker/overlay2/<container-id>/merged/container/path

# 2. volume（Docker 管理）
# 创建 volume
docker volume create myvol
# 实际位置：/var/lib/docker/volumes/myvol/_data

# 3. tmpfs（内存文件系统）
mount -t tmpfs tmpfs /container/path
```

### **Copy-on-Write (CoW) 机制**

```bash
# 当容器修改文件时：
1. 文件在只读层 → 复制到可写层（copy-up）
2. 修改在可写层进行
3. 后续读取从可写层读取

# 性能影响：
- 小文件修改：有 copy-up 开销
- 大文件修改：开销较大
- 只读文件：无开销

# 优化建议：
- 数据库数据放在 volume 中
- 日志文件放在 volume 中
- 频繁修改的文件放在 volume 中
```

## 六、**Docker 组件详细交互**

### **完整启动时序图**

```text
Docker Client
    │ 1. docker run -d nginx:alpine
    ↓
Docker Daemon (dockerd)
    │ 2. 解析命令，检查本地镜像
    ↓
containerd
    │ 3. 准备容器运行时配置
    ↓
containerd-shim
    │ 4. 作为容器的父进程
    ↓
runc
    │ 5. 创建容器进程
    ↓
Linux Kernel
    │ 6. 创建命名空间、cgroups
    ↓
容器进程 (nginx)
```

### **各组件职责**

```go
// 1. Docker Client
//    职责：命令行解析、用户交互
//    位置：/usr/bin/docker

// 2. Docker Daemon (dockerd)
//    职责：镜像管理、网络管理、REST API
//    位置：/usr/bin/dockerd
//    配置：/etc/docker/daemon.json

// 3. containerd
//    职责：容器生命周期管理、镜像分发
//    位置：/usr/bin/containerd
//    配置：/etc/containerd/config.toml

// 4. containerd-shim
//    职责：无守护进程的容器运行时
//    作用：允许重启 containerd 而不影响容器

// 5. runc
//    职责：OCI 运行时，实际创建容器
//    位置：/usr/bin/docker-runc
```

## 七、**性能优化原理**

### **1. 镜像优化**

```dockerfile
# 优化前
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y python3 python3-pip
RUN pip3 install flask gunicorn
COPY . /app
WORKDIR /app
CMD ["gunicorn", "app:app"]

# 优化后（多阶段构建）
# 阶段1：构建
FROM python:3.9-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# 阶段2：运行
FROM python:3.9-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["gunicorn", "app:app"]

# 优化结果：
# 镜像大小：1.2GB → 120MB
# 构建时间：3分钟 → 45秒
```

### **2. 存储驱动选择**

```bash
# 不同存储驱动性能对比
驱动         | 写性能 | 启动速度 | 内存使用 | 适用场景
-------------|--------|----------|----------|---------
overlay2     | 高     | 快       | 低       | 通用（默认）
devicemapper | 中     | 中       | 中       | 企业存储
btrfs        | 高     | 慢       | 高       | 快照频繁
zfs          | 高     | 慢       | 高       | 大数据

# 查看当前驱动
docker info | grep -A5 "Storage Driver"

# 修改存储驱动（在 /etc/docker/daemon.json）
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
```

### **3. 资源限制优化**

```bash
# 合理设置资源限制
docker run -d \
  --cpus="1.5" \           # 限制 1.5 个 CPU
  --memory="512m" \        # 内存限制 512MB
  --memory-swap="1g" \     # 交换内存 1GB
  --blkio-weight="500" \   # 磁盘 IO 权重
  --pids-limit="100" \     # 进程数限制
  nginx:alpine

# 监控容器资源使用
docker stats
# CONTAINER ID   CPU %   MEM USAGE / LIMIT   MEM %   NET I/O   BLOCK I/O   PIDS
# a1b2c3d4       0.15%   25MiB / 512MiB      4.89%   0B / 0B   0B / 0B     3
```

## 八、**安全机制**

### **1. 命名空间隔离**

```bash
# 每个容器都有独立的：
# - PID 空间（看不到其他进程）
# - 网络空间（独立 IP、端口）
# - 文件系统（独立根目录）
# - 用户空间（用户 ID 映射）
```

### **2. Capabilities 限制**

```bash
# Docker 默认移除大部分 Linux capabilities
# 查看容器 capabilities
docker run --rm alpine sh -c 'capsh --print'
# Current: = cap_chown,cap_dac_override,...ep

# 运行特权容器（不推荐）
docker run --privileged nginx:alpine

# 只添加需要的 capabilities
docker run --cap-add=NET_ADMIN nginx:alpine
```

### **3. Seccomp 安全配置文件**

```bash
# Docker 默认使用 seccomp 配置文件
# 限制容器可以调用的系统调用

# 查看默认配置文件
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  alpine sh -c 'wget -q -O - http://localhost/info | grep -i seccomp'

# 自定义 seccomp 配置文件
docker run --security-opt seccomp=/path/to/profile.json nginx:alpine

# 禁用 seccomp（不推荐）
docker run --security-opt seccomp=unconfined nginx:alpine
```

### **4. AppArmor/SELinux**

```bash
# AppArmor (Ubuntu/Debian)
docker run --security-opt apparmor=docker-default nginx:alpine

# SELinux (RHEL/CentOS)
docker run --security-opt label=type:container_t nginx:alpine
```

## 九、**实际调试技巧**

### **1. 查看 Docker 内部状态**

```bash
# 查看 Docker 事件流
docker events

# 查看容器详细配置
docker inspect <container>

# 查看容器日志
docker logs -f <container>

# 进入容器内部
docker exec -it <container> sh

# 检查容器进程
docker top <container>
```

### **2. 性能分析**

```bash
# 使用 ctop 查看容器状态
docker run --rm -ti \
  --name=ctop \
  --volume /var/run/docker.sock:/var/run/docker.sock:ro \
  quay.io/vektorlab/ctop:latest

# 使用 dive 分析镜像
docker run --rm -ti \
  -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive:latest <image>

# 使用 sysdig 监控
docker run -i -t --name sysdig --privileged \
  -v /var/run/docker.sock:/host/var/run/docker.sock \
  -v /dev:/host/dev -v /proc:/host/proc:ro \
  -v /boot:/host/boot:ro -v /lib/modules:/host/lib/modules:ro \
  -v /usr:/host/usr:ro \
  sysdig/sysdig
```

### **3. 网络调试**

```bash
# 查看容器网络配置
docker network inspect bridge

# 查看 iptables 规则
iptables -t nat -L -n
iptables -L -n

# 使用 nsenter 进入容器网络命名空间
PID=$(docker inspect -f '{{.State.Pid}}' <container>)
nsenter -t $PID -n ip addr show
nsenter -t $PID -n ping 8.8.8.8
```

## **总结：Docker 工作原理要点**

1. **隔离机制**：Linux 命名空间（6 种）提供隔离环境
2. **资源控制**：cgroups 限制 CPU、内存、磁盘等资源
3. **文件系统**：UnionFS 实现镜像分层和 CoW
4. **网络**：虚拟网桥、veth pair、iptables 实现网络隔离和端口映射
5. **架构**：Client/Server 架构，模块化设计（dockerd → containerd → runc）
6. **安全**：多层次安全机制（capabilities、seccomp、AppArmor/SELinux）
7. **性能**：合理的资源限制、镜像优化、存储驱动选择

Docker 的本质是 **"轻量级虚拟机"**，但实际上是在宿主机内核上运行的**隔离进程**，这是它比虚拟机更轻量、启动更快的原因。