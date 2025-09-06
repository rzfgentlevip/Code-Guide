---
title: HBASE核心原理
icon: streamline-freehand:notes-book
order: 5
author: bugcode
date: 2024-12-26T00:00:00.000Z
category:
  - 面试
tag:
  - 面试
  - 场景
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 15:13:45
permalink: /learning-notes/3r7jwjmf/
---

# HBASE核心原理

## HBase概述

HBase是以 hdfs 为数据存储的，一种分布式、非关系型的、可扩展的 NoSQL 数据库

关系型数据库和非关系型数据库的区别：

关系型数据库和非关系型数据库是两种不同的数据库类型，它们在存储方式、数据结构、查询语言等方面存在显著差异。

### 存储方式和结构

- 关系型数据库以二维表格形式存储数据，结构比较规整固定。
- 非关系型数据库的存储格式可以是key-value形式、文档形式、图片形式等，结构更加灵活和可扩展。

### 数据结构和表的关系

- 关系型数据库最典型的数据结构是表，由二维表及其之间的联系所组成的一个数据组织。在关系型数据库中，必须定义好表和字段结构后才能添加数据。
- 非关系型数据库一般不确保遵照ACID标准的数据储存系统，可以是文档或键值对等，结构更加灵活。

### 优点和应用场景

- 关系型数据库易于理解和维护，使用SQL语言通用，可用于复杂查询，适用于需要事务支持和复杂查询的应用。
- 非关系型数据库速度快，效率高，使用灵活，适用于需要大规模数据的读写和高并发访问的应用。

### 代表产品

- 关系型数据库的主要代表有SQL Server，Oracle, Mysql, PostgreSQL等。
- 非关系型数据库的代表包括MongoDB、Cassandra、Redis等。

总体来说，关系型数据库和非关系型数据库各有优势和局限，选择哪种类型的数据库取决于具体的应用需求和场景。

## 表数据模型

### Name Space

命名空间，类似于关系型数据库的 database 概念，每个命名空间下有多个表；HBase 两个自带的命名空间，分别是 hbase 和 default，hbase 中存放的是 HBase 内置的表，default表是用户默认使用的命名空间

### Table

类似于关系型数据库的表概念。不同的是，**HBase 定义表时只需要声明列族即可，不需要声明具体的列**。因为数据存储是稀疏的，所以往HBase 写入数据时，字段可以动态、按需指定。因此，和关系型数据库相比，HBase 能够轻松应对字段变更的场景

### Row

HBase 表中的每行数据都由一个 RowKey 和多个 Column（列）组成，数据是按照 RowKey的字典顺序存储的，并且查询数据时只能根据 RowKey 进行检索，所以 RowKey 的设计十分重要

### Column

HBase 中的每个列都由 Column Family(列族)和 Column Qualifier（列限定符）进行限定，例如 info:name，info:age。建表时，只需指明列族，而列限定符无需预先定义

### Time Stamp

用于标识数据的不同版本（version），每条数据写入时，系统会自动为其加上该字段，其值为写入HBase 的时间

### Cell

由{rowkey, column Family：column Qualifier, timestamp} 唯一确定的单元。cell 中的数据全部是字节码形式存储

总结：

- HBase 定义表时只需要声明列族即可，不需要声明具体的列，因为数据的存储是稀疏的；也因此能轻松应对字段变更的场景
- HBase是NoSql数据库，数据的查询只能通过RowKey 进行检索，不能通过SQL语句查询
- 通过哪些字段可以唯一确定一条数据？ RowKey（行键）、Column Family(列族)、Column Qualifier（列限定符）、timestamp（数据写入HBase的时间）

### 表结构

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image_bc70bd2.png)

行键 Row Key：与nosql数据库一样,rowkey是用来检索记录的主键。访问hbase table中的行，只有三种方式：

- 通过单个row key访问
- 通过row key的range
- 全表扫描

- Row Key 行键可以是任意字符串(最大长度是 64KB，实际应用中长度一般为 10-100bytes)，在hbase内部，rowkey保存为字节数组。Hbase会对表中的数据按照rowkey排序(字典顺序);
- 存储时，数据按照Row key的字典序(byte order)排序存储。设计key时，要充分排序存储这个特性，将经常一起读取的行存储放到一起。(位置相关性)。
- 注意： 字典序对int排序的结果是 1,10,100,11,12,13,14,15,16,17,18,19,2,20,21 …
- 要保持整形的自然序，行键必须用0作左填充。
- 行的一次读写是原子操作 (不论一次读写多少列)。这个设计决策能够使用户很容易的理解程序在对同一个行进行并发更新操作时的行为。

列族 Column Family

- HBase表中的每个列，都归属于某个列族。列族是表的schema的一部分(而列不是)，必须在使用表之前定义。
- 列名都以列族作为前缀。例如 courses:history ， courses:math 都属于 courses 这个列族。
- 访问控制、磁盘和内存的使用统计都是在列族层面进行的。列族越多，在取一行数据时所要参与IO、搜寻的文件就越多，所以，如果没有必要，不要设置太多的列族。

列 Column

- 列族下面的具体列，属于某一个ColumnFamily，类似于在mysql当中创建的具体的列。

时间戳 Timestamp

