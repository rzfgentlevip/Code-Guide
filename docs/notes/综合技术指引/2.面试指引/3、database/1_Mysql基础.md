---
title: 1、Mysql手册(基础)
icon: devicon:mysql-wordmark
order: 1
author: bugcode
date: 2024-11-16T00:00:00.000Z
copyright: bugcode
createTime: 2026/01/17 13:16:50
permalink: /compre-guide/database/mysql基础/
---

## mysql架构

和其它数据库相比,MySQL有点与众不同,它的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎的架构上,**插件式的存储引擎架构将查询处理和其它的系统任务以及数据的存储提取相分离**。这种架构可以根据业务的需求和实际需要选择合适的存储引擎。

![](./image/mysql架构.png)

- **连接层**:最上层是一些客户端和连接服务。**主要完成一些类似于连接处理、授权认证、及相关的安全方案**。在该层上引入了线程池的概念,为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

- **服务层**:第二层服务层,主要完成大部分的核心服务功能, 包括**查询解析、分析、优化、缓存、以及所有的内置函数**,所有跨存储引擎的功能也都在这一层实现,包括触发器、存储过程、视图等

- **引擎层**:第三层存储引擎层,存储引擎真正的负责了MySQL中数据的**存储和提取**,服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同,这样我们可以根据自己的实际需要进行选取

- **存储层**:第四层为数据存储层,**主要是将数据存储在运行于该设备的文件系统之上,并完成与存储引擎的交互**

## MySQL 的查询流程具体是？or 一条SQL语句在MySQL中如何执行的？

客户端请求 ---> 连接器(验证用户身份,给予权限)  ---> 查询缓存(存在缓存则直接返回,不存在则执行后续操作) ---> 分析器(对SQL进行词法分析和语法分析操作)  ---> 优化器(主要对执行的sql优化选择最优的执行方案方法)  ---> 执行器(执行时会先看用户是否有执行权限,有才去使用这个引擎提供的接口) ---> 去引擎层获取数据返回(如果开启查询缓存则会缓存查询结果)

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/27/133029-35472.png)

## Mysql常用的存储引擎有哪些,他们都有什么特点？

存储引擎是MySQL的组件,用于处理不同表类型的SQL操作。不同的存储引擎提供不同的**存储机制、索引技巧、锁定水平**等功能,使用不同的存储引擎,还可以获得特定的功能。

使用哪一种引擎可以灵活选择,**一个数据库中多个表可以使用不同引擎以满足各种性能和实际需求**,使用合适的存储引擎,将会提高整个数据库的性能 。

MySQL服务器使用**可插拔**的存储引擎体系结构,可以从运行中的 MySQL 服务器加载或卸载存储引擎 。

目前通常使用的有两种存储引擎:

**InnoDB**

- InnoDB是MySQL的默认存储引擎,支持事务、行锁和外键等操作。

**MyISAM**

- MyISAM是MySQL5.1版本前的默认存储引擎,MyISAM的并发性比较差,不支持事务和外键等操作,默认的锁的粒度为表级锁。

### 查看存储引擎

~~~java
-- 查看支持的存储引擎
SHOW ENGINES

-- 查看默认存储引擎
SHOW VARIABLES LIKE 'storage_engine'

-- 查看具体某一个表所使用的存储引擎,这个默认存储引擎被修改了
show create table tablename

-- 准确查看某个数据库中的某一表所使用的存储引擎
show table status like 'tablename'
show table status from database where name="tablename"
~~~

### 设置存储引擎

我们在进一张表的时候,可以设置表使用不同的存储引擎。

~~~java
-- 建表时指定存储引擎。默认的就是INNODB,不需要设置
CREATE TABLE t1 (i INT) ENGINE = INNODB;
CREATE TABLE t2 (i INT) ENGINE = CSV;
CREATE TABLE t3 (i INT) ENGINE = MEMORY;

-- 修改存储引擎
ALTER TABLE t ENGINE = InnoDB;

-- 修改默认存储引擎,也可以在配置文件my.cnf中修改默认引擎
SET default_storage_engine=NDBCLUSTER;
~~~

默认情况下,每当 `CREATE TABLE` 或 `ALTER TABLE` 不能使用默认存储引擎时,都会生成一个警告。为了防止在所需的引擎不可用时出现令人困惑的意外行为,可以启用 `NO_ENGINE_SUBSTITUTION SQL` 模式。如果所需的引擎不可用,则此设置将产生错误而不是警告,并且不会创建或更改表

### 常见的存储引擎

常见的存储引擎就 InnoDB、MyISAM、Memory、NDB。

```mermaid
graph TD
    Client[客户端] --> Server[MySQL Server层<br/>连接/解析/优化/缓存]
    Server --> API[存储引擎API]
    
    subgraph Engines [存储引擎层]
        API --> InnoDB[InnoDB<br/>事务型]
        API --> MyISAM[MyISAM<br/>表级锁]
        API --> Memory[Memory<br/>内存表]
        API --> Archive[Archive<br/>压缩归档]
        API --> CSV[CSV<br/>文本存储]
        API --> BlackHole[BlackHole<br/>黑洞引擎]
    end
    
    InnoDB --> Disk1[.ibd数据文件]
    MyISAM --> Disk2[.MYD/.MYI]
    Memory --> RAM[内存]
    Archive --> Disk3[.ARZ压缩]
```

InnoDB 现在是 MySQL 默认的存储引擎,支持**事务、行级锁定和外键**

下面详细介绍一下核心存储引擎:

#### InnoDB（默认引擎，5.5+）

引擎架构特征

```mermaid
graph LR
    subgraph InnoDB_Arch [InnoDB架构]
        BP[缓冲池<br/>buffer pool] --> Data[数据页]
        BP --> Index[索引页]
        
        LG[日志缓冲<br/>log buffer] --> Redo[重做日志<br/>redo log]
        Redo --> WAL[WAL机制]
        
        TB[表空间] --> IBD[.ibd文件]
        IBD --> SDI[SDI<br/>元数据]
        
        Undo[回滚段<br/>undo log] --> MVCC[MVCC]
    end
```
核心特性：

✅ 事务支持（ACID）

✅ 行级锁（高并发）

✅ 外键约束

✅ 崩溃恢复（redo + undo）

✅ MVCC（多版本并发控制）

✅ 聚簇索引（数据即索引）

适用场景：

1. 金融、电商等事务系统 
2. 高并发读写应用 
3. 需要数据强一致性


#### MyISAM（5.5前默认）

架构特征:

```mermaid
graph LR
    subgraph MyISAM_Arch [MyISAM架构]
        FRM[.frm<br/>表结构] --> Meta[元数据]
        MYD[.MYD<br/>数据文件] --> DataRows[行数据]
        MYI[.MYI<br/>索引文件] --> IndexTree[B+树索引]
        
        Cache[键缓存<br/>key buffer] --> IndexTree
    end
```

核心特性：

✅ 表级锁（读写互斥）

✅ 全文索引（早期优势）

✅ 压缩表（只读场景）

✅ 顺序存储（插入快）

❌ 无事务

❌ 无崩溃恢复

❌ 无外键

适用场景：

1. 只读/读多写少 
2. 数据仓库、日志表 
3. 全文检索（5.6后InnoDB已支持）

#### Memory（HEAP）

架构特征:

```mermaid
graph LR
    subgraph Memory_Arch [Memory架构]
        RAM[内存] --> DataHash[哈希索引]
        RAM --> DataBTree[B+树索引]
        RAM --> RowData[行数据]
        
        Disk[磁盘] --> FRM2[.frm结构定义]
    end
```

核心特性：

✅ 全内存存储（极速）

✅ 哈希索引（默认）

✅ 表级锁

❌ 重启数据丢失

❌ 不支持BLOB/TEXT

❌ 单表大小受限（max_heap_table_size）

适用场景：

1. 临时表、缓存表 
2. 中间结果集 
3. 会话状态存储

#### 存储引擎对比

