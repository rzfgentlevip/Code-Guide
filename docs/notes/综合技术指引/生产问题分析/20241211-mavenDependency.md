---
title: Maven分析项目依赖
icon: material-icon-theme:webhint
order: 5
author: bugcode
date: 2024-12-11T00:00:00.000Z
category:
  - MAVEN
tag:
  - maven
  - 场景
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 10:13:29
permalink: /compre-guide/production-issues/mavenDependency/
---

# Maven分析项目依赖

## Maven dependency

在项目开发过程中，我们经常会有分析项目依赖的需要，查找jar依赖路径，查找jar冲突等等。这时候，dependency命令会非常的有用，在此我们介绍下关于maven的dependency命令的一些用法。

## `mvn dependency:list` 列出项目的所有jar包

执行mvn dependency:list 命令，会列出项目依赖的所有jar包；

```text
mvn dependency:list

[INFO] The following files have been resolved:
[INFO]    org.springframework.boot:spring-boot-starter:jar:2.7.12:compile -- module spring.boot.starter [auto]
[INFO]    org.springframework.boot:spring-boot:jar:2.7.12:compile -- module spring.boot [auto]
[INFO]    org.springframework:spring-context:jar:5.3.27:compile -- module spring.context [auto]
[INFO]    org.springframework:spring-expression:jar:5.3.27:compile -- module spring.expression [auto]
[INFO]    org.springframework.boot:spring-boot-starter-logging:jar:2.7.12:compile -- module spring.boot.starter.logging [auto]
[INFO]    ch.qos.logback:logback-classic:jar:1.2.12:compile -- module logback.classic (auto)
[INFO]    ch.qos.logback:logback-core:jar:1.2.12:compile -- module logback.core (auto)
[INFO]    org.apache.logging.log4j:log4j-to-slf4j:jar:2.17.2:compile -- module org.apache.logging.slf4j 
```



**mvn dependency:list -Dverbose**

该命令可以列出项目依赖的所有jar包，-Dverbose参数会把被忽略的jar，即相同jar包的不同版本引入也列出来。

```text
[INFO] --- maven-dependency-plugin:3.3.0:list (default-cli) @ dynamic-thread-pool-spring-boot-starter ---
[INFO] 
[INFO] The following files have been resolved:
[INFO]    org.springframework.boot:spring-boot-starter:jar:2.7.12:compile -- module spring.boot.starter [auto]
[INFO]    org.springframework.boot:spring-boot:jar:2.7.12:compile -- module spring.boot [auto]
[INFO]    org.springframework:spring-context:jar:5.3.27:compile -- module spring.context [auto]
[INFO]    org.springframework:spring-expression:jar:5.3.27:compile -- module spring.expression [auto]
[INFO]    org.springframework.boot:spring-boot-starter-logging:jar:2.7.12:compile -- module spring.boot.starter.logging [auto]
[INFO]    ch.qos.logback:logback-classic:jar:1.2.12:compile -- module logback.classic (auto)
[INFO]    ch.qos.logback:logback-core:jar:1.2.12:compile -- module logback.core (auto)
```

## `mvn dependency:tree` 列出项目的包依赖树

这个命令跟上一个命令的区别就是，这个命令的依赖，输出来是个树，更方便看依赖关系。