- HBase中通过row和columns确定的为一个存贮单元称为cell。每个 cell都保存着同一份数据的多个版本。版本通过时间戳来索引。时间戳的类型是 64位整型。时间戳可以由hbase(在数据写入时自动)赋值，此时时间戳是精确到毫秒的当前系统时间。时间戳也可以由客户显式赋值。如果应用程序要避免数据版本冲突，就必须自己生成具有唯一性的时间戳。每个cell中，不同版本的数据按照时间倒序排序，即最新的数据排在最前面。
- 为了避免数据存在过多版本造成的的管理 (包括存贮和索引)负担，hbase提供了两种数据版本回收方式：
    - 保存数据的最后n个版本
    - 保存最近一段时间内的版本（设置数据的生命周期TTL）。
    - 用户可以针对每个列族进行设置。

Cell

- 单元 Cell 由{row key, column( = + ), version} 唯一确定的单元。cell中的数据是没有类型的，全部是字节码形式存贮。

版本号 VersionNum

- 数据的版本号，每条数据可以有多个版本号，默认值为系统时间戳，类型为Long。

### 物理存储

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241226100535389.png)



- Table 中的所有行都按照 Row Key 的字典序排列。
- Table 在**行的方向**上分割为多个 HRegion，随着表行数增多，一个表会按照行分裂为多个Region;
- HRegion按大小分割的(默认10G)，每个表一开始只有一 个HRegion，随着数据不断插入表，HRegion不断增大，当增大到一个阀值的时候，HRegion就会等分会两个新的HRegion。当Table 中的行不断增多，就会有越来越多的 HRegion。
- HRegion 是 HBase 中分布式存储和负载均衡的最小单元。最小单元就表示不同的 HRegion 可以分布在不同的 HRegion Server 上。但一个 HRegion 是不会拆分到多个 Server 上的。
- HRegion 虽然是负载均衡的最小单元，但并不是物理存储的最小单元。事实上，HRegion 由一个或者多个 Store 组成，每个 Store 保存一个 Column Family。每个 Strore 又由一个 MemStore 和0至多个 StoreFile 组成。如上图。

**文件层级结构**

- client
- ZK
- HMaster
- HRegionServer
    - HRegion=>一个table有多个HRegion,一个HRegion默认为10g
        - HStore=>对应一个列族，一个HRegion有多个HStore
            - MemStore(内存)=>Flush到StoreFile
            - StoreFile(磁盘)
                - HFile(默认128g)=>hdfs客户端，写入到hdfs中
    - HLog(容错机制，副本机制，保证数据不丢失)

### HLog(WAL log)

- WAL 意为Write ahead log，类似 mysql 中的 binlog,用来做灾难恢复时用，Hlog记录数据的所有变更,一旦数据修改，就可以从log中进行恢复。
- 每个Region Server维护一个Hlog,而不是每个Region一个。这样不同region(来自不同table)的日志会混在一起，这样做的目的是不断追加单个文件相对于同时写多个文件而言，可以减少磁盘寻址次数，因此可以提高对table的写性能。带来的麻烦是，如果一台region server下线，为了恢复其上的region，需要将region server上的log进行拆分，然后分发到其它region server上进行恢复。
- HLog文件就是一个普通的Hadoop Sequence File：
- HLog Sequence File 的Key是HLogKey对象，HLogKey中记录了写入数据的归属信息，除了table和region名字外，同时还包括 sequence number和timestamp，timestamp是”写入时间”，sequence number的起始值为0，或者是最近一次存入文件系统中sequence number。
- HLog Sequece File的Value是HBase的KeyValue对象，即对应HFile中的KeyValue

## HBase架构

HBase中核心组件：Client、Zookeeper、HMaster、RegionServer、Region、Store、HLog、HDFS等部件组成。

主要职责简单总结：

- Client：利用 RPC机制与 HMaster 和HRegionServer通信；
- Zookeeper： 协调，避免 HMaster 单点问题；HMaster没有单点问题，HBase 中可以启动多个HMaster，通过 ZooKeeper 的 Master Election 机制保证总有一个 Master 在运行。
- HMaster：负责 Table 和 Region 的管理工作；
- HRegionServer：HBase 最核心模块，响应用户IO请求，向 HDFS 中读写数据；

### 整体架构

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241226093236979.png)

- Client：包含访问hbase的接口，Client维护着一些cache来加快对hbase的访问，比如regione的位置信息.
- Zookeeper：HBase可以使用内置的Zookeeper，也可以使用外置的，在实际生产环境，为了保持统一性，一般使用外置Zookeeper。
- Zookeeper在HBase中的作用：
    - 保证任何时候，集群中只有一个master
    - 存贮所有Region的寻址入口
    - 实时监控Region Server的状态，**将Region Server的上线和下线信息实时通知给Master**
- HMaster
    - 为Region Server分配Region
    - 负责Region Rerver的负载均衡，调整Region 分布；
    - 发现失效的Region Server并重新分配其上的Region；
    - HDFS上的垃圾文件回收
    - 处理schema更新请求
    - 管理用户对 Table 的 CRUD 操作；
    - 在 RegionSplit 后，负责新Region 分配；
- HRegion Server
    - HRegion Server维护HMaster分配给它的Region，处理对这些Region的IO请求；
    - HRegion Server负责切分在运行过程中变得过大的region
    - 从图中可以看到，Client访问HBase上数据的过程并不需要HMaster参与（寻址访问Zookeeper和HRegion server，数据读写访问HRegione server）

> **HMaster仅仅维护者table和HRegion的元数据信息，负载很低。**
>
> Master：主要进程，具体实现类为HMaster,通常部署在namenode上，负责通过zk监控RegionServer进程状态，是同步所有元数据变化的接口，内部启动监控执行region的故障转移和拆分线程；
>
> RegionServer:具体实现类为HRegionServer，部署在datanode上，主要负责cell的处理，同事在执行区域拆分和合并的时候，由RegionServer来实际执行；

