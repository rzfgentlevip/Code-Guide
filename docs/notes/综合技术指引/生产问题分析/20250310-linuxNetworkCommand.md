---
title: Linux网络命令生产问题排查
icon: streamline-ultimate-color:network
order: 5
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - PROBLEM
  - LINUX
tag:
  - problen
sticky: false
star: true
footer: linux
copyright: bugcode
createTime: 2025/09/04 10:10:20
permalink: /compre-guide/production-issues/linuxNetworkCommand/
---


# Linux网络

## 网络总览

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20250307092011921.png)



## nslookup

查询dns域名解析

## route

查看内核路由表

## ifconfig命令

`ifconfig` 命令是用于查看和配置网络接口的工具，虽然在较新的Linux发行版中已经被 ip 命令取代，但在某些系统中依然可以使用。使用 ifconfig 可以查看网络接口的详细信息，如IP地址、子网掩码、MAC地址等。

**查看网络接口信息**

```shell
ifconfig

br-5636d51b8415: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.0.0  broadcast 172.18.255.255
        inet6 fe80::42:f8ff:fe9d:cb45  prefixlen 64  scopeid 0x20<link>
        inet6 fe80::1  prefixlen 64  scopeid 0x20<link>
        inet6 fc00:f853:ccd:e793::1  prefixlen 64  scopeid 0x0<global>
        ether 02:42:f8:9d:cb:45  txqueuelen 0  (Ethernet)
        RX packets 293  bytes 18720 (18.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 163  bytes 38246 (38.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

br-c1f5a8d10206: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.19.0.1  netmask 255.255.0.0  broadcast 172.19.255.255
        inet6 fe80::42:2eff:fee1:3cd4  prefixlen 64  scopeid 0x20<link>
        ether 02:42:2e:e1:3c:d4  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7  bytes 906 (906.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:29ff:fe2e:2048  prefixlen 64  scopeid 0x20<link>
        ether 02:42:29:2e:20:48  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7  bytes 906 (906.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.22.50.94  netmask 255.255.240.0  broadcast 172.22.63.255
        inet6 fe80::215:5dff:fe1d:6822  prefixlen 64  scopeid 0x20<link>
        ether 00:15:5d:1d:68:22  txqueuelen 1000  (Ethernet)
        RX packets 26877  bytes 37391926 (37.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10059  bytes 984388 (984.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

**查看某一个网络接口信息**

```shell
ifconfig eth0
```

启用或禁用网络接口：可以使用 `ifconfig` 命令启用某个网络接口，将其状态设置为 UP/DOWN：

```shell
sudo ifconfig eth0 up
sudo ifconfig eth0 down
```

为网络接口配置 IP 地址：可以使用 `ifconfig` 为某个网络接口手动配置 IP 地址、子网掩码和广播地址：

```shell
ifconfig eth0 192.168.1.100 netmask 255.255.255.0 broadcast 192.168.1.255
```

ifconfig输出内容解释

```shell
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.2  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::a00:27ff:fe4e:66a2  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:4e:66:a2  txqueuelen 1000  (Ethernet)
        RX packets 1000  bytes 123456 (123.4 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1000  bytes 123456 (123.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 1000  bytes 123456 (123.4 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1000  bytes 123456 (123.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

- eth0: 网络接口名称，eth0 通常是指以太网接口，lo 是本地主机环回接口。
- flags: 接口状态标志，比如 UP（接口已启用）、BROADCAST（支持广播）、RUNNING（接口正在运行）、MULTICAST（支持多播）等。
- mtu: 最大传输单元（Maximum Transmission Unit），表示一次可以传输的最大数据包大小,单位是字节。
- inet: IPv4 地址，表示当前接口的 IPv4 地址，例如 192.168.1.2。
- netmask: 子网掩码，例如 255.255.255.0，用于划分网络和主机部分。
- broadcast: 广播地址，例如 192.168.1.255，用于在子网中发送广播数据包。
- inet6: IPv6 地址，例如 fe80::a00:27ff:fe4e:66a2。
- prefixlen：这表示IPv6地址的前缀长度。
- scopeid 0x20< link>：IPv6地址有不同的作用范围（scope）。scopeid 定义了该地址的作用范围，0x20 是16进制格式的范围标识符。
- txqueuelen：这是网络接口的传输队列长度，表示接口能够同时排队等待传输的最大数据包数量。
- ether: 物理地址（MAC 地址），例如 08:00:27:4e:66:a2。
- RX packets 和 TX packets: 接收和发送的网络数据包数。
- errors, dropped, overruns: 分别表示接收和发送过程中出现的错误、丢包和超限。

