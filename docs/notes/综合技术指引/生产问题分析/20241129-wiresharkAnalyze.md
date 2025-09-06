---
title: Wireshark抓包分析
icon: material-icon-theme:vagrant
order: 5
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - SPRINGBOOT
tag:
  - spring
  - java
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 10:13:29
permalink: /compre-guide/production-issues/wiresharkAnalyze/
---
<!-- TOC -->

- [Wireshark抓包分析](#wireshark抓包分析)
  - [Wireshark总界面](#wireshark总界面)
  - [数据包详情](#数据包详情)
  - [wireshark过滤器表达式的规则](#wireshark过滤器表达式的规则)
  - [HTTP请求过程](#http请求过程)
  - [TCP协议工作原理](#tcp协议工作原理)
    - [三次握手](#三次握手)
  - [Wireshark抓包分析](#wireshark抓包分析-1)
    - [数据抓包](#数据抓包)
    - [三次握手分析](#三次握手分析)
    - [发送请求](#发送请求)
    - [服务器响应](#服务器响应)
    - [四次挥手分析](#四次挥手分析)
    - [tcp状态](#tcp状态)

<!-- /TOC -->

# Wireshark抓包分析

## Wireshark总界面

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/20241128192727.png)

界面中主要关注三块内容：

1. 数据列表区域，主要展示主机之间网络包数据的请求和响应信息；
2. 数据详情区，数据详情区域主要展示每一个数据包的详情信息，不同网络层的封装数据；
3. 数据字节流区：主要展示网络包中每一层详细的字节流数据；

数据包中表头主要包含以下几块内容：

1. NO：数据包编号
2. Time:数据包时间
3. Source:数据包来源地址
4. Destination:数据包目的地址
5. Protocol:数据包协议
6. Length:数据包长度
7. info：数据包详情

wireshark中对不同的协议和数据包有不同的着色规则，规则如下：

![image-20241128193119157](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241128193119157.png)

## 数据包详情

![image-20241128193429429](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241128193429429.png)

Frame:物理层数据流，传输网络数据流，比特；

Ethernet:数据链路层帧，数据包单位是帧；

Internet Protocol Version:网络层数据包；

Transmission Control Protocol:传输层数据包，tcp,udp协议；

Hypertext：应用层数据包，协议数据单元pdu;

数据包中数据分别对应网络模型中的不同层，因此下面我们详细介绍下每层包含的数据；

**网络模型**

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/76a7f779019847e4b78e1ee57dcc93bb.png)



**TCP数据包字段内容**

![image-20241129183600335](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241129183600335.png)

## wireshark过滤器表达式的规则

抓包过滤器类型

- Type（host、net、port）
- 方向Dir（src、dst）
- 协议Proto（ether、ip、tcp、udp、http、icmp、ftp等）
- 逻辑运算符（&& 与、|| 或、！非）

**过滤IP，如来源IP或者目标IP等于某个IP**

```text
ip.src eq 192.168.1.107 or ip.dst eq 192.168.1.107
ip.addr eq 192.168.1.107 // 都能显示来源IP和目标IP
```



> 提示： 在Filter编辑框中，收入过虑规则时，如果语法有误，框会显红色，如正确，会是绿色。

**端口过滤**

```text
tcp.port eq 80 // 不管端口是来源的还是目标的都显示
tcp.port == 80 // 不管端口是来源的还是目标的都显示
tcp.port eq 2722
tcp.port eq 80 or udp.port eq 80
tcp.dstport == 80 // 只显tcp协议的目标端口80
tcp.srcport == 80 // 只显tcp协议的来源端口80
udp.port eq 15000

过滤端口范围
tcp.port >= 1 and tcp.port <= 80 
```

**协议过滤**

- TCP，只显示TCP协议的数据包列表
- HTTP，只查看HTTP协议的数据包列表
- ICMP，只显示ICMP协议的数据包列表
- smtp
-  ftp
- dns

**案例**

```text
排除tcp协议：
!tcp 或者 not tcp
```

**过滤MAC地址**

```text
太以网头过滤
eth.dst == A0:00:00:04:C5:84 // 过滤目标mac
eth.src eq A0:00:00:04:C5:84 // 过滤来源mac
eth.dst==A0:00:00:04:C5:84
eth.dst==A0-00-00-04-C5-84
eth.addr eq A0:00:00:04:C5:84 // 过滤来源MAC和目标MAC都等于A0:00:00:04:C5:84的

less than 小于 < lt 
小于等于 le

等于 eq
大于 gt
大于等于 ge
不等 ne
```



**包长度过滤**

```text
udp.length == 26 这个长度是指udp本身固定长度8加上udp下面那块数据包之和
tcp.len >= 7   指的是ip数据包(tcp下面那块数据),不包括tcp本身
ip.len == 94 除了以太网头固定长度14,其它都算是ip.len,即从ip本身到最后
frame.len == 119 整个数据包长度,从eth开始到最后

eth —> ip or arp —> tcp or udp —> data
```

**http模式过滤**

```text
http.request.method == "GET"
http.request.method == "POST"
http.request.uri == "/img/logo-edu.gif"
http contains “HTTP/1.”

// GET包
http.request.method == “GET” && http contains “Host: “
http.request.method == “GET” && http contains “User-Agent: “

// POST包
http.request.method == “POST” && http contains “Host: “
http.request.method == “POST” && http contains “User-Agent: “

// 响应包
http contains “HTTP/1.1 200 OK” && http contains “Content-Type: “
http contains “HTTP/1.0 200 OK” && http contains “Content-Type: “

一定包含如下
Content-Type: 
```



## HTTP请求过程

HTTP连接本质就是使用TCP协议建立起的可靠连接进行请求发送与请求响应。HTTP请求过程大致可以分为五个步骤），每个步骤细节如下：

1. DNS解析：当客户端（如浏览器）尝试访问一个网站时，它首先会尝试从本地缓存中查找域名对应的IP地址。如果本地缓存中没有找到，客户端会发送DNS查询请求到本地的DNS服务器。
2. 建立TCP连接：客户端（如浏览器）向服务器发起连接请求，通常使用TCP/IP协议建立一个到服务器的TCP连接。这个过程称为三次握手（Three-way handshake），确保连接的可靠性。
3. 客户端发送请求：客户端向服务器发送HTTP请求消息。
4. 服务端响应请求：服务器接收到请求后，会检查请求头和请求体，然后根据请求方法（如GET、POST）和资源路径来决定如何响应。
5. 释放TCP连接：请求完成后，客户端和服务器可以关闭连接。

## TCP协议工作原理

TCP连接的建立与断开，正常过程至少需要客户	端与服务端来回发送7个包（请求）才能完成。其中客户端发送4个（1、3、4、7），服务端发送3个（2，5，6），具体原理参照下图。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241129092219415.png)

传输控制依赖首部包含的6个标志：

- URG:紧急指针
- ACK：确认序号是否有效
- PSH:接收方应**尽快**将这个报文段**交给应用层**
- RST:重建连接
- SYN:同步序号用来发起一个连接
- FIN:发送端完成发送任务

### 三次握手

三次握手建立连接

TCP建立连接的过程，包括客户端和服务器总共发送3个包，此过程称为三次握手：

```text
第一次握手：客户端发送一个带有SYN标志的TCP段，请求与服务器建立连接。

第二次握手：服务器接收到SYN请求后，发送一个带有SYN和ACK标志的TCP段作为响应。

第三次握手：客户端接收到服务器的SYN+ACK响应后，发送一个带有ACK标志的TCP段，完成连接建立。

三次握手完成后便建立了TCP连接，便可进行数据传输，TCP负责将数据分割为多个段（Segment），并按顺序发送。
```

2.3.4、四次挥手断开连接

TCP断开连接的过程，包括客户端和服务器总共发送4个包，此过程称为四次挥手：

```text
第一次挥手：客户端发送一个带有FIN标志的TCP段，请求断开连接；

第二次挥手：服务器接收到FIN请求后，发送一个带有ACK标志的TCP段作为响应；

第三次挥手：服务器发送一个带有FIN标志的TCP段，请求断开连接；

第四次挥手：客户端接收到服务器的FIN请求后，发送一个带有ACK标志的TCP段，完成连接断开。
```



## Wireshark抓包分析

### 数据抓包

**第一步**：打开wireshark主界面，连接本地网络，点击抓包按钮开始抓包；

![image-20241129093128129](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241129093128129.png)

**第二步**：ping www.baidu.com 获取目的地址的ip，方便后续网络分析；

**第三步：**执行`curl www.baidu.com`命令，让wireshark捕获本机和百度服务器之间的数据包；

经过上面三步骤，我们已经抓到本机和百度服务器之间的数据包，打开数据包如下所示:

![image-20241129093407660](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241129093407660.png)

再filter处输入获取到的百度ip，过滤处本机和百度之间的数据包，过滤命令：`ip.addr == 180.101.50.242`

![image-20241129093615939](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241129093615939.png)



### 三次握手分析

![image-20241129094300167](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241129094300167.png)

第一个包是【第一次握手】，本机:10.255.11.40 ---> 百度【180.101.20.242】发送一个SYN包，表示【新建立连接请求】

第二个包是【第二次握手】，百度【180.101.20.242】---> 本机:10.255.11.40回复一个SYN_ACK确认包，表示【接受链接请求】

第三个包是 【第三次握手】，本机:10.255.11.40 ---> 百度【180.101.20.242】本机回复ACK确认包，表示确认【建立连接请求】

> 发送完确认请求后，我开启到百度的**单向连接通道**，百度收到我的确认请求后，就开启到我这边的单向连接通道，**两边通道**都开启以后，就可以通信了。

### 发送请求

经过三次握手建立连接后，客户端向服务器发送第一个GET请求，请求序号是454，

![image-20241129095352696](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241129095352696.png)

客户端向服务器发送HTTP请求消息。请求消息通常包括以下部分：

1. 请求行：包含请求方法（如GET、POST）、请求的资源路径、HTTP版本。
2. 请求头：包含客户端信息、请求参数等，例如User-Agent、Accept、Cookie等。
3. 请求体：对于某些请求方法（如POST），可能包含额外的数据，用于向服务器发送信息。

应用层请求数据展示了用户的GET请求：

![image-20241129100946459](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241129100946459.png)

Request Method：请求方法，这里的请求方法是GET；

Request URI：请求的URI，没指定默认是/，因为我们只请求了域名www.baidu.com，并没指定要获取的资源，所以是默认的/；

Request Version：请求的版本，因为用的是HTTP协议，所以这里显示HTTP协议的版本。

**Host**：目标主机；

**User-Agent**：用户代理，也就是浏览器的类型。由于我们没用浏览器，所以这里显示的是命令curl，版本为8.9.1；

**Accept**：浏览器可接受的MIME（Multipurpose Internet Mail Extensions）类型。

> 浏览器通常使用 MIME 类型（而不是文件扩展名）来确定如何处理URL，因此 We b服务器在响应头中添加正确的 MIME 类型非常重要。如果配置不正确，浏览器可能会无法解析文件内容，网站将无法正常工作，并且下载的文件也会被[错误处理](https://marketing.csdn.net/p/3127db09a98e0723b83b2914d9256174?pId=2782?utm_source=glcblog&spm=1001.2101.3001.7020)。

### 服务器响应

![image-20241129101322155](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241129101322155.png)

状态行：包含版本和响应状态码、状态信息；

响应头：包含响应的服务器的资源信息，一行一个响应头；

响应空行：用来间隔/区分响应头和响应体；

响应体：服务器响应的内容，通常是一个HTML页面的代码或者给客户端的数据，上图中可以看到本次响应的内容是一个文件数据（File Data）。
**状态行包含以下几个字段**

- **Response Version**：响应版本，因为使用的是HTTP协议，所以这里显示了HTTP的版本；
- **Status Code**：响应状态码，这里的 200 表示请求成功；
- **Response Phrase**：响应状态码的提示信息。

**响应头信息**

- Accept-Ranges: 告知客户端资源是否支持范围请求。当前取值bytes表明资源支持范围请求，可以使用Range头进行请求。

- Cache-Control: 控制缓存行为，包括是否缓存、缓存的有效期、缓存策略等。

- Connection: 服务器是否需要保持连接。

> keep-alive表示服务器希望在发送响应后不立即关闭连接，而是保持连接状态以供后续请求复用。这种设置可以提高服务器的响应效率，减少TCP连接的建立和断开的开销，特别是在处理大量并发请求的场景下。

- Content-Length: 响应内容的字节数，本例是2381字节。

- Content-Type: 响应内容的类型和编码方式，text/html。
- Date: 指示响应消息生成的时间。
- Etag: 资源的实体标签（Entity Tag），用于缓存控制和验证。
- Last-Modified: 资源的最后修改时间
- Pragma: 历史遗留的HTTP头，它最初用于缓存控制，但现在已不推荐使用。取值no-cache表明不使用缓存。
- Server: 服务器的名称或版本。
- Set-Cookie: 用于设置客户端的Cookie

当打开响应体，会发现它的内容是明文的，对于机密通信来说HTTP通信很容易产生信息泄露。另外，以上介绍的整个交互过程中并不验证通信方的身份，第三方可以冒充他人身份参与通信。后续会发图文详解，带你搞清楚HTTPS协议的工作机制。

![image-20241129181727130](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241129181727130.png)



### 四次挥手分析

![image-20241129181852377](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241129181852377.png)

第一个包【第一次握手】，本机:10.255.11.40 ---> 百度【180.101.20.242】发送一个「FIN+ACK」，表示这是一个「释放连接」的请求；

第二个包【第二次握手】，百度【180.101.20.242】---> 本机:10.255.11.40，响应一个「ACK」，表示这是一个「确认请求」，我收到后，就会释放我到百度的单向连接；

第三个包【第三次握手】，百度【180.101.20.242】---> 本机:10.255.11.40发送一个「FIN+ACK」，表示这是一个「释放连接」的请求；

第四个包【第四次握手】，本机:10.255.11.40 ---> 百度【180.101.20.242】响应一个「ACK」，表示这是一个「确认请求」，百度收到后，就会释放到我这边的单向连接。

> 双向的连接都释放后，TCP连接就关闭了（对于本例来说就是百度响应结束），此次通信结束。

### tcp状态

在TCP层，有个FLAGS字段，这个字段有以下几个标识：SYN, FIN, ACK, PSH, RST, URG；

1. SYN表示建立连接，
2. FIN表示关闭连接，
3. ACK表示响应，
4. PSH表示有 DATA数据传输，
5. RST表示连接重置。

其中，ACK是可能与SYN，FIN等同时使用的，比如SYN和ACK可能同时为1，它表示的就是建立连接之后的响应；

如果只是单个的一个SYN，它表示的只是建立连接。

TCP的几次握手就是通过这样的ACK表现出来的。

但SYN与FIN是不会同时为1的，因为前者表示的是建立连接，而后者表示的是断开连接。

RST一般是在FIN之后才会出现为1的情况，表示的是连接重置。

一般地，当出现FIN包或RST包时，我们便认为客户端与[服务器](http://www.yunsec.net/a/zhuanti/enterprise/server/)端断开了连接；而当出现SYN和SYN＋ACK包时，我们认为客户端与服务器建立了一个连接。

PSH为1的情况，一般只出现在 DATA内容不为0的包中，也就是说PSH为1表示的是有真正的TCP数据包内容被传递。

TCP的连接建立和连接关闭，都是通过请求－响应的模式完成的。