数据存储方式：

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241226094214056.png)

### HMaster架构

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241226094407772.png)

实现类为 HMaster，通常部署在namenode上，负责监控集群中所有的 RegionServer 实例。主要作用如下：

（1）管理元数据表格 hbase:meta，接收用户对表格创建修改删除的命令并执行

（2）监控 region 是否需要进行负载均衡，故障转移和 region 的拆分。

通过启动多个后台线程监控实现上述功能：

- LoadBalancer 负载均衡器：
    - 周期性监控 region 分布在 regionServer 上面是否均衡，由参数 hbase.balancer.period控制周期时间，默认 5 分钟。

- CatalogJanitor 元数据管理器：
    - 定期检查和清理 hbase:meta 中的数据

- MasterProcWAL master 预写日志处理器：
    - 把 master 需要执行的任务记录到预写日志 WAL 中，如果 master 宕机，让 backupMaster（高可用）读取日志继续工作



> 什么时候客户端连接master？
>
> 在客户端对元数据进行操作的时候才会连接 master，如果对数据进行读写，直接连接zookeeper读取目录/hbase/meta-region-server 节点信息，会记录 meta 表格的位置。直接读取即可，不需要访问 master，这样可以减轻 master 的压力，相当于 master 专注 meta 表的写操作，客户端可直接读取 meta 表

### Region Server架构

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241226094727378.png)

Region Server 实现类为 HRegionServer，通常部署在datanode上，主要作用如下:

（1）负责数据 cell 的处理，例如写入数据 put，查询数据 get 等

（2）拆分合并 region 的实际执行者，由 master 监控，由 regionServer 执行

HRegionServer 内部管理了一系列 HRegion对象，每个 HRegion 对应 Table 中的一个Region(一个table可以分裂为多个Region)，HRegion 由多个 HStore 组成，每个 HStore 对应 Table 中的一个 Column Familiy 的存储。

HStore 是 HBase 存储的核心，其中由两部分构成，一部分是 MemStore，一部分是 StoreFile。StoreFile 文件数量增长到一定阈值后，会触发 Compact合并操作，将多个StoreFile 合并成一个 StoreFile，合并过程中会进行版本合并和数据删除。StoreFile 在完成 Compact 合并操作后，会逐步形成越来越大的 StoreFile，当单个StoreFile 大小超过一定阈值后，会触发 Split 操作，同时把当前Region 分裂成2个Region，父Region 会下线，新分裂出的2个孩子Region 会被 HMaster 分配到相应的 HRegionServer 上，使得原先1个Region 压力得以分流到2个Region 上。

> 如何理解Region？
>
> Region是HBase中数据管理的基本单位，每个Region由其所属的表、第一行和最后一行组成，每个Region都有一个唯一的RegionID来标识；Region代表特定rowkey区间内的数据片段，每个Region存储着1到多个存储Store，每个Store对应Table中的一个ColumnFamily(即每一个Region包含所有行)，并且每个Store中包含一个MemStore的写缓存
>
> MemStore 写入缓存，用于存储尚未写入磁盘的新数据。一个区域中的每个列族都有一个MemStore。减少磁盘浪费

### Table 与Region

> Region是 HBase集群分布数据的最小单位。

Region 是部分数据，所以是所有数据的一个子集，但Region包括完整的行，所以Region 是行为单位表的一个子集。

每个Region 有三个主要要素:

- 它所属于哪张表
- 它所包含的的第一行(第一个Region 没有首行)
- 他所包含的最后一行(末一个Region 没有末行)

当表初写数据时，此时表只有一个Region，当随着数据的增多，Region 开始变大，等到它达到限定的阀值大小时，便会把Region 分裂为两个大小基本相同的Region，而这个阀值就是StoreFile 的设定大小(参数:hbase.hRegion.max.filesize 新版本默认10G) ,在第一次分裂Region之前，所有加载的数据都放在原始区域的那台服务器上，随着表的变大，Region 的个数也会相应的增加，而Region 是HBase集群分布数据的最小单位。

(但Region 也是由block组成，Region是属于单一的RegionServer，除非这个RegionServer 宕机，或者其它方式挂掉，再或者执行balance时，才可能会将这部分Region的信息转移到其它机器上。)

这也就是为什么 Region比较少的时候，导致Region 分配不均，总是分派到少数的节点上，读写并发效果不显著，也即HBase 读写效率比较低的原因。

### 元数据表 .META.和 -ROOT-

HBase内部维护着两个元数据表，分别是-ROOT- 和 .META. 表。他们分别维护者当前集群所有Region 的列表、状态和位置。

(1) .META. 记录用户表的 Region 信息，可以有多个 Region，.META.会随需要被Split。

(2) -ROOT- 记录 .META. 表的 Region 信息，只有一个 Region，-ROOT-永不会被Split。

(3) ZooKeeper 中记录 -ROOT- 表的 Location，.META. 和 -ROOT- 的关系见下图。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/image-20241226141922815.png)

-ROOT- 表包含.META. 表的Region 列表，因为.META. 表可能会因为超过Region 的大小而进行分裂，所以-ROOT-才会保存.META.表的Region索引，-ROOT-表是不会分裂的。而.META. 表中则包含所有用户Region（user-space Region）的列表。表中的项使用Region 名作为键。Region 名由所属的表名、Region 的起始行、创建的时间以及对其整体进行MD5 hash值。

### Region定位流程

算法：B+树定位，通过ZooKeeper 来查找 -ROOT-，然后是.META.，然后找到Table里的Region。

