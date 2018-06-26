---
show: step
version: 1.0
enable_checker: true
---
# Supervisor 安装与配置

## 1. 实验介绍

#### 1.1 实验内容

本节实验和大家一起来学习另一个运维管理方式——Supervisor。Supervisor 是一个 `client/server` 模式的系统，他允许用户在类 Unix 的操作系统上控制多个进程。

#### 1.2 实验知识点

+ Supervisor 简介

+ Supervisor 安装

+ Supervisor 运行

+ Supervisor 配置

#### 1.3 推荐阅读

+ [Supervisor 官方文档](http://supervisord.org/index.html)

+ [Fork–exec](https://en.wikipedia.org/wiki/Fork%E2%80%93exec)

## 2. Supervisor 简介

#### 2.1 概述

Supervisor 是一个 `client/server` 模式的系统，他允许用户在类 Unix 的操作系统上控制多个进程。在许多 `VPS(Virtual Private Servers)` 环境中，我们想要运行一些小程序、shell 脚本，应用程序或者是一些大型的软件包，通常会为这些程序编写一个初始化脚本，但是这就可能会耗费很多的管理时间。而 Supervisor 作为一个进程管理工具可以通过提供一个一致的接口来管理这些长时间运行的程序，同时对它们进行监听和控制。

#### 2.2 特点

+ 简单

Supervisor 通过 [INI-style](https://en.wikipedia.org/wiki/INI_file) 来进行配置，这种方式比较简单，易于学习。提供了许多进程选项，可以让你像重启失败进程那样很容易的进行操作。

+ 集中管理

Supervisor 提供了一个启动、停止和监控进程的地方，其中进程可以单独或分组进行控制，也可以通过配置 Supervisor 来提供一个本地或远程的命令，以及 web 接口。

+ 高效率

Supervisor 通过 [fork/exec](http://blog.csdn.net/zhoubangtao/article/details/53888792) 来启动子进程，而子进程不用守护，一旦进程终止，操作系统就会向 Supervisor 发送信号。不用像一些麻烦的 PID 文件和定期轮询来重启失败进程的解决方案。

+ 可扩展

Supervisor 有一个简单的事件通知协议，任何语言编写的程序都可以用来监控它。并且还有一个用于控制的 [XML-RPC](http://supervisord.org/api.html) 接口。

+ 兼容性（多平台）

除了 Windows 以外，Supervisor 支持在 `Linux`、`Mac OS X`、`Solaris` 和 `FreeBSD` 上进行测试和运行。

#### 2.3 组件

+ Supervisord

这是 `Supervisor server` 端的进程，负责在服务器端启动子程序，同时响应客户端的命令，重启崩掉或退出的子程序，记录子进程的标准输出和标准错误输出（stdout、stderr），并且生成和处理与子进程生命周期对应的事件。

Supervisord 进程的配置文件默认存放在 `/etc/supervisord.conf` 中，同样也是用 `INI-style` 格式来写的，该配置文件最主要的一点就是通过文件系统权限来保证其安全性，因为这其中可能包含了未加密的用户名和密码。

+ Supervisorctl

这是 `Supervisor client` 端的进程，为 Supervisord 的功能提供一个类似 shell 的界面。从 Supervisorctl ，用户可以连接到不同的 Supervisord 进程，通过 Supervisord 的控制在子进程中获得状态，停止或启动子进程，以及获得运行进程的列表。

Supervisorctl 不仅可以通过 UNIX socket 连接到本机的 Supervisord 上，还可以通过 TCP socket 连接到远程的 Supervisord。

Supervisorctl 进程通常使用和 server 端是相同的配置文件，其中包含有 `[supervisorctl]` 部分的任何配置文件都可以使用。

+ Web Server

是一个和 Supervisorctl 相媲美的 web 用户界面，可以通过浏览器进行访问。如果你开启 Supervisord 监听网络套接字，激活配置文件中的 `[inet_http_server]` 部分，在访问服务器 URL 就可以通过 web 界面来查看和控制进程的状态了。

+ XML-RPC Interface

服务于 web 界面的 HTTP 服务器提供的一个 XML-RPC 界面，可以用于询问和控制 Supervisor 和它运行的程序。

## 3. Supervisor 的安装

在前面我们知道了 Supervisor 支持多种平台的运行，因此在不同系统上安装也会有所不同，这里我们根据官方文档上提供的几种方法做讲解说明。

第一种是系统能够访问到 Internet 时，可以通过如下方法进行安装。这也是官方推荐的一种安装方法。

通过使用 `setuptools` 的一个功能：`easy_install` 来进行安装。

> 说明：`setuptools` 是对 `Python distutils` 的改进，可以更轻松地构建和分发 Python 包（尤其是依赖关系在其他包的那种）。用户不需要安装甚至不需要了解 `setuptools` 就可以使用它们，而且不必将整个 `setuptools` 软件包包含在发行版中。通过只包含一个引导程序模块（一个 8K 的 `.py` 文件），如果用户从源代码构建你的软件包并没有安装合适的版本，那么你的软件包将自动下载并安装 `setuptools`。
>
> `Easy Install` ：是一个与 `setuptools` 捆绑在一起的 `Python` 模块（`easy_install`），可以自动下载，构建，安装和管理 Python 包。该工具支持通过 `HTTP`，`FTP`，`Subversion` 和 `SourceForge` 进行下载，并自动扫描从 [PyPI](https://pypi.python.org/pypi/supervisor) 链接的网页以查找下载链接。（这是目前可用于 Python 最接近的 CPAN ）

```bash
# 由于实验环境的原因，这种方法大家可以在自己的本地进行尝试

1. 若系统中没有安装 setuptool，可以选择安装，安装方法参考 PEAK install setuptools 官网说明。

2. 若不想去安装 setuptools 可以通过下载 Supervisor 发行版并进行手动安装，Supervisor 的 releases 可以从 PyPI 上下载。解压存档后，运行 `python setup.py install` 命令。

3.前面的准备好之后，通过 `easy_install` 工具来安装 Supervisor 即可。

easy_install supervisor

注：根据系统的 python 的权限，需要使用 root 用户的权限才能执行。
```

> 补充：[PEAK install setuptools](http://peak.telecommunity.com/DevCenter/setuptools#installing-setuptools) 官网以及 [PyPI](https://pypi.python.org/pypi/supervisor) 官网。

第二种方法是通过下载 Supervisor 的软件包来进行安装

Linux 提供了通过系统软件包管理器来安装 Supervisor。这些软件包由第三方制定，不由 Supervisor 开发人员完成。其中他们的一个特点就是，可以集成到分发的服务管理基础设备中，例如，允许 Supervisord 在系统引导时自动启动。

补充：这种安装方法和官方推荐的不同在于，分发包的版本往往会落后于 Pypi 上发布的 Supervisor 软件包。

基于本实验环境是 Ubuntu 的环境，预编译的软件包已经存在于发行版的仓库中，所以只用执行如下命令：

```bash
$ sudo apt-get install supervisor
```

```checker
- name: check pkg
  script: |
    #!/bin/bash
      dpkg -l supervisor
  error: 没有安装 supervisor
```

可以发现在实验楼的环境中已经安装并且运行了 supervisor。

```bash
$ sudo service supervisor status
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517470153876.png-wm)

不过我们可以通过包管理工具来检查他们的可用性，例如，在 Ubuntu 上可以运行 `apt-cache show supervisor`，在 CentOS 上你可以运行 `yum info supervisor`。

```bash
$ sudo apt-cache show supervisor
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517470246288.png-wm)

除了通过软件包管理器来安装，还可以使用 pip 工具来安装，因为 Supervisor 主要是由 python 来编写的。

```bash
pip install supervisor
```

Supervisor 安装操作视频：


`@
http://labfile.oss.aliyuncs.com/courses/980/week10/7-1.mp4
@`

## 4. Supervisor 的配置

下面我们将会学习 Supervisor 的配置。

### 4.1 配置文件说明

完成了 Supervisor 的安装，我们可以运行 `echo_supervisord_conf` 命令来打印出一个示例的 Supervisor 配置文件，如下所示：

```bash
$ echo_supervisord_conf
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2767331517471131358-wm)

这个配置文件包括了所有需要配置的部分的参数示例，大家可以把它当作一个参考文件来学习，但是我们平时的配置不用包含所有的部分，所以我们这里可以简化一点，拷贝环境中已经启动的那个 Supervisor 的配置文件来学习如何配置。Supervisor 的配置文件的名称为 `Supervisord.conf` ，前面也提到说 Supervisord 和 Supervisorctl 是共同使用配置文件的。配置文件通常位于 `/etc/supervisor/` 目录下。

```bash
$ ll /etc/supervisor/
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517472936070.png-wm)

我们将 `supervisord.conf` 这个文件复制到当前目录下进行操作。

```bash
$ sudo cp /etc/supervisor/supervisord.conf ./
```

```checker
- name: check file
  script: |
    #!/bin/bash
	ls /home/shiyanlou/supervisord.conf
  error: /home/shiyanlou 目录下没有 supervisord.conf 文件
```

然后，我们就来配置 supervisor 以及相关参数说明。

```bash
$ sudo vim supervisord.conf
```

下面我们讲解几个比较常用的配置块，如下：

这部分主要用于监听 UNIX 域套接字的 HTTP 服务器的配置，如果不配置这部分，那么就不会启动 UNIX 域套接字的 HTTP 服务器。也就是 Supervisorctl 不能通过 UNIX 域套接字连接到本机的 Supervisord 上。参数说明如下：

```bash
[unix_http_server]
file=/var/run/supervisor.sock   # ; socket 文件的路径
chmod=0700                      # ; sockef 文件的权限(默认为 0700)
```

这部分主要用于配置文件在插入和管理过程中的全局配置。参数说明如下

```bash
[supervisord]
logfile=/var/log/supervisor/supervisord.log # ; 主要的 log 文件路径，默认为 $CWD/supervisord.log
pidfile=/var/run/supervisord.pid            # ; supervisord 的 pidfile，默认为 $CWD/supervisord.pid
childlogdir=/var/log/supervisor             # ; 用于 'AUTO' 子日志文件目录，默认项
logfile_maxbytes=10MB       # ; 活动日志文件在旋转之前可能消耗的最大字节数，默认 50MB
logfile_backups=5           # ; 活动日志文件循环产生的备份数量。如果设置为 0，则不会保留备份。默认为 10
loglevel=info               # ; 日志级别，指定写入 supervisord 活动日志的内容。默认是 info，其他: debug，warn，trace
nodaemon=false              # ; 如果是 true，supervisord 将在前台开始，而不是守护进程。默认 false
minfds=1024                 # ; 在 supervisord 程序开始之前可用的文件描述符的最小数目。默认 1024
minprocs=200                # ; 在 supervisord 程序开始之前可用的进程描述符的最小数目。默认 200
```

这部分配置主要是用于 supervisorctl 交互式的 shell 程序设置。

```bash
[supervisorctl]
serverurl=unix:///var/run//supervisor.sock  # ; 用来访问 supervisord 服务器的 URL ，对于 UNIX 套接字使用： unix://URL
```

这部分主要是把我们配置的信息写到多个文件中，然后 include 进来，其中递归文件是不支持的，这里有用到了一个 glob 文件模式来使用特定规则进行匹配。

```bash
[include]
files = /etc/supervisor/conf.d/supervisor_service.conf
```

还有一个部分在这个配置文件中没有列出，是 `[inet_http_server]` 部分，主要插入一个在 `TCP(internet)` 套接字上监听的 HTTP 服务器配置。如果没有这部分将不会启动 inet HTTP 服务器。配置和 `unix_http_server` 相似，如下：

```bash
[inet_http_server]
file = /tmp/supervisor.sock # 套接字路径
chmod = 0777            # 权限值
chown= nobody:nogroup   # 更改套接字文件的用户和组
username = user         # 对 HTTP 服务器进行身份验证所需的用户名
password = 123          # 验证 HTTP 服务器所需的密码
```

其他的配置部分不是常用这里就不做多讲，大家可以去参考官方说明或者之前打印出来的配置示例的说明。

### 4.2 修改配置

大致了解了 supervisor 的相关配置后，我们就可以在环境中已有配置文件基础上进行简单的修改。因为环境中已经启动了 supervisor 程序，所以为了避免冲突，我们只需要进行如下修改：

```bash
1. 将 [unix_http_server] 部分中的 supervisor.sock 修改为其他名称，如：

file=/var/run/supervisor1.sock

2. 对 [supervisord] 部分也做相似操作，如：

logfile=/var/log/supervisor/supervisord1.log
pidfile=/var/run/supervisord1.pid

3. 对 [supervisorctl] 部分也做相似操作，如：

serverurl=unix:///var/run//supervisor1.sock

4. 对于 [include] 部分，因为环境中已经启动了，再次执行时会有问题，所以这里我们最好把这部分注释掉，在语句前面添上分号（;）即可。但是你在本地若要配置 ssh 服务时还是需要进行配置说明。
```

```checker
- name: check content
  script: |
    #!/bin/bash
	grep supervisord1 /home/shiyanlou/supervisord.conf
  error: /home/shiyanlou/supervisord.conf file 内容不对
```

修改配置完成后保存退出。

到此配置其实还没有完成，因为我们是在实验环境已有的基础进行学习的，所以如果我们是在本地配置时就还有一个重要的部分没有涉及到。大家通过 `cat /etc/supervisor/conf.d/supervisor_service.conf` 这个命令就可以在终端打印出这个重要的部分了——`[program:x] 配置部分`。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517481676643.png-wm)

在配置文件中必须包含一个或更多的 `program` 项来让 supervisord 知道应该启动或者控制哪些程序。标题 `program` 的冒号后面是程序的名称，该名称用来控制由此配置而创建的进程。`program` 部分的参数较多，大家可以去官方文档中查看，这里我们简单讲解几个常用的参数，下面是环境中已经配置好的一个 program 的文件，如下：

```bash
[program:ssh]
command=/usr/sbin/sshd -D  # 启动这个程序时会执行的命令，命令可以是绝对路径或者相对的。程序可以接受参数，也可以用双引号来把带空格的参数分组传递给程序等等
stdout_events_enabled=true # 如果为 true，则会在进程写入 stdout 文件描述符时发出 PROCESS_LOG_STDOUT 事件。其中只有在接收数据时文件描述符不处于捕获模式时才会发出事件。
stderr_events_enabled=true # 如果为 true，则在进程写入 stderr 文件描述符时将发出 PROCESS_LOG_STDERR 事件。只有在接收数据时文件描述符不处于捕获模式时才会发出事件。
autorestart=true  # 指定如果 supervisord 程序在运行状态下退出，是否应该自动重启进程。
```

program 配置部分我们是单独保存在 `conf.d` 路径下的，为了能够读取到这部分的配置我们就需要在 `supervisor.conf` 配置文件中进行说明，也就是我们之前讲到的 include 部分的配置的。因此，当我们配置了一个 program 文件之后需要在 `supervisor.conf` 配置文件中的 `include` 部分进行说明，这样才能在后面执行时进行调用。

### 4.3 执行命令

当我们配置好了以上几个部分之后，supervisor 的配置部分就基本完成，这时我们就可以来运行 `supervisord` 和 `supervisorctl` 的命令了。

#### 4.3.1 运行 supervisord

启动 supervisord 需要运行 `$BINDIR/supervisord`，通过参数传递启动 supervisord 的可执行文件。当 supervisord 启动时会在默认位置或当前工作目录中搜素配置文件，而参数的不同也会有不同的执行效果。这里我们列出几个常用的参数进行说明，如下：

| 参数      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| `-c file` | supervisord 配置文件的路径                                   |
| `-n`      | 在前台运行 supervisord                                       |
| `-h`      | 帮助命令                                                     |
| `-u`      | UNIX 用户名或数字用户标识                                    |
| `-d PATH` | 当 supervisord 作为守护进程运行时，在守护进程之前 cd 到这个目录 |
| `-l file` | 用作 supervisord 活动日志的文件名路径                        |
| ...       | ...                                                          |

我们通常选择参数 `-c` 来指定 supervisord 命令的配置文件的绝对路径，这样可以确保安全性。

```bash
$ sudo supervisord -c supervisor.conf
```

#### 4.3.2 运行 supervisorctl

启动 supervisord 之后就可以去运行 supervisorctl，运行命令为 `$BINDIR/supervisorctl`。执行之后会显示一个 supervisor 的 shell 界面，允许你控制当前有 supervisord 管理的进程。可以通过 `help` 或者 `?` 来获取相关支持的命令。如下所示：

```bash
$ sudo supervisorctl
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517565772622.png-wm)

+ status：打印出状态

+ stop/start：停止/开始

+ exit：退出

因为在这里配置文件下我们没有定义什么服务，所以这里的状态就不会显示出什么来。

`supervisorctl` 命令也有许多参数来执行，如下说明：

| 参数     | 说明                                                    |
| -------- | ------------------------------------------------------- |
| `-c`     | 配置文件的路径，默认为 /etc/supervisor.conf             |
| `-i`     | 执行命令后启动交互式 shell                              |
| `-h`     | 打印使用情况并退出                                      |
| `-u`     | 用于与服务器进行验证的用户名                            |
| `-p`     | 用于与服务器进行身份验证的密码                          |
| `-r`     | 保持 readline 历史记录                                  |
| `-s URL` | supervisord 正在监听的 URL，默认：http://localhost:9001 |

比如我们可以通过参数 `-c` 来看看环境中默认启动的那个 supervisor 的服务。

```bash
$ sudo supervisorctl -c /etc/supervisor/supervisor.conf
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517565779558.png-wm)

可以看到原本环境中启动了 ssh 服务。

从上面的执行命令知道在参数 `-c` 的情况下会显式的指定执行的配置文件的路径，但是如果没有参数选项的时候，程序就会按照如下这个顺序来查找 `supervisor.conf` 配置文件：

+ 1. $CWD/supervisord.conf

+ 2. $CWD/etc/supervisord.conf

+ 3. /etc/supervisord.conf

> $CWD：当前目录

### 4.4 配置实例

通过前面的学习我们大致了解了 supervisor 的相关配置和如何运行，但是我们还没有自己动手编写一个服务来实践实践，所以这里我们重新编写一个配置实例来完成项目。

主要的就是编写一个 program 文件，然后在主配置文件中指定匹配该 program 文件即可。

下面我们先来编写一个 supervisor 管理 nginx 的例子，首先添加一个 nginx 的 program 到 `/etc/supervisor/conf.d/` 文件下。

```bash
$ sudo vim /etc/supervisor/conf.d/supervisor_nginx.conf

[program:nginx]
command=/usr/sbin/nginx
autostart=true                # 随着 supervisord 的启动而启动
autorestart=true              # 自动重启
startretries=3                # 启动失败时的最多重试次数
stopwaitsecs=10               # 发送 SIGKILL 前的等待时间
```

```checker
- name: check file
  script: |
    #!/bin/bash
	ls /etc/supervisor/conf.d/supervisor_nginx.conf
  error: /etc/supervisor/conf.d 目录下没有 supervisor_nginx.conf 文件
- name: check content
  script: |
    #!/bin/bash
	grep nginx /home/shiyanlou/supervisor.conf
  error: /home/shiyanlou/supervisor.conf 内容不对
```

然后，我们需要去配置文件中添加 `supervisor_nginx.conf` 这个 program 项。

```bash
$ sudo vim supervisor.conf
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517561509430.png-wm)

保存退出。

这里我们再补充一个 supervisorctl 的知识点——动作参数，说明如下：

| Action   | Describe                               |
| -------- | -------------------------------------- |
| `help`   | 打印出可用动作                         |
| `add`    | 激活进程/组的配置中的任何更新          |
| `remove` | 从激活的配置中删除进程/组              |
| `update` | 重新加载配置，然后根据需要添加和删除   |
| `clear`  | 清除进程的日志文件                     |
| `reread` | 重新加载守护进程的配置文件，不重新启动 |
| `status` | 获取所有进程的状态信息                 |
| `tail`   | 输出过程日志的最后一部分               |
| ...      | ...                                    |

这里我们配置完后，需要用到 `reread` 选项来重新加载一下配置文件。

```bash
$ sudo supervisorctl reread
$ sudo supervisorctl update
```

然后启动 supervisorctl 就可以进入到交互界面，也就可以对 nginx 进行控制管理了。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517561867135.png-wm)

我们在 `supervisor_nginx.conf` 文件中定义了一个自动重启的功能，我们这里可以来验证一下：

首先，另起一个终端，查看 nginx 是否已经启动了。

```bash
$ ps -ef | grep nginx
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep nginx
  error: 没有启动 nginx
```

已经启动，然后我们在 supervisor shell 交互界面也查看一下 nginx 的状态。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517564569732.png-wm)

接着我们在另起的那个终端将 nginx 进程 kill 掉，并查看 nginx 的状态。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517564578435.png-wm)

最后，回到 supervisor shell 交互界面再次查看状态，可以看到 PID 已经发生变化，说明自动重启了 nginx 服务了。

Supervisor 配置操作视频：



`@
http://labfile.oss.aliyuncs.com/courses/980/week10/7-2.mp4
@`

## 5. 总结

本节实验主要讲解了 supervisor 的安装和相关配置实例。