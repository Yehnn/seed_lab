---
show: step
version: 1.0
enable_checker: true
---
# Saltstack 安装与配置

## 1. 实验介绍

#### 1.1 实验内容

本节实验节点的是一种全新的运维管理系统——Saltstack ，它非常的简单，能够在几分钟内运行起来，足以用速度来取代复杂性，并且可以扩展来管理数以万计的服务器，同时在几秒内就能和每个系统进行通信。

#### 1.2 实验知识点

+ Saltstack 简介

+ Saltstack 安装

+ Saltstack 配置

#### 1.3 推荐阅读

+ [SaltStack 官方文档](https://docs.saltstack.com/en/latest/contents.html)

+ [SaltStack 中文文档](http://docs.saltstack.cn/)

+ [中国 SaltStack 用户组](http://www.saltstack.cn/kb/cssug-conf-2016-1-beijing/)

## 2. 简介

#### 2.1 Saltstack 概述

Saltstack 是一种全新的基础设施管理方式，可以作为一个配置管理系统，用来维护预定义状态的远程节点，也可以作为一个分布式远程执行系统，用来在远程节点上执行命令和查询数据等。

Saltstack 主要还是采用的是 `server/client` 模式，需求的功能内建在一组 daemon 中，使用的是轻量级的通信器（[ZeroMQ](http://zeromq.org/intro:start)），采用 python 来编写，完全开源，和 `Puppet` 、[Chef](https://learn.chef.io/modules/learn-the-basics#/) 功能类似。同时，Saltstack 提供了一个可插式的架构，使得系统具有较好的扩展性。

> `ZMQ(ZeroMQ)`：是一个消息处理队列库，可在多个线程、内核和主机盒之间弹性伸缩。也是一个简单好用的传输层，像框架一样的一个 socket library，使得Socket 编程更加简单、简洁和性能更高。

> `Chef`：是一个用 Ruby 和 Erlang 编写的配置管理工具。可以简化配置和维护公司服务器的任务，并且可以与众多云平台进行集成，以自动配置和配置新机器。

#### 2.2 基本组件

+ `Master`：是 Salt 的主服务器

+ `Minions`：是提供的目标机器

+ `Keys`：密钥用于安全通信

+ `States`：是 Salt 的核心，定义了一种状态，states 模块可以用来查看系统是否已经处于正确的状态中。同时 states 可以管理各种各样的东西，包括但不限于文件、MySQL 数据库、PostgreSQL 数据库、系统包、系统服务、ssh 密钥、用户、python 虚拟环境、rvm 环境、pip 包、时区、邮件别名和 heck 等。

+ `Grains`：是用来获取有关系统的数据。Grains 是关于底层操作系统，内存，磁盘和许多其他系统属性的静态信息。当 minion 启动并定期刷新或使用远程执行命令时，会自动收集 Grains。

+ `Pillar`：用于将数据传送到你的系统。在配置一个简单的系统时需要考虑不同的自定义数据。通常这些值对于每个系统或系统角色是不同的。定义完这些数据值，然后将它们分配给一个或多个的目标 minions。最后使用变量将值插入到 Salt states 中。其中 pillar 会作为 Jinja2 模板存储在 master 中，当它的信息需要时，会被传递给 minions，呈现并解析为 YAML 格式。

下图是 `Saltstack` 的基本架构：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4131timestamp1516702041965.png/wm)
*（图片来自官方文档）*

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4147timestamp1516786305554.png/wm)

从这张各组件间的结构图可以知道，states 定义了一些适用的事情状态， Pillar 和 Grains 确定了 minion 会受到那种 states 以及 states 本身的参数化。如图所示，Pillar 模板首先会被 Jinjia2 进行渲染，然后和 Grains 传送给 states 中进行渲染，完成后会被解析为 YAML 格式来确定什么样的 states 适用有并运用于 minion 中。

#### 2.3 Saltstack 通信

Saltstack 采用的是一个 `server-agent` 通信模型，server 组件被称为 `Salt Master` ，而 agent 称为 `Salt Minions`。Master 负责发送命令给 Minions ，然后聚合并显示这些命令的结果，一个 Master 可以管理数千个系统。同时通过消息队列 `ZeroMQ` 在 master 端与 minion 端之间建立消息发布连接。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4147timestamp1516784063789.png/wm)

Saltstack 的通信方式采用的是 `publish-subscribe` 这种模式的管理。连接由 minions 发起，就不用这些系统打开任何入站端口（可以减少攻击向量），而 master 需要打开 `4505` 和 `4506` 端口来接受连接。如下图是通信模式：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516697770655.png-wm)
*（图片来自官方文档）*

**通信安全**

当 minion 第一次启动时，它会在网络中搜索名为 salt 的系统，发现后 minion 进行握手，然后将其公钥发送给 master。初始连接之后，Salt minion 的公钥存储在服务器上，并且必须通过 `Salt-key` 命令来接受 master。（其中 minion 在其密钥被接受之前不会运行任何命令）。在 minion 密钥被接受之后，master 返回它的公钥以及一个旋转的 AES 密钥，该密钥用于加密和解密由 master 发送的消息。`Salt master` 和 `Salt minion` 之间的所有进一步通信都使用 AES 密钥进行加密。