Client 访问用户数据之前需要访问 ZooKeeper，然后访问 -ROOT- 表，接着访问 .META. 表，最后才能找到用户数据的位置去访问。中间需要多次网络操作，不过 Client 端会执行 Cache 缓存。

(1) 客户端client首先连接到ZooKeeper这是就要先查找-ROOT-的位置。

(2) 然后client通过-ROOT-获取所请求行所在范围所属的.META.Region的位置。

(3) client接着查找.META.Region来获取user-space Region所在的节点和位置。

(4) 接着client就可以直接和管理者那个Region的RegionServer进行交互。

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/9a5de9469e88b49976ba8dec6aaddbf8.jpeg)

每个行操作可能要访问三次远程节点，为了节省这些代价，client会缓存他们遍历-ROOT-和.META. 的位置以及user-space Region的开始行和结束行，这样每次访问就不会再从表中去查询了，但如果变动了怎么办？却是存在这个问题，这样的话client 会出现错误，那此时Region毫无疑问是移动了，这时，client 会再次从.META.查找Region 的新位置并再次将其放入到缓存中去，周而复始。同样道理如果.META.的Region移动了，client 也才会去-ROOT-表查询.META.Region的新位置。

HMaster是Hbase主/从集群架构中的中央节点。通常一个HBase集群存在多个HMaster节点,其中一个为Active Master,其余为Backup Master；

Hbase每时每刻只有一个hmaster主服务器程序在运行，hmaster将region分配给region服务器，协调region服务器的负载并维护集群的状态。Hmaster不会对外提供数据服务，而是由region服务器负责所有regions的读写请求及操作。

由于hmaster只维护表和region的元数据，而不参与数据的输入/输出过程，hmaster失效仅仅会导致所有的元数据无法被修改，但表的数据读/写还是可以正常进行的。

HMaster的作用：

- 为Region server分配region
- 负责Region server的负载均衡
- 发现失效的Region server并重新分配其上的region
- HDFS上的垃圾文件回收
- 处理schema更新请求


HRegionServer作用：

- 维护master分配给他的region，处理对这些region的io请求
- 负责切分正在运行过程中变的过大的region


可以看到，client访问hbase上的数据并不需要master参与（寻址访问zookeeper和region server，数据读写访问region server），master仅仅维护table和region的元数据信息（table的元数据信息保存在zookeeper上），负载很低。

注意：master上存放的元数据是region的存储位置信息，但是在用户读写数据时，都是先写到region server的WAL日志中，之后由region server负责将其刷新到HFile中，即region中。所以，用户并不直接接触region，无需知道region的位置，所以其并不从master处获得什么位置元数据，而只需要从zookeeper中获取region server的位置元数据，之后便直接和region server通信。

HRegionServer存取一个子表时，会创建一个HRegion对象，然后对表的每个列族创建一个Store实例，每个Store都会有一个MemStore和0个或多个StoreFile与之对应，每个StoreFile都会对应一个HFile， HFile就是实际的存储文件。因此，一个HRegion有多少个列族就有多少个Store。

一个HRegionServer会有多个HRegion和一个HLog。

当HRegionServer意外终止后，HMaster会通过Zookeeper感知到。

Zookeeper作用在于：

1. hbase regionserver 向zookeeper注册，提供hbase regionserver状态信息（是否在线）。
2. hmaster启动时候会将hbase系统表-ROOT- 加载到 zookeeper cluster，通过zookeeper cluster可以获取当前系统表.META.的存储所对应的regionserver信息。


zookeeper是hbase集群的"协调器"。由于zookeeper的轻量级特性，因此我们可以将多个hbase集群共用一个zookeeper集群，以节约大量的服务器。多个hbase集群共用zookeeper集群的方法是使用同一组ip，修改不同hbase集群的"zookeeper.znode.parent"属性，让它们使用不同的根目录。比如cluster1使用/hbase-c1,cluster2使用/hbase-c2，等等。

## HBase写流程

### 写流程

**总体流程**

1. Client 先访问 zookeeper，获取 hbase:meta 表位于哪个 Region Server。
2. 访问对应的 Region Server，获取 hbase:meta 表，根据读请求的 namespace:table/rowkey，查询出目标数据位于哪个 Region Server 中的哪个 Region 中。并将该 table 的 region 信息以 及 meta 表的位置信息缓存在客户端的 meta cache，方便下次访问。
3. 与目标 Region Server 进行通讯；
4. 将数据顺序写入（追加）到 WAL；
5. 将数据写入对应的 MemStore，数据会在 MemStore 进行排序；
6. 向客户端发送 ack；
7. 等达到 MemStore 的刷写时机后，将数据刷写到 HFile。

**详细流程**

