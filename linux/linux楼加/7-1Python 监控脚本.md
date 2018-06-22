---
show: step
version: 1.0
enable_checker: true
---
# Python 监控脚本

## 1. 实验介绍

#### 1.1 实验内容

本实验中我们将使用 python 实现 netstat 端口信息的查看。

#### 1.2 实验知识点

- 了解实验目的
- 解析实现功能
- 实现 netstat 功能

#### 1.3 推荐阅读

- [netstat 实现原理](https://github.com/ecki/net-tools/blob/master/netstat.c)
- [psutil 实现 netstat](https://github.com/giampaolo/psutil/blob/master/scripts/netstat.py)
- [listdir、walk、glob 速度对比](https://stackoverflow.com/questions/8931099/quicker-to-os-walk-or-glob)
- [tcp 文件解析](https://www.kernel.org/doc/Documentation/networking/proc_net_tcp.txt)
- [文件描述的理解](http://shanks.leanote.com/post/5908aa3ab757)
- [proc 目录详情](http://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/proc.html)
- [proc 官方说明](https://github.com/torvalds/linux/blob/master/Documentation/filesystems/proc.txt)
- [fd 的源码](https://github.com/torvalds/linux/blob/master/fs/proc/fd.c)

#### 1.4 效果展示

1.在非 root 权限情况下执行脚本：

![show-without-sudo](https://dn-anything-about-doc.qbox.me/document-uid113508labid2213timestamp1502334248424.png/wm)

2.使用 sudo 执行脚本：

![show-with-sudo](https://dn-anything-about-doc.qbox.me/document-uid113508labid2213timestamp1502334280529.png/wm)

## 2. 实验目的

本实验的目的在于用 python 实现一个 `netstat`。

在日常运维时我们用到 `netstat` 命令的频率会非常高，例如这样的一些场景：

1. 配置 docker 服务端绑定的端口与地址，我们需要验证一下是否配置正确，使用 `netstat` 命令查看相关的端口是否开放，相关的程序是不是我们预期的那个（当然不止 docker，也可以是 nginx、apache、mysql 等等的服务应用）。

2. 当我们启动某个服务之后，需要查看有哪些 IP 地址连接上了端口，与我们建立了连接，发送了数据包，这个时候我们可能会用到 `netstat` 命令。

3. 若是我们受到了 SYN 泛洪攻击，在我们不知情的情况下，首先会查看 CPU 信息，紧接着可能会想到使用 `netstat` 查看此时半连接数多不多。

4. 有时候我们感觉到服务器无缘无故非常卡顿，或者从监控中查看到有异常的指标等等，排除自身原因可能会猜想服务器被人攻击了，留下了后门，若是依赖通过 `ps auxf` 或者 `ps -ef` 命令来查找异常的进程宛如大海捞针，很难定位而且非常的低效，并且对方也可能通过 `mount-bind` 的方式隐藏进程相关信息，但是我们若是通过 `netstat` 命令查看异常的端口再结合 `lsof` 与 `ps` 或许能帮我们更快的定位问题程序的所在。

诸如此类的情况我们都会用到 `netstat` 命令,`netstat` 来自于 `net-tool` 工具集，在 2001 年便停止维护了，很多同学可能说用 `netstat` 命令已经 out 了，更多的人会选择 `ss` 命令，因为这个命令可以展示更多的网络相关信息，而且因为使用的是 `tcp_diag` 模块效率会高不少。

二者的区别并不是本实验的重点，而选择 `netstat` 只是为了在自己建立运维平台时想获取端口信息，或者在自定义显示端口信息时提供一种思路，同时因为其实现简单，能帮助我们快速上手。

## 3. netstat 功能实现

下面我们将会学习netstat 功能实现。

### 3.1 netstat 简单解析

有兴趣的同学可以看看推荐阅读中 github 上 `netstat` 的源码，这里不做过多的讲解只是简单说说思路。

在 netstat 中首先会去解析各种参数，以此来判断我们需要怎样的信息。然后若需要查看 tcp 的相关信息，`netstat` 会去读取 `/proc/net/tcp` 中的内容，同理若是 tcp6 则读取 `/proc/net/tcp6`，若是 udp 则去读取 `/proc/net/udp`，若是 udp6 则读取 `/proc/net/udp6` 中的信息：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081502334367442-wm)

在文件中的第一行已经给出了一部分内容的解释，但是 inode 值后面的内容并没有逐行注释，以下将给予大家完整的信息解析(将所有的信息分解为 3 部分)：

```bash
It will first list all listening TCP sockets, and next list all established TCP connections. A typical entry of /proc/net/tcp would look like this (split up into 3 parts because of the length of the line):

   46: 010310AC:9C4C 030310AC:1770 01
   |      |      |      |      |   |--> 连接状态
   |      |      |      |      |------> 远程TCP端口号
   |      |      |      |-------------> 远程 IPv4 地址
   |      |      |--------------------> 本地 TCP 端口号
   |      |---------------------------> 本地 IPv4 地址
   |----------------------------------> 条目的数量

   00000150:00000000 01:00000019 00000000
      |        |     |     |       |--> 未恢复的RTO超时数量
      |        |     |     |----------> 在计时器到期之前的 jiffies 数量
      |        |     |----------------> timer_active(见原文)
      |        |----------------------> 接受队列
      |-------------------------------> 传输队列

   1000        0 54165785 4 cd1e6040 25 4 27 3 -1
    |          |    |     |    |     |  | |  | |--> 慢启动阈值大小
    |          |    |     |    |     |  | |  |      or -1 if the threshold
    |          |    |     |    |     |  | |  |      is >= 0xFFFF
    |          |    |     |    |     |  | |  |----> 发送拥塞窗口
    |          |    |     |    |     |  | |-------> (ack.quick<<1)|ack.pingpong
    |          |    |     |    |     |  |---------> Predicted tick of soft clock
    |          |    |     |    |     |              (延迟的ACK控制数据)
    |          |    |     |    |     |------------> 重新传输超时
    |          |    |     |    |------------------> 套接字在内存中的位置
    |          |    |     |-----------------------> 套接字引用计数
    |          |    |-----------------------------> inode
    |          |----------------------------------> 未答复的0窗口探针
    |---------------------------------------------> uid
```

（此段来自于 [kernel.org](https://www.kernel.org/doc/Documentation/networking/proc_net_tcp.txt)）

在读取文件中的相关信息之后我们会发现其中部分信息是 16 进制的表示方式，直接查看可读性会很差，所以 `netstat` 会将信息做一定的筛选与转码，最后呈现出我们能够轻易解读的和我们关心的相关信息。

在使用 `netstat` 的时候，通过添加 `-p` 参数可以获取相关进程的 PID 与进程的程序名，这一实现是通过在读取的信息中有一项是 inode 值，这个 inode 是相关进程的 socket 的文件所在的 inode 值，所以可以通过 inode 值再去查找相关的进程信息。

> *inux 中内核为每个进程维护一个打开文件的列表，该表被称为文件表（`file table`）。该表由一些叫做文件描述符（`file descriptors`）的非负整数进行索引，一个文件描述符对应进程中一个打开的文件，无论是进程间的 pipe 通信，还是 socket 通信，还是标准的输入输出在进程中都有对应的文件描述符。一般情况下 0、1、2 索引对应的是标准输入、标准输出、标准错误。

> 本质上这些索引都是一些软链接，我们可以查看其真正打开的文件，而一般进程间 pipe 通信与 socket 通信打开的文件并没有实际的文件名，所以会直接对应到文件对应的 inode 上，展示的效果是这样的：`6 -> pipe:[3142927]`、`7 -> socket:[3142930]`。

(对这一部分不太清楚的同学，又想进一步了解的同学，可以查看推荐阅读中文件描述的理解的文章)

这就是我们使用 netstat 查看相关信息的来源。

### 3.2 netstat 的实现

根据上一小节的叙述，我们了解了 `netstat` 的原理，因此我们可以归纳出这样一个流程图：

![process](https://dn-simplecloud.shiyanlou.com/uid/113508/1502334449352.png-wm)

根据这样的流程，我们便可以开始逐步的实现：

1.判断是否有参数

因为这是一段小程序我们会以脚本的形式来执行，我们可以使用 `sys` 模块获取命令行执行脚本时所有用的参数，从而判断我们需要哪种类型的数据，若是没有则默认选择查看 tcp 相关的端口信息：

```python
if __name__ == '__main__':
    choose = 'tcp'
    if len(sys.argv) > 1:
        choose = sys.argv[1]
    main(choose)
```

2.在获取用户的选择之后便开始获取数据内容：

剩下的操作可以说基本上是本工具的主要操作，所以我们将其放置于 `main()` 方法中，流程第一步就是读取相关类型文件的内容：

```python
PROC_FILE = {
    'tcp': '/proc/net/tcp',
    'tcp6': '/proc/net/tcp6',
    'udp': '/proc/net/udp',
    'udp6': '/proc/net/udp6'
}


def get_content(type):
    ''' 读取文件内容
    '''
    with open(PROC_FILE[type], 'r') as file:
        content = file.readlines()
        content.pop(0)  # 去除文章的第一行抬头
    return content


def main(choose):
    '''获取以及展示端口链接相关信息
    '''
    templ = "%-5s %-30s %-30s %-13s %-6s %s"
    print(templ % (
        "Proto", "Local address", "Remote address", "Status", "PID",
        "Program name"))
    content = get_content(choose)
```

在这里我们为了增加展示内容的可读性，我们定义了一个输出的模版，以及先输出每一列展示内容的意义，再根据选择读取相关文件的内容。

同时我们通过 `pop(0)` 的方法去除了第一行的内容，因为从之前的查看中我们了解到内容中的第一行起一个解释的作用，对我们信息的提取与筛选展示并没有帮助。

3.分离数据

在获取所有内容之后，就是对信息的筛选，用更合理的方式展现我们需要的信息。

(1)将每一行的数据进行拆分

之前我们在读取数据的时候是一行一行读取的，每一行数据在一个列表中，现在我们需要对每一个数据都做处理，所以需要将一行数据进行整体拆分。

```python
    for info in content:
        iterms = info.split()
```

（2）获取每一列的数据

protocol 的类型（我们这里偷懒），因为我们只是查看单个文件，所以可以直接使用 `choose`。

`local address` 和 `remote address` ：我们只需要将其 `16` 进制的表示方式转换成 10 进制的点分表示方式即可。

status 也是通过 16 进制的形式展现出来的，每一个值都代表一种状态：

```
STATUS = {
    '01': 'ESTABLISHED',
    '02': 'SYN_SENT',
    '03': 'SYN_RECV',
    '04': 'FIN_WAIT1',
    '05': 'FIN_WAIT2',
    '06': 'TIME_WAIT',
    '07': 'CLOSE',
    '08': 'CLOSE_WAIT',
    '09': 'LAST_ACK',
    '0A': 'LISTEN',
    '0B': 'CLOSING'
}
```

我们只需要通过一个字典获取对应的状态即可。

PID 的获取比较麻烦，因为我们只是知道 socket 文件的 inode 值，上文我们提到过每个进程都会维护一个文件表，表中会有多个文件描述符，只要进行了 socket 的进程，那文件描述符中的索引一定会有体现，所以我们只要查找所有文件描述符中有没有等值的 inode 值即可。

那么现在的问题就是如何查找所有的文件描述符？在 `/proc` 目录中以数字命名的目录便是进程相关的目录，而数字便是其进程 ID，在目录中会有相关进程的信息，其中有一个 `fd` 的目录就是文件描述符中索引的目录，在该目录中我们可以看到该进程打开的所有文件。

所以我们完成这一功能的思路就是遍历这样的目录 `/proc/数字/fd/`,然后通过获取软链接对应的文件，判断有没有 inode 对应的值，若是找到了自然就获取到了 PID。

> 对于 `/proc` 目录不太了解的同学，又想进一步了解该目录中会有哪些信息可以查看推荐阅读中的 proc 目录详情。

```python
def get_pid(inode):
    for path in glob.glob('/proc/[0-9]*/fd/[0-9]*'):
        try:
            if str(inode) in os.readlink(path):
                return path.split('/')[2]
            else:
                continue
        except:
            pass
    return None
```

这里使用 `glob` 模块是因为只有它支持通配符的方式来遍历我们所需要的目录，并且返回的是一个完整的路径，虽然 `listdir` 模块也可以通过列表生成式的方式遍历所有的进程目录的 fd 目录，但是返回的是文件名，我们还需要进行拼接，较为麻烦，所以用 glob 比较方便。

同时这里会使用到 try ，因为进程本来就是一个程序执行的过程，其具有动态性，可以被创建、撤销，所以 `/proc` 目录中的数字命名的目录也是动态的，可能当我们通过 glob 获取所有的进程的文件描述符的路径之后，其中某个进程就完成任务，然后被撤销了，其相关的目录也被删除了，此时我们再通过 `os.readlink` 获取软链接对应的源文件时就会报错，毕竟目录都不存在了。

通过获取到 PID 之后我们只剩下最后一项信息，便是 program name 的获取。若是对进程目录较为熟悉的同学会知道目录中有一个 `comm` 文件，其记录的就是此进程的程序名，还有一个文件是 `cmdline`，这个文件中记录的是该进程的启动目录。

所以我们获取进程的 program name 只需要读取该文件即可。

### 3.3 完整代码展示

提示：查看 `/proc/PID/fd/` 目录下的文件时需要 root 权限，所以在没有使用 root 权限执行脚本时，是获取不到 PID 的。本实验环境中我们没有相关权限，所以获取不到 PID 与 program name，大家可以在本地尝试。

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
import sys
import glob

PROC_FILE = {
    'tcp': '/proc/net/tcp',
    'tcp6': '/proc/net/tcp6',
    'udp': '/proc/net/udp',
    'udp6': '/proc/net/udp6'
}

STATUS = {
    '01': 'ESTABLISHED',
    '02': 'SYN_SENT',
    '03': 'SYN_RECV',
    '04': 'FIN_WAIT1',
    '05': 'FIN_WAIT2',
    '06': 'TIME_WAIT',
    '07': 'CLOSE',
    '08': 'CLOSE_WAIT',
    '09': 'LAST_ACK',
    '0A': 'LISTEN',
    '0B': 'CLOSING'
}


def get_content(type):
    ''' 读取文件内容
    '''
    with open(PROC_FILE[type], 'r') as file:
        content = file.readlines()
        content.pop(0)  # 去除文件的第一行抬头
    return content


def get_program_name(pid):
    '''获取程序名
    '''
    path = '/proc/' + str(pid) + '/comm'
    with open(path, 'r') as file:
        content = file.read()
    content = content.strip()
    return content


def convert_ip_port(ip_port):
    '''转换
    '''
    ip, port = ip_port.split(':')
    port = int(port, 16)
    ip = [str(int(ip[6:8], 16)), str(int(ip[4:6], 16)), str(int(ip[2:4], 16)),
          str(int(ip[0:2], 16))]
    ip = '.'.join(ip)
    return ip, port


def get_pid(inode):
    '''
        获取 PID 的原理：
            内核为每个进程维护一个打开文件的列表，该表被称为文件表（file table）。该表由一些叫做文件描述符（file descriptors）的非负整数进行索引，一个文件描述符对应进程中一个打开的文件。
            在 /proc/net/tcp 可以得到 inode 值，而这个 inode 就是相关进程的 socket 文件所在的 inode 值
        获取的方法：
            1. 首先遍历所有的 /proc/ 进程 id/fd/ 文件夹
                遍历所有的目录有三种方式 walk，listdir，glob
                三者遍历的速度 listdir > glob > walk
                并且只有 glob 支持通配符遍历，而在 /proc/ 目录下有很多系统性能信息相关的目录与文件，没必要全部遍历，只需要找进程相关的目录即可
                当然也可以使用 psutil.pids() 获取所有的 pid，然后用双重循环来遍历所有文件描述符的索引，但是我们若是用了 psutil 模块的话，很多地方获取起来就会简单很多了
            2. 因为 socket 的通信通道没有文件名，所以文件对应的软连接是 socket:inode 值，所以查看有没有匹配的值
            3. 然后获取相关文件的 path 中的 pid 即可
    '''
    for path in glob.glob('/proc/[0-9]*/fd/[0-9]*'):
        try:
            if str(inode) in os.readlink(path):
                return path.split('/')[2]
            else:
                continue
        except:
            pass
    return None


def main(choose):
    '''获取以及展示端口链接相关信息
    '''
    templ = "%-5s %-30s %-30s %-13s %-6s %s"
    print(templ % (
        "Proto", "Local address", "Remote address", "Status", "PID",
        "Program name"))
    content = get_content(choose)

    for info in content:
        iterms = info.split()
        proto = choose
        local_address = "%s:%s" % convert_ip_port(iterms[1])
        status = STATUS[iterms[3]]
        if status == 'LISTEN':
            remote_address = '-'
        else:
            remote_address = "%s:%s" % convert_ip_port(iterms[2])
        pid = get_pid(iterms[9])
        program_name = ''
        if pid:
            program_name = get_program_name(pid)
        print(templ % (
            proto,
            local_address,
            remote_address,
            status,
            pid or '-',
            program_name or '-',
        ))


if __name__ == '__main__':
    choose = 'tcp'
    if len(sys.argv) > 1:
        choose = sys.argv[1]
    main(choose)

```

Python 监控脚本完整代码讲解视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week6/2-1.mp4
@`

### 3.4 使用 psutil 实现

以上的方式能够帮我们进一步的了解 Linux，但是实现起来一点都不简单，在 psutil 模块的 github 中我们会发现这样一份代码，同样也是实现 netstat 相关功能，相对来说简单不少：

```python
#!/usr/bin/env python

# Copyright (c) 2009, Giampaolo Rodola'. All rights reserved.
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

"""
A clone of 'netstat -antp' on Linux.
$ python scripts/netstat.py
Proto Local address      Remote address   Status        PID    Program name
tcp   127.0.0.1:48256    127.0.0.1:45884  ESTABLISHED   13646  chrome
tcp   127.0.0.1:47073    127.0.0.1:45884  ESTABLISHED   13646  chrome
tcp   127.0.0.1:47072    127.0.0.1:45884  ESTABLISHED   13646  chrome
tcp   127.0.0.1:45884    -                LISTEN        13651  GoogleTalkPlugi
tcp   127.0.0.1:60948    -                LISTEN        13651  GoogleTalkPlugi
tcp   172.17.42.1:49102  127.0.0.1:19305  CLOSE_WAIT    13651  GoogleTalkPlugi
tcp   172.17.42.1:55797  127.0.0.1:443    CLOSE_WAIT    13651  GoogleTalkPlugi
...
"""

import socket
from socket import AF_INET, SOCK_STREAM, SOCK_DGRAM

import psutil


AD = "-"
AF_INET6 = getattr(socket, 'AF_INET6', object())
proto_map = {
    (AF_INET, SOCK_STREAM): 'tcp',
    (AF_INET6, SOCK_STREAM): 'tcp6',
    (AF_INET, SOCK_DGRAM): 'udp',
    (AF_INET6, SOCK_DGRAM): 'udp6',
}


def main():
    templ = "%-5s %-30s %-30s %-13s %-6s %s"
    print(templ % (
        "Proto", "Local address", "Remote address", "Status", "PID",
        "Program name"))
    proc_names = {}
    for p in psutil.process_iter():
        try:
            proc_names[p.pid] = p.name()
        except psutil.Error:
            pass
    for c in psutil.net_connections(kind='inet'):
        laddr = "%s:%s" % (c.laddr)
        raddr = ""
        if c.raddr:
            raddr = "%s:%s" % (c.raddr)
        print(templ % (
            proto_map[(c.family, c.type)],
            laddr,
            raddr or AD,
            c.status,
            c.pid or AD,
            proc_names.get(c.pid, '?')[:15],
        ))


if __name__ == '__main__':
    main()

```

## 4. 总结

本节实验中我们学习了以下内容：

- 了解实验目的
- 解析实现功能
- 实现 netstat 功能

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。

## 5. 作业

本实验中我们简单实现了 netstat 的功能，但是我们一次只能查看一个文件，如 tcp 或者 tcp6 等等，你能否尝试一下默认情况查看所有 tcp、tcp6、udp、udp6 的信息呢？