| 特性         | InnoDB  | MyISAM | Memory        | Archive | CSV  |
| :----------- | :------ | :----- | :------------ | :------ | :--- |
| **事务**     | ✅       | ❌      | ❌             | ❌       | ❌    |
| **锁粒度**   | 行锁    | 表锁   | 表锁          | 行锁    | 表锁 |
| **MVCC**     | ✅       | ❌      | ❌             | ❌       | ❌    |
| **外键**     | ✅       | ❌      | ❌             | ❌       | ❌    |
| **崩溃恢复** | ✅       | ❌      | 丢失          | ✅       | ❌    |
| **压缩**     | ✅       | ✅      | ❌             | ✅       | ❌    |
| **全文索引** | ✅(5.6+) | ✅      | ❌             | ❌       | ❌    |
| **空间索引** | ✅(5.7+) | ✅      | ❌             | ❌       | ❌    |
| **内存使用** | 缓冲池  | 键缓存 | 全表内存      | 低      | 低   |
| **存储限制** | 64TB    | 256TB  | max_heap_size | 无      | 4GB  |
| **速度-读**  | 高      | 很高   | 极高          | 低      | 低   |
| **速度-写**  | 中      | 高     | 高            | 极高    | 中   |

### 文件存储结构对比

在 MySQL中建立任何一张数据表,在其数据目录对应的数据库目录下都有对应表的 `.frm` 文件,`.frm` 文件是用来保存每个数据表的元数据(meta)信息,包括表结构的定义等,与数据库存储引擎无关,也就是任何存储引擎的数据表都必须有`.frm`文件,命名方式为 数据表名.frm,如user.frm。

查看MySQL 数据保存在哪里:`show variables like 'data%'`

MyISAM 物理文件结构为:

- `.frm`文件:与表相关的元数据信息都存放在frm文件,包括表结构的定义信息等
- `.MYD` (`MYData`) 文件:MyISAM 存储引擎专用,用于存储MyISAM 表的数据
- `.MYI` (`MYIndex`)文件:MyISAM 存储引擎专用,用于存储MyISAM 表的索引相关信息

InnoDB 物理文件结构为:

- `.frm` 文件:与表相关的元数据信息都存放在frm文件,包括表结构的定义信息等
- `.ibd` 文件或 `.ibdata` 文件: 这两种文件都是存放 InnoDB 数据的文件,之所以有两种文件形式存放 InnoDB 的数据,是因为 InnoDB 的数据存储方式能够通过配置来决定是使用**共享表空间**存放存储数据,还是用**独享表空间**存放存储数据。
- 独享表空间存储方式使用`.ibd`文件,并且每个表一个`.ibd`文件, 共享表空间存储方式使用`.ibdata`文件,所有表共同使用一个`.ibdata`文件(或多个,可自己配置)

> 面试中如何回答这些问题？

**事务方面**

InnoDB 支持事务,MyISAM 不支持事务。这是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一;

**是否支持外键**

InnoDB 支持外键,而 MyISAM 不支持。对一个包含外键的 InnoDB 表转为 MYISAM 会失败;

**聚簇索引和非聚簇索引**

InnoDB 是聚簇索引,MyISAM 是非聚簇索引。聚簇索引的文件存放在主键索引的叶子节点上,因此 InnoDB 必须要有主键,通过主键索引效率很高。但是辅助索引需要两次查询,先查询到主键,然后再通过主键查询到数据。因此,主键不应该过大,因为主键太大,其他索引也都会很大。而 MyISAM 是非聚集索引,数据文件是分离的,索引保存的是数据文件的指针。主键索引和辅助索引是独立的。

**是否保存表的行数**

InnoDB 不保存表的具体行数,执行`select count(*) from table` 时需要全表扫描。而 MyISAM 用一个变量保存了整个表的行数,执行上述语句时只需要读出该变量即可,速度很快;

**锁粒度**

InnoDB 最小的锁粒度是行锁,MyISAM 最小的锁粒度是表锁。一个更新语句会锁住整张表,导致其他查询和更新都会被阻塞,因此并发访问受限。这也是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一;

**是否支持数据库异常崩溃后的安全恢复**

MyISAM 不支持,而 InnoDB 支持。

使用 InnoDB 的数据库在异常崩溃后,数据库重新启动的时候会保证数据库恢复到崩溃前的状态。这个恢复的过程依赖于 `redo log` 。

🌈 拓展一下:

- MySQL InnoDB 引擎使用 **redo log(重做日志)** 保证事务的**持久性**,使用 **undo log(回滚日志)** 来保证事务的**原子性**。
- MySQL InnoDB 引擎通过 **锁机制**、**MVCC** 等手段来保证事务的隔离性( 默认支持的隔离级别是 **`REPEATABLE-READ`** )。
- 保证了事务的持久性、原子性、隔离性之后,一致性才能得到保障。

两种引擎对比:

![1631753040455](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/084403-805112.png)

![1640656100727](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/28/094822-164211.png)

> 一张表,里面有ID自增主键,当insert了17条记录之后,删除了第15,16,17条记录,再把Mysql重启,再insert一条记录,这条记录的ID是18还是15 ？

如果表的类型是MyISAM,那么是18。因为MyISAM表会把自增主键的最大ID 记录到**数据文件中**,重启MySQL自增主键的最大ID也不会丢失;

如果表的类型是InnoDB,那么是15。因为InnoDB 表只是把自增主键的最大ID记录到**内存中**,所以重启数据库或对表进行OPTION操作,都会导致最大ID丢失。

> 哪个存储引擎执行 select count(*) 更快,为什么?

MyISAM更快,因为MyISAM内部维护了一个计数器,可以直接调取。

- 在 MyISAM 存储引擎中,把表的总行数存储在磁盘上,当执行 select count(*) from t 时,直接返回总数据。
- 在 InnoDB 存储引擎中,跟 MyISAM 不一样,没有将总行数存储在磁盘上,当执行 select count(*) from t 时,会先把数据读出来,一行一行的累加,最后返回总数量。

InnoDB 中 count(*) 语句是在执行的时候,全表扫描统计总数量,所以当数据越来越大时,语句就越来越耗时了,为什么 InnoDB 引擎不像 MyISAM 引擎一样,将总行数存储到磁盘上？这跟 InnoDB 的事务特性有关,由于多版本并发控制(MVCC)的原因,InnoDB 表“应该返回多少行”也是不确定的。

## 锁机制与 InnoDB 锁算法

**MyISAM 和 InnoDB 存储引擎使用的锁:**

- MyISAM 采用**表级锁**(table-level locking)。
- InnoDB 支持**行级锁**(row-level locking)和**表级锁**,默认为行级锁

**表级锁和行级锁对比:**

- **表级锁:** MySQL 中锁定 **粒度最大** 的一种锁,对当前操作的整张表加锁,实现简单,资源消耗也比较少,加锁快,不会出现死锁。其锁定粒度最大,触发锁冲突的概率最高,并发度最低,MyISAM 和 InnoDB 引擎都支持表级锁。
- **行级锁:** MySQL 中锁定 **粒度最小** 的一种锁,只针对当前操作的行进行加锁。 行级锁能大大减少数据库操作的冲突。其加锁粒度最小,并发度高,但加锁的开销也最大,加锁慢,会出现死锁。

**InnoDB 存储引擎的锁的算法有三种:**

- Record lock:记录锁,单个行记录上的锁
- Gap lock:间隙锁,锁定一个范围,不包括记录本身
- Next-key lock:record+gap 临键锁,锁定一个范围,包含记录本身

## 介绍一下Innodb存储引擎锁算法

### InnoDB 锁体系架构

```mermaid
graph TB
    subgraph InnoDB_Locks [InnoDB 锁体系]
        Lock[锁管理器] --> Type[锁类型]
        Lock --> Mode[锁模式]
        Lock --> Algo[锁算法]
        Lock --> Gap[间隙锁/临键锁]
        
        Type --> Row[行锁]
        Type --> Table[表锁]
        
        Mode --> S[共享锁 S]
        Mode --> X[排他锁 X]
        Mode --> IS[意向共享锁 IS]
        Mode --> IX[意向排他锁 IX]
        
        Algo --> Record[记录锁<br/>Record Lock]
        Algo --> Gap[间隙锁<br/>Gap Lock]
        Algo --> NK[临键锁<br/>Next-Key Lock]
        Algo --> PI[插入意向锁<br/>Insert Intention]
        
        Gap --> Phantom[解决幻读]
        NK --> RR[可重复读隔离级别]
    end
```

下面详细介绍一下每一种锁算法:

### 记录锁（Record Lock）