1. HBase使用MemStore和StoreFile存储对表的更新。
2. 数据在更新时首先写入Log(WAL log)和内存(MemStore)中，MemStore中的数据是排序的，当MemStore累计到一定阈值时，就会创建一个新的MemStore，并且将老的MemStore添加到flush队列，由单独的线程flush到磁盘上，成为一个StoreFile。于此同时，系统会在zookeeper中记录一个redo point，表示这个时刻之前的变更已经持久化了。
3. 当系统出现意外时，可能导致内存(MemStore)中的数据丢失，此时使用Log(WAL log)来恢复checkpoint之后的数据。
4. StoreFile是只读的，一旦创建后就不可以再修改。因此HBase的更新其实是不断追加的操作。当一个Store中的StoreFile达到一定的阈值后，就会进行一次合并(minor_compact, major_compact),将对同一个key的修改合并到一起，形成一个大的StoreFile，当StoreFile的大小达到一定阈值后，又会对 StoreFile进行split，等分为两个StoreFile。
5. 由于对表的更新是不断追加的，compact时，**需要访问Store中全部的 StoreFile和MemStore，**将他们按row key进行合并，由于StoreFile和MemStore都是经过排序的，并且StoreFile带有内存中索引，合并的过程还是比较快。
6. Hbase写数据是先将数据写入wal(预写日志),再写入内存。在代码实现中，首先要获得锁，（因为要写数据）。然后写入日志文件。但并不同步（也就是不写入hdfs中）。再写入内存，之后进行释放锁（写完数据了）。之后同步日志文件（写入hdfs中）。如果wal同步失败了，就进行回滚操作单元格，将keys从内存中移除。所以最后要判断是否日志文件写入成功，如果成功，内存中数据就保留；如果不成功，内存中数据就删除。保证了数据一致性。

总结：用户向Region Server发出请求，Region Server响应， 先安排数据存入Memstore，并备份日志到WAL，当Memstore数据量到一定数量后将创建新的HFile，然后将数据转移至Hfile写入磁盘。写入完毕（如果不用Memstore,直接写HFile将浪费磁盘空间并且无法更新，如果没有WAL，在数据写入Memstore时如果Region Server崩溃，则这部分写入缓存数据将丢失）。

> 重要信息
>
> - Hbase表非常稀疏，但是不占存储空间。
> - DDL：是表的增删改查:describe
> - DML：是数据的增删改查：put(增，改）,scan，get,delete
> - HMaster将与客户端交互的部分交给了ZK，客户端对标数据的增改不需要访问Hmaster，只需要请求ZK
> - 一个regionServer存在多个region，一个region是一个切片，包括多个列族，也就是多个store，一个store中保存一个memstore。一个列族对应多个store。但一个region中肯定是不同的列族

### 刷写flush

memstore memstore向hdfs中保存数据。两个标注：

- 时间
- 大小
- 预刷写

1. 按时间

（hbase.regionserver.optionalcacheflushinterval）默认：1h，内存中的文件在自动刷新之前能够存活的最长时间

```java
<!-- 内存中的文件在自动刷新之前能够存活的最长时间，默认是1h -->  
    <property>  
        <name>hbase.regionserver.optionalcacheflushinterval</name>  
        <value>3600000</value>  
        <description>  
            Maximum amount of time an edit lives in memory before being automatically  
            flushed.  
            Default 1 hour. Set it to 0 to disable automatic flushing.  
        </description>  
    </property>
```

2. 按大小

一种是regionserver范围内的size；一种是region范围内的size。在regionserver中求的是所有region的memstore的总和hbase.regionserver.global.memstore.size，如果大于限定值，那么所有region都需要执行刷写操作，并且为了保护内存安全，在刷写操作过程中需要阻塞客户端读写（所有region中的memstore)，防止客户端操作比刷写的操作还快，容易造成堆溢出。注意默认为*Defaults to 40% of heap (0.4).。刷写过程是按照memstore从大到小的顺序进行刷写的。

```java
<!-- regionServer的全局memstore的大小，超过该大小会触发flush到磁盘的操作,默认是堆大小的40%,而且regionserver级别的   
        flush会阻塞客户端读写 -->  
    <property>  
        <name>hbase.regionserver.global.memstore.size</name>  
        <value></value>  
        <description>Maximum size of all memstores in a region server before  
            new  
            updates are blocked and flushes are forced. **Defaults to 40% of heap (0.4).  **
            Updates are blocked and flushes are forced until size of all  
            memstores  
            in a region server hits  
            hbase.regionserver.global.memstore.size.lower.limit.  
            The default value in this configuration has been intentionally left  
            emtpy in order to  
            honor the old hbase.regionserver.global.memstore.upperLimit property if  
            present.  
        </description>  
    </property>
```

（hbase.regionserver.global.memstore.size.lower.limit）默认：堆大小 * 0.4 * 0.95
有时候集群的“写负载”非常高，写入量一直超过flush的量，这时，我们就希望memstore不要超过一定的安全设置。在这种情况下，写操作就要被阻塞一直到memstore恢复到一个“可管理”的大小, 这个大小就是默认值是堆大小 * 0.4 * 0.95，*也就是当regionserver级别的flush操作发送后,会阻塞客户端写,一直阻塞到整个regionserver级别的memstore的大小为 堆大小 * 0.4 0.95为`止

```java
<!--可以理解为一个安全的设置，有时候集群的“写负载”非常高，写入量一直超过flush的量，这时，我们就希望memstore不要超过一定的安全设置。   
        在这种情况下，写操作就要被阻塞一直到memstore恢复到一个“可管理”的大小, 这个大小就是默认值是堆大小 * 0.4 * 0.95，也就是当regionserver级别   
        的flush操作发送后,会阻塞客户端写,一直阻塞到整个regionserver级别的memstore的大小为 堆大小 * 0.4 *0.95为止 -->  
    <property>  
        <name>hbase.regionserver.global.memstore.size.lower.limit</name>  
        <value></value>  
        <description>Maximum size of all memstores in a region server before  
            flushes are forced.  
            Defaults to 95% of hbase.regionserver.global.memstore.size (0.95).  
            A 100% value for this value causes the minimum possible flushing to  
            occur when updates are  
            blocked due to memstore limiting.  
            The default value in this configuration has been intentionally left  
            emtpy in order to  
            honor the old hbase.regionserver.global.memstore.lowerLimit property if  
            present.  
        </description>  
    </property>  