#### 2.4 Saltstack 插件

插件（`plug-ins`）对于 Saltstack 来说十分重要，学习插件较学习可插式架构来说就如同发展传播和做研究的类比。

Saltstack 的核心框架提供了高速通信和事件总线。而在这个核心框架之上，剩余是特性就被公开为一组松散耦合的可插式的子系统。

Saltstack 包含了超过 20 个的可插式的子系统，但常用的也就是少数感兴趣的。

下图是官方提供的常用子系统和每个子系统常用的插件。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516697284070.png-wm)

#### 2.5 Saltstack 特点

+ **实时通信**：所有的 Salt minions 会同时接受命令，可以在几秒内完成对数千系统的查询，获取信息的方式是实时查询，而不是依赖数据库。

+ **没有 Freeloader**：Salt minions 会自己做自己的工作，收到 Salt master 的通信是一组轻量级的指令集。也就是说，具有这些属性的 minions 就会用这些参数来运行这个命令。而每个 minions 已经拥有它需要存储在本地的所有命令，因此执行命令后很快就能返回给 master。

+ **高性能**：通过使用 ZeroMQ 的通信系统在主从控制之间建立一个持续的数据通道，并且使用多线程和并发的方法来进行调优。

+ **可扩展性**：能够在一台或多台 minion 机器上快速执行命令。运行速度快，灵活，用相同的远程执行架构可以满足不同数量的服务器的需求。

+ **自动化**：通过基础架构可以进行自动化初始系统配置，同时还可以碎石自动进行扩展、修复和执行管理等。

+ **容易管理**：salt 可以在几乎所有 python 能运行的地方运行，对于唯一需要由 Salt 管理的需求就是支持的网络协议（因为它支持任何网络协议，即使是自定义的协议也支持）。

+ **标准化**：规范化是 Salt 跨平台管理功能的关键。无论是针对 Linux，Windows，MacOS，FreeBSD，Solaris 还是 AIX，在物理硬件上或在云中，或者是容器，Salt 的命令和状态都是相同的。

## 3. Saltstack 安装

SaltStack 已经被设计的十分容易安装和启动，官方的安装文档也包含了所有支持平台的安装方法。

安装的方法有多种，可以选择从软件包来进行安装，或者通过 `pip` 直接从源来安装 SaltStack ，也可以通过使用引导脚本（[BOOTSTRAP](https://docs.saltstack.com/en/latest/topics/tutorials/salt_bootstrap.html)），以及用 SaltStack 提供的专用工具来创建计算机并在云上（`salt-cloud` 或 `salt-virt`）安装 Salt 等等。并且 Saltstack 也支持在不同平台上进行安装，包括：`Fedora`、`macOS`、`Solaris`、`RHEL`、`CentOS`、`Ubuntu` 以及 `Windows` 等等。

在这里我们主要介绍在 `Ubuntu14（Trusty）` 上使用软件包来安装 SaltStack。

通过下面命令可以查看环境

```bash
$ lsb_release -a
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516779189866.png-wm)

运行下面的命令导入 SaltStack 的存储库密钥

```bash
$ wget -O - https://repo.saltstack.com/apt/ubuntu/14.04/amd64/latest/SALTSTACK-GPG-KEY.pub | sudo apt-key add -
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516779215259.png-wm)

将以下源信息内容保存到 `/etc/apt/sources.list.d/saltstack.list` 文件中，以便获取该源上的包信息：

```bash
# 默认没有这个文件，这是我们单独新建的一个
$ sudo vim /etc/apt/sources.list.d/saltstack.list

# 将下面内容写到文件中保存
deb http://repo.saltstack.com/apt/ubuntu/14.04/amd64/latest trusty main
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516779239573.png-wm)

> 补充：`source.list` 文件格式：
> `deb  [地址] [版本代号] [限定词]`

更新源，然后分别安装 saltstack 的组件，这里我们只安装了 master 和 minion ，其他组件需要用时再进行安装即可。（因为实验楼环境的原因，我们选择将 master 端和 minion 端安在同一台机器上，在本地大家可以选择分开安装都是一样的，不过需要分别对上面的源进行更新后再进行安装组件就可以了）

```bash
$ sudo apt-get update
$ sudo apt-get install salt-master
$ sudo apt-get install salt-minion
```

安装完成后，启动服务即可。

```bash
$ sudo service salt-master start
$ sudo service salt-minion start
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep salt
  error: 没有启动 salt
- name: check port1
  script: |
    #!/bin/bash
	netstat -luntp|grep 4505
  error: 4505 端口没有监听
- name: check port1
  script: |
    #!/bin/bash
	netstat -luntp|grep 4506
  error: 4506 端口没有监听