定义： 对索引记录加锁，锁定具体的某一行

```sql
-- 示例：锁定id=5的记录
SELECT * FROM users WHERE id = 5 FOR UPDATE;
-- 在id=5的索引记录上加X锁
```

锁算法特征:

```mermaid
graph LR
    subgraph Record_Lock [记录锁示例]
        Index[索引页] --> R1[记录 id=1]
        R1 --> R2[记录 id=3]
        R2 --> R3[记录 id=5]
        R3 --> R4[记录 id=7]
        
        Lock[🔒 X锁] -.-> R3
    end
```
特性：

✅ 锁定单条索引记录

✅ 必须通过索引条件（非索引条件升级为表锁）

✅ 锁定索引项，而非数据行本身

### 间隙锁（Gap Lock）

定义： 锁定索引记录之间的间隙，不包含记录本身

```sql
-- RR隔离级别下，范围条件会触发电间隙锁
SELECT * FROM users WHERE id BETWEEN 3 AND 7 FOR UPDATE;
-- 锁定(3,7)之间的间隙，防止插入id=4,5,6
```

算法模型:

```mermaid
graph TB
    subgraph Gap_Lock [间隙锁范围]
        direction LR
        
        G1[间隙] --> R1[记录 id=1]
        R1 --> G2[间隙 🔒]
        G2 --> R2[记录 id=3]
        R2 --> G3[间隙 🔒]
        G3 --> R3[记录 id=5]
        R3 --> G4[间隙 🔒]
        G4 --> R4[记录 id=7]
        
        style G2 fill:#ffcccc
        style G3 fill:#ffcccc
        style G4 fill:#ffcccc
    end
```
间隙范围：
- (-∞, 1)：最小间隙
- (1, 3)：中间间隙
- (3, 5)：中间间隙
- (5, 7)：中间间隙
- (7, +∞)：最大间隙

特性：

✅ 仅在 RR（可重复读） 隔离级别生效

✅ 解决幻读问题的核心机制

✅ 锁定的是"空隙"，不是记录

✅ 不同事务的间隙锁不互斥

### 临键锁（Next-Key Lock）

定义： 记录锁 + 间隙锁的组合，左开右闭区间

```sql
-- 临键锁实际锁定的范围
SELECT * FROM users WHERE id > 3 AND id < 7 FOR UPDATE;
-- 实际锁定区间：(3,5] 和 (5,7]
```

```mermaid
sequenceDiagram
    participant C as 客户端
    participant P as 解析器
    participant L as 锁管理器
    participant I as 索引

    C->>P: SELECT * FROM users WHERE id BETWEEN 3 AND 7 FOR UPDATE

    P->>I: 扫描索引范围

    I->>L: 找到记录 id=3
    L->>L: 加临键锁 (1,3]
    Note over L: 记录锁 id=3 + 间隙锁 (1,3)

    I->>L: 找到记录 id=5
    L->>L: 加临键锁 (3,5] 🔒
    Note over L: 记录锁 id=5 + 间隙锁 (3,5)

    I->>L: 找到记录 id=7
    L->>L: 加临键锁 (5,7] 🔒
    Note over L: 记录锁 id=7 + 间隙锁 (5,7)

    L-->>C: 返回结果集
```

1. 临键锁 = 记录锁 + 间隙锁
2. 锁定区间 = (前一条记录, 当前记录]
3. 生效条件 = RR隔离级别 + (范围查询|非唯一索引查询)
4. 解决幻读 = 阻止其他事务在锁定区间插入新记录
5. 内存结构 = 哈希表 + 锁对象 + 等待队列

锁定范围公式:临键锁区间 = (前一条记录, 当前记录]
示例：

- 记录id=5，临键锁锁定(3,5]
- 间隙锁锁定(3,5)
- 记录锁锁定id=5

查询 id > 3 AND id < 7 锁定 (3,5] 和 (5,7] 的原因：

1. 索引驱动：临键锁是基于已存在的索引记录划分区间 
2. 记录锚点：id=5是满足条件的唯一记录，产生(3,5]
3. 边界保护：id=7是下一个记录，产生(5,7]防止插入幻影行 
4. 间隙锁定：锁定的是插入数据的可能性，不是记录本身

记住这个核心原则：
- 临键锁 = (前一个存在的索引记录, 当前存在的索引记录]
- 分界点永远是指南针——索引中实际存在的记录

### 插入意向锁（Insert Intention Lock）

定义： 特殊的间隙锁，表示等待插入的意图

```mermaid
sequenceDiagram
    participant T1 as 事务1
    participant T2 as 事务2
    participant DB as InnoDB
    
    T1->>DB: 加间隙锁(3,7)
    DB-->>T1: 获取成功
    
    T2->>DB: 请求插入意向锁(id=5)
    DB-->>T2: 等待（被间隙锁阻塞）
    
    T1->>DB: 提交/回滚
    DB->>DB: 释放间隙锁
    
    DB-->>T2: 获取插入意向锁
    T2->>DB: 插入数据
```

特性：

✅ 插入操作前必须获取

✅ 多个插入意向锁可以共存

✅ 与间隙锁互斥

✅ 解决"幻象插入"问题

### 隔离级别与锁算法

```mermaid
flowchart TD
    subgraph Isolation [隔离级别与锁策略]
        RU[读未提交] -->|不加锁| Read1[脏读可能]
        
        RC[读已提交] -->|仅记录锁| Algo1[Record Lock]
        RC -->|无间隙锁| Phantom1[幻读可能]
        
        RR[可重复读] -->|临键锁| Algo2[Next-Key Lock]
        RR -->|间隙锁| Algo3[Gap Lock]
        RR --> Phantom2[无幻读]
        
        Serial[可串行化] -->|所有读加S锁| Algo4[Record + Gap]
        Serial -->|锁升级| Algo5[表锁可能]
    end
```

### 不同隔离级别的锁策略

| 隔离级别     | 读操作          | 范围查询锁 | 间隙锁 | 幻读 |
| :----------- | :-------------- | :--------- | :----- | :--- |
| **读未提交** | 无锁            | 无锁       | ❌      | ✅    |
| **读已提交** | 快照读/一致性读 | 仅记录锁   | ❌      | ✅    |
| **可重复读** | 快照读          | **临键锁** | ✅      | ❌    |
| **可串行化** | 共享锁          | 临键锁     | ✅      | ❌    |

### 锁算法优化与避坑

#### 索引失效导致锁升级

```sql
-- ❌ 危险：非索引字段查询
BEGIN;
SELECT * FROM users WHERE name = 'Tom' FOR UPDATE;
-- 无索引 → 全表扫描 → 所有记录加锁 → 实际表锁

-- ✅ 正确：索引字段
BEGIN;
SELECT * FROM users WHERE id = 1 FOR UPDATE;
-- 有索引 → 精准记录锁
```

#### 间隙锁导致的死锁

![](./image/间隙锁导致的死锁.png)

解决方案:

```sql
-- 降低隔离级别（业务允许时）
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 使用唯一索引
ALTER TABLE users ADD UNIQUE INDEX idx_unique_col(column);

-- 优化事务大小
-- 减少锁定范围，尽快提交
```

### 小结

| 锁类型         | 锁定对象     | 隔离级别 | 用途        | 互斥对象          |
| :------------- | :----------- | :------- | :---------- | :---------------- |
| **记录锁**     | 单条索引记录 | 所有级别 | 行级锁定    | 其他记录锁        |
| **间隙锁**     | 索引间隙     | RR       | 防止幻读    | 插入意向锁        |
| **临键锁**     | 间隙+记录    | RR       | 行锁+防幻读 | 记录锁+插入意向锁 |
| **插入意向锁** | 间隙         | RR/RC    | 插入等待    | 间隙锁            |


1. **RR如何解决幻读？**
    - Next-Key Lock = Record Lock + Gap Lock
    - 锁定已存在记录和记录之间的空隙
2. **间隙锁什么时候生效？**
    - RR隔离级别
    - 范围查询或非唯一索引等值查询
3. **为什么RC级别没有幻读风险仍有间隙锁？**
    - 业务在RC下可接受幻读
    - 性能优先，减少锁范围
4. **如何优化锁冲突？**
    - 索引下推，精确命中
    - 降低隔离级别
    - 拆分大事务
    - 合理设计索引