```text
mvn dependency:tree

[INFO] --- maven-dependency-plugin:3.3.0:tree (default-cli) @ dynamic-thread-pool-spring-boot-starter ---
[INFO] bugcode.online:dynamic-thread-pool-spring-boot-starter:jar:1.0
[INFO] +- org.springframework.boot:spring-boot-starter:jar:2.7.12:compile
[INFO] |  +- org.springframework.boot:spring-boot:jar:2.7.12:compile
[INFO] |  |  \- org.springframework:spring-context:jar:5.3.27:compile
[INFO] |  |     \- org.springframework:spring-expression:jar:5.3.27:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter-logging:jar:2.7.12:compile
[INFO] |  |  +- ch.qos.logback:logback-classic:jar:1.2.12:compile
[INFO] |  |  |  \- ch.qos.logback:logback-core:jar:1.2.12:compile
[INFO] |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.17.2:compile
[INFO] |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.17.2:compile
[INFO] |  |  \- org.slf4j:jul-to-slf4j:jar:1.7.36:compile
[INFO] |  +- jakarta.annotation:jakarta.annotation-api:jar:1.3.5:compile
[INFO] |  +- org.springframework:spring-core:jar:5.3.27:compile
[INFO] |  |  \- org.springframework:spring-jcl:jar:5.3.27:compile
[INFO] |  \- org.yaml:snakeyaml:jar:1.30:compile
```

`mvn dependency:tree -Dverbose`
`-Dverbose`参数会把被忽略的jar，即相同jar包的不同版本引入也列出来。

`dependency:tree`有几个比较重要的参数，非常有用:

- includes,说明：该参数可以列出指定要求的jar，其他的忽略
    - 示例：`-Dincludes=velocity:velocity`，只列出velocity的依赖关系
    - 参数值：`[groupId]:[artifactId]:[type]:[version]`，参数格式就是这样的，没有的值可以留空，
    - 举例`-Dincludes=:spring-aop`，`-Dincludes=:::5.0.6.RELEASE`，`-Dincludes=org.springframework`
    - 通配符：在参数中可以使用通配符，例如`org.apache.*`, `:::*-SNAPSHOT`
    - 多个参数值：参数后面可以跟多个参数值，以英文逗号分隔，举例`-Dincludes=org.apache.maven*,org.codehaus.plexus`
- excludes
    - 说明：该参数的用法跟`includes`是一样的，不过这个参数的作用是排除指定的jar

### 依赖分析案例

我们以spring-boot-starter来分析其依赖关系:

```
指定具体的坐标分析依赖
mvn dependency:tree -Dverbose -Dincludes=org.springframework.boot:spring-boot-starter
只指定artifactId分析依赖
mvn dependency:tree -Dverbose -Dincludes=:spring-boot-starter

分析结果如下:
[INFO] bugcode.online:dynamic-thread-pool-spring-boot-starter:jar:1.0
[INFO] +- org.springframework.boot:spring-boot-starter:jar:2.7.12:compile
[INFO] +- org.springframework.boot:spring-boot-starter-aop:jar:2.7.12:compile
[INFO] |  \- (org.springframework.boot:spring-boot-starter:jar:2.7.12:compile - version managed from 2.7.12; omitted for duplicate)
[INFO] +- org.springframework.boot:spring-boot-starter-test:jar:2.7.12:test
[INFO] |  \- (org.springframework.boot:spring-boot-starter:jar:2.7.12:test - version managed from 2.7.12; omitted for duplicate)
[INFO] \- org.redisson:redisson-spring-boot-starter:jar:3.26.0:compile
[INFO]    +- org.springframework.boot:spring-boot-starter-actuator:jar:2.7.12:compile (version managed from 3.2.0)
[INFO]    |  \- (org.springframework.boot:spring-boot-starter:jar:2.7.12:compile - version managed from 2.7.12; omitted for duplicate)
[INFO]    \- org.springframework.boot:spring-boot-starter-data-redis:jar:2.7.12:compile (version managed from 3.2.0)
[INFO]       \- (org.springframework.boot:spring-boot-starter:jar:2.7.12:compile - version managed from 2.7.12; omitted for duplicate)

```

此处一定不要省略`-Dverbose`参数，要不然是不会显示被忽略的包

## `dependency:analyze-only`分析依赖

`dependency:analyze-only`命令可以分析整个项目，并且找出项目中依赖有如下情况的：

- 声明了并且使用了的依赖
- 没有声明但是使用了的依赖
- 声明了但是没有使用的依赖

输出示例：