```

可以通过 ps 命令来查看进程是否跑起来了

```bash
$ ps -ef | grep salt
```

通过 netstat 来查看网络状态，master 启动后默认会监听 `4505` 和 `4506` 这两个端口，4505(`publish_port`)为 salt 消息发布系统的端口，4506(`ret_port`)为 salt minion 端与 master 端通信的端口。

```bash
$ sudo netstat -nltp
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516779321894.png-wm)

Saltstack 安装操作视频：


`@
http://labfile.oss.aliyuncs.com/courses/980/week10/4-1.mp4
@`

## 4. Saltstack 配置

Saltstack 的配置非常简单，主服务器（master）的默认配置适用于大多数的安装，而配置一个 minion 的唯一需求只是在 minion 的配置文件中设置 master 的位置即可。

配置文件是在 `/etc/salt` 下的的相应组件 `/etc/salt/master` 和 `/etc/salt/minion` 中。

我们先来配置 master 的相关配置项，如下：

```bash
$ sudo vim /etc/salt/master

# 修改下面几项配置

# 将默认的所有网络接口（0.0.0.0）都可访问修改为本地接口
- # interface: 0.0.0.0
+ interface: 127.0.0.1

# 设置默认 root 用户运行 salt 进程
- # user: root
+ user: root

# 启用这个设置将自动接受所有来自 minion 的公共密钥
- # auto_accept: False
+ auto_accept: False
```

> 这里只配置我们需要的配置项，其他配置项大家可以从配置文件中了解或者查看官方文档中 [master 配置项](https://docs.saltstack.com/en/latest/ref/configuration/master.html#configuration-salt-master)的说明。

然后，配置 minion 的配置项

```bash
$ sudo vim /etc/salt/minion

# 修改如下配置

- # master: salt
+ master: 127.0.0.1
```

> 这里只配置我们需要的配置项，其他配置项大家可以从配置文件中了解或者查看官方文档中 [minions 配置项](https://docs.saltstack.com/en/latest/ref/configuration/minion.html#configuration-salt-minion)的说明。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516779359579.png-wm)

配置完成后我们需要重启一下服务：

```bash
$ sudo service salt-master restart
$ sudo service salt-minion restart
```

```checker
- name: check content
  script: |
    #!/bin/bash
	cat /etc/salt/master | grep -e 'interface' -e 'user' -e 'auto_accept'
  error: /etc/salt/master 没有配置
- name: check content
  script: |
    #!/bin/bash
	cat /etc/salt/minion | grep 'master'
  error: /etc/salt/minion 没有配置
```

Saltstack 配置操作视频：


`@
http://labfile.oss.aliyuncs.com/courses/980/week10/4-2.mp4
@`

## 5. 测试

完成上面操作后，我们通过 salt 的相关语句来测试一下 master 端和 minion 端是否成功连接并且能够通信。

我们先使用 `salt-key` 命令来简单管理认证的 salt 服务器公钥。

Salt minion 的 keys 可以处于如下几种状态：

+ `unaccepted`：等待被接受的 key

+ `accepted`：key 已经被接受，并且 minion 能够被连接到 master 上

+ `rejected`：key 被拒绝使用 salt-key 命令，这种状态下 minion 拒绝任何来自 master 的通信

+ `denied`：key 被 master 自动拒绝

这里我们简单使用几个参数来测试，详细参数说明可以参考 [salt-key](https://docs.saltstack.com/en/latest/ref/cli/salt-key.html) 命令说明。

常用参数：

+ -L：列出所有公钥

+ -A：接受所有挂起的密钥，即接收 Unaccepted Keys 状态下所有的 minion

+ -a：指定接收某台 minion 的 key

+ -D：删除所有密钥

```bash
$ sudo salt-key -L
$ sudo salt-key -A
$ sudo salt-key -L
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516779420506.png-wm)

验证 master 和 minion 的通信（在 master 端进行）：

```bash
$ sudo salt '*' test.ping
```

```checker
- name: check ping
  script: |
    #!/bin/bash
	salt '*' test.ping|grep True
  error: master 和 minion 通信不成功
```

这里我们使用了 salt 的基本语法来进行测试，在后面一节我们将会详细讲解它的语法内容。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516779432708.png-wm)

返回的是 `True` 说明通信成功。

补充：在通过密钥与测试成功中，我们可以看到由一串数字代表的主机，这是因为当前主机的主机名是这一串数字，所以都显示的是一串数字。

通过下面命令可以查看主机名与 IP 的对应关系：

```bash
$ cat /etc/hosts
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516779402760.png-wm)

当然我们也可以直接通过 `hostname` 命令来查看当前机器的主机名。

验证发现 master 端和 minion 端是可以正常通信，至此 saltstack 的安装以及配置完成，后面将继续讲解 salt 的使用。

Saltstack 测试操作视频：


`@
http://labfile.oss.aliyuncs.com/courses/980/week10/4-3.mp4
@`

## 6. 总结

本节主要讲解了 Saltstack 的基本原理以及如何安装和配置的详细操作，在后面我们将继续对 salt 进行深入学习。