**一句话总结：**
**InnoDB通过临键锁（记录锁+间隙锁）在RR级别完美解决幻读，核心在于锁定索引范围而非单行，这是InnoDB实现高并发一致性的精髓。**

## 数据类型

主要包括以下五大类:

- 整数类型:BIT、BOOL、TINY INT、SMALL INT、MEDIUM INT、 INT、 BIG INT
- 浮点数类型:FLOAT、DOUBLE、DECIMAL
- 字符串类型:CHAR、VARCHAR、TINY TEXT、TEXT、MEDIUM TEXT、LONGTEXT、TINY BLOB、BLOB、MEDIUM BLOB、LONG BLOB
- 日期类型:Date、DateTime、TimeStamp、Time、Year
- 其他数据类型:BINARY、VARBINARY、ENUM、SET、Geometry、Point、MultiPoint、LineString、MultiLineString、Polygon、GeometryCollection等

![1640656330188](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/095211-484648.png)

![1640656346954](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/28/095239-888739.png)

> CHAR 和 VARCHAR 的区别？

char是固定长度,varchar长度可变:

char(n) 和 varchar(n) 中括号中 n 代表**字符**的个数,并不代表字节个数,比如 CHAR(30) 就可以存储 30 个字符。

存储时,前者不管实际存储数据的长度,直接按 char 规定的长度分配存储空间;而后者会根据实际存储的数据分配最终的存储空间

相同点:

1. char(n),varchar(n)中的n都代表字符的个数
2. 超过char,varchar最大长度n的限制后,字符串会被截断。

不同点:

1. char不论实际存储的字符数都会占用n个字符的空间,而varchar只会占用实际字符应该占用的字节空间加1(实际长度length,0<=length<255)或加2(length>255)。因为varchar保存数据时除了要保存字符串之外还会加一个字节来记录长度(如果列声明长度大于255则使用两个字节来保存长度)。
2. 能存储的最大空间限制不一样:char的存储上限为255字节。
3. char在存储时会截断尾部的空格,而varchar不会。

char是适合存储很短的、一般固定长度的字符串。例如,char非常适合存储密码的MD5值,因为这是一个定长的值。对于非常短的列,char比varchar在存储空间上也更有效率。

> 列的字符串类型可以是什么？

字符串类型是:SET、BLOB、ENUM、CHAR、TEXT、VARCHAR

> BLOB和TEXT有什么区别？

BLOB是一个二进制对象,可以容纳可变数量的数据。有四种类型的BLOB:TINYBLOB、BLOB、MEDIUMBLO和 LONGBLOB

TEXT是一个不区分大小写的BLOB。四种TEXT类型:TINYTEXT、TEXT、MEDIUMTEXT 和 LONGTEXT。

BLOB 保存二进制数据,TEXT 保存字符数据。

## 数据库的三大范式

数据冗余直观上可以说就是一张表里不同位置有大量重复的数据,这种冗余不仅仅增加了存储量,也使得我们会更容易遇到三种异常(**插入异常,更新异常和删除异常**)。

### 第一范式

- 第一范式:确保每列保持原子性,数据表中的所有字段值都是不可分解的原子值。在任何一个关系型数据库中,满足第一范式是最基本的要求,
    - 1NF是对属性的**原子性**,要求属性具有原子性,不可再分解;
    - 如学生(学号,姓名,性别,出生日期),可以看到每一个列属性都是不可在分割的单位,这就是原子性的含义。
    - 解决方法:对可以再分的列字段,进行拆分,划分为多个表。

![1640651441705](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/28/083043-534139.png)

但是,就算是这样符合1NF的表仍然会有很多,因为1NF其实是最为基本的要求,不满足1NF的数据库甚至不一定能建立成功。

比如,上面提到的三个异常:

1. 插入异常。如果学校建了新系,但还没有招生,这个系就不能被插入数据表里
2. 更新异常。如果Akon转系了,那在上表中需要更改两行院系&课程记录
3. 删除异常。如果所有学生记录被删除,院系记录和课程也就不复存在了

> 1NF解决的是数据冗余问题。

### 第二范式

我们先来看看什么是依赖:

对于一个表来说,如果通过其中一个属性可以找到唯一对应的一条记录,那么我们可以说它为本表的主键(Primary Key)。 比如下表中,每个学生的学号是存在且唯一的,但是名字可能会有重名存在,学号为主键(Primary Key),而姓名就不是。 同时,我可以通过【学号】查到任何一行的任何一列属性,比如通过【学号】查【院系】,通过【学号】查【课程】,通过【学号】查【姓名】。这时我们可以说,其他的这三个属性**依赖**于学号。

**那什么是部分依赖呢？**
候选键(Candidate Key)就是,当两个属性结合在一起可以唯一确定任何一条记录的情况。比如在一张学生成绩表中。

![1640655170465](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/28/093301-442493.png)

【学号+课程号】 一起可以确定任何一条分数或是学号或是教师,所以 【学号+课程号】 就是本表的候选键。
一张表可以有多个键(key),一般我们会选择其中一个作为主键。
这个时候我们可以看到,教师这一属性其实只由课程号决定,而课程号只是 候选键 的一部分,因此这时我们就说,教师属性存在部分函数依赖。

- 第二范式:确保表中的每列都和主键相关
    - 2NF是对记录的**唯一性**,要求记录有唯一标识,即实体的唯一性,即**不存在部分依赖,必须是完全依赖,也就是所有的非主属性必须完全依赖于主属性**;
    - 如表:(学号,课程号,姓名,学分),这个表明显说明了两个事务:学生信息, 课程信息;由于非主键字段必须依赖主键,这里**学分依赖课程号**,**姓名依赖与学号**,所以不符合二范式。所以我们要对表进行拆分,让其符合第二范式,学生:`Student`(学号, 姓名);课程:`Course`(课程号, 学分);选课关系:`StudentCourse`(学号, 课程号, 成绩)。
    - 解决方法:一般是把表拆分为两个表,一张表只管理一件事。
    - 如果不符合第二范式,可能会存在的问题:
        - `数据冗余:`,每条记录都含有相同信息;
        - `删除异常:`删除所有学生成绩,就把课程信息全删除了;
        - `插入异常:`学生未选课,无法记录进数据库;
        - `更新异常:`调整课程学分,所有行都调整。

**拆分表**

1. 先找出所有的非主属性(不是主键也不是候选键包含部分的属性),在这个例子里,键为【学号+课程号】,那么他俩为主属性,剩下的都是非主属性
2. 检查这些非主属性是否存在部分函数依赖。【姓名】只依赖于【学号】,存在;【分数】非得要【学号+课程号】一起才能确定,不存在;【教师】只依赖于【课程】号,存在
    1. 将这些存在部分函数依赖的属性分出去建立满足2NF的新表,切分方法并不唯一。

**分数表去掉 【姓名】 和 【教师】 属性:**

![1640655497010](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/28/093817-893822.png)

**为 姓名 建立学生表:**

![1640655526626](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/28/093848-181211.png)

**为 教师 建立 课程表:**

![1640655547943](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/28/093913-281644.png)

第二范式存在的异常:

- 插入异常。招新生的话,学生信息可以单独插入,有改进。
- 更新异常。如果L1号课换老师了,只用修改一次,有改进。
- **删除异常**。如果删除所有的学生信息,教师信息还在,分数信息也还在;但是如果我从教师表里删掉课程L1的记录,教师Mr.x以及CS院系信息就不复存在了,这是个大问题。

> 满足2NF的前提是必须满足1NF。
>
> 2NF解决的是部分函数依赖问题。
>
> 数据冗余问题变少了。

### 第三范式

比如上面一节提到,如果删除教师表里课程L1的信息,教师Mr.x以及CS院系信息就不复存在了,同时,如果一名新来的教师还没有被分配到任何课,他就不能被加入到教师表里。

这是因为,在教师表里:

1. 课程号可以决定教师. A → B
2. 教师不能决定课程号,因为一个教师可以教多门课. B not→ A
3. 教师决定院系,因为一个教师只能属于一个院系. B → C

这时我们就发现,非主属性 【院系】,也依赖于另一个非主属性 【教师】,这种情况就叫做传递依赖。
而3NF的条件,就是要去除这种传递依赖。

