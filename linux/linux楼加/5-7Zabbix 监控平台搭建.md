---
show: step
version: 1.0
enable_checker: true
---
# Zabbix 监控平台搭建

## 1 实验介绍

#### 1.1 实验内容

老牌的系统监控工具有两个最著名的，分别是 `Nagios` 和 `Zabbix`。Nagios 是免费开源软件，功能简单轻量，上手快，但是解决复杂问题需要自己做很多工作。Zabbix 是商业开源软件，功能齐全，图形展示功能强大，插件多，但上手较慢，需要花较多时间才能熟练掌握。这里推荐使用 Zabbix，虽然上手较慢，但一旦熟悉之后，会节省后面很多时间。本节实验主要介绍 Zabbix 的基本架构以及如何搭建。

#### 1.2 实验知识点

+ Zabbix 的介绍

+ Zabbix 的架构原理

+ Zabbix 的平台的搭建

#### 1.3 推荐阅读

+ [Zabbix 中文社区](http://www.zabbix.org.cn/)

+ [Zabbix 官网](https://www.zabbix.com/)

+ [Zabbix wiki](https://en.wikipedia.org/wiki/Zabbix)

+ [Zabbix.org](http://www.zabbix.org/wiki/Main_Page)

## 2 Zabbix 的介绍

首先介绍 Zabbix。

### 2.1 概述

Zabbix 是一个基于 Web 界面的企业级的分布式开源监控方法，对各种网络参数以及服务器的健康性和完整性进行监控，保证系统的安全运行。它是由 Alexei Vladishev 创建的，目前是 [Zabbix SIA](https://www.zabbix.com/pr) 在进行持续开发和支持。

Zabbix 的通知机制较为灵活，能够允许用户给触发事件配置基于邮件、手机、MSN 等方式发送警告，来让系统管理员快速定位并解决发生的问题。而对于数据存储方面，Zabbix 提供了详细的报告和对数据进行可视化的操作功能。

**Zabbix** 的源代码是免费发行，可供公众任意使用。

### 2.2 组成结构

Zabbix 由以下几个主要软件组件构成：

+ **Server**

Zabbix server 是监控代理程序、报告系统可用性、系统完成整性和统计信息的核心组件。具有网络状态的监控，数据收集等功能，可以运行在 Linux、 Solaris、 HP-UX、 AIX、OS X 等平台上。

+ **Agent 监控代理**

Zabbix Agents 监控代理部署在监控目标上，能够主动监控本地资源和应用程序，并将收集到的数据报告给 Zabbix Server，完成信息的收集。

> Zabbix Server 不仅可以单独对服务器状态进行监控，还可以和 Zabbix Agent 配合，获取更多的服务监控数据。

+ **Proxy 代理服务器**

Zabbix proxy 可以替 Zabbix Server 收集性能和可用性数据。`Proxy` 代理服务器是 Zabbix 软件可选择部署的一个部分。

+ **数据库存储**

Zabbix 可使用 `MySQL`，`PostgreSQL`，`SQLite`，`Oracle` 或 `IBM DB2` 来存储数据。所有配置信息和 Zabbix 收集到的数据都被存储在数据库中。

+ **Web 界面**

Zabbix 提供了基于 `Web` 的界面。这个界面是 Zabbix Server 的一部分，通常(但不一定)跟 Zabbix Server 运行在同一台物理机器上。

> 注：如果使用 SQLite，Zabbix Web 界面必须要跟 Zabbix Server 运行在同一台物理机器上。

#### 2.3 Zabbix 的架构原理

#### 2.3.1 架构

常见的架构模式有以下两种：

① **server-agentd** 模式

这是最简单的架构，常用于监控主机比较少的情况。

② **server-proxy-agentd** 模式

这个常用于比较多的机器，使用 proxy 进行分布式监控，有效的减轻 server 端的压力。

这里给出第二种 zabbix 架构，如下图：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4092timestamp1512038285485.png/wm)

Zabbix 的架构中主要包括了：**Zabbix Server**、**Zabbix Proxy（可选部分）** 和**被监控主机群**。其中核心部分是 Zabbix Server ，主要包含三个组件：`Zabbix Server`、`Zabbix database` 和 `Zabbix Web GUI` 。

+ **Zabbix Server**

主要是负责接收 `agent` 发送的报告信息，所有配置、统计数据及操作数据均由其组织进行。在 `Zabbix Server` 中有一个 `item` 监控项主要是用来实现数据收集 。而调用的进程工具是 `zabbix_get`（在 `server` 或者 `proxy` 端执行获取远程客户端信息的命令）

+ **Zabbix database**

主要用于实现数据库服务功能，专用于存储所有配置信息，以及由 `zabbix` 收集的数据。

+ **Zabbix Web GUI**

`zabbix` 的 `GUI` 接口，通常与 `server` 运行在同一台机器上 。主要基于 `Apache PHP` 。

#### 2.3.2 进程构成

默认情况下 `zabbix` 包含 5 个程序：`zabbix_agentd`、`zabbix_get`、`zabbix_proxy`、`zabbix_sender`、`zabbix_server`，另外 `zabbix_java_gateway` 是可选，需要另外安装。

+ **zabbix_server**

服务端守护进程。通过执行轮询和对数据的捕获，来计算是否满足触发器的条件从而向用户发出通知。同时 server 进程也是所有配置、操作数据的存储库，并且所有进程的数据最终都是提交到 server 上。

+ **zabbix_agentd**

客户端守护进程，主要用于收集客户端数据，并将数据报告给 server 进行处理。

+ **zabbix_get**

单独使用的命令，在 server 或者 proxy 端执行获取远程客户端信息的命令。也可以用来做用户排错。

+ **zabbix_sender**

主要用于发送数据给 server 或者 proxy，通常用于耗时比较长的检查，定期发送可用性和性能数据。

+ **zabbix_proxy**

zabbix 代理守护进程。功能类似 server，唯一不同的是它只是一个中转站，可以收集来自一个或多个受监控设备的监控数据，并将的数据信息发送给 Zabbix 服务器。需要使用独立的数据库。

+ **zabbix_java_gateway**（可选）

zabbix 2.0 之后引入的一个功能。Java 网关，类似 agentd，但是只用于 Java 方面。

### 2.4 功能特点

Zabbix 的主要功能有：对 CPU 的负荷情况、内存和磁盘的使用、网络状态以及端口情况和日志的监控等。

Zabbix 是一个高度集成的网络监控解决方案，一个简单的安装包里可以提供多样性的功能。下面是它的一些功能特点：

+ **数据收集**

  + 可用性和性能检查
  + 支持 SNMP ，IPMI（[智能平台管理界面](https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface)），JMX（[Java 管理扩展](https://en.wikipedia.org/wiki/Java_Management_Extensions)），[VMware](https://en.wikipedia.org/wiki/VMware) 监控
  + 自定义检查
  + 按照自定义的间隔收集需要的数据

+ **灵活的阀值定义**

可以灵活的定义问题阈值，称之为触发器，触发器从后端数据库可以获取参考值。

+ **高度可配置化的警告**

可以根据升级计划，接收方和媒体类型来定制发送警告通知。

+ **实时图表绘制**

使用内置图表绘制功能可以将监控项的内容立即绘制成图表。

+ **Web 监控功能**

Zabbix 可以模拟鼠标在 Web 网站上的操作来检查 Web 的功能和响应时间

+ **多样的可视化选项**

  + 能够创建自定义的图形，图形中可将多个监控项组合在一个视图展示
  + 网络拓扑图
  + 以仪表盘的样式展现自定义的展现和幻灯片
  + 报告

+ **配置简单**

  + 添加受监控的设备作为主机

+ **Zabbix API**

Zabbix API 为 Zabbix 提供了对外的可编程接口，用于批量操作，第三方软件集成和其他目的

+ **权限管理系统**

  + 安全用户认证
  + 特定用户可以限制访问特定的视图

+ **功能强大并易于扩展的监控代理**

  + 部署在被监控对象上
  + 可以部署在 Linux 和 Windows 上

+ **二进制代码**

为了性能和更少内存的占用，采用 C 语言编写，便于移植

+ **为复杂环境准备**

使用 Zabbix proxy 代理服务器，使得远程监控更简单

*（此段功能特点源自 zabbix 官方文档手册）*

## 3 Zabbix 的平台的搭建

### 3.1 安装

安装部署 `zabbix` 有四种方法：

+ **从部署包进行安装**
+ 下载最新的源代码安装，并自行编译
+ 从容器安装
+ 下载虚拟应用

> 这里我们主要从部署包来进行安装。

1.**安装 zabbix**

其中部署包包含了配置文件。

因为我们的环境是 Ubuntu 14.04 ，如果你的环境是 Ubuntu 16.04 版本，则把下面的命令中的 `trusty` 替换成 `xenial`。

```bash
$ wget http://repo.zabbix.com/zabbix/3.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.4-1+trusty_all.deb
$ sudo dpkg -i zabbix-release_3.4-1+trusty_all.deb
$ sudo apt-get update # 需要更新源，才能安装最新的版本。版本差异会导致一些较大的不同表现
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514530735040.png/wm)

若你的环境是 centos 则按照下面命令进行安装。

```bash
yum update
rpm -ivh http://repo.zabbix.com/zabbix/3.4/rhel/7/x86_64/zabbix-release-3.4-1.el7.noarch.rpm
```

2.**安装 Zabbix 其他部署包**

安装支持 MySQL 的 Zabbix 服务器

>注意：下列两个 mysql 的包不能在一条命令中同时安装，因为他们直接的依赖会有冲突

```bash
$ sudo apt-get install -y zabbix-server-mysql
```

安装支持 MySQL 的 Zabbix 代理

```bash
$ sudo apt-get install -y zabbix-proxy-mysql
```

安装支持的 Zabbix 前端

```bash
$ sudo apt-get install -y zabbix-frontend-php
```

安装 Zabbix 代理

> 从上文中得知 agent 一般安装于被监控端，类似与 nagios 的 nrpe，此处我们本地即是 server 也是被监控端，所以安装

```bash
$ sudo apt-get install -y zabbix-agent
```

```checker
- name: check pkg
  script: |
    #!/bin/bash
      dpkg -l zabbix-server-mysql
  error: 没有安装 zabbix-server-mysql
- name: check pkg
  script: |
    #!/bin/bash
      dpkg -l zabbix-proxy-mysql
  error: 没有安装 zabbix-proxy-mysql
- name: check pkg
  script: |
    #!/bin/bash
      dpkg -l zabbix-frontend-php
  error: 没有安装 zabbix-frontend-php
- name: check pkg
  script: |
    #!/bin/bash
      dpkg -l zabbix-frontend-php
  error: 没有安装 zabbix-frontend-php
```

3.**安装初始化数据库 Mysql**

在 MySQL 上创建 Zabbix 初始化数据库和用户。

需要先创建 MySQL 数据库：

```bash
$ sudo service mysql start # 启动 MySQL 服务
$ mysql -uroot
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep mysql
  error: 没有启动 mysql
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531032245.png/wm)

```mysql
# 创建一个名叫 zabbix 的数据库，字符集设置为 utf8 ，将字符串中每一个字符用二进制数据存储，区分大小写。
mysql> create database zabbix character set utf8 collate utf8_bin;

# 这是一个授权的操作，意为允许用户 zabbix 从 localhost 的主机连接到 mysql 服务器的 zabbix 数据库，并使用 <password> 作为密码 
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by '<password>'; # <password> 记得修改，这里我的密码是 `zabbix`。

mysql> quit;
```

```checker
- name: check sql
  script: |
    #!/bin/bash
	mysql -u root shiyanlou001 -e "show databases"|grep "zabbix"
  error: 没有创建 zabbix 数据库
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531074814.png/wm)

然后导入初始架构（Schema）和数据。在此步骤之前，zabbix 数据库的 table 表中是没有任何东西的。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514534237690.png/wm)

但是使用了如下命令导入了初始架构和数据之后，我们可以看到 zabbix 数据库的 table 表中已经有了导入的数据。

```bash
# 使用 zcat 初始化查看其中数据，通过 zabbix 用户初始化名为 zabbix 的数据库
$ zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz | mysql -uzabbix -pzabbix zabbix
```

```bash
$ mysql -uzabbix -pzabbix # 登录 zabbix 数据库
mysql> use zabbix;
mysql> show tables; # 可以看到 zabbix 的表中已经导入了数据。
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531116047.png/wm)

4.**为 Zabbix server 配置数据库**

在 zabbix_server.conf 中编辑数据库配置

```
sudo vim /etc/zabbix/zabbix_server.conf
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
```

```checker
- name: check content
  script: |
    #!/bin/bash
	grep DBName /etc/zabbix/zabbix_server.conf | grep zabbix
  error: /etc/zabbix/zabbix_server.conf 内容不对
```

注：DBPassword 中使用的是 MySQL 的 Zabbix 数据库密码。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531163471.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531155001.png/wm)

5.**启动 Zabbix Server 进程**

```
sudo service zabbix-server start
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep zabbix-server
  error: 没有启动 zabbix-server
```

启动 zabbix-agent 代理

```
sudo service zabbix-agent start
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep zabbix-agent
  error: 没有启动 zabbix-agent
```

6.**Zabbix 前端配置**

Zabbix 前端的 Apache 配置文件位于 `/etc/apache2/conf-enabled/zabbix.conf` 或 `/etc/apache2/conf.d/zabbix.conf` 中。

需要先启动一下 Apache2

```
$ sudo service apache2 start
$ sudo vim /etc/apache2/conf-enabled/zabbix.conf
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep apache2
  error: 没有启动 apache2
```

配置文件如下：

```
php_value max_execution_time 300
php_value memory_limit 128M
php_value post_max_size 16M
php_value upload_max_filesize 2M
php_value max_input_time 300
php_value always_populate_raw_post_data -1
# php_value date.timezone Europe/Riga
```

注：PHP 的一些设置已经配置好了，但需要取消 `date.timezone` 的注释，并设置所在地正确的时区。时区设置可参考这个链接 [List of Supported Timezones](http://php.net/manual/en/timezones.php)。

这里我们设置为 `Asia/Shanghai`。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531103035.png/wm)

7.**SELinux 配置**

**在本实验环境中无需配置 SELinux ，可跳过此步**。但是如果你是在本地 centos 中搭建，需要在强制模式下启用 SELinux 状态后，通过下面的命令来启用 Zabbix 前端和服务器之间的通信。

RHEL 7 及更高版本：

```
setsebool -P httpd_can_connect_zabbix on

# 如果数据库可以通过网络访问，还需要允许 Zabbix 前端连接到数据库：
setsebool -P httpd_can_network_connect_db on
```

RHEL 7 之前：

```
setsebool -P httpd_can_network_connect on
setsebool -P zabbix_can_network on
```

8.**重启 Apache Web 服务器**

前端配置完成后，需要重新启动 Apache Web 服务器：

```
$ sudo service apache2 restart
```

9.**访问前端浏览器**

Zabbix 前端可以在浏览器中通过 `http://localhost/zabbix` 进行访问。

在浏览页面中大家按照提示框中一步一步进行 zabbix 初始安装。步骤如下：

(1) 左边红色框中是前端登录界面的步骤，选择右下角的 `Next step` 按钮：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531432699.png/wm)

(2) 然后是一些先决条件，可以看到我们之前配置的 `date.timezone` 显示的是 `Asia/Shanghai`。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531441256.png/wm)

