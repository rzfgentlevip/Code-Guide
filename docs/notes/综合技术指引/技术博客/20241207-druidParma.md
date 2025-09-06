---
title: Druid参数说明
icon: material-icon-theme:webhint
order: 5
author: bugcode
date: 2024-12-07T00:00:00.000Z
category:
  - DRUID
tag:
  - druid
  - 场景
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 10:31:15
permalink: /compre-guide/technical-blog/druidParma/
---


# Druid参数说明

## KeepAlive

- 连接池中的minIdle数量以内的连接，空闲时间超过minEvictableIdleTimeMillis，则会执行keepAlive操作。

- keepAlive是Druid用来保持连接有效性的，只有空闲时间大于keepAliveBetweenTimeMillis(默认2分钟)并且小于minEvictableIdleTimeMillis该参数才会有用；

- 当连接的空闲时间大于keepAliveBetweenTimeMillis(默认2分钟)，但是小于minEvictableIdleTimeMillis(默认30分钟)，Druid会通过调用validationQuery保持该连接的有效性。

  ***当连接空闲时间大于minEvictableIdleTimeMillis，Druid会直接将该连接关闭，keepAlive会无效。\***



## timeBetweenEvictionRunsMillis

**timeBetweenEvictionRunsMillis**默认值是60s,主要作用在两处地方

作为DestroyTask执行的时间周期，DestroyTask主要有两个作用：

- 判断连接池的连接空闲数是否大于minIdle，如果是则关闭多余的连接，反之则新增连接
- 回收连接池泄露的连接，removeAbandoned。

作为验证连接是否有效的时间周期，如果testOnBorrow==false并且testWhileIdle==true,则在应用获取连接的时候会判断连接的空闲时间是否大于timeBetweenEvictionRunsMillis，如果大于则会验证该连接是否有效。



- timeBetweenEvictionRunsMillis可以用来控制空闲连接数的回收周期
- timeBetweenEvictionRunsMillis可以用来控制回收泄露连接的周期
- timeBetweenEvictionRunsMillis连接的空闲时间大于该值testWhileIdle才起作用



## InitialSize

**initialSize**：连接池初始化时初始化的数据库连接数

initialSize在哪个阶段会起作用？

当项目第一次进行增，删，改，查的时候，连接池会初始化，这个时候会根据initialSize参数初始化数据库连接放入连接池中。

> initialSize的作用是告诉连接池初始化时应该初始化的物理连接数，要注意的是这个值越大，第一次调用数据库时越慢。



## minIdle

**minIdle**：连接池中的最小空闲连接数，Druid会定时扫描连接池的连接，如果空闲的连接数大于该值，则关闭多余的连接，反之则创建更多的连接以满足最小连接数要求。

为什么要设置这个参数？

设置这个参数可以应对突发流量，如果没有设置空闲连接，当有多个请求同时调用数据库，但是连接池中并没有可用连接，这时就必须创建连接，创建连接是一个非常耗时的操作，有可能会导致请求超时。

minIdle是怎么起作用的？

当连接池初始化时，会初始化一个定时清除空闲连接的任务DestroyTask，该任务默认是1分钟执行一次(使用**timeBetweenEvictionRunsMillis**参数设置)，定时任务清除空闲连接会进行两个判断：

1. 空闲时间大于minEvictableIdleTimeMillis(默认30分钟)，并且空闲连接数大于minIdle；
2. 空闲时间大于maxEvictableIdleTimeMillis(默认7小时)；

minIdle的意义是预防突发流量；

数据源会在DestroyTask定时检查池中的空闲连接，会关闭多余的连接或者创建新的连接。

数据源并不能确保空闲连接一定等于minIdle，可以通过设置timeBetweenEvictionRunsMillis参数调整定时任务的执行间隔，从而控制数据源中空闲连接数更加接近minIdle；



## maxActive

**maxActive**：连接池中的最大连接数，连接池中的连接包含3部分：

1. **activeCount**：正在使用的连接；
2. **poolingCount**：连接池中的空闲连接；
3. **createTaskCount**：正在生成的连接；

这三部分的连接总和不能超过maxActive；

为什么要设置这个参数？

数据库的连接总数是有限制的，有时候僧多粥少，只能限制每个应用的连接数了。

maxActive是怎么起作用的？

maxActive在Druid中有多处使用，最主要的一处是在CreateConnectionTask中

## maxWait

maxWait是什么意思？

**maxWait**：从连接池中获取连接的最大等待时间，单位ms，默认-1，即会一直等待下去

为什么要设置这个参数？

笔者在使用Druid时都会设置这个参数，这样如果是获取连接超时，更容易从日志中获取调用失败的原因。

maxWait默认是不超时，即如果连接池没有空闲连接，则会一直等待下去，但是一般的接口都是有超时时间的，如果接口超时，不方便定位出来是获取不到连接导致的，最好设置maxWait，并且小于接口的超时时间。

## removeAbandoned

## removeAbandoned，logAbandoned，removeAbandonedTimeoutMillis是什么意思？

**removeAbandoned**：如果连接泄露，是否需要回收泄露的连接，默认false；
**logAbandoned**：如果回收了泄露的连接，是否要打印一条log，默认false；
**removeAbandonedTimeoutMillis**：连接回收的超时时间，默认5分钟；