- 第三范式:确保每列都和主键列直接相关而不是间接相关
    - 3NF是对字段的**冗余性**,要求任何字段不能由其他字段派生出来,它要求字段没有冗余,即**不存在传递依赖**,不存在属性对主键的**传递依赖**。;
    - 表: 学号, 姓名, 年龄, 学院名称, 学院电话,因为存在**依赖传递**: (学号) → (学生)→(所在学院) → (学院电话) 。所以对表进行拆分,消除传递函数依赖,学生:(学号, 姓名, 年龄, 所在学院);学院:(学院, 电话)。
    - **可能会存在问题:**
        - `数据冗余:`有重复值;
        - `更新异常:`有重复的冗余信息,修改时需要同时修改多条记录,否则会出现**数据不一致的情况** 。
        - `插入异常`:学生未选课,无法记录进数据库;
        - `删除异常`:删除所有学生成绩,就把课程信息全删除了;

**拆分表**

解决方法有多种,这里可以将院系信息分表。

课程表只有课程号和教师信息:

![1640655793724](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/28/094315-520507.png)

而教师表 只有教师和院系信息:

![1640655818425](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/28/094347-409638.png)

这样我们再检查上面的问题, 删除L1课程信息,Mr.X老师的信息仍然保存的很好,有改进。 新老师Mr.Z可以被插入教师表,哪怕他还没有被分配任何课程。

> 3NF解决的是传递依赖关系
>
> 满足3NF的前提是必须满足2NF。
>
> 3NF解决了插入异常。

## Mysql的数据类型都有哪些？

**整数**

- TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT分别占用8、16、24、32、64位存储空间。值得注意的是,INT(10)中的10只是表示显示字符的个数,并无实际意义。一般和UNSIGNED ZEROFILL配合使用才有实际意义,例如,数据类型INT(3),属性为UNSIGNED ZEROFILL,如果插入的数据为3的话,实际存储的数据为003。

**浮点数**

- FLOAT、DOUBLE及DECIMAL为浮点数类型,DECIMAL是利用字符串进行处理的,能存储精确的小数。相比于FLOAT和DOUBLE,DECIMAL的效率更低些。FLOAT、DOUBLE及DECIMAL都可以指定列宽,例如FLOAT(5,2)表示一共5位,两位存储小数部分,三位存储整数部分。

**字符串**

- 字符串常用的主要有CHAR和VARCHAR,VARCHAR主要用于存储可变长字符串,相比于定长的CHAR更节省空间。CHAR是定长的,根据定义的字符串长度分配空间。
    - 应用场景:对于经常变更的数据使用CHAR更好,CHAR不容易产生碎片。对于非常短的列也是使用CHAR更好些,CHAR相比于VARCHAR在效率上更高些。**一般避免使用TEXT/BLOB等类型,因为查询时会使用临时表,造成严重的性能开销。**

**日期**

- 比较常用的有year、time、date、datetime、timestamp等,datetime保存从1000年到9999年的时间,精度位秒,使用8字节的存储空间,与时区无关。timestamp和UNIX的时间戳相同,保存从1970年1月1日午夜到2038年的时间,精度到秒,使用四个字节的存储空间,并且与时区相关。
    - 应用场景:尽量使用timestamp,相比于datetime它有着更高的空间效率。 

## 索引的使用场景有哪些？

- 对于中大型表建立索引非常有效,对于非常小的表,一般全部表扫描速度更快些。
- 对于超大型的表,建立和维护索引的代价也会变高,这时可以考虑分区技术。
- 如果表的增删改非常多,而查询需求非常少的话,那就没有必要建立索引了,因为维护索引也是需要代价的。
- 一般不会出现再where条件中的字段就没有必要建立索引了。
- 多个字段经常被查询的话可以考虑联合索引。
- 字段多且字段值没有重复的时候考虑唯一索引。
- 字段多且有重复的时候考虑普通索引

## 哪些情况需要创建索引

1. 主键自动建立唯一索引
2. 频繁作为查询条件的字段
3. 查询中与其他表关联的字段,外键关系建立索引
4. 单键/组合索引的选择问题,高并发下倾向创建组合索引
5. 查询中排序的字段,排序字段通过索引访问大幅提高排序速度
6. 查询中统计或分组字段


### 哪些情况不要创建索引

1. 表记录太少
2. 经常增删改的表
3. 数据重复且分布均匀的表字段,只应该为最经常查询和最经常排序的数据列建立索引(如果某个数据类包含太多的重复数据,建立索引没有太大意义)
4. 频繁更新的字段不适合创建索引(会加重IO负担)
5. where条件里用不到的字段不创建索引

## 索引的设计原则有哪些？

1. 最适合索引的列是在***where*后面出现的列或者连接句子中指定的列**,而不是出现在SELECT关键字后面的选择列表中的列。
2. **索引列的基数越大,索引的效果越好**,换句话说就是索引列的区分度越高,索引的效果越好。比如使用性别这种区分度很低的列作为索引,效果就会很差,因为列的基数最多也就是三种,大多不是男性就是女性。
3. 尽量使用短索引,对于较长的字符串进行索引时应该指定一个较短的前缀长度,因为较小的索引涉及到的磁盘I/O较少,并且索引高速缓存中的块可以容纳更多的键值,会使得查询速度更快。尽量利用最左前缀。
4. 不要过度索引,每个索引都需要额外的物理空间,维护也需要花费时间,所以索引不是越多越好。

## 索引的类型有哪些

MySQL主要的索引类型主要有FULLTEXT,HASH,BTREE,RTREE。

**FULLTEXT**

- FULLTEXT即全文索引,MyISAM存储引擎和InnoDB存储引擎在MySQL5.6.4以上版本支持全文索引,一般用于查找文本中的关键字,而不是直接比较是否相等,多在CHAR,VARCHAR,TAXT等数据类型上创建全文索引。全文索引主要是用来解决WHERE name LIKE "%zhang%"等针对文本的模糊查询效率低的问题。

**HASH**

- HASH即哈希索引,哈希索引多用于等值查询,时间复杂夫为o(1),效率非常高,但不支持排序、范围查询及模糊查询等。

**BTREE**

- BTREE即B+树索引,INnoDB存储引擎默认的索引,支持排序、分组、范围查询、模糊查询等,并且性能稳定。

**RTREE**

- RTREE即空间数据索引,多用于地理数据的存储,相比于其他索引,空间数据索引的优势在于范围查找

## 如何对索引进行优化

1. 对索引的优化其实最关键的就是要符合索引的设计原则和应用场景,将不符合要求的索引优化成符合索引设计原则和应用场景的索引。
2. 除了索引的设计原则和应用场景那几点外,还可以从以下两方面考虑。
3. 在进行查询时,索引列不能是表达式的一部分,也不能是函数的参数,因为这样无法使用索引。例如 `select * from table_name where a + 1 = 2`。
4. 将区分度最高的索引放在前面
5. 尽量少使用`select*`。

索引的使用场景、索引的设计原则和如何对索引进行优化可以看成一个问题。


## 什么是临时表,什么时候会使用到临时表,什么时候删除临时表？

MySQL在执行SQL语句的时候会临时创建一些存储中间结果集的表,这种表被称为临时表,临时表只对当前连接可见,在连接关闭后,临时表会被删除并释放空间。

临时表主要分为内存临时表和磁盘临时表两种。内存临时表使用的是MEMORY存储引擎,磁盘临时表使用的是MyISAM存储引擎。

一般在以下几种情况中会使用到临时表:

1. FROM中的子查询
2. DISTINCT查询并加上ORDER BY
3. ORDER BY和GROUP BY的子句不一样时会产生临时表
4. 使用UNION查询会产生临时表

## 大表数据查询如何进行优化？

1. 索引优化
2. SQL语句优化
3. 水平拆分
4. 垂直拆分
5. 建立中间表
6. 使用缓存技术
7. 固定长度的表访问起来更快
8. 越小的列访问越快

## 了解慢日志查询吗？统计过慢查询吗？对慢查询如何优化？

慢查询一般用于记录执行时间超过某个临界值的SQL语句的日志。

**相关参数:**