```

按大小2：（hbase.hregion.memstore.flush.size）默认：128M。单个region里memstore的缓存大小，超过那么整个HRegion就会flush。但只会阻止向该memstore的读写操作
**预刷写**

(hbase.hregion.preclose.flush.size）默认为：5M

- 当一个 region 中的 memstore 的大小大于这个值的时候，我们又触发了region的 close时，会先运行“pre-flush”操作，清理这个需要关闭的memstore，然后 将这个 region 下线。
- 当一个 region下线了，我们无法再进行任何写操作。如果一个 memstore 很大的时候，flush 操作会消耗很多时间。"pre-flush"操作意味着在 region 下线之前，会先把 memstore 清空。这样在最终执行 close 操作的时候，flush 操作会很快。

## HBase读流程

- LRU:最近最少使用，Block Cache是块缓存，因为要存最近最常用的数据。删除数据的话就遵循LRU原则，删除最近最少使用的数据。
- 因为数据的顺序是按照时间戳控制的，所以需要比较磁盘和内存中的数据。所以不是先访问内存，而是两个要一起读，要比较时间戳。从磁盘的数据放在cache，节省时间。以后blockcache有就不读磁盘里相应的文件了。因为磁盘里的其他文件也可能包含当前读取对象。
- Hbase是读比写慢的，因为无论如何都需要读磁盘。
- block cache只存放磁盘里的文件

用户向Region Server发出请求，Region Server 去 BlockCache里查看最近频繁查看的数据有没有用户需要的，如果没有再去Hfile里查询，Region Server 返回用户响应，用户拿到查询数据。（因为去Hfile所在磁盘里查询需要花费不少时间，查询BlockCache时间少，所以BlockCache大大减少了延迟。）

## StoreFile Compaction（小文件合并）

由于memstore每次刷写都会生成一个新的HFile，且同一个字段的不同版本（timestamp） 和不同类型（Put/Delete）有可能会分布在不同的 HFile 中，因此查询时需要遍历所有的 HFile。为了减少 HFile 的个数，以及清理掉过期和删除的数据，会进行 StoreFile Compaction。 Compaction 分为两种，分别是 Minor Compaction（小合并） 和 Major Compaction（大合并）。Minor Compaction 会将临近的若干个较小的 HFile合并成一个较大的 HFile，但不会清理过期和删除的数据。 Major Compaction 会将一个 Store 下的所有的 HFile合并成一个大 HFile，并且会清理掉过期 和删除的数据。

> **当文件数超过三个的时候，走的也是大合并**
>  一个region进行 major compaction合并的周期,在这个点的时候， 这个region下的所有hfile会进行合并,默认是7天,**major compaction非常耗资源,建议生产关闭(设置为0)，在应用空闲时间手动触发**

```java
 <!-- 一个region进行 major compaction合并的周期,在这个点的时候， 这个region下的所有hfile会进行合并,默认是7天,major   
        compaction非常耗资源,建议生产关闭(设置为0)，在应用空闲时间手动触发 -->  
    <property>  
        <name>hbase.hregion.majorcompaction</name>  
        <value>604800000</value>  
        <description>The time (in miliseconds) between 'major' compactions of  
            all  
            HStoreFiles in a region. Default: Set to 7 days. Major compactions tend to  
            happen exactly when you need them least so enable them such that they  
            run at  
            off-peak for your deploy; or, since this setting is on a periodicity that is  
            unlikely to match your loading, run the compactions via an external  
            invocation out of a cron job or some such.  
        </description>  
    </property>  
```

一个store里面允许存的hfile的个数，超过这个个数会被写到新的一个hfile里面 也即是每个region的每个列族对应的memstore在fulsh为hfile的时候，默认情况下当达到3个hfile的时候就会对这些文件进行合并重写为一个新文件，设置个数越大可以减少触发合并的时间，但是每次合并的时间就会越长（hbase.hstore.compactionThreshold）

```java
    <!-- 一个store里面允许存的hfile的个数，超过这个个数会被写到新的一个hfile里面 也即是每个region的每个列族对应的memstore在fulsh为hfile的时候，默认情况下当达到3个hfile的时候就会对这些文件进行合并重写为一个新文件，设置个数越大可以减少触发合并的时间，但是每次合并的时间就会越长 -->  
    <property>  
        <name>hbase.hstore.compactionThreshold</name>  
        <value>3</value>  
        <description>  
            If more than this number of HStoreFiles in any one HStore  
            (one HStoreFile is written per flush of memstore) then a compaction  
            is run to rewrite all HStoreFiles files as one. Larger numbers  
            put off compaction but when it runs, it takes longer to complete.  
        </description>  
    </property>  