举个栗子：笔者当初做过单个事务使用多数据源的功能，重写了Spring的ConnectionHolder，由于笔者的疏忽，每次从ConnectionHolder中获取连接时都获取到了两条连接，但是只是使用了其中的一条，相当于另一条连接只是从连接池中拿出来了，但是再也不会还回去了，这样就导致了连接池中的连接很快就消耗光了，即activeCount=maxActive。

如果当时我设置了removeAbandoned就不会出现这个问题，应该Druid会定期检查池中借出去的连接是否处于运行状态，如果不是处于运行状态，并且借出时间超过**removeAbandonedTimeoutMillis**(默认5分钟)就会回收该连接。

连接是怎么回收的？

Druid每隔**timeBetweenEvictionRunsMillis**(默认1分钟)会调用DestroyTask，在这里会判断是否可以回收泄露的连接

## minEvictableIdleTim

## 连接池是怎么判断一条连接是Idle状态的？

就是通过这两个参数进行判断的

- **minEvictableIdleTimeMillis**：最小空闲时间，默认30分钟，如果连接池中非运行中的连接数大于minIdle，并且那部分连接的非运行时间大于minEvictableIdleTimeMillis，则连接池会将那部分连接设置成Idle状态并关闭；也就是说如果一条连接30分钟都没有使用到，并且这种连接的数量超过了minIdle，则这些连接就会被关闭了。
- **maxEvictableIdleTimeMillis**：最大空闲时间，默认7小时，如果minIdle设置得比较大，连接池中的空闲连接数一直没有超过minIdle，这时那些空闲连接是不是一直不用关闭？当然不是，如果连接太久没用，数据库也会把它关闭，这时如果连接池不把这条连接关闭，系统就会拿到一条已经被数据库关闭的连接。为了避免这种情况，Druid会判断池中的连接如果非运行时间大于maxEvictableIdleTimeMillis，也会强行把它关闭，而不用判断空闲连接数是否小于minIdle；

minEvictableIdleTimeMillis连接空闲时间大于该值并且池中空闲连接大于minIdle则关闭该连接

maxEvictableIdleTimeMillis连接空闲时间大于该值，不管minIdle都关闭该连接

## testOnBorrow

**testOnBorrow**：如果为true(默认false)，当应用向连接池申请连接时，连接池会判断这条连接是否是可用的。

testOnBorrow什么时候会用到？

如果是常用的数据库，则使用${DBNAME}ValidConnectionChecker进行判断，比如Mysql数据库，使用MySqlValidConnectionChecker的isValidConnection进行判断；

如果是其他数据库，则使用validationQuery判断；

testOnBorrow能够确保我们每次都能获取到可用的连接，但如果设置成true，则每次获取连接的时候都要到数据库验证连接有效性，这在高并发的时候会造成性能下降，可以将testOnBorrow设成false，testWhileIdle设置成true这样能获得比较好的性能。

## testOnReturn

**testOnReturn**：如果为true(默认false)，当应用使用完连接，连接池回收连接的时候会判断该连接是否还可用。

testOnReturn什么时候会用到？

当连接使用完，调用commit或者rollback方法后，连接池会回收该连接，该参数主要在DruidDataSource的recycle方法中用到

## testWhileIdle

testWhileIdle是什么意思？

**testWhileIdle**：如果为true(默认true)，当应用向连接池申请连接，并且testOnBorrow为false时，连接池将会判断连接是否处于空闲状态，如果是，则验证这条连接是否可用。

testWhileIdle什么时候会起作用？

1. 获取连接时；
2. testOnBorrow==false;
3. testWhileIdle==true;

使用代码在DruidDataSource的getConnectionDirect方法
***注意：此时判断连接空闲的依据是空闲时间大于timeBetweenEvictionRunsMillis(默认1分钟)，并不是使用minEvictableIdleTimeMillis跟maxEvictableIdleTimeMillis\***，*也就是说如果连接空闲时间超过一分钟就测试一下连接的有效性，但并不是直接剔除；而如果空闲时间超过了minEvictableIdleTimeMillis则会直接剔除。*

testWhileIdle的作用跟testOnBorrow是差不多的，都是在获取连接的时候测试连接的有效性，如果两者都为true，则testOnBorrow优先级高，则不会使用到testWhileIdle。

## validationQuery

validationQuery是什么意思？

**validationQuery**：Druid用来测试连接是否可用的SQL语句,默认值每种数据库都不相同：
Mysql:SELECT 1;
SQLSERVER:SELECT 1;
ORACLE:SELECT 'x' FROM DUAL;
PostGresql:SELECT 'x';

validationQuery什么时候会起作用？

当Druid遇到testWhileIdle，testOnBorrow，testOnReturn时，就会验证连接的有效性，验证规则如下：
如果有相关数据库的ValidConnectionChecker，则使用ValidConnectionChecker验证(Druid提供常用数据库的ValidConnectionChecker，包括MSSQLValidConnectionChecker，MySqlValidConnectionChecker，OracleValidConnectionChecker，PGValidConnectionChecker)；

如果没有ValidConnectionChecker，则直接使用validationQuery验证；

ValidConnectionChecker是如何验证的？

MySqlValidConnectionChecker会使用Mysql独有的ping方式进行验证，其他数据库其实也都是使用validationQuery进行验证

- 不同数据库的默认值不同；
- 如果是Mysql数据库，则validationQuery不会起作用，Mysql会使用ping的方式验证；