- slow_query_log:是否开启慢日志查询,1表示开启,0表示关闭。
- slow_query_log_file:MySQL数据库慢查询日志存储路径。
- long_query_time:慢查询阈值,当SQL语句查询时间大于阈值,会被记录在日志上。
- log_queries_not_using_indexes:未使用索引的查询会被记录到慢查询日志中。
- log_output:日志存储方式。“FILE”表示将日志存入文件。“TABLE”表示将日志存入数据库。

**如何对慢查询进行优化？**

- 分析语句的执行计划,查看SQL语句的索引是否命中
- 优化数据库的结构,将字段很多的表分解成多个表,或者考虑建立中间表。
- 优化LIMIT分页。


## 主键一般用自增ID还是UUID？

**使用自增ID的好处:**

1. 字段长度较uuid会小很多。
2. 数据库自动编号,按顺序存放,利于检索
3. 无需担心主键重复问题

**使用自增ID的缺点:**

1. 因为是自增,在某些业务场景下,容易被其他人查到业务量。
2. 发生数据迁移时,或者表合并时会非常麻烦
3. 在高并发的场景下,竞争自增锁会降低数据库的吞吐能力

UUID:通用唯一标识码,UUID是基于当前时间、计数器和硬件标识等数据计算生成的。

**使用UUID的优点:**

1. 唯一标识,不会考虑重复问题,在数据拆分、合并时也能达到全局的唯一性。
2. 可以在应用层生成,提高数据库的吞吐能力。
3. 无需担心业务量泄露的问题。

**使用UUID的缺点:**

1. 因为UUID是随机生成的,所以会发生随机IO,影响插入速度,并且会造成硬盘的使用率较低。
2. UUID占用空间较大,建立的索引越多,造成的影响越大。
3. UUID之间比较大小较自增ID慢不少,影响查询速度。

最后说下结论,一般情况MySQL推荐使用自增ID。因为在MySQL的InnoDB存储引擎中,主键索引是一种聚簇索引,主键索引的B+树的叶子节点按照顺

序存储了主键值及数据,如果主键索引是自增ID,只需要按顺序往后排列即可,如果是UUID,ID是随机生成的,在数据插入时会造成大量的数据移动,产生大量的内存碎片,造成插入性能的下降。

## 字段为什么要设置成not null?

首先说一点,NULL和空值是不一样的,空值是不占用空间的,而NULL是占用空间的,所以字段设为NOT NULL后仍然可以插入空值。

字段设置成not null主要有以下几点原因:

1. NULL值会影响一些函数的统计,如count,遇到NULL值,这条记录不会统计在内。
2. B树不存储NULL,所以索引用不到NULL,会造成第一点中说的统计不到的问题
3. NOT IN子查询在有NULL值的情况下返回的结果都是空值。

例如user表如下 :

![1631772205116](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/140348-975886.png)

```sql
select * from `user` where username NOT IN (select username from `user` where id != 0) ,
--这条查询语句应该查到zhangsan这条数据,但是结果显示为null。
```

MySQL在进行比较的时候,NULL会参与字段的比较,因为NULL是一种比较特殊的数据类型,数据库在处理时需要进行特数处理,增加了数据库处理记录的复杂性。

## 如何优化查询过程中的数据访问？

从减少数据访问方面考虑:

- 正确使用索引,尽量做到索引覆盖
- 优化SQL执行计划

从返回更少的数据方面考虑:

- 数据分页处理
- 只返回需要的字段

从减少服务器CPU开销方面考虑:

- 合理使用排序
- 减少比较的操作
- 复杂运算在客户端处理

从增加资源方面考虑:

- 客户端多进程并行访问
- 数据库并行处理

## 如何优化长难的查询语句？

- 将一个大的查询分解为多个小的查询
- 分解关联查询,使缓存的效率更高

## 如何优化LIMIT分页？

- 在LIMIT偏移量较大的时候,查询效率会变低,可以记录每次取出的最大ID,下次查询时可以利用ID进行查询
- 建立复合索引

## 如何优化UNION查询

- 如果不需要对结果集进行去重或者排序建议使用UNION ALL,会好一些。

## 如何优化WHERE子句

1. 不要在where子句中使用!=和<>进行不等于判断,这样会导致放弃索引进行全表扫描。
2. 不要在where子句中使用null或空值判断,尽量设置字段为not null。
3. 尽量使用union all代替or
4. 在where和order by涉及的列建立索引
5. 尽量减少使用in或者not in,会进行全表扫描
6. 在where子句中使用参数会导致全表扫描
7. 避免在where子句中对字段及进行表达式或者函数操作会导致存储引擎放弃索引进而全表扫描

## SQL语句执行的很慢原因是什么？

1. 如果SQL语句只是偶尔执行很慢,可能是执行的时候遇到了锁,也可能是redo log日志写满了,要将redo log中的数据同步到磁盘中去。
2. 如果SQL语句一直都很慢,可能是字段上没有索引或者字段有索引但是没用上索引。

## SQL语句的执行顺序?

```sql
SELECT DISTINCT
	select_list
FROM
	left_table
LEFT JOIN
	right_table ON join_condition
WHERE
	where_condition
GROUP BY
	group_by_list
HAVING
	having_condition
ORDER BY
	order_by_condition
```

执行顺序如下:

![1631772654483](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/141057-986872.png)

1. FROM:对SQL语句执行查询时,首先对关键字两边的表以笛卡尔积的形式执行连接,并产生一个虚表V1。虚表就是视图,数据会来自多张表的执行结果。
2. ON:对FROM连接的结果进行ON过滤,并创建虚表V2
3. JOIN:将ON过滤后的左表添加进来,并创建新的虚拟表V3
4. WHERE:对虚拟表V3进行WHERE筛选,创建虚拟表V4
5. GROUP BY:对V4中的记录进行分组操作,创建虚拟表V5
6. HAVING:对V5进行过滤,创建虚拟表V6
7. SELECT:将V6中的结果按照SELECT进行筛选,创建虚拟表V7
8. DISTINCT:对V7表中的结果进行去重操作,创建虚拟表V8,如果使用了GROUP BY子句则无需使用DISTINCT,因为分组的时候是将列中唯一的值分成一组,并且每组只返回一行记录,所以所有的记录都h是不同的。
9. ORDER BY:对V8表中的结果进行排序。

## 索引的种类有哪些？

1. 主键索引:数据列不允许重复,不能为NULL,一个表只能有一个主键索引
2. 组合索引:由多个列值组成的索引。
3. 唯一索引:数据列不允许重复,可以为NULL,索引列的值必须唯一的,如果是组合索引,则列值的组合必须唯一。
4. 全文索引:对文本的内容进行搜索。
5. 普通索引:基本的索引类型,可以为NULL 。


## 唯一索引和主键索引的区别？

1. 主键是一种约束,唯一索引是一种索引,两者在本质上是不同的。
2. 主键创建后一定包含一个唯一性索引,唯一性索引并不一定就是主键。
3. 唯一性索引列允许空值,而主键列不允许为空值。
4. 主键列在创建时,已经默认为空值 + 唯一索引了。
5. 主键可以被其他表引用为外键,而唯一索引不能。
6. 一个表最多只能创建一个主键,但可以创建多个唯一索引。
7. 主键更适合那些不容易更改的唯一标识,如自动递增列、身份证号等。

## 如何创建删除索引？

**创建索引:**

1. 使用CREATE INDEX 语句

```sql
CREATE INDEX index_name ON table_name (column_list);
```

2. 在CREATE TABLE时创建

```sql
CREATE TABLE user(
  id INT PRIMARY KEY,
  information text,
  FULLTEXT KEY (information)
);
```

3. 使用ALTER TABLE创建索引

```sql
ALTER TABLE table_name ADD INDEX index_name (column_list);
```

**删除索引**

1. 删除主键索引

```sql
alter table 表名 drop primary key
```

2. 删除其他索引

```sql
alter table 表名 drop key 索引名
```

## 使用索引查询时性能一定会提升吗？

不一定,前面在索引的使用场景和索引的设计原则中已经提到了如何合理地使用索引,因为创建和维护索引需要花费空间和时间上的代价,如果不合理地使用索引反而会使查询性能下降 。

## 什么是前缀索引

**前缀索引是指对文本或者字符串的前几个字符建立索引**,这样索引的长度更短,查询速度更快。

- 使用场景:前缀的区分度比较高的情况下。
- 建立前缀索引的方式

```sql
ALTER TABLE table_name ADD KEY(column_name(prefix_length));
```

