---
title: Spark 开发规范
icon: emojione-v1:straight-ruler
order: 5
author: bugcode
date: 2024-11-19T00:00:00.000Z
category:
  - PROBLEM
tag:
  - problem
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 10:13:29
permalink: /compre-guide/production-issues/DevelopmentExperience/
---

# 1、Join规范

1. Spark sql中Join默认是inner join。
2. 如果join的字段在两个表中都有，那么可以使用Seq(“字段名”)指定join的字段，这样两个表的公共字段不需要做去重处理。
    1. 如果join的两个表字段不一致，可以使用tabNm(“col1”)===tabNm(“col2”)这种方式进行不同字段之间的join操作，但是在结果表中可能回产生重复列，所以需要对结果表中的列进行删除处理，如果drop某一列，那么结果表中如果有多个列明一样的列，都会删除掉，所以在join之前，尽量将另一个表中的字段重命名，保证join后不会产生重复列。
3. Join两个表的on条件，尽量直接写在on条件里，不要将条件提升到where过滤后面，将条件写在on后面，不会影响整个结果表的数据量，但是回应i想右表的数据量，加快join速度，但是将条件写在where过滤后面，回影响结果表中的数据量，也就是影响左表的数据量。
4. Select表达式selectExpr，在使用select的时候，要求select里面的列类型相同，比如字符串，可以使用selectExpr给所有列进行重命名，方便之后表之间做子连接，自连接会产生重复列。

# **2、函数使用规范**

## 1.2、时间函数规范

### 1.2.1、**spar计算时间差精度问题**

spark不支持毫秒级别精度，比如2023-04-04 09:23:56.329和2023-04-04 10:46:36.789在计算时间差的时候，只会计算到秒级别的时间差，毫秒会自动截取，对于精确计算时间差值的场景，在spark中可以使用以下方法计算毫秒级时间差。

```java
.withColumn("strtMills",
	when(instr(col("strt_tm"),".") === 0,0)
	.otherwise(rpad(subing_index(col("strt_tm"),".",-1),3,"0").cast("float")/1000)

函数说明：
instr(col("strt_tm"),".")函数：
def instr(str:Column,subString:String):Column
函数会判断字串subString在字符串str中是否出现，如果出现，就返回1，否则就返回0

subing_index(col("strt_tm"),".",-1)

subing_index(str:String,delim:String,count:Int):Column
截取字符串str第count个分隔符之前的字符串。如果count为正，则从左边开始截取。如果count为负，则从右边开始截取。

rpad(subing_index(col("strt_tm"),".",-1),3,"0")
rpad(str:Column,len:Int,pad:String):Column
函数对字符串str从右向左填充0字符，保证字符串的长度为3.
```

### 1.2.2、**时间日期函数**

1. unix_timestamp()函数返回标准unix时间戳，标准格式是：yyyy-MM-dd HH:mm:ss，`unix_timestamp('2020-11-25','yyyy-MM-dd HH:mm:ss'),指定时间格式。`
2. from_unixtime()将时间戳换算成当前时间，可自定义输出日期格式，若不定义则为默认`yyyy-MM-dd HH:mm:ss`，`select from_unixtime('1606303452')`。
3. 日期转换函数，to_date()函数可以将日期时间戳转换为日期，格式为yyyy-MM-dd标准格式，`select to_date('2020-11-25','yyyy-MM-dd')`。
4. hour()，minute()，second()函数可以从时间戳中提取到时分秒数值，提取出来的是两位数形式。
5. 从数据源查询时间类型的数据，需要做时区转换：（mkt_data_upd_tm AT TIME ZONE ‘UTC-8’）

## 1.3、字符串操作

1. 强转字符串类型

```java
buy_stlmnt_spd_nm::varchar  //::，postgre中的类型墙砖符号
```

## 1.4、spark对null类型的支持’

有时候需要给dataframe中某一列的空类型字段设置为null类型，但是spark中不支持直接写为null，所以可以使用selectExpr()表达式设置为null类型。

```java
withColumn("col",lit("null"))// 错误写法，报错spark不支持的null类型'

//正确写法
selectExpr("null as col") // col列的类型是null类型
```

# **3、编码规范**

对于常量，单个字符或数值类型，使用col(“”)将其转换为Column类型在做计算。

代码中的大小写要统一，一般情况使用大写形式。

常量尽量定义在包文件中，然后在需要的地方引用即可。

长的sqll代码使用”””sql”””包裹书写。

将数据保存到hbase的时候，hbase中的列明要和最终select里面的列明一样。

postgre数据库中做字符串拼接操作

```java
ordr_cd || '_' || row_number(partition by ... order by ...)
```

# Spark任务执行缓慢

## 1.4、任务执行缓慢排查及解决办法

大概率是发生数据倾斜，详细解决办法参考下列文档：

### **1.4、设计规范**

## 5、**开发步骤总结**

## **2.1、需求理解**

## **2.2、需求评审**

## **2.3、模型设计**

## **2.4、编码开发**

## **2.5、服务测试**

测试升级前找广亮哥review代码

找治昊review代码逻辑

## **2.6、手册编写**

概要设计文档撰写

应急手册撰写

交付物撰写

Jar包

初始化脚本

创建模型脚本