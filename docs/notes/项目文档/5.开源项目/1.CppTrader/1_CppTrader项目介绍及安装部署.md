---
title: 1、CppTrader项目介绍及安装部署
createTime: 2026/03/23 11:30:57
permalink: /project-docs/Opensource/cpptrader/CppTrader项目介绍及安装部署/
icon: pajamas:deployments
order: 1
---


## CppTrader项目介绍及地址

CppTrader是一个开源的、专为金融交易设计的C++高性能框架，由ChronoXOR开发并维护。该项目提供了一套完整的交易系统组件，旨在帮助开发者快速构建低延迟、高吞吐量的交易平台，适用于股票、期货、外汇等金融产品的交易系统开发;

项目地址:[CppTrader](https://github.com/chronoxor/CppTrader)

## 核心定位

1. 高性能交易系统构建基石：专为追求极致速度和可靠性的交易平台设计

2. 事件驱动架构：支持高并发场景下的高效处理

3. 跨平台支持：覆盖Linux、macOS、Windows三大操作系统


## 主要组件

CppTrader包含三个核心功能模块：

1. 超快速匹配引擎：交易系统的核心订单匹配组件
2. 订单簿处理器：维护和管理委托账本
3. NASDAQ ITCH协议处理器：处理ITCH市场数据流

## 不同模块功能详解

### 匹配引擎（Matching Engine）

匹配引擎是交易系统的核心，负责订单的撮合逻辑：

2. 订单类型支持：市价单、限价单、止损单等常见类型
3. 价格-时间优先原则：按价格优先、同价时间优先的顺序撮合
4. 多级市场深度：维护多个价格档位的订单队列
5. 事件通知机制：订单接受、成交、取消等事件实时回调


### 订单簿处理器（Order Book Processor）

订单簿是市场深度的核心数据结构：

1. 买卖盘维护：分别维护买单队列和卖单队列
2. 实时更新：处理新订单、撤单、成交等操作
3. 快照生成：支持市场深度快照输出

### ITCH协议处理器

NASDAQ ITCH是交易所常用的市场数据协议：

1. 消息解析：解析ITCH 5.0协议的各种消息类型
2. 流式处理：处理TCP流式数据的粘包/拆包问题（与前述ITCHHandler设计一致）
3. 高吞吐量：达到纳秒级别的消息处理速度

## 引入的技术特性

CppTrader在性能优化方面采用了多种技术手段：

| 优化技术         | 说明                                 |
| :--------------- | :----------------------------------- |
| **零拷贝**       | 减少数据在内存中的复制次数           |
| **内存池**       | 预分配内存，避免频繁的new/delete操作 |
| **无锁数据结构** | 使用原子操作替代互斥锁，降低竞争开销 |
| **NUMA感知**     | 优化内存访问局部性，提升多核性能     |
| **CPU亲和性**    | 绑定线程到特定核心，减少上下文切换   |


## 安装部署

为防止不同组件版本导致环境污染，本部署方案使用docker环境，在docker环境中下载源码，然后编译，安装，部署，测试等；

### 基础镜像

本部署案例基于Ubuntu:22.04版本，因此首先在服务器上下载Ubuntu基础镜像:

```shell
docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/ubuntu:22.04
docker tag  swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/ubuntu:22.04  ubuntu:22.04
```

### 安装部署工具包

在构建和编译源代码之前，需要安装基础环境，因此，为方便调试和测试，首先基于基础镜像构建一个包含基础环境的镜像,Dockerfile文件如下:


```Dockerfile
FROM ubuntu:22.04


WORKDIR /home/app/

RUN apt update; \
    apt makecache ; \
    apt install -y cmake gcc g++ python3 pip uuid-dev git vim tree wget curl iproute2 net-tools inetutils-ping; \
    pip3 install gil

CMD []
```

执行以下构建命令构建镜像:

```shell
docker build -f Dockerfile-tools -t ubuntu-tools-22.04:1.0.0 .
```


### 下载及编译源码

基础环境安装完成之后，接下来要在基础环境的容器中下载CppTrade项目源码进行编译和安装，因此将克隆，编译，安装等步骤封装在DockerFile文件中，进一步构建出应用镜像;

```Dockerfile
FROM ubuntu-tools-22.04:1.0.0

RUN git clone https://github.com/chronoxor/CppTrader.git ; \
    cd CppTrader && gil update; \
    cd build && sed -i 's/\[\[/\[/g' ./unix.sh && sed -i 's/\]\]/\]/g' ./unix.sh && sh unix.sh

CMD []
```

执行build脚本构建编译好的应用镜像;

```shell
docker build -f Dockerfile-compile -t cpptradeubuntu-tools-22.04:1.0.0 .
```

最后，执行完上面的构建步骤后，我们就得到一个`cpptradeubuntu-tools-22.04:1.0.0`应用镜像，此镜像内包含了我们已经编译和安装好的CppTrader项目。接下来我们可以运行该容器并且测试项目；

```shell
# 运行并且进入容器
docker run -it --rm -v /home/data:/home/app/CppTrader/data cpptradeubuntu-tools-22.04:1.0.0  /bin/bash
```

执行测试脚本:

```shell
cpptrader-performance-itch_handler < /home/app/CppTrader/data/01302019.NASDAQ_ITCH50
```

注意：在执行测试之前，首先要将测试文件下载到本地然后挂载到容器内部;