这里面有个prefix_length参数很难确定,这个参数就是前缀长度的意思。通常可以使用以下方法进行确定,先计算全列的区分度。

```sql
SELECT COUNT(DISTINCT column_name) / COUNT(*) FROM table_name;
```

然后在计算前缀长度为多少时和全列的区分度最相似。

```sql
SELECT COUNT(DISTINCT LEFT(column_name, prefix_length)) / COUNT(*) FROM table_name
```

不断地调整`prefix_length`的值,直到和全列计算出区分度相近。

## 什么是最左匹配原则

最左匹配原则:从最左边为起点开始连续匹配,遇到范围查询(<、>、between、like)会停止匹配。

例如建立索引(a,b,c),大家可以猜测以下几种情况是否用到了索引

**第一种**

```sql
select * from table_name where a = 1 and b = 2 and c = 3
select * from table_name where b = 2 and a = 1 and c = 3 -- 调动索引字段顺序不会影响查询效率
```

上面两次查询过程中所有值都用到了索引,where后面字段调换不会影响查询结果,因为MySQL中的优化器会自动优化查询顺序。

**第二种**

```sql
select * from table_name where a = 1
select * from table_name where a = 1 and b = 2
select * from table_name where a = 1 and b = 2 and c = 3
```

答案是三个查询语句都用到了索引,因为三个语句都是从最左开始匹配的。

**第三种**

```sql
select * from table_name where b = 1
select * from table_name where b = 1 and c = 2
```

答案是这两个查询语句都没有用到索引,因为不是从最左边开始匹配的 ,建立的索引a没有生效,所以索引b,c都失效。

**第四种**

```sql
select * from table_name where a = 1 and c = 2
```

这个查询语句只有a列用到了索引,c列没有用到索引,因为中间跳过了b列,不是从最左开始连续匹配的,这里索引c失效。

**第五种**

```sql 
select * from table_name where a = 1 and b < 3 and c < 1
```

这个查询中只有a列和b列使用到了索引,而c列没有使用索引,因为根据最左匹配查询原则,遇到范围查询会停止。

**第六种**

```sql 
select * from table_name where a like 'ab%';
select * from table_name where a like '%ab'
select * from table_name where a like '%ab%'
```

对于列为字符串的情况,只有前缀匹配可以使用索引,中缀匹配和后缀匹配只能进行全表扫描 .

> 对于字符串两端都有%的情况,可以使用覆盖索引。

## 索引在什么情况下会失效？

在上面介绍了几种不符合最左匹配原则的情况会导致索引失效,除此之外,以下这几种情况也会导致索引失效。

1. 条件中有or,例如 `select * from table_name where a = 1 or b = 3`
2. 在索引上进行计算会导致索引失效,例如 `select * from table_name where a + 1 = 2`
3. 在索引的类型上进行数据类型的隐形转换,会导致索引失效,例如字符串一定要加引号,假设`select * from table_name where a = '1' `会使用到索引,如果写成 `select * from table_name where a = 1 `则会导致索引失效。
4. 在索引中使用函数会导致索引失效,例如 `select * from table_name where abs(a) = 1`
5. 在使用like查询时以%开头会导致索引失效
6. 索引上使用`！、=、<>`进行判断时会导致索引失效,例如` select * from table_name where a!= 1`
7. 索引字段上使用` is null/is not null`判断时会导致索引失效,例如 `select * from table_namewhere a is null `

## Mysql查询

> count(*) 和 count(1)和count(列名)区别   ps:这道题说法有点多

执行效果上:

- count(*)包括了所有的列,相当于行数,在统计结果的时候,不会忽略列值为NULL
- count(1)包括了所有列,用1代表代码行,在统计结果的时候,不会忽略列值为NULL
- count(列名)只包括列名那一列,在统计结果的时候,会忽略列值为空(这里的空不是只空字符串或者0,而是表示null)的计数,即某个字段值为NULL时,不统计。

执行效率上:

- 列名为主键,count(列名)会比count(1)快
- 列名不为主键,count(1)会比count(列名)快
- 如果表多个列并且没有主键,则 count(1) 的执行效率优于 count(*)
- 如果有主键,则 select count(主键)的执行效率是最优的
- 如果表只有一个字段,则 select count(*) 最优。

> MySQL中 in和 exists 的区别？

- exists:exists对外表用loop逐条查询,每次查询都会查看exists的条件语句,当exists里的条件语句能够返回记录行时(无论记录行是的多少,只要能返回),条件就为真,返回当前loop到的这条记录;反之,如果exists里的条件语句不能返回记录行,则当前loop到的这条记录被丢弃,exists的条件就像一个bool条件,当能返回结果集则为true,不能返回结果集则为false
- in:in查询相当于多个or条件的叠加

```sql
SELECT * FROM A WHERE A.id IN (SELECT id FROM B);
SELECT * FROM A WHERE EXISTS (SELECT * from B WHERE B.id = A.id);
```

**如果查询的两个表大小相当,那么用in和exists差别不大**。

如果两个表中一个较小,一个是大表,则子查询表大的用exists,子查询表小的用in:

> UNION和UNION ALL的区别?

UNION和UNION ALL都是将两个结果集合并为一个,**两个要联合的SQL语句 字段个数必须一样,而且字段类型要“相容”(一致);**

- UNION在进行表连接后会筛选掉重复的数据记录(效率较低),而UNION ALL则不会去掉重复的数据记录;
- UNION会按照字段的顺序进行排序,而UNION ALL只是简单的将两个结果合并就返回;

## SQL执行顺序

- 手写

  ```sql
  SELECT DISTINCT <select_list>
  FROM  <left_table> <join_type>
  JOIN  <right_table> ON <join_condition>
  WHERE  <where_condition>
  GROUP BY  <group_by_list>
  HAVING <having_condition>
  ORDER BY <order_by_condition>
  LIMIT <limit_number>
  复制代码
  ```

- 机读

  ```sql
  FROM  <left_table>
  ON <join_condition>
  <join_type> JOIN  <right_table> 
  WHERE  <where_condition>
  GROUP BY  <group_by_list>
  HAVING <having_condition>
  SELECT
  DISTINCT <select_list>
  ORDER BY <order_by_condition>
  LIMIT <limit_number>
  复制代码
  ```

- 总结

![1640658451255](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/28/102732-201961.png)

> mysql 的内连接、左连接、右连接有什么区别？
>
> 什么是内连接、外连接、交叉连接、笛卡尔积呢？

#### join图

![1640658483243](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/082650-591926.png)


### Sql基础

#### SQL语句主要分为哪几类

1. 数据据定义语言DDL(Data Definition Language):主要有CREATE,DROP,ALTER等对逻辑结构有操作的,包括表结构、视图和索引。
2. 数据库查询语言DQL(Data Query Language):主要以SELECT为主
3. 数据操纵语言DML(Data Manipulation Language):主要包括INSERT,UPDATE,DELETE
4. 数据控制功能DCL(Data Control Language):主要是权限控制能操作,包括GRANT,REVOKE,COMMIT,ROLLBACK等。

#### SQL约束有哪些

- 主键约束:主键为在表中存在一列或者多列的组合,能唯一标识表中的每一行。一个表只有一个主键,并且主键约束的列不能为空。
- 外键约束:外键约束是指用于在两个表之间建立关系,需要指定引用主表的哪一列。只有主表的主键可以被从表用作外键,被约束的从表的列可以不是主键,所以创建外键约束需要先定义主表的主键,然后定义从表的外键。
- 唯一约束:确保表中的一列数据没有相同的值,一个表可以定义多个唯一约束。
- 默认约束:在插入新数据时,如果该行没有指定数据,系统将默认值赋给该行,如果没有设置没默认值,则为NULL。
- Check约束:Check会通过逻辑表达式来判断数据的有效性,用来限制输入一列或者多列的值的范围。在列更新数据时,输入的内容必须满足Check约束的条件

#### 什么是子查询

子查询:把一个查询的结果在另一个查询中使用。

子查询可以分为以下几类:

- 标量子查询:指子查询返回的是一个值,可以使用 =,>,<,>=,<=,<>等操作符对子查询标量结果进行比较,一般子查询会放在比较式的右侧。

```sql
SELECT * FROM user WHERE age = (SELECT max(age) from user) //查询年纪最大的人
```