(3) 接着是连接 DB 数据库，需要输入之前设定的数据库密码。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531456175.png/wm)

(4) 之后是一些服务细节的确认（默认设置即可），看到这个界面就表示成功了，点击：`finish`。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531461570.png/wm)

(5) 下面这个页面就是 `zabbix` 的登录页面，默认登录名是 `Admin` 和密码是 `zabbix` 

> 可以看到 Sign in 下面还有一个选项是 `sign in as guest`（以游客身份登录，初次登录最好选择管理员身份默认登录，方便后面涉及到权限的操作）

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531468800.png/wm)

(6) 登录成功后就进入到 `zabbix` 的监控界面。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531473831.png/wm)

(7) 监控界面其实是一个仪表盘（Dashboard），上面有各种监控项，我们找到 `Status of Zabbix` 这一栏，可以看到 zabbix 的状态是正在运行（`yes`）。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514534052541.png/wm)

如果你搭建的 zabbix 状态显示的是 `no`，很有可能是因为 SELinux 没有关闭造成的。

至此，zabbix 的搭建就完成了，在下一节中将继续给大家讲解如何去使用 zabbix 。此环境大家记得保存下来，避免以后反复安装和配置。

Zabbix 服务安装部署操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week6/7-1.mp4
@`

## 5 总结

通过本节实验的学习可以认识到 zabbix 监控工具的原理以及基本架构，也学习了如何搭建一个 zabbix。在后续的学习中将继续为大家讲解 zabbix 的一些详细用法以及如何用它来实现实现系统监控的。
