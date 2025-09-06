---
title: Linux命令手册
icon: material-icon-theme:folder-command
order: 1
author: bugcode
date: 2020-01-01T00:00:00.000Z
category:
  - LINUX
  - JAVA
tag:
  - linux
sticky: false
star: true
footer: maven
copyright: bugcode
createTime: 2025/09/04 14:06:06
permalink: /learning-notes/linux/basic/
---

<!-- TOC -->

- [Linux命令手册](#linux命令手册)
  - [基础命令](#基础命令)
    - [复制拷贝](#复制拷贝)
    - [查看ip地址](#查看ip地址)
    - [防火墙](#防火墙)
    - [开关机](#开关机)
    - [切换用户](#切换用户)
    - [设置普通用户权限](#设置普通用户权限)
    - [修改文件所属](#修改文件所属)
    - [**帮助**命令](#帮助命令)
    - [help **获得**shell内置命令的帮助信息](#help-获得shell内置命令的帮助信息)
    - [Linux目录结构](#linux目录结构)
    - [用户管理](#用户管理)
  - [vim编辑器](#vim编辑器)
    - [概述](#概述)
    - [一般模式](#一般模式)
    - [编辑模式](#编辑模式)
    - [指令模式](#指令模式)
  - [grep用法](#grep用法)
    - [基础用法](#基础用法)
    - [grep基本查找匹配](#grep基本查找匹配)
    - [正则表达式](#正则表达式)
  - [文件目录类操作](#文件目录类操作)
    - [PWD](#pwd)
    - [ls 列出目录内容](#ls-列出目录内容)
    - [mkdir](#mkdir)
    - [cd 切换目录](#cd-切换目录)
    - [cp文件复制](#cp文件复制)
    - [rm删除文件或者目录](#rm删除文件或者目录)
    - [mv](#mv)
  - [查看文件内容](#查看文件内容)
    - [cat](#cat)
    - [tac](#tac)
    - [more](#more)
    - [less:查看文件内容](#less查看文件内容)
    - [head:查看文件内容：](#head查看文件内容)
    - [tail:查看文件内容](#tail查看文件内容)
    - [nl](#nl)
    - [重定向文件](#重定向文件)
    - [echo](#echo)
    - [ln软连接](#ln软连接)
  - [时间日期类](#时间日期类)
  - [用户组命令](#用户组命令)
    - [groupadd 新增组](#groupadd-新增组)
    - [groupdel删除组](#groupdel删除组)
    - [groupmod修改组](#groupmod修改组)
    - [cat  /etc/group 查看创建了哪些组](#cat--etcgroup-查看创建了哪些组)
    - [案例演示](#案例演示)
    - [查看用户所属用户组](#查看用户所属用户组)
  - [用户管理](#用户管理-1)
    - [useradd 添加新用户](#useradd-添加新用户)
    - [password设置用户密码](#password设置用户密码)
    - [id判断用户是否存在](#id判断用户是否存在)
    - [su切换用户](#su切换用户)
    - [userdel删除用户](#userdel删除用户)
    - [who查看登录信息](#who查看登录信息)
    - [设置普通用户具有root权限](#设置普通用户具有root权限)
    - [cat  /etc/passwd 查看创建了哪些组](#cat--etcpasswd-查看创建了哪些组)
    - [usermod修改用户](#usermod修改用户)
  - [文件权限](#文件权限)
    - [文件属性](#文件属性)
    - [chomd修改文件权限](#chomd修改文件权限)
  - [磁盘分区类](#磁盘分区类)
    - [基本用法](#基本用法)
    - [查看分区情况](#查看分区情况)
  - [搜索查找类](#搜索查找类)
    - [find命令](#find命令)
    - [grep在文件内搜索字符匹配的行并且输出](#grep在文件内搜索字符匹配的行并且输出)
    - [which](#which)
    - [whereis查找](#whereis查找)
    - [基础语法](#基础语法)
    - [Locate查找](#locate查找)
    - [查找总结](#查找总结)
  - [进程线程类](#进程线程类)
    - [使用ps -aux](#使用ps--aux)
    - [使用ps -elf](#使用ps--elf)
    - [使用pstree -aup命令来查看](#使用pstree--aup命令来查看)
    - [top：查看系统健康状态](#top查看系统健康状态)
    - [查看系统进程中所有的线程：](#查看系统进程中所有的线程)
    - [pstree：查看进程树](#pstree查看进程树)
    - [kill:杀死进程](#kill杀死进程)
  - [网络命令](#网络命令)
    - [基础语法](#基础语法-1)
      - [列出所有端口信息](#列出所有端口信息)
    - [列出所有tcp端口](#列出所有tcp端口)
    - [列出所有监听端口](#列出所有监听端口)
    - [禁止域名解析](#禁止域名解析)
    - [netstat:显示网络统计信息情况](#netstat显示网络统计信息情况)
    - [查看端口占用情况](#查看端口占用情况)
    - [找出应用的端口](#找出应用的端口)
  - [解压类](#解压类)
    - [gzip/gunzip压缩](#gzipgunzip压缩)
    - [zip/unzip压缩](#zipunzip压缩)
    - [tar打包](#tar打包)
  - [后台服务管理](#后台服务管理)
    - [service后台服务管理](#service后台服务管理)
    - [chkconfig:设置后台服务自启配置](#chkconfig设置后台服务自启配置)
  - [crond系统定时服务](#crond系统定时服务)
    - [crond服务管理：](#crond服务管理)
    - [crontab定时任务设置](#crontab定时任务设置)
  - [软件包管理](#软件包管理)
    - [RPM概述](#rpm概述)
    - [RPM查询命令（rpm -qa）](#rpm查询命令rpm--qa)
    - [**RPM卸载命令（rpm -e）**](#rpm卸载命令rpm--e)
    - [**RPM安装命令（rpm -ivh）**](#rpm安装命令rpm--ivh)
  - [shell编程](#shell编程)
    - [概述](#概述-1)
    - [shell脚本的执行方式](#shell脚本的执行方式)
    - [yum仓库配置](#yum仓库配置)

<!-- /TOC -->


# Linux命令手册

| Tab            | 命令或路径等的补全键                          |
| -------------- | --------------------------------------------- |
| Ctrl + A       | 光标回到命令首行                              |
| Ctrl + E       | 光标回到命令行尾                              |
| Ctrl + F       | 光标向右移动一个字符，相当于方向键→           |
| Ctrl + B       | 光标向左移动一个字符，相当于方向键←           |
| Ctrl + Insert  | 复制命令行内容                                |
| Shift + Insert | 粘贴命令行内容                                |
| Ctrl + K       | 剪切(删除)光标处到行尾的字符                  |
| Ctrl + U       | 剪切(删除)光标处到行首的字符                  |
| Ctrl + W       | 剪切(删除)光标前的一个单词                    |
| Ctrl + Y       | 粘贴剪切(删除)的文字/粘贴最后一次被删除的单词 |
| Ctrl + C       | 中断终端正在执行的任务或者删除整行            |
| Ctrl + H       | 删除光标所在处的前一个字符，相当于退格键      |
| Ctrl + R       | 搜索命令行使用过的历史命令记录                |
| Ctrl + G       | 退出搜索历史命令模式                          |
| Ctrl + L       | 清除所有内容，相当于clear命令                 |
| Ctrl + S       | 锁定终端，使其无法输入内容                    |
| Ctrl + Q       | 解锁锁定状态                                  |
| Ctrl + Alt + d | 挂起当前会话窗口                              |
|                |                                               |

## 基础命令

### 复制拷贝

拷贝文件命令：cp 拷贝文件的目录 拷贝到的文件路径

```text
cp  etc/hadoop/*.xml input/   # 把etc/hadoop/*.xml路径下所有xml结尾的文件拷贝到input/目录下面
```

### 查看ip地址

- ifconfig:查看ip地址
- hostname -i:查看所有ip地址，不会显示详细信息
- hostname -I

### 防火墙

不管是云主机还是虚拟机，都需要把防火墙进行关闭，否则无法使用远程链接。

```text
1）基本语法：
service iptables status	（功能描述：查看防火墙状态）
service iptables start   启动防火墙
chkconfig iptables –list	（功能描述：查看防火墙开机启动状态）（双横线）
service iptables stop	（功能描述：临时关闭防火墙）
chkconfig iptables off	（功能描述：关闭防火墙开机启动）
chkconfig iptables on	（功能描述：开启防火墙开机启动）
service iptables restart  保存退出后重启防火墙
```

**`firewall`防火墙**

```text
systemctl status firewalld 查看firewall服务状态
	出现Active: active (running)切高亮显示则表示是启动状态。
	出现 Active: inactive (dead)灰色表示停止，看单词也行。
 firewall-cmd --state 查看firewall的状态
 开启、重启、关闭、firewalld.service服务
 	service firewalld start 开启
 	service firewalld restart 重启
 	service firewalld stop 停止
 	firewall-cmd --list-all 查看防火墙规则
 查询、开放、关闭端口
 	firewall-cmd --query-port=8080/tcp 查询端口是否开放
 	firewall-cmd --permanent --add-port=80/tcp 开放80端口
 	firewall-cmd --permanent --remove-port=8080/tcp  移除端口
 	firewall-cmd --reload 重启防火墙(修改配置后要重启防火墙)
# 参数解释
1、firwall-cmd：是Linux提供的操作firewall的一个工具；
2、--permanent：表示设置为持久；
3、--add-port：标识添加的端口；
```

### 开关机

在linux领域内大多用在服务器上，很少遇到关机的操作。毕竟服务器上跑一个服务是永无止境的，除非特殊情况下，不得已才会关机 

正确的关机流程为：sync > shutdown > reboot > halt

```text
1）基本语法：

	（1）sync （功能描述：将数据由内存同步到硬盘中）

	（2）shutdown [选项] 时间	

			选项：

			-h：关机

			-r：重启

	（3）halt 			（功能描述：关闭系统，等同于shutdown –h now 和 poweroff）

	（4）reboot 			（功能描述：就是重启，等同于 shutdown –r now）
```

**案例**

```text
//将数据由内存同步到硬盘中
sync  
//计算机将在10分钟后关机，并且会显示在登录用户的当前屏幕中
shutdown –h 10 ‘This server will shutdown after 10 mins’
//立马关机
shutdown –h now 
//系统立马重启
shutdown –r now
//重启（等同于 shutdown –r now）
reboot 
//关机（等同于shutdown –h now 和 poweroff）
halt 
//注意：不管是重启系统还是关闭系统，首先要运行sync命令，把内存中的数据写到磁盘中。
```

- 关机：halt
- 重启：reboot
- 把内存里面的内容加载到硬盘：sync

### 切换用户

切换用户：su 用户名，切换其他用户

su root:切换到root用户

### 设置普通用户权限

允许普通用户具有root权限

```text
编辑配置文件：vi /etc/sudoers

找到 %wheel        ALL=(ALL)       ALL

rzf  ALL=(ALL)       ALL即让rzf用户具有root权限
```

### 修改文件所属

- 修改文件所属的用户：chown 用户名 文件名

- chown -R 用户名:用户权限 文件名

例如：chown -R rui:+x hadoop

ls -al:查看文件所属的用户

修改创建文件的账户和拥有者：sudo chown 账户名:拥有者 文件名

### **帮助**命令

man获取帮助信息

```text
1）基本语法：

man [命令或配置文件]（功能描述：获得帮助信息）

（1）显示说明

NAME  命令的名称和单行描述

SYNOPSIS 怎样使用命令

DESCRIPTION 命令功能的深入讨论

EXAMPLES  怎样使用命令的例子

SEE ALSO  相关主题（通常是手册页）
```

**数字说明**

1. 用户在shell环境中可以操作的命令或是可执行的文件

2. 系统内核(kernel)可以调用的函数

3. 常用的函数or函数库

4. 设备配置文件

5. 配置文件的格式

6. 游戏相关

7. linux网络协议和文件系统

8. 系统管理员可以用的命令

9. 跟内核有关系的文件

### help **获得**shell内置命令的帮助信息

​	1）基本语法：

​	help 命令	（功能描述：获得shell内置命令的帮助信息）

### Linux目录结构

![](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/082838-590394.jpeg)



目录介绍：

- `/bin`：是`Binary`的缩写，这个目录存放着系统必备执行命令
- `/boot`：这里存放的是启动`Linux`时使用的一些核心文件，包括一些连接文件以及镜像文件，自己的安装别放这里
- `/dev`：`Device`(设备)的缩写，该目录下存放的是`Linux`的外部设备，在`Linux`中访问设备的方式和访问文件的方式是相同的。
- `/etc`：所有的系统管理所需要的配置文件和子目录。
- `/home`：存放普通用户的主目录，在`Linux`中每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。
- `/lib`：系统开机所需要最基本的动态连接共享库，其作用类似于`Windows`里的`DLL`文件。几乎所有的应用程序都需要用到这些共享库。
- `/lost+found`：这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。
- `/media`：`linux`系统会自动识别一些设备，例如U盘、光驱等等，当识别后，`linux`会把识别的设备挂载到这个目录下。
- `/misc`: 该目录可以用来存放杂项文件或目录，即那些用途或含义不明确的文件或目录可以存放在该目录下。
- `/mnt`：系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在`/mnt/`上，然后进入该目录就可以查看光驱里的内容了。
- `/net` :存放着和网络相关的一些文件.
- `/opt`：这是给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。
- `/proc`：这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。
- `/root`：该目录为系统管理员，也称作超级权限者的用户主目录。`
- `/sbin`：s就是`Super User`的意思，这里存放的是系统管理员使用的系统管理程序。
- `/selinux`：这个目录是`Redhat/CentOS`所特有的目录，`Selinux`是一个安全机制，类似于`windows`的防火墙
- `/srv`：`service`缩写，该目录存放一些服务启动之后需要提取的数据。
- `/sys`： 这是`linux2.6`内核的一个很大的变化。该目录下安装了`2.6`内核中新出现的一个文件系统` sysfs 。`
- `/tmp`：这个目录是用来存放一些临时文件的。
- `/usr`： 这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似与`windows`下的`program files`目录。
- `/var`：这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。

**补充路径切换**

```text
~ 表示代码主目录，也就是当前登录用户的用户目录。
比如：我登录用户是chen
cd ~
~ 代表的就是 /home/chen/
当然前提是有用户目录，如果没有/home/chen目录的话情况就比较多了。
总之
cd ~
再
pwd
就知道了。
//pwd表示打印目录路径
```

### 用户管理

```text
//添加新用户，useradd ss
useradd 添加新用户
//设置密码，passwd ss
passwd
//判断用户是否存在,id ss,查看用户信息
id
//切换用户su ss,切换普通用户，切换root用户：su -
su 切换用户
//删除用户exit到root用户后在执行删除userdel ss
//删除用户但是保留用户主目录
	userdel 用户名
//删除用户，用户和用户主目录全部删除
	userdel -r 用户名
//查看用户登录信息
who：//三种用法
	whoimi//显示自身用户名称
	who im i//显示登录用户的用户名
	who//查看当前有哪些用户登录本台机器
//设置普通用户具有root权限
/*在etc/sudoers/文件夹下面找到root那一行，在下面添加即可*/
root ALL=(ALL) ALL
rr ALL=(ALL) ALL
//然后用sudo命令加上操作即可，sodu标示提升权限

```



## vim编辑器

### 概述

所有的 `Unix Like` 系统都会内建` vi `文书编辑器，其他的文书编辑器则不一定会存在。但是目前我们使用比较多的是 `vim `编辑器。

`Vim `具有程序编辑的能力，可以主动的以字体颜色辨别语法的正确性，方便程序设计。`Vim`是从` vi `发展出来的一个文本编辑器。代码补完、编译及错误跳转等方便编程的功能特别丰富，在程序员中被广泛使用。

简单的来说`vi `是老式的字处理器，不过功能已经很齐全了，但是还是有可以进步的地方。`vim `则可以说是程序开发者的一项很好用的工具。

**三种模式**

- `Command（命令）模式，用于输入命令;
- `Insert（插入）模式，用于插入文本;`
- `Visual（可视）模式，用于视化的的高亮并选定正文；`

**文件保存和退出**

Command 模式是vi或vim的默认模式，如果我们处于其它命令模式时，要通过ESC键切换过来。当我们按ESC键后，接着再输入:号时，vi会在屏幕的最下方等待我们输入命令

```text
:w  保存；
:w  filename 另存为filename；
:wq! 保存退出；
:wq! filename 注：以filename为文件名保存后退出；
:q! 不保存退出；
:x 应该是保存并退出 ，功能和:wq!相同
```

### 一般模式

以 `vi `打开一个档案就直接进入一般模式了(这是默认的模式)。在这个模式中， 你可以使用『上下左右』按键来移动光标，你可以使用『删除字符』或『删除整行』来处理档案内容， 也可以使用『复制、贴上』来处理你的文件数据。

> ① 移动光标 ② 复制 粘贴 ③ 剪切 粘贴 删除 ④ 撤销与恢复

**语法介绍**

```text
1. yy（功能描述：复制光标当前一行）

  y数字y（功能描述：复制一段(从第几行到第几行)）

2. p（功能描述：箭头移动到目的行粘贴）

3. u（功能描述：撤销上一步）

4. dd（功能描述：删除光标当前行）

	d数字d	（功能描述：删除光标(含)后多少行）

5. x（功能描述：删除一个字母，相当于del）

   X（功能描述：删除一个字母，相当于Backspace）

6. yw（功能描述：复制一个词）

7. dw（功能描述：删除一个词）

8. shift+^（功能描述：移动到行头）

9. shift+$（功能描述：移动到行尾）

1. 1+shift+g（功能描述：移动到页头，数字）

11. shift+g（功能描述：移动到页尾）

12. 数字N+shift+g（功能描述：移动到目标行）

13. gg:移动到行首

14. G：移动到行尾
```

**复制黏贴**

```text
复制当前行（光标所在那一行）

按键：yy

粘贴：在想要粘贴的地方按下p 键【将粘贴在光标所在行的下一行】,如果想粘贴在光标所在行之前，则使用P键


从当前行开始复制指定的行数，如复制5行，5yy

粘贴：在想要粘贴的地方按下p 键【将粘贴在光标所在行的下一行】,如果想粘贴在光标所在行之前，则使用P键
```

**剪切和删除**

```text
在VIM编辑器中，剪切与删除都是dd

如果剪切了文件，但是没有使用p进行粘贴，就是删除操作

如果剪切了文件，然后使用p进行粘贴，这就是剪切操作

1. 剪切/删除当前光标所在行

按键：dd （删除之后下一行上移）

粘贴：p

注意：dd 严格意义上说是剪切命令，但是如果剪切了不粘贴就是删除的效果。

2. 剪切/删除多行（从当前光标所在行开始计算）

按键：数字dd

粘贴：p

特殊用法：

3. 剪切/删除光标所在的当前行（光标所在位置）之后的内容，但是删除之后下一行不上移

按键：D （删除之后当前行会变成空白行）
```

**撤销和恢复**

```text
撤销：u（undo）

恢复：ctrl + r 恢复（取消）之前的撤销操作【重做，redo】
```

**翻屏指令**

```text
向上 翻屏，按键：ctrl + b （before） 或 PgUp

向下 翻屏，按键：ctrl + f （after） 或 PgDn

向上翻半屏，按键：ctrl + u （up）

向下翻半屏，按键：ctrl + d （down）
```

**快速定位光标到指定行**

行号 + G，如150G代表快速移动光标到第150行。

### 编辑模式

在一般模式中可以进行删除、复制、贴上等等的动作，但是却无法编辑文件内容的！ 要等到你按下`『i, I, o, O, a, A, r, R』`等任何一个字母之后才会进入编辑模式。

注意了！通常在` Linux `中，按下这些按键时，在画面的左下方会出现`『INSERT 或 REPLACE 』`的字样，此时才可以进行编辑。而如果要回到一般模式时， 则必须要按下`『Esc』`这个按键即可退出编辑模式。

**语法说明**

```text
1）进入编辑模式

（1）i 当前光标前

（2）a 当前光标后

（3）o 当前光标行的下一行

2）退出编辑模式

按『Esc』键
```

### 指令模式

在一般模式当中，输入`『 : / ?』`3个中的任何一个按钮，就可以将光标移动到最底下那一行。

在这个模式当中， 可以提供你『搜寻资料』的动作，而读取、存盘、大量取代字符、离开 `vi `、显示行号等动作是在此模式中达成的！

> 文件保存、退出、查找与替换、显示行号、paste模式等等

**语法说明**

```text
1）基本语法 shift+:金融指令模式

（1）: 选项

  w	代表对当前文件进行保存操作，但是其保存完成后，并没有退出这个文件

  q	代表退出当前正在编辑的文件，但是一定要注意，文件必须先保存，然后才能退出

  wq 代表文件先保存后退出（保存并退出）
  如果一个文件在编辑时没有名字，则可以使用:wq 文件名称，代表把当前正在编辑的文件保存到指定的名称中，然后退出

  q! => 代表强制退出但是文件未保存（不建议使用）

  ！  感叹号强制执行

（2）/  查找，/被查找词，n是查找下一个，shift+n是往上查找
  存在多个满足条件的结果。在搜索结果中切换上/下一个结果：N/n （大写N代表上一个结果，小写n代表next）
  如果需要取消高亮，则需要在末行模式中输入:noh【no highlight】

（3）?  查找，?被查找词，n是查找上一个，shift+n是往下查找

2）案例

:wq!		强制保存退出
:set nu 显示行号
```

**显示行号**

```text
:set nu
【nu = number】，行号

取消行号 => :set nonu
```

**set paste模式**

为什么要使用paste模式？

问题：在终端Vim中粘贴代码时，发现插入的代码会有多余的缩进，而且会逐行累加。原因是终端把粘贴的文本存入键盘缓存（Keyboard Buffer）中，Vim则把这些内容作为用户的键盘输入来处理。导致在遇到换行符的时候，如果Vim开启了自动缩进，就会默认的把上一行缩进插入到下一行的开头，最终使代码变乱。

在粘贴数据之前，输入下面命令开启paste模式
:set paste

粘贴完毕后，输入下面命令关闭paste模式
:set nopaste


## grep用法

### 基础用法

```java
grep  <Options> <Search String>  <File-Name>

-a --text  # 不要忽略二进制数据。
-A <显示行数>   --after-context=<显示行数>   # 除了显示符合范本样式的那一行之外，并显示该行之后的内容。
-b --byte-offset                           # 在显示符合范本样式的那一行之外，并显示该行之前的内容。
-B<显示行数>   --before-context=<显示行数>   # 除了显示符合样式的那一行之外，并显示该行之前的内容。
-c --count    # 计算符合范本样式的列数。
-C<显示行数> --context=<显示行数>或-<显示行数> # 除了显示符合范本样式的那一列之外，并显示该列之前后的内容。
-d<进行动作> --directories=<动作>  # 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep命令将回报信息并停止动作。
-e<范本样式> --regexp=<范本样式>   # 指定字符串作为查找文件内容的范本样式。
-E --extended-regexp             # 将范本样式为延伸的普通表示法来使用，意味着使用能使用扩展正则表达式。
-f<范本文件> --file=<规则文件>     # 指定范本文件，其内容有一个或多个范本样式，让grep查找符合范本条件的文件内容，格式为每一列的范本样式。
-F --fixed-regexp   # 将范本样式视为固定字符串的列表。
-G --basic-regexp   # 将范本样式视为普通的表示法来使用。
-h --no-filename    # 在显示符合范本样式的那一列之前，不标示该列所属的文件名称。
-H --with-filename  # 在显示符合范本样式的那一列之前，标示该列的文件名称。
-i --ignore-case    # 忽略字符大小写的差别。
-l --file-with-matches   # 列出文件内容符合指定的范本样式的文件名称。
-L --files-without-match # 列出文件内容不符合指定的范本样式的文件名称。
-n --line-number         # 在显示符合范本样式的那一列之前，标示出该列的编号。
-P --perl-regexp         # PATTERN 是一个 Perl 正则表达式
-q --quiet或--silent     # 不显示任何信息。
-R/-r  --recursive       # 此参数的效果和指定“-d recurse”参数相同。
-s --no-messages  # 不显示错误信息。
-v --revert-match # 反转查找。
-V --version      # 显示版本信息。   
-w --word-regexp  # 只显示全字符合的列。
-x --line-regexp  # 只显示全列符合的列。
-y # 此参数效果跟“-i”相同。
-o # 只输出文件中匹配到的部分。
-m <num> --max-count=<num> # 找到num行结果后停止查找，用来限制匹配行数
```

> grep是以行以单位进行输出

**规则表达式**

```text
^    # 锚定行的开始 如：'^grep'匹配所有以grep开头的行。    
$    # 锚定行的结束 如：'grep$' 匹配所有以grep结尾的行。
.    # 匹配一个非换行符的字符 如：'gr.p'匹配gr后接一个任意字符，然后是p。    
*    # 匹配零个或多个先前字符 如：'*grep'匹配所有一个或多个空格后紧跟grep的行。    
.*   # 一起用代表任意字符。   
[]   # 匹配一个指定范围内的字符，如'[Gg]rep'匹配Grep和grep。    
[^]  # 匹配一个不在指定范围内的字符，如：'[^A-FH-Z]rep'匹配不包含A-R和T-Z的一个字母开头，紧跟rep的行。    
(..)  # 标记匹配字符，如'(love)'，love被标记为1。    
<      # 锚定单词的开始，如:'<grep'匹配包含以grep开头的单词的行。    
>      # 锚定单词的结束，如'grep>'匹配包含以grep结尾的单词的行。    
x{m}  # 重复字符x，m次，如：'0{5}'匹配包含5个o的行。    
x{m,}   # 重复字符x,至少m次，如：'o{5,}'匹配至少有5个o的行。    
x{m,n}  # 重复字符x，至少m次，不多于n次，如：'o{5,10}'匹配5--10个o的行。   
\w    # 匹配文字和数字字符，也就是[A-Za-z0-9]，如：'G\w*p'匹配以G后跟零个或多个文字或数字字符，然后是p。   
\W    # \w的反置形式，匹配一个或多个非单词字符，如点号句号等。   
\b    # 单词锁定符，如: '\bgrep\b'只匹配grep。  
```

### grep基本查找匹配

在文件中搜索一个单词，命令会返回一个包含 “match_pattern” 的文本行：

```text
grep match_pattern file_name
grep "match_pattern" file_name
```

在多个文件中查找:

```text
grep "match_pattern" file_1 file_2 file_3 ...
```

输出除之外的所有行 -v 选项:

```text
grep -v "kafka" nohup-2024-11-25.log

输出的内容是不包含kafka的内容
```

统计文件或者文本中包含匹配字符串的行数 -c 选项：
```text
 grep -c "kafka" nohup-2024-11-25.log
```

略匹配样式中的字符大小写：
```text
grep -i "KAFKA" nohup-2024-11-25.log
```

显示带有行号：
```text
grep -in "KAFKA" nohup-2024-11-25.log
```


### 正则表达式

```text
//1，查找特定字符串：
//选出文件中有字符串“name"的行
[rzf@hadoop01 module]$ grep -n(表示输出行号) 'name' rzf.txt 
//输出没有“name"的行
[rzf@hadoop01 module]$ grep -n 'name' rzf.txt  //-v表示反向输出
/,2，使用[]查找字符
//如果我们要查找test或者taste这两个关键词，可以发现同时含有“t?st",所以可以使用通配符
[rzf@hadoop01 module]$ grep -n 't[ae]st' rzf.txt//[]里面只有一个字符起作用
//例如想要查找到有“Na"的字符时候
[rzf@hadoop01 module]$ grep -n 'Na' rzf.txt 
//如果我们不想要待查找的字符前面有某一个字符的话，可以这样做
[rzf@hadoop01 module]$ grep -n [^s]et rzf.txt //表示查找字符串et,但是前面不能含有s字符，
//如果我们想带查找的字符前面不能有小写字符，可以这样做
[rzf@hadoop01 module]$ grep -n [^a-z]a rzf.txt //表示查找字符a,但是前面不可以出现小写字符
连续字符可以这样使用：[a-z],[A-Z],[1-9]
也可以将上面三者连起来使用：[a-zA-Z0-9]
//查找有数字的哪一行
[rzf@hadoop01 module]$ grep -n [0-9] rzf.txt 
/,3，行首与行尾：^ &
[rzf@hadoop01 module]$ grep -n '^return' rzf.txt    //表示查找含有return词语的行并且必须在行首出现
//开头是小写的所有行输出^ 表示行首。千万记住:^在[]表示取反，而在[]外部表示行首
[rzf@hadoop01 module]$ grep -n '[^a-z]' rzf.txt 
//如果不希望开头是英文字母
[rzf@hadoop01 module]$ grep -n '^[^a-zA-Z]' rzf.txt //说明：^ 在[]内部和外部的含义是不同的，在[]里面表示的是取反向的意思，在[]外面表示的是开头的意思，也就是行首
//找出以.结尾的行
[rzf@hadoop01 module]$ grep -n '\.$' rzf.txt //&表示结尾符号，其中\表示转义字符
//查找空白行
[rzf@hadoop01 module]$ grep -n '^$' rzf.txt 
/,4，任意一个字符：.与重复字符：*
在正则表达式中：.代表：一定有一个任意字符
							*代表：重复前一个字符，0到无穷多次的意思
[rzf@hadoop01 module]$ grep -n 'g..d' rzf.txt //表示查找以g开头并且以d结尾的字符，中间只能有两个字符
[rzf@hadoop01 module]$ grep -n 'go*d' rzf.txt //代表输出以g开头，并且以d结尾，中间有0个或者以上的o字符的字符串，切记，*表示重复*前面的一个字符从0到无穷次，包括0次
//输出以g开头，并且以g结尾，中间包括一个或者超过1个o的字符串
grep -n 'goo*g' rzf.txt
//特别注意：.*代表0个或者多个任意字符
//限定连续RE字符范围{}
[rzf@hadoop01 module]$ grep -n 'o\{2\}' rzf.txt //表示查找有两个连续的oo的字符串，\\表示是转义字符
[rzf@hadoop01 module]$ grep -n 'go\{2,5\}' rzf.txt//查找g后面有2-5个o的字符串输出
[rzf@hadoop01 module]$ grep -n 'go\{2,\}' rzf.txt //查找g后面有多于两个o的字符串
```

## 文件目录类操作

### PWD

pwd 显示当前工作目录的绝对路径
基本语法：
pwd:（功能描述：显示当前工作目录的==绝对路径==）

~~~ java
pwd
./  //标示当前目录
~~~

### ls 列出目录内容

```text
基本语法：

ls [选项] [目录或是文件]

选项：

-a ：全部的文件，连同隐藏档( 开头为 . 的文件) 一起列出来(常用)

-d：仅仅显示目录本身，而不是列出目录内的文件数据

-l ：长数据串列出，包含文件的属性与权限等等数据；(常用)

-al：列出所有文件，包括隐藏文件的全部详细信息
```

> 每行列出的信息依次是： 文件类型与权限 链接数 文件属主 文件属组 文件大小用byte来表示 建立或最近修改的时间 名字 

### mkdir

mkdir 创建一个新的文件夹

```text
1、基本语法：

mkdir [-p] 要创建的目录

选项：

-p：创建多层目录

2、rmdir：删除一个空的文件夹

基本语法：

rmdir 要删除的空目录

3、touch：创建空文件

基本语法：

touch 文件名称
```



### cd 切换目录

```text
1）基本语法：

	（1）cd 绝对路径

	（2）cd 相对路径

	（3）cd ~或者cd（功能描述：回到自己的家目录，即主目录）

	（4）cd -	（功能描述：回到上一次所在目录）

	（5）cd ..（功能描述：回到当前目录的上一级目录）

	（6）cd -P （功能描述：跳转到实际物理路径，而非快捷方式路径）
```

### cp文件复制

```text
cp：文件的复制

1）基本语法：

（1）cp source dest 	（功能描述：复制source文件到dest）

（2）cp -r sourceFolder targetFolder	（功能描述：递归复制整个文件夹）
```

### rm删除文件或者目录

```text
rm:移除文件或者目录

1）基本语法

	（1）rmdir deleteEmptyFolder	（功能描述：删除空目录）

	（2）rm -rf deleteFile			（功能描述：递归删除目录中所有内容）
```

### mv

mv:移动文件与目录或者重命名

```text
1）基本语法：

	（1）mv oldNameFile newNameFile	（功能描述：重命名）

	（2）mv /temp/movefile /targetFolder	（功能描述：移动文件）
```

## 查看文件内容

```text
cat     由第一行开始显示内容，并将所有内容输出

tac     从最后一行倒序显示内容，并将所有内容输出

more    根据窗口大小，一页一页的现实文件内容

less    和more类似，但其优点可以往前翻页，而且进行可以搜索字符

head    只显示头几行

tail    只显示最后几行

nl      类似于cat -n，显示时输出行号

tailf   类似于tail -f 
```

### cat

```text
查看文件内容，从第一行开始显示。
功能：查看文件内容

语法：cat [选项] [文件]

选项：

-b ：对非空输出行编号
-n ：对输出的所有行编号
-s ：不输出多行空行

1）基本语法

	cat  [选项] 要查看的文件

选项：

-A ：相当于 -vET 的整合选项，可列出一些特殊字符而不是空白而已；

-b ：列出行号，仅针对非空白行做行号显示，空白行不标行号！

-E ：将结尾的断行字节 $ 显示出来；

-n ：列出行号，连同空白行也会有行号，与 -b 的选项不同；

-T ：将 [tab] 按键以 ^I 显示出来；

-v ：列出一些看不出来的特殊字符
```

### tac

查看文件内容，从最后一行开始显示，可以看出 tac 是 cat 的倒写。

```text
1）基本语法：

	tac  [选项参数] 要查看的文件
```

cat的功能是将文件从第一行开始连续的将内容输出在屏幕上。但是cat并不常用，原因是当文件大，行数比较多时，屏幕无法全部容下时，只能看到一部分内容。

cat语法：cat [-n]  文件名 （-n ： 显示时，连行号一起输出）:cat -n spark.log

tac的功能是将文件从最后一行开始倒过来将内容数据输出到屏幕上。我们可以发现，tac实际上是cat反过来写。这个命令也不常用。

tac语法：tac 文件名。

### more

```text
查看文件内容，一页一页的显示文件内容。

1）基本语法：

more 要查看的文件

2）功能使用说明

空白键 (space)：代表向下翻一页；

Enter:代表向下翻『一行』；

q:代表立刻离开 more ，不再显示该文件内容。

Ctrl+F 向下滚动一屏

Ctrl+B 返回上一屏

= 输出当前行的行号

:f 输出文件名和当前行的行号

v 调用vi编辑器； 

! 命令 调用Shell，并执行命令； 

q 退出more
```

### less:查看文件内容

`less `的作用与` more` 十分相似，都可以用来浏览文字档案的内容，不同的是` less `允许使用`[pageup] [pagedown]`往回滚动。

```text
1）基本语法：
	less 要查看的文件
2）功能使用说明


[pagedown]：向下翻动一页；

[pageup] ：向上翻动一页；

空白键 (space)：代表向下翻一页；

Enter:代表向下翻『一行』；

q:代表立刻离开 more ，不再显示该文件内容。

Ctrl+F 向下滚动一屏

Ctrl+B 返回上一屏

/字串    ：向下搜寻『字串』的功能；n：向下查找；N：向上查找；
?字串    ：向上搜寻『字串』的功能；n：向上查找；N：向下查找；
q        ：离开 less 这个程序；
```

1. less的功能和more相似，但是使用more无法向前翻页，只能向后翻。
2. less可以使用【pageup】和【pagedown】键进行前翻页和后翻页，这样看起来更方便。
3. less还有一个功能，可以在文件中进行搜索你想找的内容，假设你想在passwd文件中查找有没有weblogic字符串，那么你可以这样来做：

```text
 less passwd

然后输入：

/weblogic

回车

此时如果有weblogic字符串，linux会把该字符已高亮方式显示。

退出查看页面，请按“q”键。
```

### head:查看文件内容：

查看文件内容，只看头几行。

```text
1）基本语法

head -n 10 文件      （功能描述：查看文件头10行内容，10可以是任意行数）
```

### tail:查看文件内容

```text
//查看文件内容，只看尾巴几行。

1）基本语法

（1）tail  -n 10 文件 		（功能描述：查看文件头10行内容，10可以是任意行数）

（2）tail  –f  文件		（功能描述：实时追踪该文档的所有更新）
```

**head和tail**

1. head和tail通常使用在只需要读取文件的前几行或者后几行的情况下使用。head的功能是显示文件的前几行内容
2. head的语法：head [n number] 文件名 (number 显示行数)
3. tail的功能恰好和head相反，只显示最后几行内容
4. tail的语法:tail [-n number] 文件名

### nl

nl的功能和cat -n一样，同样是从第一行输出全部内容，并且把行号显示出来

nl的语法：nl 文件名

### 重定向文件

1）基本语法：（也就是把查询到的内容写到文件中）

（1）ls –l >文件		（功能描述：列表的内容写入文件a.txt中（覆盖写））

（2）ls –al >>文件	（功能描述：列表的内容追加到文件aa.txt的末尾）

### echo

```text
1）基本语法：

（1）echo 要显示的==内容 >> 存储==内容的的文件	（功能描述：将要显示的内容，存储到文件中追加写)

（2）echo 要显示的==内容 >存储==内容的的文件	（功能描述：将要显示的内容，存储到文件中覆盖写),

（2）echo 变量		（功能描述：显示变量的值）
```

### ln软连接

```text
1）基本语法：

ln –s [原文件] [目标文件]		（功能描述：给原文件创建一个软链接，软链接存放在目标文件目录）

2）案例：

[root@hadoop101 module]# ln -s /opt/module/test.txt /opt/t.txt

[root@hadoop101 opt]# ll

lrwxrwxrwx. 1 root    root      20 6月  17 12:56 t.txt -> /opt/module/test.txt

相当于t.txt的实际路径是->后面的路径。

cd -P 文件夹名字，指向的是实际的物理地址。，不加-P指向的是虚的地址。

//创建一个软链接
[atguigu@hadoop103 opt]$ ln -s /opt/module/hadoop-2.7.2/ /opt/software/hadoop
cd不加参数进入是软链接的地址
[atguigu@hadoop103 software]$ cd hadoop
[atguigu@hadoop103 hadoop]$ pwd
/opt/software/hadoop

cd加参数进入是实际的物理地址
[atguigu@hadoop103 software]$ cd -P hadoop
[atguigu@hadoop103 hadoop-2.7.2]$ pwd
/opt/module/hadoop-2.7.2
```

## 时间日期类

```text
1）基本语法

date [OPTION]... [+FORMAT]

1. date:显示当前日期：

   1）基本语法：

  	（1）date								（功能描述：显示当前时间）

   	（2）date +%Y							（功能描述：显示当前年份）

   （3）date +%m							（功能描述：显示当前月份）

   （4）date +%d							（功能描述：显示当前是哪一天）

   （5）date +%Y%m%d   date +%Y/%m/%d …	（功能描述：显示当前年月日各种格式 ）

   （6）date "+%Y-%m-%d %H:%M:%S"		（功能描述：显示年月日时分秒）

date "+%Y-%m-%d-%H-%M-%s"//显示年月日时分秒

2. date显示非当前时间

   1）基本语法：

   （1）date -d '1 days ago'			（功能描述：显示前一天日期）

   （2）date -d yesterday +%Y%m%d	（同上）

   （3）date -d next-day +%Y%m%d	（功能描述：显示明天日期）

   （4）date -d 'next monday'			（功能描述：显示下周一时间）

   （5）date -d "next day"   //显示下一天时间

3. date设置系统当前时间

   1）基本语法：

   	date -s 字符串时间

4. cal:查看日历

   1）基本语法：

   cal [选项]			（功能描述：不加选项，显示本月日历）

   选项：

   -3 ，显示系统前一个月，当前月，下一个月的日历

   cal 2020：具体某一年，显示这一年的日历。
```

## 用户组命令

每个用户都有一个用户组，系统可以对一个用户组中的所有用户进行集中管理。不同`Linux` 系统对用户组的规定有所不同，

如`Linux`下的用户属于与它同名的用户组，这个用户组在创建用户时同时创建。

用户组的管理涉及用户组的添加、删除和修改。组的增加、删除和修改实际上就是对`/etc/group`文件的更新。

### groupadd 新增组

~~~ java
1）基本语法
groupadd 组名
//添加一个用户组
[root@VM-8-10-centos /]# groupadd myGroup
~~~

### groupdel删除组

~~~ java
1）基本语法：
groupdel 组名
//删除一个用户组
[root@VM-8-10-centos /]# groupdel MYgROUP
~~~

###  groupmod修改组

~~~ java
1）基本语法：
groupmod -n 新组名 老组名
4. 创建一个用户并且属于一个组：usermod -g 组名 用户名:usermod -g bb cc:将用户cc分到bb用户组里面

 usermod -g cc cc：重新修改cc用户为cc组里面。
    

 groupmod -n 新组名 老组名：groupmod -n bb aa：给组aa重新命名为bb

 创建一个用户并且属于一个组：usermod -g 组名 用户名:usermod -g bb cc:将用户cc分到bb用户组里面

 usermod -g cc cc：重新修改cc用户为cc组里面
~~~

### cat  /etc/group 查看创建了哪些组

~~~ java
cat  /etc/group
~~~

### 案例演示

~~~ java
//创建一个新的组
[root@VM-8-10-centos /]# groupadd myGroup
//修改组名
[root@VM-8-10-centos /]# groupmod -n group myGroup 
[root@VM-8-10-centos /]# cat /etc/group
//切换用户
[root@VM-8-10-centos /]# su rzf
//创建一个文件
[rzf@VM-8-10-centos ~]$ mkdir rr
//将rzf用户添加到group组中，但是要在root用户权限下
[root@VM-8-10-centos ~]# usermod -g group rzf
//注意
[root@hadoop101 rr]# groupdel rr
groupdel：不能移除用户“rr”的主组
[root@hadoop101 rr]# rserdel -r rr
-bash: rserdel: command not found
[root@hadoop101 rr]# userdel -r rr
//在删除用户组的时候不能删除时，先删除用户即可
-rw-r--r-- 1 root root    0 3月  19 2020 rzf.txt
第一个root标示文件的创建者
第二个root标示文件创建者属于哪一个组
~~~

### 查看用户所属用户组

- groups命令
- id命令

~~~ java
//查看当前用户属于哪一个用户组
[rzf@hadoop01 hive]$ groups
wheel
//查看特定用户属于的用户组
groups 用户名
[rzf@hadoop01 hive]$ groups root
root : root
//也可以通过id命令查看
[rzf@hadoop01 hive]$ id
uid=1001(rzf) gid=10(wheel) groups=10(wheel)
[rzf@hadoop01 hive]$ id rzf
uid=1001(rzf) gid=10(wheel) groups=10(wheel)
    
[root@hadoop101 rr]# groupdel rr
groupdel：不能移除用户“rr”的主组
[root@hadoop101 rr]# rserdel -r rr
-bash: rserdel: command not found
[root@hadoop101 rr]# userdel -r rr
//在删除用户组的时候不能删除时，先删除用户即可
-rw-r--r-- 1 root root    0 3月  19 2020 rzf.txt
第一个root标示文件的创建者
第二个root标示文件创建者属于哪一个组
~~~

## 用户管理

### useradd 添加新用户

~~~ java
1）基本语法：
	useradd 用户名		（功能描述：添加新用户）
//注意，新添加的用户的目录在home文件夹下面
~~~

### password设置用户密码

~~~ java
1）基本语法：
	passwd 用户名	（功能描述：设置用户密码）

~~~

### id判断用户是否存在

~~~ java
1）基本语法：
	id 用户名
~~~

### su切换用户

~~~~ java
1）基本语法：
su 用户名称     				（功能描述：切换用户）
在这里普通用户如何切换root用户,输入：su
root用户切换普通用户：su 用户名
普通用户切换root用户：sudo -i
~~~~

### userdel删除用户

~~~ java
1）基本语法：
	（1）userdel  用户名		（功能描述：删除用户但保存用户主目录）
（2）userdel -r 用户名		（功能描述：用户和用户主目录，都删除）
~~~

### who查看登录信息

~~~ java
(1）基本语法
	（1）whoami			（功能描述：显示自身用户名称）
（2）who am i		（功能描述：显示登录用户的用户名）
（3）who			（功能描述：看当前有哪些用户登录到了本台机器上）
~~~

### 设置普通用户具有root权限

~~~ java
1）修改配置文件

修改 /etc/sudoers 文件，找到下面一行，在root下面添加一行，如下所示：
Allow root to run any commands anywhere
root    ALL=(ALL)     ALL
rzf   ALL=(ALL)     ALL
修改完毕，现在可以用rzf帐号登录，然后用命令 su - ，即可获得root权限进行操作。
//在用户状态下创建一个目录，在创建的时候需要添加sudo命令
[rzf@VM-8-10-centos opt]$ sudo mkdir rr
~~~

### cat  /etc/passwd 查看创建了哪些组

~~~ java
cat  /etc/passwd
~~~

###  usermod修改用户 

~~~ java
1）基本语法：
usermod -g 用户组 用户名
//将用户rzf加入dev用户组
usermod –g dev rzf
~~~

## 文件权限

### 文件属性

`Linux`系统是一种典型的多用户系统，不同的用户处于不同的地位，拥有不同的权限。为了保护系统的安全性，`Linux`系统对不同的用户访问同一文件（包括目录文件）的权限做了不同的规定。在`Linux`中我们可以使用`ll`或者`ls –l`命令来显示一个文件的属性以及文件所属的用户和组。

~~~ java
1）从左到右的10个字符表示：

如果没有权限，就会出现减号[ - ]而已。从左至右用0-9这些数字来表示:

（1）0首位表示类型

在Linux中第一个字符代表这个文件是目录、文件或链接文件等等

     - 代表文件

     d 代表目录

     c 字符流，装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)

     s socket

     p 管道

     l 链接文档(link file)；

     b 设备文件，装置文件里面的可供储存的接口设备(可随机存取装置)

（2）第1-3位确定属主（该文件的所有者）拥有该文件的权限。---User

（3）第4-6位确定属组（所有者的同组用户）拥有该文件的权限，---Group

（4）第7-9位确定其他用户拥有该文件的权限 ---Other
~~~

| 文件类型 | 属主权限     | 属组权限     | 其他用户权限 |
| -------- | ------------ | ------------ | ------------ |
| 0        | 1    2   3   | 4   5   6    | 7   8   9    |
| d        | R   w   x    | R   -   x    | R   -   x    |
| 目录文件 | 读  写  执行 | 读  写  执行 | 读  写  执行 |

~~~ java
2）rxw作用文件和目录的不同解释

    （1）作用到文件：

    [ r ]代表可读(read): 可以读取，查看

    [ w ]代表可写(write): 可以修改，但是不代表可以删除该文件,删除一个文件的前提条件是对该文件所在的目录有写权限，才能删除该文件.

    [ x ]代表可执行(execute):可以被系统执行

    （2）作用到目录：

    [ r ]代表可读(read): 可以读取，ls查看目录内容

    [ w ]代表可写(write): 可以修改，目录内创建+删除+重命名目录

    [ x ]代表可执行(execute):可以进入该目录
~~~

![graphic](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/linux/202012/13/112231-458638.jpeg)

### chomd修改文件权限

~~~ java
1. 基本语法：

	chmod  [{ugoa}{+-=}{rwx}] [文件或目录] [mode=421 ]  [文件或目录] 

    2）功能描述

    u代表用户，g代表组，o代表其他，a代表all所有人，增加权限用+，减少权限用-

    改变文件或者目录权限

    文件: r-查看；w-修改；x-执行文件

    目录: r-列出目录内容；w-在目录中创建和删除；x-进入目录

    删除一个文件的前提条件:该文件所在的目录有写权限，你才能删除该文件。

    chmod u+x aa.txt //对aa文件修改有x可执行权限

    chmod g-w aa.txt：//对组收回w写权限
2. 改变文件拥有者

    1）基本语法：

    chown [最终用户] [文件或目录]		（功能描述：改变文件或者目录的所有者）

    chown 文件所属用户：文件所属组 文件名
    [root@VM-8-10-centos ~]# chown rzf aa
		[root@VM-8-10-centos ~]# ll
	total 4
	drwxr-xr-x 2 rzf root 4096 Dec 13 10:56 aa

3. chgrp：改变组所属

       1）基本语法：

       chgrp [最终用户组] [文件或目录]	（功能描述：改变文件或者目录的所属组）
       [root@VM-8-10-centos ~]# chgrp group aa
		[root@VM-8-10-centos ~]# ll
		total 4
		drwxr-xr-x 2 rzf group 4096 Dec 13 10:56 aa
~~~

![img](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/linux/202012/13/114914-680469.png)

注意：`rzf group`文件后的这两个参数，第一个表示文件所属的用户，第二个表示文件所属的组。

## 磁盘分区类

### 基本用法

```text
df查看硬盘

1）基本语法：

	df  参数		（功能描述：列出文件系统的整体磁盘使用量，检查文件系统的磁盘空间占用情况）

参数：

-a ：列出所有的文件系统，包括系统特有的 /proc 等文件系统；

-k ：以 KBytes 的容量显示各文件系统；

-m ：以 MBytes 的容量显示各文件系统；

-h ：以人们较易阅读的 GBytes, MBytes, KBytes 等格式自行显示；

-H ：以 M=1000K 取代 M=1024K 的进位方式；

-T ：显示文件系统类型，连同该 partition 的 filesystem 名称 (例如 ext3) 也列出；

-i ：不用硬盘容量，而以 inode 的数量来显示
//案例演示
[root@VM-8-10-centos ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        908M     0  908M   0% /dev
tmpfs           919M   24K  919M   1% /dev/shm
tmpfs           919M  480K  919M   1% /run
tmpfs           919M     0  919M   0% /sys/fs/cgroup
/dev/vda1        40G  2.5G   36G   7% /
tmpfs           184M     0  184M   0% /run/user/0

du -skh /*  查看当前目录下所有文件的大小
```

### 查看分区情况

1）基本语法：

​	fdisk –l			（功能描述：查看磁盘分区详情）

​	注意：在root用户下才能使用

2）功能说明：

​	（1）Linux分区

这个硬盘是20G的，有255个磁面；63个扇区；2610个磁柱；每个 cylinder（磁柱）的容量是 8225280 bytes=8225.280 K（约为）=8.225280M（约为）；

| Device   | Boot | Start       | End         | Blocks | Id         | System   |
| -------- | ---- | ----------- | ----------- | ------ | ---------- | -------- |
| 分区序列 | 引导 | 从X磁柱开始 | 到Y磁柱结束 | 容量   | 分区类型ID | 分区类型 |

df查看硬盘

```text
1）基本语法：

	df  参数		（功能描述：列出文件系统的整体磁盘使用量，检查文件系统的磁盘空间占用情况）

参数：

-a ：列出所有的文件系统，包括系统特有的 /proc 等文件系统；

-k ：以 KBytes 的容量显示各文件系统；

-m ：以 MBytes 的容量显示各文件系统；

-h ：以人们较易阅读的 GBytes, MBytes, KBytes 等格式自行显示；

-H ：以 M=1000K 取代 M=1024K 的进位方式；

-T ：显示文件系统类型，连同该 partition 的 filesystem 名称 (例如 ext3) 也列出；

-i ：不用硬盘容量，而以 inode 的数量来显示
```

## 搜索查找类

### find命令

**基础语法**

```text
语法：find [路径] [选项] [表达式]
完整命令：find   path  -option  【 -print 】  【 -exec   -ok   |xargs  |grep  】 【  command  {} \;  】

功能：查找文件及其绝对路径。

path：要查找的目录路径。 

  ~ 表示$HOME目录
  . 表示当前目录
  / 表示根目录

print：表示将结果输出到标准输出。 

exec：对匹配的文件执行该参数所给出的shell命令。 
形式为command {} \;，注意{}与\;之间有空格 

ok：与exec作用相同，区别在于，在执行命令之前，都会给出提示，让用户确认是否执行 

|xargs  与exec作用相同 ，起承接作用，区别在于 |xargs 主要用于承接删除操作 ，而 -exec 都可用 如复制、移动、重命名等

options ：表示查找方式

常用选项：

  -name   filename  #查找名为filename的文件
  -perm   #按执行权限来查找
  -user username  #按文件属主来查找
  -group groupname #按组来查找
  -mtime   -n +n   #按文件更改时间来查找文件，-n指n天以内，+n指n天以前
  -atime    -n +n   #按文件访问时间来查找文件，-n指n天以内，+n指n天以前
  -ctime   -n +n  #按文件创建时间来查找文件，-n指n天以内，+n指n天以前
  -nogroup  #查无有效属组的文件，即文件的属组在/etc/groups中不存在
  -nouser #查无有效属主的文件，即文件的属主在/etc/passwd中不存
  -type    b/d/c/p/l/f   #查是块设备、目录、字符设备、管道、符号链接、普通文件
  -size  n[c] #查长度为n块[或n字节]的文件
  -mount  #查文件时不跨越文件系统mount点
  -follow  #如果遇到符号链接文件，就跟踪链接所指的文件
  -prune #忽略某个目录

参数说明：

    [路径]：查找的目录路径。
    [选项]：命令的可选参数，用于控制查找条件。
    [表达式]：查找的具体条件，如文件名、类型等。
```

**基本用法**

**按照文件名字查找**

```text
在当前目录及子目录中，查找小写字母开头的txt文件 

find . -name '[a-z]*.sh' -print

在/etc及其子目录中，查找host开头的文件 
find /etc -name 'host*' -print
```

**按照权限查找**
```text
在当前目录及子目录中，查找属主具有读写执行，其他具有读执行权限的文件
 find . -perm 755 -print

查找用户有写权限或者组用户有写权限的文件或目录
find ./ -perm /220
find ./ -perm /u+w,g+w
find ./ -perm /u=w,g=w
```

> 参考:https://blog.csdn.net/l_liangkk/article/details/81294260


### grep在文件内搜索字符匹配的行并且输出

```text
1）基本语法，相当于二次检索

grep+参数+查找内容+源文件
grep [-abcEFGhHilLnqrsvVwxy][-A<显示列数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]
https://linux.cn/article-1672-1.html//find和grep命令详解
参数：
－c：只输出匹配行的计数。
－I：不区分大小写(只适用于单字符)。
－h：查询多文件时不显示文件名。
－l：查询多文件时只输出包含匹配字符的文件名。
－n：显示匹配行及行号。
－s：不显示不存在或无匹配文本的错误信息。
－v：显示不包含匹配文本的所有行。
```

(global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来)是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

1）基本语法，相当于二次检索

grep+参数+查找内容+源文件

~~~ java
grep [-abcEFGhHilLnqrsvVwxy][-A<显示列数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]
https://linux.cn/article-1672-1.html//find和grep命令详解

参数：

－c：只输出匹配行的计数。

－I：不区分大小写(只适用于单字符)。

－h：查询多文件时不显示文件名。

－l：查询多文件时只输出包含匹配字符的文件名。

－n：显示匹配行及行号。

－s：不显示不存在或无匹配文本的错误信息。

－v：显示不包含匹配文本的所有行。
~~~

**案例**

```text
 ls -l |grep aa.txt//ls -l查看目录下面文件，| grep对文件进行过滤，过滤出aa.txt文件

 ls -l |grep -n aa.txt
2:-rwxr--r-- 1 rui rui 0 3月  19 00:58 aa.txt//-n表示把匹配的行号输出，隐藏文件也算
```

### which

which命令在Linux中是一个非常有用的命令，用于定位系统中的可执行文件。如果你想知道某个程序究竟在哪里，只需使用which命令即可。which命令会在你的shell的搜索路径中定位一个可执行文件。

```text
语法：which [选项] 文件
功能：用于查找可执行文件的位置。
常用选项：
  -a：显示所有匹配的路径，而不仅仅是第一个。
  --version：显示版本信息。
参数说明：
  [选项]：命令的可选参数，可以指定显示所有匹配的路径或版本信息。
  文件：需要查找的可执行文件名。
```

**擦护照单个命令的位置**

```text
which java

/home/jdk1.8.0_161//bin/java
这告诉我们java命令的位置在/home/jdk1.8.0_161//bin/java;
```

**查找多个命令位置**

```shell
which java man ls cat
/home/jdk1.8.0_161//bin/java
/usr/bin/man
/usr/bin/ls
/usr/bin/cat
```

**显示所有匹配的路径**

如果一个程序在两个地方都有可执行文件，比如在/usr/bin/program和/usr/local/bin/program，你可以使用-a选项显示所有的路径。

```shell
which -a program
```

**查找自定义脚本位置**

假设你有一个名为myscript的自定义脚本，并且它在你的PATH中。你可以使用which命令来找到它的位置：

```shell
which myscript
```

> which命令只适用于可执行文件。
> 如果which命令在当前路径中找不到可执行文件，它将返回空。


### whereis查找

### 基础语法

```text
语法：whereis [选项] 文件
功能：查找特定文件名的文件，包括可执行文件、源代码文件和man手册。

常用选项：
    -b：仅查找二进制文件。
    -m：仅查找手册文件。
    -s：仅查找源代码文件。
    -B：指定要查找二进制文件的路径。
    -M：指定要查找手册文件的路径。
    -S：指定要查找源代码文件的路径。

参数说明：
    [选项]：命令的可选参数，用于指定查找的文件类型和路径。
    文件：需要查找的文件名。
```

whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz


### Locate查找

```text
语法：locate [选项] 模式  locate [options] [pattern]

功能：locate是一个Unix实用程序，用于快速查找文件和目录。它是find命令的一个更方便和高效的替代品，find命令搜索整个文件系统，而locate命令则查看系统中定期更新的文件数据库。因此，搜索完成得更快。

常用选项：
  -A, --all:显示匹配所有指定模式的条目
  -b, --basename:只将基本名称与指定的模式匹配
  -c, --count:输出匹配条目的数量，而不是文件名
  -i：忽略大小写。
  -r：使用正则表达式。
  -n：指定显示的最大条目数。
  -b, --basename  # 仅匹配路径名的基本名称
-c, --count     # 只输出找到的数量
-d, --database DBPATH # 使用DBPATH指定的数据库，而不是默认数据库 /var/lib/mlocate/mlocate.db
-e, --existing  # 仅打印当前现有文件的条目
-1 # 如果 是 1．则启动安全模式。在安全模式下，使用者不会看到权限无法看到 的档案。这会始速度减慢，因为 locate 必须至实际的档案系统中取得档案的  权限资料。
-0, --null            # 在输出上带有NUL的单独条目
-S, --statistics      # 不搜索条目，打印有关每个数据库的统计信息
-q                    # 安静模式，不会显示任何错误讯息。
-P, --nofollow, -H    # 检查文件存在时不要遵循尾随的符号链接
-l, --limit, -n LIMIT # 将输出（或计数）限制为LIMIT个条目
-n                    # 至多显示 n个输出。
-m, --mmap            # 被忽略，为了向后兼容
-r, --regexp REGEXP   # 使用基本正则表达式
    --regex           # 使用扩展正则表达式
-q, --quiet           # 安静模式，不会显示任何错误讯息
-s, --stdio           # 被忽略，为了向后兼容
-o                    # 指定资料库存的名称。
-h, --help            # 显示帮助
-i, --ignore-case     # 忽略大小写
-V, --version         # 显示版本信息


参数说明：

  [选项]：命令的可选参数，用于控制查找行为。
  模式：匹配文件名的模式，可以包含通配符或正则表达式。
```

**基本使用**

```text
locate filename:这个命令会在文件数据库中查找名为filename的文件。

locate -i filename:这个命令会忽略大小写，查找名为filename的文件。

locate *pattern*:这个命令会在文件数据库中查找包含pattern的文件,会匹配文件名字；

locate -n 10 filename：这个命令会在文件数据库中查找名为filename的文件，并且只显示前10个结果。

sudo updatedb：这个命令会更新locate命令的数据库，以便包含最新的文件信息。
```


### 查找总结

1. which：常用于查找可直接执行的命令。只能查找可执行文件，该命令基本只在$PATH路径中搜索，查找范围最小，查找速度快。默认只返回第一个匹配的文件路径，通过选项 -a 可以返回所有匹配结果。

2. whereis：不只可以查找命令，其他文件类型都可以（man中说只能查命令、源文件
和man文件，实际测试可以查大多数文件）。在$PATH路径基础上增加了一些系统目录的查找，查找范围比which稍大，查找速度快。可以通过 -b 选项，限定只搜索二进制文件；

3. locate：超快速查找任意文件。它会从linux内置的索引数据库查找文件的路径，索引速度超快。刚刚新建的文件可能需要一定时间才能加入该索引数据库，可以通过执行updatedb命令来强制更新一次索引，这样确保不会遗漏文件。该命令通常会返回大量匹配项，可以使用 -r 选项通过正则表达式来精确匹配。

4. find：直接搜索整个文件目录，默认直接从根目录开始搜索，建议在以上命令都无法解决问题时才用它，功能最强大但速度超慢。除非你指定一个很小的搜索范围。通过 -name 选项指定要查找的文件名，支持通配符。

## 进程线程类

```text
ps命令查找与进程相关的PID号：
 
ps a 显示现行终端机下的所有程序，包括其他用户的程序。
 
ps -A 显示所有程序。
 
ps c 列出程序时，显示每个程序真正的指令名称，而不包含路径，参数或常驻服务的标示。
 
ps -e 此参数的效果和指定"A"参数相同。
 
ps e 列出程序时，显示每个程序所使用的环境变量。
 
ps f 用ASCII字符显示树状结构，表达程序间的相互关系。
 
ps -H 显示树状结构，表示程序间的相互关系。
 
ps -N 显示所有的程序，除了执行ps指令终端机下的程序之外。
 
ps s 采用程序信号的格式显示程序状况。
 
ps S 列出程序时，包括已中断的子程序资料。
 
ps -t<终端机编号> 指定终端机编号，并列出属于该终端机的程序的状况。
 
ps u 以用户为主的格式来显示程序状况。
 
ps x 显示所有程序，不以终端机来区分。
   
最常用的方法是ps aux,然后再通过管道使用grep命令过滤查找特定的进程,然后再对特定的进程进行操作。
 
ps aux | grep program_filter_word,ps -ef |grep tomcat
 
ps -ef|grep java|grep -v grep 显示出所有的java进程，去处掉当前的grep进程。
```

### 使用ps -aux

使用ps aux命令来查看，能以简单列表的形式显示出进程信息

 ps aux用于报告当前系统的进程状态。可以搭配kill指令随时中断、删除不必要的程序.
  ps命令是最基本同时也是非常强大的进程查看命令，使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多的资源等等，总之大部分信息都是可以通过执行该命令得到的.

  - a显示当前终端下的所有进程信息，包括其他用户的进程.
  - u使用以用户为主的格式输出进程信息.
  - x显示当前用户在所有终端下的进程

```text
ps查看系统中的所有进程

1）基本语法：
ps -aux - 显示进程信息，包括无终端的（x）和针对用户（u）的进程：如USER, PID, %CPU, %MEM等;

​ps –aux（功能描述：查看系统中所有进程）

ps -aux |grep 10216 #查看某一个进程相信情况

2）功能说明

​USER：该进程是由哪个用户产生的

​PID：进程的ID号

%CPU：该进程占用CPU资源的百分比，占用越高，进程越耗费资源；

%MEM：该进程占用物理内存的百分比，占用越高，进程越耗费资源；

VSZ：该进程占用虚拟内存的大小，单位KB；

RSS：该进程占用实际物理内存的大小，单位KB；

TTY：该进程是在哪个终端中运行的。其中tty1-tty7代表本地控制台终端，tty1-tty6是本地的字符界面终端，tty7是图形终端。pts/0-255代表虚拟终端。

STAT：进程状态。常见的状态有：R：运行、S：睡眠、T：停止状态、s：包含子进程、+：位于后台

START：该进程的启动时间

TIME：该进程占用CPU的运算时间，注意不是系统时间

COMMAND：产生此进程的命令名


//案例演示
[root@VM-8-10-centos bin]# ps -aux

//查看系统中所有的进程

ps -ef 
//通过管道查找某一个进程
[rzf@hadoop01 hive]$ ps -ef | grep hive
//杀死某一个进程
[rzf@hadoop01 hive]$ kill -9 14390

```

### 使用ps -elf

使用ps -elf命令来查看

 - e:显示系统内的所有进程信息.
 - l:使用长（long）格式显示进程信息.
 - -f:使用完整的（full）格式显示进程信息

### 使用pstree -aup命令来查看

> pstree以树状显示正在运行的进程。树的根节点为pid或init。如果指定了用户名，进程树将以用户所拥有的进程作为根节点。

使用pstree -aup命令来查看

可以带上|grep查询特定进程。例如 pstree -aup | grep php，以树状图的方式展现进程之间的派生关系，显示效果比较直观

- a显示每个程序的完整指令，包含路径，参数或是常驻服务的标示
- c不使用精简标示法
- G使用VT100终端机的列绘图字符
- h列出树状图时，特别标明现在执行的程序
- H<程序识别码>此参数的效果和指定-h参数类似，但特别标明指定的程序
- l采用长列格式显示树状图
- n用程序识别码排序。预设是以程序名称来排序
- p显示程序识别码
- u显示用户名称

### top：查看系统健康状态

```text
1）基本命令

​	top [选项]	

​	（1）选项：

        top –d 1//设置1秒刷新一次

        -d 秒数：指定top命令每隔几秒更新。默认是3秒在top命令的交互模式当中可以执行的命令：

        -i：使top不显示任何闲置或者僵死进程。

        -p：通过指定监控进程ID来仅仅监控某个进程的状态。

        -s ： 使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险。

(2）操作选项：

    P：		以CPU使用率排序，默认就是此项 

    M：		以内存的使用率排序 

    N：		以PID排序 

    q：		退出top
（3）查询结果字段解释
```

第一行信息为任务队列信息	

| 内容                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| 12:26:46                         | 系统当前时间                                                 |
| up 1 day, 13:32                  | 系统的运行时间，本机已经运行1天13小时32分钟                  |
| 2 users                          | 当前登录了两个用户                                           |
| load  average:  0.00, 0.00, 0.00 | 系统在之前1分钟，5分钟，15分钟的平均负载。一般认为小于1时，负载较小。如果大于1，系统已经超出负荷。 |

第二行为进程信息

| Tasks:  95 total | 系统中的进程总数                          |
| ---------------- | ----------------------------------------- |
| 1 running        | 正在运行的进程数                          |
| 94 sleeping      | 睡眠的进程                                |
| 0 stopped        | 正在停止的进程                            |
| 0 zombie         | 僵尸进程。如果不是0，需要手工检查僵尸进程 |

第三行为CPU信息

| Cpu(s):  0.1%us | 用户模式占用的CPU百分比                                      |
| --------------- | ------------------------------------------------------------ |
| 0.1%sy          | 系统模式占用的CPU百分比                                      |
| 0.0%ni          | 改变过优先级的用户进程占用的CPU百分比                        |
| 99.7%id         | 空闲CPU的CPU百分比                                           |
| 0.1%wa          | 等待输入/输出的进程的占用CPU百分比                           |
| 0.0%hi          | 硬中断请求服务占用的CPU百分比                                |
| 0.1%si          | 软中断请求服务占用的CPU百分比                                |
| 0.0%st          | st（Steal  time）虚拟时间百分比。就是当有虚拟机时，虚拟CPU等待实际CPU的时间百分比。 |

第四行为物理内存信息

| Mem:    625344k total | 物理内存的总量，单位KB                                       |
| --------------------- | ------------------------------------------------------------ |
| 571504k used          | 已经使用的物理内存数量                                       |
| 53840k free           | 空闲的物理内存数量，我们使用的是虚拟机，总共只分配了628MB内存，所以只有53MB的空闲内存了 |
| 65800k buffers        | 作为缓冲的内存数量                                           |

第五行为交换分区（swap）信息

| Swap:   524280k total | 交换分区（虚拟内存）的总大小 |
| --------------------- | ---------------------------- |
| 0k used               | 已经使用的交互分区的大小     |
| 524280k free          | 空闲交换分区的大小           |
| 409280k cached        | 作为缓存的交互分区的大小     |

退出操作：q或者ctrl+c或者ctrl+z或者esc

行上述命令后，可以按P、M、N对查询出的进程结果进行排序。

### 查看系统进程中所有的线程：

1）通过pstree命令（根据pid）进行查询：

```
ps -ef|grep java     //查找进程pid（比如这里查找java（tomcat）进程的pid）
pstree -p pid
//统计线程的个数
pstree -p 19135|wc -l
```

2）或者使用top命令查看（可以查看到线程情况）

```
top -Hp 19135       //下面结果中的Tasks 对应的47即是线程的个数

top - 14:05:55 up 391 days, 20:59,  1 user,  load average: 0.00, 0.00, 0.00

Tasks:  47 total,   0 running,  47 sleeping,   0 stopped,   0 zombie

Cpu(s):  0.2%us,  0.1%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st

Mem:   8058056k total,  7718656k used,   339400k free,   354216k buffers

Swap:        0k total,        0k used,        0k free,  4678160k cached
```

### pstree：查看进程树

~~~ java
1）基本语法：

	pstree [选项]

	选项

  -p：  显示进程的PID 

  -u：  显示进程的所属用户
//案例
pstree -u
pstree -d
~~~

### kill:杀死进程

~~~ java
1）基本语法：

kill -9 pid进程号

选项

-9 表示强迫进程立即停止
~~~

## 网络命令

### 基础语法

`netstat[<options>]`

**可选项**

```text
常用参数：
-a或--all：显示所有连线中的Socket；监听和未监听都会输出
-t或--tcp：显示TCP传输协议的连线状况；
-x或--unix：此参数的效果和指定"-A unix"参数相同；
-u或--udp：显示UDP传输协议的连线状况；
-l或--listening：显示监控中的服务器的Socket；
-p或--programs：显示正在使用Socket的程序识别码和程序名称；
-n或--numeric：直接使用ip地址，而不通过域名服务器；
-c或--continuous：持续列出网络状态；
-s或--statistice：显示网络工作信息统计表；
-r或--route：显示Routing Table；

其他参数；
-e或--extend：显示网络其他相关信息；
-A<网络类型>或--<网络类型>：列出该网络类型连线中的相关地址；
-C或--cache：显示路由器配置的快取信息；
-F或--fib：显示FIB；
-g或--groups：显示多重广播功能群组组员名单；
-h或--help：在线帮助；
-i或--interfaces：显示网络界面信息表单；
-M或--masquerade：显示伪装的网络连线；
-N或--netlink或--symbolic：显示网络硬件外围设备的符号连接名称；
-o或--timers：显示计时器；
-v或--verbose：显示指令执行过程；
-V或--version：显示版本信息；
-w或--raw：显示RAW传输协议的连线状况；
--ip或--inet：此参数的效果和指定"-A inet"参数相同。
```

netstat -ntlp //查看当前所有tcp端口
netstat -ntulp |grep 80 //查看所有80端口使用情况
netstat -an | grep 3306 //查看所有3306端口使用情况
netstat -lanp //查看一台服务器上面哪些服务及端口
ps -ef |grep mysqld //查看一个服务有几个端口。比如要查看mysqld
netstat -pnt |grep :3306 |wc //查看某一端口的连接数量,比如3306端口
netstat -anp |grep 3306 //查看某一端口的连接客户端IP 比如3306端口


#### 列出所有端口信息

> 包括监听和未监听的

```text
netsta -a 

输出:

netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 localhost:41835         0.0.0.0:*               LISTEN
tcp        0      0 localhost:33060         0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:9000            0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.53:domain       0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:8011            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:8001            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:redis           0.0.0.0:*               LISTEN
tcp        0      0 localhost:mysql         0.0.0.0:*               LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
```


### 列出所有tcp端口

```text
netstat -at

输出:

Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 localhost:41835         0.0.0.0:*               LISTEN
tcp        0      0 localhost:33060         0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:9000            0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.53:domain       0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:8011            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:8001            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:redis           0.0.0.0:*               LISTEN
tcp        0      0 localhost:mysql         0.0.0.0:*               LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
tcp6       0      0 [::]:9092               [::]:*                  LISTEN
tcp6       0      0 [::]:9000               [::]:*                  LISTEN
tcp6       0      0 [::]:45693              [::]:*                  LISTEN
tcp6       0      0 [::]:http-alt           [::]:*                  LISTEN
tcp6       0      0 [::]:36803              [::]:*                  LISTEN
tcp6       0      0 [::]:8011               [::]:*                  LISTEN
tcp6       0      0 [::]:8001               [::]:*                  LISTEN
tcp6       0      0 [::]:2181               [::]:*                  LISTEN
tcp6       0      0 [::]:redis              [::]:*                  LISTEN
tcp6       0      0 localhost:46842         127.0.1.1:9092          ESTABLISHED
tcp6       0      0 127.0.1.1:9092          localhost:47864         ESTABLISHED
tcp6       0      0 localhost:47864         127.0.1.1:9092          ESTABLISHED
tcp6       0      0 localhost:46848         127.0.1.1:9092          ESTABLISHED

列出所有udp端口：

netstat -au
```

### 列出所有监听端口

```text
netsta -l

只列出所有监听 tcp 端口:netstat -lt
只列出所有监听 udp 端口:netstat -lu
只列出所有监听 UNIX 端口:netstat -lx
```


### 禁止域名解析

**正常输出**

```text
 netstat -ltp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 localhost:41835         0.0.0.0:*               LISTEN      886/docker-proxy
tcp        0      0 localhost:33060         0.0.0.0:*               LISTEN      355/mysqld
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN      295/sshd: /usr/sbin
tcp        0      0 0.0.0.0:9000            0.0.0.0:*               LISTEN      863/docker-proxy
tcp        0      0 127.0.0.53:domain       0.0.0.0:*               LISTEN      153/systemd-resolve
tcp        0      0 0.0.0.0:8011            0.0.0.0:*               LISTEN      900/docker-proxy
tcp        0      0 0.0.0.0:8001            0.0.0.0:*               LISTEN      923/docker-proxy
tcp        0      0 0.0.0.0:redis           0.0.0.0:*               LISTEN      256/redis-server *:
tcp        0      0 localhost:mysql         0.0.0.0:*               LISTEN      355/mysqld
```

**禁止域名解析**

```text
netstat -nltp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:41835         0.0.0.0:*               LISTEN      886/docker-proxy
tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN      355/mysqld
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      295/sshd: /usr/sbin
tcp        0      0 0.0.0.0:9000            0.0.0.0:*               LISTEN      863/docker-proxy
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      153/systemd-resolve
tcp        0      0 0.0.0.0:8011            0.0.0.0:*               LISTEN      900/docker-proxy
tcp        0      0 0.0.0.0:8001            0.0.0.0:*               LISTEN      923/docker-proxy
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      256/redis-server *:
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      355/mysqld
tcp6       0      0 :::22                   :::*                    LISTEN      295/sshd: /usr/sbin
tcp6       0      0 :::9092                 :::*                    LISTEN      6300/java
tcp6       0      0 :::9000                 :::*                    LISTEN      870/docker-proxy
tcp6       0      0 :::45693                :::*                    LISTEN      6300/java
tcp6       0      0 :::8080                 :::*                    LISTEN      6936/java
tcp6       0      0 :::36803                :::*                    LISTEN      5800/java
tcp6       0      0 :::8011                 :::*                    LISTEN      909/docker-proxy
tcp6       0      0 :::8001                 :::*                    LISTEN      930/docker-proxy
tcp6       0      0 :::2181                 :::*                    LISTEN      5800/java
tcp6       0      0 :::6379                 :::*                    LISTEN      256/redis-server *:
```

### netstat:显示网络统计信息情况

~~~ java
1）基本语法：

netstat –anp
    
（功能描述：此命令用来显示整个系统目前的网络情况。例如目前的连接、数据包传递数据、或是路由表内容）

选项：

-an 按一定顺序排列输出

-p  表示显示哪个进程在调用

-nltp 查看tcp协议进程端口号
 
//案例，擦好看具体进程的网络情况
[root@VM-8-10-centos bin]# netstat -anp | grep 80
    
功能描述：netstat命令是一个监控TCP/IP网络的非常有用的工具，它可以显示路由表、实际的网络连接以及每一个网络接口设备的状态信息。 

基本语法：netstat [选项]

选项参数：

​	-t或--tcp：显示TCP传输协议的连线状况； 

	-u或--udp：显示UDP传输协议的连线状况；

​	-n或--numeric：直接使用ip地址，而不通过域名服务器； 

​	-l或--listening：显示监控中的服务器的Socket； 

​	-p或--programs：显示正在使用Socket的程序识别码（PID）和程序名称；
~~~

### 查看端口占用情况

**查看所有端口占用情况**

~~~ java
 netstat -nultp
 //杀死断后号所在的进程
 kill -9 pid
~~~

### 找出应用的端口

```text
并不是所有的进程都能找到，没有权限的会不显示，使用 root 权限查看所有的信息。
# netstat -ap | grep ssh
 
找出运行在指定端口的进程
# netstat -an | grep ':80'
```

## 解压类

### gzip/gunzip压缩

~~~ java
1）基本语法：

    gzip+文件		（功能描述：压缩文件，只能将文件压缩为*.gz文件）

    gunzip+文件.gz	（功能描述：解压缩文件命令）

2）特点：

（1）只能压缩文件不能压缩目录

（2）不保留原来的文件
    
gzip -d file.tar.gz
~~~

### zip/unzip压缩

~~~ java
1）基本语法：

zip + 参数 + XXX.zip + 将要压缩的内容 （功能描述：压缩文件和目录的命令，window/linux通用且可以压缩目录且保留源文件）

参数：

-r 压缩目录
~~~

- 案例演示

~~~ java
//压缩文件
[root@VM-8-10-centos rzf]# ll
total 4
-rw-r--r-- 1 root root     0 Dec 13 12:14 aa.txt
-rw-r--r-- 1 root root     0 Dec 13 12:14 bb.txt
drwxrwxr-x 2 rzf  group 4096 Dec 13 11:10 rr
[root@VM-8-10-centos rzf]# zip /home/rzf/file.zip aa.txt bb.txt 
  adding: aa.txt (stored 0%)
  adding: bb.txt (stored 0%)
[root@VM-8-10-centos rzf]# ll
total 8
-rw-r--r-- 1 root root     0 Dec 13 12:14 aa.txt
-rw-r--r-- 1 root root     0 Dec 13 12:14 bb.txt
-rw-r--r-- 1 root root   302 Dec 13 12:16 file.zip
drwxrwxr-x 2 rzf  group 4096 Dec 13 11:10 rr
//将aa.txt bb.txt文件压缩到/home/rzf/file.zip目录下，-r参数可以省略
//文件解压缩
[root@VM-8-10-centos rzf]# unzip f.zip 
Archive:  f.zip
replace aa.txt? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
 extracting: aa.txt                  
replace bb.txt? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
 extracting: bb.txt                  
[root@VM-8-10-centos rzf]# ll
total 12
-rw-r--r-- 1 root root     0 Dec 13 12:14 aa.txt
-rw-r--r-- 1 root root     0 Dec 13 12:14 bb.txt
-rw-r--r-- 1 root root   302 Dec 13 12:16 file.zip
-rw-r--r-- 1 root root   302 Dec 13 12:18 f.zip
drwxrwxr-x 2 rzf  group 4096 Dec 13 11:10 rr
[root@VM-8-10-centos rzf]# 
//如果想使用unzip解压文件到其他的目录，需要使用-d参数
[rzf@hadoop01 software]$ unzip mysql-libs.zip -d /opt/module/
~~~

### tar打包

~~~ java
1）基本语法：

tar + 参数 + XXX.tar.gz + 将要打包进去的内容		（功能描述：打包目录，压缩后的文件格式.tar.gz）

参数：

    -c 产生.tar打包文件

    -v 显示详细信息

    -f 指定压缩后的文件名

    -z 打包同时压缩

    -x 解包.tar文件

（1）压缩：tar -zcvf  XXX.tar.gz   n1.txt    n2.txt

	压缩多个文件

（2）解压：tar -zxvf  XXX.tar.gz

	解压到当前目录

	解压到/opt目录： tar -zxvf test.tar.gz –C /opt
~~~

- 案例演示

~~~ java
//压缩文件
[root@VM-8-10-centos rzf]# ll
total 20
-rw-r--r-- 1 root root   263 Dec 13 12:20 aa.txt
-rw-r--r-- 1 root root     0 Dec 13 12:14 bb.txt
-rw-r--r-- 1 root root   302 Dec 13 12:16 file.zip
-rw-r--r-- 1 root root   302 Dec 13 12:18 f.zip
drwxrwxr-x 2 rzf  group 4096 Dec 13 11:10 rr
-rw-r--r-- 1 root root   562 Dec 13 12:21 test.tzr
[root@VM-8-10-centos rzf]# tar -zcvf my.tar aa.txt bb.txt f.zip test.tzr 
aa.txt
bb.txt
f.zip
test.tzr
[root@VM-8-10-centos rzf]# ll
total 24
-rw-r--r-- 1 root root   263 Dec 13 12:20 aa.txt
-rw-r--r-- 1 root root     0 Dec 13 12:14 bb.txt
-rw-r--r-- 1 root root   302 Dec 13 12:16 file.zip
-rw-r--r-- 1 root root   302 Dec 13 12:18 f.zip
-rw-r--r-- 1 root root  1138 Dec 13 12:21 my.tar
drwxrwxr-x 2 rzf  group 4096 Dec 13 11:10 rr
-rw-r--r-- 1 root root   562 Dec 13 12:21 test.tzr
[root@VM-8-10-centos rzf]# 
//解压文件：tar -zxvf  XXX.tar.gz，表示解压到当前文件，解压到其他文件需要带上参数-C
[root@VM-8-10-centos rzf]# ll
total 24
-rw-r--r-- 1 root root   263 Dec 13 12:20 aa.txt
-rw-r--r-- 1 root root     0 Dec 13 12:14 bb.txt
-rw-r--r-- 1 root root   302 Dec 13 12:16 file.zip
-rw-r--r-- 1 root root   302 Dec 13 12:18 f.zip
-rw-r--r-- 1 root root  1138 Dec 13 12:21 my.tar
drwxrwxr-x 2 rzf  group 4096 Dec 13 11:10 rr
-rw-r--r-- 1 root root   562 Dec 13 12:21 test.tzr
[root@VM-8-10-centos rzf]# tar -zxvf my.tar -C /home/
aa.txt
bb.txt
f.zip
test.tzr
[root@VM-8-10-centos rzf]# cd ..
[root@VM-8-10-centos home]# ll
total 20
-rw-r--r-- 1 root       root        263 Dec 13 12:20 aa.txt
-rw-r--r-- 1 root       root          0 Dec 13 12:14 bb.txt
-rw-r--r-- 1 root       root        302 Dec 13 12:18 f.zip
drwx------ 5 lighthouse lighthouse 4096 Dec 12 23:10 lighthouse
drwx------ 5 rzf        group      4096 Dec 13 12:27 rzf
-rw-r--r-- 1 root       root        562 Dec 13 12:21 test.tzr
[root@VM-8-10-centos home]# 

~~~

## 后台服务管理

### service后台服务管理

~~~ java
//network服务也可以换成其他的服务，比如mysql
1）service network status   查看指定服务的状态

2）service network stop    停止指定服务

3）service network start    启动指定服务

4）service network restart   重启指定服务

5）service --status-all      查看系统中所有的后台服务
//可以通过进程查看是否启动
ps -ef |grep mysql(服务名)
~~~

### chkconfig:设置后台服务自启配置

~~~ java
1）chkconfig   			查看所有服务器自启配置

2）chkconfig iptables off   关掉指定服务的自动启动

3）chkconfig iptables on   开启指定服务的自动启动
~~~

## crond系统定时服务

### crond服务管理：

~~~ java
service crond restart 			（重新启动服务）
~~~

### crontab定时任务设置

~~~ java
1）基本语法

crontab [选项]

选项： 

  -e：    编辑crontab定时任务 

  -l：    查询crontab任务 

  -r：    删除当前用户所有的crontab任务
2)参数说明
	crontab -e 
	（1）进入crontab编辑界面。会打开vim编辑你的工作。
~~~

（1）执行的任务

| 项目      | 含义                 | 范围                    |
| --------- | -------------------- | ----------------------- |
| 第一个“*” | 一小时当中的第几分钟 | 0-59                    |
| 第二个“*” | 一天当中的第几小时   | 0-23                    |
| 第三个“*” | 一个月当中的第几天   | 1-31                    |
| 第四个“*” | 一年当中的第几月     | 1-12                    |
| 第五个“*” | 一周当中的星期几     | 0-7（0和7都代表星期日） |

（2）特殊符号

| 特殊符号 | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| *        | 代表任何时间。比如第一个“*”就代表一小时中每分钟都执行一次的意思。 |
| ，       | 代表不连续的时间。比如“0 8,12,16 * * * 命令”，就代表在每天的8点0分，12点0分，16点0分都执行一次命令 |
| -        | 代表连续的时间范围。比如“0 5  *  *  1-6命令”，代表在周一到周六的凌晨5点0分执行命令 |
| */n      | 代表每隔多久执行一次。比如“*/10  *  *  *  *  命令”，代表每隔10分钟就执行一遍命令 |

（3）特定时间执行命令

| 时间              | 含义                                                         |
| ----------------- | ------------------------------------------------------------ |
| 45 22 * * * 命令  | 在22点45分执行命令                                           |
| 0 17 * * 1 命令   | 每周1 的17点0分执行命令                                      |
| 0 5 1,15 * * 命令 | 每月1号和15号的凌晨5点0分执行命令                            |
| 40 4 * * 1-5 命令 | 每周一到周五的凌晨4点40分执行命令                            |
| */10 4 * * * 命令 | 每天的凌晨4点，每隔10分钟执行一次命令                        |
| 0 0 1,15 * 1 命令 | 每月1号和15号，每周1的0点0分都会执行命令。注意：星期几和几号最好不要同时出现，因为他们定义的都是天。非常容易让管理员混乱。 |

3）案例：

~~~ java
*/5 * * * * /bin/echo ”11” >> /tmp/test
~~~



1. service后台服务管理

   1）service network status   查看指定服务的状态

   2）service network stop    停止指定服务

   3）service network start    启动指定服务

   4）service network restart   重启指定服务

   5）service --status-all      查看系统中所有的后台服务

2. chkconfig:设置后台服务自启配置

   1）chkconfig   			查看所有服务器自启配置

   2）chkconfig iptables off   关掉指定服务的自动启动

   3）chkconfig iptables on   开启指定服务的自动启动

3. 功能描述：netstat命令是一个监控TCP/IP网络的非常有用的工具，它可以显示路由表、实际的网络连接以及每一个网络接口设备的状态信息。 

   基本语法：netstat [选项]

   选项参数：

   ​	-t或--tcp：显示TCP传输协议的连线状况； 

   -u或--udp：显示UDP传输协议的连线状况；

   ​	-n或--numeric：直接使用ip地址，而不通过域名服务器； 

   ​	-l或--listening：显示监控中的服务器的Socket； 

   ​	-p或--programs：显示正在使用Socket的程序识别码（PID）和程序名称；

4. crond系统定时服务

   1. crond服务管理：service crond restart 			（重新启动服务）

   2. crontab定时任务设置

      1）基本语法

      crontab [选项]

      选项： 

        -e：    编辑crontab定时任务 

        -l：    查询crontab任务 

        -r：    删除当前用户所有的crontab任务

   （1）进入crontab编辑界面。会打开vim编辑你的工作。

   \* * * * * 执行的任务

   | 项目      | 含义                 | 范围                    |
   | --------- | -------------------- | ----------------------- |
   | 第一个“*” | 一小时当中的第几分钟 | 0-59                    |
   | 第二个“*” | 一天当中的第几小时   | 0-23                    |
   | 第三个“*” | 一个月当中的第几天   | 1-31                    |
   | 第四个“*” | 一年当中的第几月     | 1-12                    |
   | 第五个“*” | 一周当中的星期几     | 0-7（0和7都代表星期日） |

   （2）特殊符号

   | 特殊符号 | 含义                                                         |
   | -------- | ------------------------------------------------------------ |
   | *        | 代表任何时间。比如第一个“*”就代表一小时中每分钟都执行一次的意思。 |
   | ，       | 代表不连续的时间。比如“0 8,12,16 * * * 命令”，就代表在每天的8点0分，12点0分，16点0分都执行一次命令 |
   | -        | 代表连续的时间范围。比如“0 5  *  *  1-6命令”，代表在周一到周六的凌晨5点0分执行命令 |
   | */n      | 代表每隔多久执行一次。比如“*/10  *  *  *  *  命令”，代表每隔10分钟就执行一遍命令 |

   （3）特定时间执行命令

   | 时间              | 含义                                                         |
   | ----------------- | ------------------------------------------------------------ |
   | 45 22 * * * 命令  | 在22点45分执行命令                                           |
   | 0 17 * * 1 命令   | 每周1 的17点0分执行命令                                      |
   | 0 5 1,15 * * 命令 | 每月1号和15号的凌晨5点0分执行命令                            |
   | 40 4 * * 1-5 命令 | 每周一到周五的凌晨4点40分执行命令                            |
   | */10 4 * * * 命令 | 每天的凌晨4点，每隔10分钟执行一次命令                        |
   | 0 0 1,15 * 1 命令 | 每月1号和15号，每周1的0点0分都会执行命令。注意：星期几和几号最好不要同时出现，因为他们定义的都是天。非常容易让管理员混乱。 |

   3）案例：

   */5 * * * * /bin/echo ”11” >> /tmp/test

## 软件包管理

### RPM概述

`RPM（RedHat Package Manager）`，`RedHat`软件包管理工具，类似`windows`里面的`setup.exe` 是`Linux`这系列操作系统里面的打包安装工具，它虽然是`RedHat`的标志，但理念是通用的。

- RPM包的名称格式

~~~ java
Apache-1.3.23-11.i386.rpm

\- “apache” 软件名称

\- “1.3.23-11”软件的版本号，主版本和此版本

\- “i386”是软件所运行的硬件平台，Intel 32位微处理器的统称

\- “rpm”文件扩展名，代表RPM包
~~~

### RPM查询命令（rpm -qa）

如果软件使用rpm命令安装，那么就可以使用rpm命令查询。

~~~ java
1．基本语法

rpm -qa				（功能描述：查询所安装的所有rpm软件包）

2．经验技巧

由于软件包比较多，一般都会采取过滤。rpm -qa | grep -i(-i可以添加，也可以不添加) rpm软件包
rpm -qa |grep firefox（关键字） 
~~~

### **RPM卸载命令（rpm -e）**

~~~ java
1．基本语法

（1）rpm -e RPM软件包   

（2） rpm -e --nodeps 软件包  
~~~

- 选项说明

| 选项     | 功能                                                         |
| -------- | ------------------------------------------------------------ |
| -e       | 卸载软件包                                                   |
| --nodeps | 卸载软件时，不检查依赖。这样的话，那些使用该软件包的软件在此之后可能就不能正常工作了。 |

- 案例实操

~~~ java
//卸载firefox软件 
rpm -e firefox
~~~

### **RPM安装命令（rpm -ivh）**

~~~ java
1．基本语法

​	rpm -ivh RPM包全名
~~~

- 选项说明

| 选项     | 功能                     |
| -------- | ------------------------ |
| -i       | -i=install，安装         |
| -v       | -v=verbose，显示详细信息 |
| -h       | -h=hash，进度条          |
| --nodeps | --nodeps，不检测依赖进度 |

~~~ java
 rpm -ivh firefox-45.0.1-1.el6.centos.x86_64.rpm 
~~~

## shell编程

### 概述

Shell是一个命令行解释器，它为用户提供了一个向Linux内核发送请求以便运行程序的界面系统级程序，用户可以用Shell来启动、挂起、停止甚至是编写一些程序。Shell还是一个功能相当强大的编程语言，易编写、易调试、灵活性强。Shell是解释执行的脚本语言，在Shell中可以调用Linux系统命令。

### shell脚本的执行方式

1）echo输出命令

​	（1）基本语法：

​		echo [选项] [输出内容]

选项： 

  -e：  支持反斜线控制的字符转换

| 控制字符 | 作        用                                                 |
| -------- | ------------------------------------------------------------ |
| \\       | 输出\本身                                                    |
| \a       | 输出警告音                                                   |
| \b       | 退格键，也就是向左删除键                                     |
| \c       | 取消输出行末的换行符。和“-n”选项一致                         |
| \e       | ESCAPE键                                                     |
| \f       | 换页符                                                       |
| \n       | 换行符                                                       |
| \r       | 回车键                                                       |
| \t       | 制表符，也就是Tab键                                          |
| \v       | 垂直制表符                                                   |
| \0nnn    | 按照八进制ASCII码表输出字符。其中0为数字零，nnn是三位八进制数 |
| \xhh     | 按照十六进制ASCII码表输出字符。其中hh是两位十六进制数        |

~~~ jva
echo -e "hello\tword"//单引号也可以
hello   word
~~~

2. 第一个Shell脚本

~~~ java
//在etc目录下面创建helloword.sh脚本，
//在里面输入echo "helloword"
//升级权限
chmod 777 helloword.sh
//执行脚本
./helloword.sh//./标示在当前目录执行
//也可以用绝对路径执行
//也可以用命令执行
[root@hadoop101 sh]# sh ./helloword.sh 
helloword
[root@hadoop101 sh]# bash ./helloword.sh //也可以添加绝对路径
helloword
~~~

脚本的常用执行方式

第一种：输入脚本的绝对路径或相对路径

（1）首先要赋予helloworld.sh 脚本的+x权限

[atguigu@hadoop102 datas]$ chmod 777 helloworld.sh

（2）执行脚本

​	/root/helloWorld.sh

​	./helloWorld.sh

第二种：bash或sh+脚本（不用赋予脚本+x权限）

​	sh /root/helloWorld.sh

​	sh helloWorld.sh

3. ##  **shell**中的变量

   1. Linux Shell中的变量分为“系统变量”和“用户自定义变量”，可以通过set命令查看系统变量。

   2）系统变量：$HOME、$PWD、$SHELL、$USER等等

   3）显示当前shell中所有变量：set

~~~ java
//获取当前用户的家目录
[root@hadoop101 sh]# echo $HOME
/root
//获取并打印当前的绝对路径值
[root@hadoop101 sh]# echo $PWD
/etc/sh
//获取shell语言脚本的启动命令
[root@hadoop101 sh]# echo $SHELL
/bin/bash
//获取当前登录的用户
[root@hadoop101 sh]# echo $USER
root
~~~

2. 定义变量

1）基本语法：

​	变量=值，注意，等号两边没有空格

2）变量定义规则

​	（1）变量名称可以由字母、数字和下划线组成，但是不能以数字开头。

​	（2）等号两侧不能有空格

​	（3）变量名称一般习惯为大写

​	（4）双引号和单引号有区别，双引号仅将空格脱意，单引号会将所有特殊字符脱意

~~~ java
//（1）定义变量并且打印变量值
[root@hadoop101 sh]# ss=8
[root@hadoop101 sh]# echo $ss
8
//撤销变量
unset ss     
//（3）声明静态的变量B=2，不能unset,只可以读取，不允许修改
readonly B=2  
//（4）可把变量提升为全局环境变量，可供其他shell程序使用
export 变量名  
~~~

3. 单引号和双引号的区别

~~~ java
//双引号仅仅将空格脱意，而单引号将所有特殊字符脱意
//如果加上-e,单双引号都脱意
[root@hadoop101 sh]# ss="xxx\tsss"
[root@hadoop101 sh]# echo -e $ss
xxx     sss
[root@hadoop101 sh]# ss="xxx xxx"
[root@hadoop101 sh]# echo $ss
xxx xxx
//如果不添加-e，两者都会原样输出
~~~

4. 将命令的返回值赋给变量

~~~ java
//A=`ls -la` 反引号，运行里面的命令，并把结果返回给变量A
//A=$(ls -la) 等价于反引号
[root@hadoop101 sh]# a=`ll`
[root@hadoop101 sh]# echo $a
总用量 4 -rwxrwxrwx. 1 root root 29 4月 4 16:26 helloword.sh
//第二种用法
[root@hadoop101 sh]# a=$(ll)
[root@hadoop101 sh]# echo $a
总用量 4 -rwxrwxrwx. 1 root root 29 4月 4 16:26 helloword.sh
~~~

5. 设置环境变量

~~~ java
（1）export 变量名=变量值	（功能描述：设置环境变量的值）
（2）echo $变量名			（功能描述：查询环境变量的值）
（3）source 配置文件			（功能描述：让修改后的配置信息立即生效）
~~~

6. 位置参数变量

   1）基本语法

   ​	$n	（功能描述：n为数字，$0代表命令本身，$1-$9代表第一到第九个参数，十以上的参数，十以上的参数需要用大括号包含，如${10}）

   ​	$*	（功能描述：这个变量代表命令行中所有的参数，$*把所有的参数看成一个整体）

   ​	$@	（功能描述：这个变量也代表命令行中所有的参数，不过$@把每个参数区分对待）

   ​	$#	（功能描述：这个变量代表命令行中所有参数的个数）

~~~ java
echo "$1"//输出第一个参数
echo "$2"//输出第二个参数
echo "$*"//输出所有参数
echo "$#"//输出参数的个数
//$*和$@的区别
#!/bin/bash

for i in "$*"
do
        echo "$i"
done

for i in "$@"
do
        echo "$i"
done
~           
[root@hadoop101 sh]# ./fo.sh 2 4 6 8
2 4 6 8
2
4
6
8
//$*标示把参数全部一次性输出，而$@标示把参数一次一个的输出。
~~~

7. 预定义变量

   1）基本语法：

   ​	$？		（功能描述：最后一次执行的命令的返回状态。如果这个变量的值为0，证明上一个命令正确执行；如果这个变量的值为非0（具体是哪个数，由命令自己来决定），则证明上一个命令执行不正确了。）

   $$		（功能描述：当前进程的进程号（PID））

   $!		（功能描述：后台运行的最后一个进程的进程号（PID））

~~~ java
echo "$$" //打印当前的进程号
./helloword.sh & //调用别的脚本，并且变成后台运行程序
 echo "$!" //打印后台运行程序最有一个进程号
~~~

8. 运算符

   1）基本语法：

   （1）“$((运算式))”或“$[运算式]”

   （2）expr m + n 

   注意expr运算符间要有空格

~~~ java
[root@hadoop101 sh]# a=$((2+3))
[root@hadoop101 sh]# echo $a
5
[root@hadoop101 sh]# a=$[5*8]
[root@hadoop101 sh]# echo $a
40
[root@hadoop101 sh]# expr 2 + 3
5//注意加号两端有空格
    [root@hadoop101 sh]# a=$[(2+3)*4]
[root@hadoop101 sh]# echo $a
20//不需要空格
//expr分布计算
S=`expr 2 + 3`
expr $S \* 4
//expr一步完成计算
expr `expr 2 + 3` \* 4
echo `expr \`expr 2 + 3\`\*4`
~~~

8. 条件判断

   基本语法：

   [ condition ]（注意condition前后要有空格）

   \#非空返回true，可使用$?验证（0为true，>1为false）

9. 常用判断谈条件

   1）两个整数之间比较

   = 字符串比较

   -lt 小于

   -le 小于等于

   -eq 等于

   -gt 大于

   -ge 大于等于

   -ne 不等于

   2）按照文件权限进行判断

   -r 有读的权限

   -w 有写的权限

   -x 有执行的权限

   3）按照文件类型进行判断

   -f 文件存在并且是一个常规的文件

   -e 文件存在

   -d 文件存在并是一个目录

~~~ java
//23是否大于等于22
[ 23 -ge 22 ]
//student.txt是否具有写权限
[ -w student.txt ]
//root/install.log目录中的文件是否存在
 [ -e /root/install.log ]
~~~

10. if判断

    1）基本语法：

    if [ 条件判断式 ];then 

      程序 

    fi 

    或者 

    if [ 条件判断式 ] 

      then 

    ​    程序 

    fi

    ​	注意事项：（1）[ 条件判断式 ]，中括号和条件判断式之间必须有空格

~~~ java
#!/bin/bash


if [ $1 -eq "123" ]//if和条件表达之间有空格
        then
        echo "123"
elif [ $1 -eq "456" ]
        then
        echo "456"
fi
~            
~~~

11. case语句

    1）基本语法：

    case $变量名 in 

      "值1"） 

    ​    如果变量的值等于值1，则执行程序1 

    ​    ;; 

      "值2"） 

    ​    如果变量的值等于值2，则执行程序2 

    ​    ;; 

      …省略其他分支… 

      *） 

    ​    如果变量的值都不是以上的值，则执行此程序 

    ​    ;; 

    esac

12. for循环

    1）基本语法1：

    for 变量 in 值1 值2 值3… 

      do 

    ​    程序 

      done

~~~ java
//计算100累加和
sum=0
for((i=1; i<="$1"; i++))
do
        sum=$[$sum+$i]
done
echo "$sum"//只要用变量的地方就要添加$,并且在使用变量前要进行定义
~~~

13. while循环

    1）基本语法：

    while [ 条件判断式 ] 

      do 

    ​    程序 

      done

~~~ JAVA
sum=0
i=0
while [ $i -le "$1" ]
do
        sum=$[$sum+$i]
        i=$[$i+1]
done
echo "$sum"
~~~

14. read读取控制台输入

    1）基本语法：

    ​	read(选项)(参数)

    ​	选项：

    -p：指定读取值时的提示符；

    -t：指定读取值时等待的时间（秒）。

    参数	

    ​	变量：指定读取值的变量名

~~~ java
 read -p "please input your name:" NAME
please input your name:rzf
[root@hadoop101 sh]# echo $NAME
rzf
//提示在3秒内输入内容，没输入的话是空值
[root@hadoop101 sh]# read -t 3 -p "please input your name:" NAME
~~~

15. 系统函数

    1）basename基本语法

    basename [pathname] [suffix]		

    basename [string] [suffix]  	（功能描述：basename命令会删掉所有的前缀包括最后一个（‘/’）字符，然后将字符串显示出来。

    选项：

    suffix为后缀，如果suffix被指定了，basename会将pathname或string中的suffix去掉。

~~~ java
//没有指定suffix
[root@hadoop101 sh]# basename /etc/sh/while.sh 
while.sh
//指定后缀sh
[root@hadoop101 sh]# basename /etc/sh/while.sh sh
while.
~~~

​	dirname基本语法

​	dirname 文件绝对路径		（功能描述：从给定的包含绝对路径的文件名中去除文件名（非目录的部分），然后返回剩下的路径（目录的部分））

~~~ java
[root@hadoop101 sh]# dirname  /etc/sh/while.sh          
/etc/sh
~~~

16. 自定义函数

~~~ java
[ function ] funname[()]
	{
		Action;
		[return int;]
	}
	
	function start() / function start / start()
~~~

注意：

​	（1）必须在调用函数地方之前，先声明函数，shell脚本是逐行运行。不会像其它语言一样先编译。

​	（2）函数返回值，只能通过$?系统变量获得，可以显示加：return返回，如果不加，将以最后一条命令运行结果，作为返回值。return后跟数值n(0-255)

~~~ java
//打印比你输入小的所有数字
i=0
while [ $i -le "$1" ]
do
        echo "$i"
        i=$[ $i+1 ]
        sleep 1
done
//求两个数的和
function getSum()
{
        sum=0
        sum=$[ $1+$2 ]
        echo "$sum"

}

read -p "Please input number1:" NUM1
read -p "please input number2:" NUM2
getSum $NUM1 $NUM2;
~~~

### yum仓库配置

1. 概述

   YUM（全称为 Yellow dog Updater, Modified）是一个在Fedora和RedHat以及CentOS中的Shell前端软件包管理器。基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

   在Linux上使用源码的方式安装软件非常满分，使用yum可以简化安装的过程

2. 常用命令：

   1）基本语法：

   yum install -y httpd			（功能描述：安装httpd并确认安装）

   yum list					（功能描述：列出所有可用的package和package组）

   yum clean all				（功能描述：清除所有缓冲数据）

   yum deplist httpd			（功能描述：列出一个包所有依赖的包）

   yum remove httpd			（功能描述：删除httpd）

   2）案例实操

   ​	yum install -y tree

3. 制作网络版yum源。

   （1）前提条件linux系统必须可以联网

   （2）在Linux环境环境中访问该网络地址：<http://mirrors.163.com/.help/centos.html>，在使用说明中点击CentOS6->再点击保存

   （3）**在打开的终端中输入如下命令，就可以找到文件的保存位置。**

   （4）替换本地yum文件

   1. 把下载的文件移动到/etc/yum.repos.d/目录

   ```java
   sudo cp CentOS6-Base-163.repo /etc/yum.repos.d/
   ```

   2. 进入到/etc/yum.repos.d/目录,用CentOS6-Base-163.repo替换CentOS-Base.rep

   （5）安装命令

   ```java
   [root@hadoop101 yum.repos.d]# yum clean all
   [root@hadoop101 yum.repos.d]# yum makecache
   //安装tree插件试试
   yum install -y tree
   //tree插件可以把目录文件以树形结构显示出来
   ```

   4. 制作本地yum源

      1）为什么要制作本地YUM源

      YUM源虽然可以简化我们在Linux上安装软件的过程，但是生成环境通常无法上网，不能连接外网的YUM源，说以接就无法使用yum命令安装软件了。为了在内网中也可以使用yum安装相关的软件，就要配置yum源。

      YUM源其实就是一个保存了多个RPM包的服务器，可以通过http的方式来检索、下载并安装相关的RPM包

   2）制作本地YUM源

   （1）准备一台Linux服务器，版本CentOS-6.8-x86_64-bin-DVD1.iso

   （2）配置好这台服务器的IP地址

   （3）将CentOS-6.8-x86_64-bin-DVD1.iso镜像挂载到/mnt/cdrom目录

   ==应为镜像文件中包含了很多各种各样的安装包==

   ```java
   //建立挂载目录
   mkdir /mnt/cdrom
   //挂载
    mount -t iso9660 /dev/cdrom /mnt/cdrom
   ```

   （4）修改本机上的YUM源配置文件，将源指向自己

   备份原有的YUM源的配置文件

   ~~~ java
   cd /etc/yum.repos.d/
   cp CentOS-Base.repo  CentOS-Base.repo.bak
   //进行备份
   cp -r CentOS-Base.repo.bak CentOS-Base.repo
   //修改内容如下
   name=CentOS-Local
   baseurl=file:///var/iso
   gpgcheck=1
   enabled=1   #增加改行，使能
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
   //清除YUM缓冲
   yum clean all
   //列出可用的YUM源
   yum repolist
   //安装相应的软件
   yum install -y httpd//httpd类似于http协议
   //开启httpd使用浏览器访问http://hadoop101:80（如果访问不通，检查防火墙是否开启了80端口或关闭防火墙）
   //如果访问不到就执行：service httpd start
   //将YUM源配置到httpd（Apache Server）中，其他的服务器即可通过网络访问这个内网中的YUM源了
   cp -r /mnt/cdrom/ /var/www/html/CentOS
   //取消先前挂载的镜像
   umount /mnt/cdrom
   //在浏览器中访问http://192.168.11.101/CentOS/
   //让其他需要安装RPM包的服务器指向这个YUM源，准备一台新的服务器，备份或删除原有的YUM源配置文件
   备份原有的YUM源的配置文件
   cd /etc/yum.repos.d/
   cp CentOS-Base.repo  CentOS-Base.repo.bak
   //编辑CentOS-Base.repo文件
   [base]
   name=CentOS-hadoop101
   baseurl=http://192.168.11.101/CentOS
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
   //在这台新的服务器上执行YUM的命令
   yum clean all
   yum repolist
   ~~~

   