- 列子查询:指子查询的结果是n行一列,一般应用于对表的某个字段进行查询返回。可以使用IN、ANY、SOME和ALL等操作符,不能直接使用

```sql 
SELECT num1 FROM table1 WHERE num1 > ANY (SELECT num2 FROM table2)
```

- 行子查询:指子查询返回的结果一行n列

```sql 
SELECT * FROM user WHERE (age,sex) = (SELECT age,sex FROM user WHERE name="zhangsan")
```

- 表子查询:指子查询是n行n列的一个数据表

```sql
SELECT * FROM student WHERE (name,age,sex) IN (SELECT name,age,sex FROM class1) --在学生表中找到班级在1班的学生
```

#### 了解MySQL的几种连接查询吗？

MySQl的连接查询主要可以分为外连接,内连接,交叉连接

**外连接**

- 外连接主要分为左外连接(LEFT JOIN)、右外连接(RIGHT JOIN)、全外连接。
- 左外连接:显示左表中所有的数据及右表中符合条件的数据,右表中不符合条件的数据为null。

![1631770721000](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/133844-129480.png)

- 右外连接:显示左表中所有的数据及右表中符合条件的数据,右表中不符合条件的数据为null。

![1631770758813](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/133921-621905.png)

> MySQL中不支持全外连接。

- 内连接:只显示符合条件的数据

![1631770802566](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/134004-950015.png)

- 交叉连接:使用笛卡尔积的一种连接。

  笛卡尔积,百度百科的解释:两个集合X和Y的笛卡尔积表示为X × Y,第一个对象是X的成员而第二个对象是Y的所有可能有序对的其中一个成员 。例如:`A={a,b},B={0,1,2},A × B = {(a,0),(a,1),(a,2),(b,0),(b,1),(b,2)} `

**举例如下:有两张表分为L表和R表。 **

![1631770886115](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/134129-364648.png)

左外连接 : select L.`*`,R.`*` from L left join R on L.b=R.b

![1631770941267](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/134237-74660.png)

右外连接: select L.`*`,R.`*` from L right join R on L.b=R.b

![1631770987482](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/134309-60073.png)

内连接: select L.`*`,R.`*` from L inner join R on L.b=R.b

![1631771057312](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/134420-270738.png)

交叉连接: select L.`*`,R.`*` from L,R

![1631771090537](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/134453-312816.png)

#### mysql中in和exists的区别？

in和exists一般用于子查询。

- 使用exists时会先进行外表查询,将查询到的每行数据带入到内表查询中看是否满足条件;使用in一般会先进行内表查询获取结果集,然后对外表查询匹配结果集,返回数据。
- in在内表查询或者外表查询过程中都会用到索引。
- exists仅在内表查询时会用到索引
- 一般来说,当子查询的结果集比较大,外表较小使用exist效率更高;当子查询寻得结果集较小,外表较大时,使用in效率更高。
- 对于not in和not exists,not exists效率比not in的效率高,与子查询的结果集无关,因为not in对于内外表都进行了全表扫描,没有使用到索引。not exists的子查询中可以用到表上的索引。

#### varchar和char的区别？

- varchar表示变长,char表示长度固定。当所插入的字符超过他们的长度时,在严格模式下,会拒绝插入并提示错误信息,在一般模式下,会截取后插入。如char(5),无论插入的字符长度是多少,长度都是5,插入字符长度小于5,则用空格补充。对于varchar(5),如果插入的字符长度小于5,则存储的字符长度就是插入字符的长度,不会填充。
- 存储容量不同,对于char来说,最多能存放的字符个数为255。对于varchar,最多能存放的字符个数是65532。
- 存储速度不同,char长度固定,存储速度会比varchar快一些,但在空间上会占用额外的空间,属于一种空间换时间的策略。而varchar空间利用率会高些,但存储速度慢,属于一种时间换空间的策略。

#### MySQL中int(10)和char(10)和varchar(10)的区别？

int(10)中的10表示的是显示数据的长度,而char(10)和varchar(10)表示的是存储数据的大小。

#### drop、delete和truncate的区别？

![1631771405632](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/16/135008-639495.png)

> 一般来讲,删除整个表,使用drop,删除表的部分数据使用delete,保留表结构删除表的全部数据使用truncate

#### UNION和UNION ALL的区别？

union和union all的作用都是将两个结果集合并到一起。

- union会对结果去重并排序,union all直接直接返回合并后的结果,不去重也不进行排序。
- union all的性能比union性能好。 


## 一张表,里面有ID自增主键,当insert了17条记录之后,删除了第15,16,17条记录,再把Mysql重启,再insert一条记录,这条记录的ID是18还是15 ？

1. 如果表的类型是MyISAM,那么是18,因为MyISAM表会把自增主键的最大ID记录到数据文件里,重启MySQL自增主键的最大ID也不会丢失
2. 如果表的类型是InnoDB,那么是15,InnoDB表只是把自增主键的最大ID记录到内存中,所以重启数据库或者是对表进行OPTIMIZE操作,都会导致最大ID丢失


## 请简洁描述Mysql中InnoDB支持的四种事务隔离级别名称,以及逐级之间的区别？

SQL标准定义的四个隔离级别为:

- read uncommited :读到未提交数据,在该隔离级别,所有事务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用,因为它的性能也不比其他级别好多少。读取未提交的数据,也被称之为脏读(Dirty Read)。
- read committed:脏读,不可重复读,这是大多数数据库系统的默认隔离级别(但不是MySQL默认的)。它满足了隔离的简单定义:一个事务只能看见已经提交事务所做的改变。这种隔离级别也支持所谓的不可重复读(Nonrepeatable Read),因为同一事务的其他实例在该实例处理其间可能会有新的commit,所以同一select可能返回不同结果。
- repeatable read:可重读,这是MySQL的默认事务隔离级别,它确保同一事务的多个实例在并发读取数据时,会看到同样的数据行。不过理论上,这会导致另一个棘手的问题:幻读(PhantomRead)。简单的说,幻读指当用户读取某一范围的数据行时,另一个事务又在该范围内插入了新行,当用户再读取该范围的数据行时,会发现有新的“幻影” 行。InnoDB和Falcon存储引擎通过多版本并发控制(MVCC,Multiversion Concurrency Control 间隙锁)机制解决了该问题。注:其实多版本只是解决不可重复读问题,而加上间隙锁(也就是它这里所谓的并发控制)才解决了幻读问题。
- serializable :串行事物,这是最高的隔离级别,它通过强制事务排序,使之不可能相互冲突,从而解决幻读问题。简言之,它是在每个读的数据行上加上共享锁。在这个级别,可能导致大量的超时现象和锁竞争。

## 主键和候选键有什么区别？

- 超键(super key): 在关系中能唯一标识元组的属性集称为关系模式的超键
- 候选键(candidate key): 不含有多余属性的超键称为候选键。也就是在候选键中,若再删除属性,就不是键了！
- 主键(primary key): 用户选作元组标识的一个候选键程序主键
- 外键(foreign key):如果关系模式R中属性K是其它模式的主键,那么k在模式R中称为外键。

> 学生信息(学号 身份证号 性别 年龄 身高 体重 宿舍号)和 宿舍信息(宿舍号 楼号)
>
> 超键:只要含有“学号”或者“身份证号”两个属性的集合就叫超键,例如R1(学号 性别)、R2(身份证号 身高)、R3(学号 身份证号)等等都可以称为超键！
>
> 候选键:不含有多余的属性的超键,比如(学号)、(身份证号)都是候选键,又比如R1中学号这一个属性就可以唯一标识元组了,而有没有性别这一属性对是否唯一标识元组没有任何的影响！
>
> 主键:就是用户从很多候选键选出来的一个键就是主键,比如你要求学号是主键,那么身份证号就不可以是主键了！
>
> 外键:宿舍号就是学生信息表的外键
>

## 数据库备份

必须要在未登录状态下:

- 导出整个数据库:`mysqldump -u 用户名 -p 数据库名 > 导出的文件名`
- 导出一张表:`mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名`
- d导出一个数据库结构:`mysqldump -u dbuser -p -d --add-drop-table dbname >d:/dbname_db.sql`,-d 没有数据 --add-drop-table 在每个 create 语句之前增加一个 drop table