```

## 数据的删除

正常的数据读写操作不需要访问master。但元数据还需要查看master。但也是从ZK找master，master节点是高可用的，位置会发生变化，也是通过ZK进行管理。

**删除无用数据**

删除数据除了在大合并storeFile时，还有一种情况是flush，也就是在内存中的数据会进行合并。例如同一rowkey首先put 1，再put 2，然后flush进行刷写。再查所有版本的信息会只存在2,1的插入信息根本就不会刷写进磁盘。但注意这是因为在同一内存中的操作进行合并，只关心内存空间，不关心磁盘空间。

那操作标记怎么删除呢？

在flush的时候不删，在compact（major)的时候才会删除。因为flush时候是只关心内存空间的，但在磁盘中会存在多个文件，可能当前操作对应的数据还存在其他文件中，会造成数据混乱。例如向10001里put数据1，然后flush，此时硬盘上会出现一个文件。再向10001里put数据2，然后再删除10002的该数据。此时扫面版本是只存在delete操作，进行flush后，写入磁盘文件，该文件就包含delete操作。因为flush里不能删除，该操作对应的数据也存在第一次刷写的硬盘中。若删除了这个数据，那么在之后compact(major)时，10001为1的数据又会出现了。（例子进行简化了，方便解释）

## Region Split

默认情况下，每个 Table 起初只有一个 Region，随着数据的不断写入，Region 会自动进行拆分。刚拆分时，两个子 Region 都位于当前的 Region Server，但处于负载均衡的考虑，HMaster 有可能会将某个 Region 转移给其他的 RegionServer。

Region Split 时机：

- 当1个region中的某个Store下所有StoreFile的总大小超过hbase.hregion.max.filesize， 该 Region 就会进行拆分（0.94 版本之前）。
- 当 1 个 region 中 的 某 个 Store 下所有 StoreFile 的 总 大 小 超 过 Min(R^2 * “hbase.hregion.memstore.flush.size”,hbase.hregion.max.filesize")，该Region 就会进行拆分，其中 R 为当前 Region Server 中属于该 Table 的个数（0.94 版本之后）。但这样就会造成数据倾斜，每个region的负载不一样，会造成负载不均衡。

## HRegion管理

任何时刻，一个HRegion只能分配给一个HRegion Server。HMaster记录了当前有哪些可用的HRegion Server。以及当前哪些HRegion分配给了哪些HRegion Server，哪些HRegion还没有分配。当需要分配的新的HRegion，并且有一个HRegion Server上有可用空间时，HMaster就给这个HRegion Server发送一个装载请求，把HRegion分配给这个HRegion Server。HRegion Server得到请求后，就开始对此HRegion提供服务。

HRegion Server上线

HMaster使用zookeeper来跟踪HRegion Server状态。当某个HRegion Server启动时，会首先在zookeeper上的server目录下建立代表自己的znode。由于HMaster订阅了server目录上的变更消息，当server目录下的文件出现新增或删除操作时，HMaster可以得到来自zookeeper的实时通知。因此一旦HRegion Server上线，HMaster能马上得到消息。

HRegion Server下线

当HRegion Server下线时，它和zookeeper的会话断开，zookeeper而自动释放代表这台server的文件上的独占锁。HMaster就可以确定：HRegion Server和zookeeper之间的网络断开了。

HRegion Server挂了。

无论哪种情况，HRegion Server都无法继续为它的HRegion提供服务了，此时HMaster会删除server目录下代表这台HRegion Server的znode数据，并将这台HRegion Server的HRegion分配给其它还活着的节点。

## HMaster工作机制

master上线

master启动进行以下步骤:

- 从zookeeper上获取唯一一个代表active master的锁，用来阻止其它HMaster成为master。
- 扫描zookeeper上的server父节点，获得当前可用的HRegion Server列表。
- 和每个HRegion Server通信，获得当前已分配的HRegion和HRegion Server的对应关系。
- 扫描.META.region的集合，计算得到当前还未分配的HRegion，将他们放入待分配HRegion列表。

master下线

由于HMaster只维护表和region的元数据，而不参与表数据IO的过程，HMaster下线仅导致所有元数据的修改被冻结(无法创建删除表，无法修改表的schema，无法进行HRegion的负载均衡，无法处理HRegion 上下线，无法进行HRegion的合并，唯一例外的是HRegion的split可以正常进行，因为只有HRegion Server参与)，表的数据读写还可以正常进行。因此HMaster下线短时间内对整个HBase集群没有影响。
从上线过程可以看到，HMaster保存的信息全是可以冗余信息（都可以从系统其它地方收集到或者计算出来）因此，一般HBase集群中总是有一个HMaster在提供服务，还有一个以上的‘HMaster’在等待时机抢占它的位置。

## RowKey 设计

    一条数据的唯一标识就是 RowKey，那么这条数据存储于哪个分区，取决于 RowKey 处 于哪个一个预分区的区间内，设计 RowKey 的主要目的 ，就是让数据均匀的分布于所有的 region 中，在一定程度上防止数据倾斜。接下来我们就谈一谈 RowKey 常用的设计方案。
    三个原则:
    
        散列性
        唯一性
        长度原则
    
    1．生成随机数、hash、散列值

## 内存优化

HBase 操作过程中需要大量的内存开销，毕竟 Table 是可以缓存在内存中的，一般会分 配整个可用内存的 70%给 HBase 的 Java 堆。但是不建议分配非常大的堆内存，因为 GC 过 程持续太久会导致 RegionServer 处于长期不可用状态，一般 16~48G 内存就可以了，如果因为框架占用内存过高导致系统内存不足，框架一样会被系统服务拖死。

因为内存有个RegionServer 级别的刷写，限制值是堆大小 * 0.4 。如果它比较大，那么在进行阻塞时，会一直把读写阻塞到低于堆大小0.40.38。如果堆内存比较大，那么这个值也会比较大，那么阻塞的时间就会比较久。

### 允许在 HDFS 的文件中追加内容

```text
hdfs-site.xml、hbase-site.xml
属性：dfs.support.append
```

解释：开启 HDFS 追加同步，可以优秀的配合 HBase 的数据同步和持久化。默认值为 true。

### 优化 DataNode 允许的最大文件打开数

```test
hdfs-site.xml
属性：dfs.datanode.max.transfer.threads
```

解释：HBase 一般都会同一时间操作大量的文件，根据集群的数量和规模以及数据动作，设置为 4096 或者更高。默认值：4096。因为有RegionServer 级别的刷写，同时会刷写多个文件。

### 优化延迟高的数据操作的等待时间

```test
hdfs-site.xml
属性：dfs.image.transfer.timeout
```

解释：如果对于某一次数据操作来讲，延迟非常高，socket 需要等待更长的时间，建议把该值设置为更大的值（默认 60000 毫秒），以确保 socket 不会被 timeout 掉。为了确保有些进行慢的任务也能正常进行（也即是延迟高）

### 优化数据的写入效率

```java
mapred-site.xml
属性：
mapreduce.map.output.compress
mapreduce.map.output.compress.codec
```

解释：开启这两个数据可以大大提高文件的写入效率，减少写入时间。第一个属性值修改为true，第二个属性值修改为：org.apache.hadoop.io.compress.GzipCodec 或者其他压缩方式。通过压缩文件来提高存储效率例如storeFile就是以HFile的文件格式进行存储的，可以进行文件压缩。

### 设置 RPC 监听数量

```test
hbase-site.xml
属性：Hbase.regionserver.handler.count
```

解释：默认值为 30，用于指定 RPC 监听的数量，可以根据客户端的请求数进行调整，读写请求较多时，增加此值。（客户端和服务端之间的通信使用的是RPC）

### 优化 HStore 文件大小

```java
hbase-site.xml
属性：hbase.hregion.max.filesize
```

解释：默认值 10737418240（10GB），如果需要运行 HBase 的 MR 任务，可以减小此值，因为一个 region 对应一个 map 任务，如果单个 region 过大，会导致 map 任务执行时间过长。该值的意思就是，如果 HFile 的大小达到这个数值，则这个 region 会被切分为两个 Hfile。

### 优化 HBase 客户端缓存

```java
hbase-site.xml
属性：hbase.client.write.buffer
```

解释：用于指定 Hbase 客户端缓存，增大该值可以减少 RPC 调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到减少 RPC 次数的目的。

### 指定 scan.next 扫描 HBase 所获取的行数

```java
hbase-site.xml
属性：hbase.client.scanner.caching
```

解释：用于指定 scan.next 方法获取的默认行数，值越大，消耗内存越大。
上面两个都是客户端的优化。

### flush、compact、split 机制

当 MemStore 达到阈值，将 Memstore 中的数据 Flush 进 Storefile；compact 机制则是把 flush出来的小文件合并成大的 Storefile 文件。split 则是当 Region 达到阈值，会把过大的 Region一分为二。



## HBase为什么查询效率高

### 实时查询

实时查询，可以认为是从内存中查询，一般响应时间在1秒内。HBase的机制是数据先写入到内存中，当数据量达到一定的量（如128M），再写入磁盘中， 在内存中，是不进行数据的更新或合并操作的，只增加数据，这使得用户的写操作只要进入内存中就可以立即返回，保证了HBase I/O的高性能。

实时查询，即反应根据当前时间的数据，可以认为这些数据始终是在内存的，保证了数据的实时响应。

### Hbase数据查询过程

第1步：

项目有100亿业务数据，存储在一个Hbase集群上（由多个服务器数据节点构成），每个数据节点上有若干个Region（区域），每个Region实际上就是Hbase中一批数据的集合（比如20万条数据）。

我们现在开始根据主键RowKey来查询对应的记录，Hbase的Master帮我们迅速定位到该记录所在的数据节点，以及数据节点中的Region，目前我们有100亿条记录，占空间10TB。所有记录被切分成5000个Region，那么现在，每个Region就是2G。

由于记录在1个Region中，所以现在我们只要查询这2G的记录文件，就能找到对应记录。

##### 第2步：

由于Hbase存储数据是按照列族存储的。比如一条记录有300个字段，前100个字段是人员信息相关，这是一个列簇（列的集合）；中间100个字段是公司信息相关，是一个列簇。最后100个字段是人员交易信息相关，也是一个列簇。

这三个列簇是分开存储的。这样的存储结构就保证了Hbase可支持的表的宽度（字段数）可达到百万个。

这时，假设2G的Region文件中，分为4个列族，那么每个列族就是500M。

到这里，我们只需要遍历这500M的列簇就可以找到对应的记录。

##### 第3步：

如果要查询的记录在其中1个列族上，1个列族在底层，包含1个或者多个HFile。

HFile可以理解为列簇底层更细粒度的存储文件。

如果一个HFile一般的大小为100M，那么该列族包含5个HFile在磁盘上或内存中。

由于Hbase的内存进而磁盘中的数据是排好序的，要查询的记录有可能在最前面，也有可能在最后面，按平均来算，我们只需遍历2.5个HFile共250M，即可找到对应的记录。

##### 第4步：

每个HFile中，是以键值对(key/value)方式存储，只要遍历文件中的key位置即可，并判断符合条件可以了。

一般key是有限的长度，假设key/value比是1:25，最终只需要10M的数据量，就可获取的对应的记录。

如果数据在机械磁盘上，按其访问速度100M/S，只需0.1秒即可查到。

如果是SSD的话，0.01秒即可查到。

当然，Hbase是有内存缓存机制的，如果数据在内存中，效率会更高。

正因为以上大致的查询思路，保证了Hbase即使随着数据量的剧增，也不会导致查询性能的下降。

同时，HBase是一个面向列存储的数据库（列簇机制），当表字段非常多时，可以把其中一些字段独立出来放在一部分机器上，而另外一些字段放到另一部分机器上，分散存储，分散列查询。

正由于这样复杂的存储结构和分布式的存储方式，保证了Hbase海量数据下的查询效率。