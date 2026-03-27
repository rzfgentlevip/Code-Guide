---
title: 1、LiquiBook项目介绍及安装包部署
createTime: 2026/03/24 15:24:07
permalink: /project-docs/Opensource/liquibook/LiquiBook项目介绍及安装包部署/
order: 1
icon: glyphs-poly:border-outer
---



## 安装及部署


根据项目介绍，在编译和测试前需要安装编译环境和测试工具，[LiquiBook 项目依赖关系](https://github.com/enewhuis/liquibook?tab=readme-ov-file#dependencies)，因此为防止环境污染，本次编译安装在 Ubuntu:22.04版本的docker镜像中安装部署；


### 构建基础工具镜像包

```Dockerfile
FROM ubuntu:22.04


WORKDIR /home/app/

RUN apt update; \
    apt makecache ; \
    apt install -y mpc-ace build-essential python3 libbz2-dev libz-dev libicu-dev libboost-all-dev cmake gcc g++ python3 pip uuid-dev git vim tree wget curl iproute2 net-tools inetutils-ping; \
    pip3 install gil

CMD []
```

构建命令:

```shell
docker build -f Dockerfile-tools -t liquibook-ubuntu-tools-22.04:1.0.0 .
```

### 构建应用镜像

```Dockerfile
FROM liquibook-ubuntu-tools-22.04:1.0.0


ADD ./liquibook-2.0.0.tar.gz .

# 设置环境变量
ENV BOOST_VERSION=1.74.0 \
    BOOST_CFG=-gcc114-mt-1_74 \
    BOOST_ROOT=/usr \
    MPC_ROOT=/usr/lib/ace/MPC/ \
    LIQUIBOOK_ROOT=/home/app/liquibook-2.0.0 \
    QUICKFAST_ROOT=/home/app/liquibook-2.0.0/noQuickFAST

CMD []
```

> docker镜像中设置环境变量，需要将环境便令指定的版本替换为安装的组件的版本，QUICKFAST_ROOT环境变量设置为

构建命令:

```shell
docker build -f Dockerfile-compile -t liquibookapp-ubuntu-tools-22.04:1.0.0 .
```

### 项目编译安装

```shell
$ cd liquibook
$ . ./env.sh
$ $MPC_ROOT/mwc.pl -type make liquibook.mwc
$ make depend
$ make all
```