```text
[INFO] --- maven-dependency-plugin:3.3.0:analyze-only (default-cli) @ dynamic-thread-pool-spring-boot-starter ---
[WARNING] Used undeclared dependencies found:
[WARNING]    org.slf4j:slf4j-api:jar:1.7.36:compile
[WARNING]    org.redisson:redisson:jar:3.26.0:compile
[WARNING]    org.springframework:spring-core:jar:5.3.27:compile
[WARNING]    org.springframework:spring-context:jar:5.3.27:compile
[WARNING]    org.springframework.boot:spring-boot:jar:2.7.12:compile
[WARNING] Unused declared dependencies found:
[WARNING]    org.springframework.boot:spring-boot-starter:jar:2.7.12:compile
[WARNING]    org.springframework.boot:spring-boot-configuration-processor:jar:2.7.12:compile
[WARNING]    org.springframework.boot:spring-boot-autoconfigure:jar:2.7.12:compile
[WARNING]    org.springframework.boot:spring-boot-starter-aop:jar:2.7.12:compile
[WARNING]    org.springframework.boot:spring-boot-starter-test:jar:2.7.12:test
[WARNING]    junit:junit:jar:4.13.1:test
[WARNING]    org.redisson:redisson-spring-boot-starter:jar:3.26.0:compile
[WARNING]    org.projectlombok:lombok:jar:1.18.4:compile
[INFO] ------------------------------------------------------------------------
```

需要注意的是，如果你要查看声明了并且使用了的依赖，必须加上参数`-Dverbose`。

## `dependency:analyze-duplicate`分析 and

这个命令会查找`<dependencies/> 和 <dependencyManagement/>`中重复声明的依赖

```text
[INFO] --- maven-dependency-plugin:3.3.0:analyze-duplicate (default-cli) @ dynamic-thread-pool-spring-boot-starter ---
[INFO] No duplicate dependencies found in <dependencies/> or in <dependencyManagement/>
```

## `dependency:list-repositories` 列出所有的远程repositories

命令: `mvn dependency:list-repositories`

```text
[INFO] --- maven-dependency-plugin:3.3.0:list-repositories (default-cli) @ dynamic-thread-pool-spring-boot-starter ---
[INFO] Repositories used by this build:
[INFO]        id: sonatype-nexus-snapshots
      url: https://oss.sonatype.org/content/repositories/snapshots
   layout: default
snapshots: [enabled => true, update => daily]
 releases: [enabled => false, update => daily]

[INFO]        id: sonatype-releases
      url: https://oss.sonatype.org/content/repositories/releases
   layout: default
snapshots: [enabled => true, update => daily]
 releases: [enabled => true, update => daily]
```

## `dependency:purge-local-repository` 清理本地repository

这个命令的会首先解析整个项目的依赖，然后从本地repository中清理这些依赖，重新从远程repository下载。

- 直接依赖
    - 有一点要说清楚，这个命令默认的对所有的依赖项进行操作。所以它会在清除操作之前，下载某些缺失的依赖来收集完整的依赖树信息。为了避免这些预下载的操作，你可以设置参数`-DactTransitively=false`，仅对项目的直接依赖进行操作。
- 指定/排除依赖
    - 你也可以有针对性的只操作某些包，需要添加参数`-Dincludes`，明确的声明包，这个是可以传多个值的，用英文逗号分隔，举例:`dependency:purge-local-repository -Dincludes=org.slf4j:slf4j-api,org.slf4j:log4j-over-slf4j`。`-Dexcludes`也是一样的道理，只不过是排除某些依赖。
- 自定义清理
    - 如果你想清理不在本项目中的依赖，也可以使用这个，不过参数是不一样的。`mvn dependency:purge-local-repository -DmanualIncludes=org.apache:apache`，参数`-DmanualInclude`可以让你清理不在本项目中的依赖，但是不会重新解析依赖了，因为本项目不需要这些依赖。这个对清理parent pom，导入的pom，maven插件非常有用。