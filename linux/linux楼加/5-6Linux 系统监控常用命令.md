---
show: step
version: 1.0
enable_checker: true
---
# Linux 系统监控常用命令

## 1. 实验介绍

#### 1.1 实验内容

我们的系统运行起来过后，有时候也会出现一些问题，比如说无响应，或者运行很慢等，就需要我们监控系统的运行状况，才能及时发现问题并立即进行处理。本节实验主要介绍 Linux 系统用来监控系统各项指标的常用命令。

#### 1.2 实验知识点

- 内存监控（free）
- 进程监控（ps，pstree）
- CPU 监控（top，htop）
- 网络监控（netstat，tcpdump，iftop，traceroute，mtr）
- 磁盘监控（vmstat，df，iotop）

## 2. 内存监控

下面我们将会学习内存监控命令。

### free

free 命令显示系统使用和空闲的内存情况，包括物理内存、交互区内存(swap)和内核缓冲区内存。

```bash
$ free
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1514977387886.png-wm)

> **第一部分 Mem 表示物理内存的使用情况：**
>
> | 列       | 说明       |
> | ------- | -------- |
> | total   | 内存总数     |
> | used    | 已经使用的内存数 |
> | free    | 空闲的内存数   |
> | shared  | 当前已经废弃不用 |
> | buffers | 缓冲内存数    |
> | cached  | 缓存内存数    |
>
> 关系：total = used + free
>
> **第二部分 (-/+ buffers/cache) 表示缓冲的物理内存情况:**
>
> ```
> (-buffers/cache) used 内存数：第一部分 Mem 行中的 used – buffers – cached
> (+buffers/cache) free 内存数: 第一部分 Mem 行中的 free + buffers + cached
> ```
>
> 可见 `-buffers/cache` 反映的是被程序实实在在占据的内存，而 `+buffers/cache` 反映的是可以挪用的内存总数。
>
> **第三部分 Swap 表示硬盘上交换空间的使用情况**。

- 以 MB 为单位显示内存的使用情况

```bash
$ free -m
```

- 显示汇总信息

```bash
$ free -t
```

## 3. 进程监控

下面我们将会学习进程监控命令。

### 3.1 ps

用于显示当前系统的进程状态。可以搭配 kill 指令随时中断、删除不必要的程序。使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多的资源等等。（在第一周的进程与工作中有详细介绍。）

```bash
# 显示自己这次登陆的 bash 相关的进程信息罗列出来
$ ps -l

# 显示所有用户的进程信息
$ ps aux

# 显示所有进程的 UID,PPID 与 STIME 等信息
$ ps -ef
```

![实验楼](https://dn-simplecloud.shiyanlou.com/5962221514978643215-wm)

![实验楼](https://dn-simplecloud.shiyanlou.com/5962221514978666652-wm)

### 3.2 pstree

pstree 能将当前的执行程序以树状结构显示，清楚地表达程序间的产生关系。

```bash
$ pstree
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515031303467-wm)

- 显示当前所有进程的进程号和进程 id

```bash
$ pstree -p
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515032571869-wm)

- 显示所有进程的详细信息，遇到相同的进程名可以压缩显示。

```bash
$ pstree -a
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515032602004-wm)

## 4. CPU 监控

下面我们将会学习CPU 监控命令。

### 4.1 top

 top 用于实时的显示 Linux 系统当前进程的运行情况，并按一定的顺序显示所有正在运行而且处于活动状态的进程，而且会实时更新显示结果。

```bash
$ top
```

![实验楼](https://dn-simplecloud.shiyanlou.com/5962221514966180770-wm)

显示的详细信息可参看第一周的 Linux 进程与工作中的详细介绍，与其使用方法。另外也可以使用 `man` 来查看 `top` 的详细说明：

```bash
$ man top
```

### 4.2 htop

htop 是一个高级的交互式的实时 Linux 进程监控工具。 它和 top 命令十分相似，但是它具有更丰富的特性，例如：用户可以友好地管理进程，使用快捷键，通过横向和纵向的方式显示进程等等。 htop 是一个第三方工具，所以我们需要先安装它。

```bash
$ sudo apt-get update
$ sudo apt-get install htop

$ htop
```

![实验楼](https://dn-simplecloud.shiyanlou.com/5962221514974155019-wm)

左上角： CPU 负载（因为我们是 4 核 CPU，所以这里有 4 行），内存消耗，交换空间的使用情况。

右上角：

- 第一行：总进程数，线程数，正在运行的进程数。
- 第二行：和 top 命令中的 load average 一样，也是代表的 1，5，15 分钟内 CPU 的平均负载。
- 第三行：从系统启动起到当前的运行总时间。

下方的列表代表的是当前进程的运行情况。

底部显示的是一些快捷键，可以很方便对进程进行排序，搜索，杀掉等操作。同样可以使用 `man htop`  查看它的帮助文档。

内存，进程，CPU 监控操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week6/1-1.mp4
@`

## 5. 网络监控

下面我们将会学习网络监控命令。

### 5.1 netstat

netstat 命令可以显示 linux 中当前详细的网络状态信息，包括所有的 TCP 的连接状态。我们在第 3 周网络常用命令中有介绍。

- 显示路由表

```bash
$ netstat -r
```

- 显示所有网卡列表

```bash
$ netstat -i
```

- 显示所有的 Socket 信息与连接状态

```bash
$ netstat -a
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515057667321-wm)

- 显示每个协议的统计信息

```bash
$ netstat -s
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515057745466-wm)

- 筛选监控中的服务器端口

```bash
$ netstat -lunat
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515057779800-wm)

### 5.2 tcpdump

tcpdump 是使用最广泛的网络包分析程序之一，它用于捕捉本机指定条件下的 TCP/IP 包。（在第 3 周的网络常用命令中有介绍。）

如果没有安装，就使用如下命令先安装一下：

```bash
$ sudo apt-get install tcpdump 
```

可以使用如下命令查看 tcpdump 的帮助：

```bash
$ tcpdump -h
或者
$ man tcpdump
```

```bash
$ sudo tcpdump
# 直接启动 tcpdump 将监视网络接口上所有流过的数据包。
```

捕捉到的包信息会及时输出至终端，在长期有流量的情况下输出结果会刷屏，按 `ctrl + c` 可结束数据包的捕捉。

tcpdump 的过滤器很强大，从指定 IP、端口、协议等等来过滤，例如：

- 监控指定网络接口的数据包：

```bash
#抓取 ech0 网络接口的包
$ sudo tcpdump -i eth0
```

- 设置只抓取 10 个数据包：

```bash
$ sudo tcpdump -c 10
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515060318676-wm)

- 监控指定主机的数据包

先用 netstat 来看一下当前网络状态

```bash
$ netstat -lunat
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515057779800-wm) 

利用前面输出中显示的那两个 ip 地址来实践（注意，你自己的输出结果和截图的可能不相同，根据你自己的输出结果来实验。）

我们这里就来捕获主机 `192.168.42.6` 与主机 `192.168.42.2` 之间的通信

```bash
$ sudo tcpdump host 192.168.42.6 and 192.168.42.2 |less
```

> 输出结果很多，可以使用 less 通过翻页键查看上下页的内容

还可以捕获和实验楼之间的通信，我们用浏览器打开实验楼的地址 `www.shiyanlou.com` ，然后再运行 `netstat -lunat` 

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515058915561-wm)

那个 foreign address 是 115.29.233.149 的连接就是与实验楼建立的通信。

然后来捕获和实验楼之间的通信，执行完过后去刷新一下页面就可以看到数据包信息：

```bash
$ sudo tcpdump host 192.168.42.6 and 115.29.233.149
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515059217605-wm)

另外它还可以使用 or ，not（或者 `!` ）

```bash
#主机 192.168.42.6 或者主机 192.168.42.2 收到和发出的所有数据包
$ sudo tcpdump host 192.168.42.6 or 192.168.42.2

#主机 192.168.42.6 和除了主机 192.168.42.2 之外的所有主机通信的数据包
$ sudo tcpdump host 192.168.42.6 and not 192.168.42.2
或者
$ sudo tcpdump host 192.168.42.6 and ! 192.168.42.2
```

- 监控指定端口的数据包

```bash
$ sudo tcpdump port 5901
```

### 5.3 iftop

iftop 是一个实时流量监控工具。iftop 监控的是网络的使用情况，而 top 监控的是 CPU 的使用情况。iftop 监控一个选定的接口并且显示两台主机之间当前宽带的使用情况。

```bash
$ sudo apt-get install iftop

$ sudo iftop
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515049618884-wm)

界面相关说明:

| 关键词   | 说明                      |
| ----- | ----------------------- |
| <= => | 流量的方向                   |
| TX    | 发送流量                    |
| RX    | 接收流量                    |
| Cum   | 运行 iftop 到目前时间的总流量      |
| peak  | 流量峰值                    |
| rates | 分别表示过去 2s 10s 40s 的平均网速 |
| TOTAL | 总流量                     |

界面操作：

| 按键   | 说明                           |
| ---- | ---------------------------- |
| h    | 显示按键帮助                       |
| n    | 切换显示本机的 IP 或主机名              |
| s    | 切换是否显示本机的 host 信息            |
| d    | 切换是否显示远端目标主机的 host 信息        |
| t    | 切换显示格式为 2 行/1 行/只显示发送流量/只显示接收流量 |

### 5.4 traceroute

traceroute 命令用于追踪数据包在网络上的传输时的全部路径，它通过发送小的数据包（默认是 40 字节）到目的设备直到其返回，来测量其需要多长时间。一条路径上的每个设备 traceroute 要测 3 次。输出结果中包括每次测试的时间（ms）和设备的名称（如有的话）及其 ip 地址。

先安装：

```bash
$ sudo apt-get install traceroute
```

例如追踪到 `www.shiyanlou.com` 的路由及速度

```bash
$ traceroute www.shiyanlou.com 
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515051018398-wm)

记录按序列号从 1 开始，每个纪录就是一跳 ，每跳表示一个网关，我们看到每行有三个时间，单位是 ms，这是探测数据包向每个网关发送三个数据包后，网关响应后返回的时间。用这三个时间来表示到达这个结点的网络速度。

我们会看到有一些行是以星号表示的。出现这样的情况，可能是防火墙封掉了 ICMP 的返回信息，所以我们得不到什么相关的数据包返回数据。

- 把跳数设置为 8 次

```bash
$ traceroute -m 8 www.shiyanlou.com
```

- 显示 IP 地址，不查主机名

```bash
$ traceroute -n www.shiyanlou.com
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515051851341-wm)

有时我们在某一网关处延时比较长，有可能是某台网关比较阻塞，也可能是物理设备本身的原因。如果某台 DNS 出现问题时，不能解析主机名、域名时，也会有延时长的现象。可以加 `-n` 参数来避免 DNS 解析，以 IP 格式输出数据。

- 探测包个数设为 4

```bash
$ traceroute -q 4 www.shiyanlou.com
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515051933901-wm)

### 5.5 mtr

`mtr` 是 Linux 中一个判断网络连通性的工具。一般 `ping` 命令可以用来判断丢包率，`traceroute` 命令可以用来追踪路由，而 `mtr` 命令结合了 ping，traceroute，nslookup 的相关特性可以用来判断网络的连通性。（在第 3 周的网络常用命令中有介绍。）

```bash
$ sudo apt-get install mtr

$ mtr -r www.shiyanlou.com
```

> `-r` 报告模式显示。

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515052498737-wm)

```bash
$ mtr -r -c 30 -s 1024 www.shiyanlou.com
```

> `-c` 设置每秒发送数据包的数量
>
> `-s` 设置 ping 包大小为多少个字节

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515052679698-wm)


网络监控操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week6/1-2.mp4
@`

## 6. 磁盘监控

下面我们将会学习磁盘监控命令。

### 6.1 vmstat

 vmstat 是 Virtual Memeory Statistics（虚拟内存统计）的缩写，命令用于显示虚拟内存、内核线程、磁盘、系统进程、I/O 块、中断、CPU 活动等的统计信息。

- 显示磁盘信息

```bash
$ vmstat -d
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515054092772-wm)

> **Reads**
>
> | 列       | 说明      |
> | ------- | ------- |
> | total   | 成功读取数   |
> | merged  | 分组读取数   |
> | sectors | 成功读取扇区数 |
> | ms      | 读取花费毫秒数 |
>
> **Writes**
>
> | 列       | 说明      |
> | ------- | ------- |
> | total   | 成功写入数   |
> | merged  | 分组写入数   |
> | sectors | 成功写入扇区数 |
> | ms      | 写入花费毫秒数 |
>
> **IO**
>
> | 列    | 说明         |
> | ---- | ---------- |
> | cur  | 正在进行中的 I/O |
> | sec    | I/O 花费时间   |

除了 `-d` 选项 ，它还有一些其他的选项：

```bash
-a：显示活动内页；
-f：显示启动后创建的进程总数；
-m：显示 slab 信息；
-n：头信息仅显示一次；
-s：以表格方式显示事件计数器和内存状态；
-d：报告磁盘状态；
-p：显示指定的硬盘分区状态；
-S：输出信息的单位。
```

- 每 2 秒显示一次系统内存的统计信息

```bash
$ vmstat 2
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515053820340-wm)

> **Procs（进程）**
>
> | 列    | 说明                                       |
> | ---- | ---------------------------------------- |
> | r    | 等待执行的任务数。展示了正在执行和等待 CPU 资源的任务个数。当这个值超过了 CPU 数目，就会出现 CPU 瓶颈了 |
> | b    | 等待 IO 的进程数量                              |
>
> **Memory（内存）**
>
> | 列     | 说明                                       |
> | ----- | ---------------------------------------- |
> | swpd  | 正在使用的虚拟内存大小。swpd 的值不为0，但是 SI，SO 的值长期为 0 的情况不会影响系统性能 |
> | free  | 空闲物理内存大小                                 |
> | buff  | 用作缓冲的内存大小                                |
> | cache | 用作缓存的内存大小。如果 cache 的值比较大，说明 cache 处的文件数多。 |
>
> **Swap（交换区）**
>
> | 列    | 说明                    |
> | ---- | --------------------- |
> | si   | 每秒从交换区写到内存的大小，由磁盘调入内存 |
> | so   | 每秒写入交换区的内存大小，由内存调入磁盘  |
>
> 内存够用的时候，这两个值都是 0，如果这两个值长期大于 0 ，系统性能会受到影响，磁盘 IO 和 CPU 资源都会被消耗。
>
> **IO（输入输出）**
>
> | 列    | 说明      |
> | ---- | ------- |
> | bi   | 每秒读取的块数 |
> | bo   | 每秒写入的块数 |
>
> 随机磁盘读写的时候，这两个值越大（如超出 1024k），能看到 CPU 在 IO 等待的值也会越大。
>
> **system（系统）**
>
> | 列    | 说明           |
> | ---- | ------------ |
> | in   | 每秒中断数，包括时钟中断 |
> | cs   | 每秒上下文切换数     |
>
> 上面两个值越大，会看到由内核消耗的 CPU 时间会越大。
>
> **CPU（以百分比表示）**
>
> | 列    | 说明                                       |
> | ---- | ---------------------------------------- |
> | us   | 用户进程执行时间百分比(user time)。us 的值比较高时，说明用户进程消耗的 CPU 时间多，但是如果长期超 50% 的使用，那么我们就该考虑优化程序算法或者进行加速。 |
> | sy   | 内核系统进程执行时间百分比(system time)。sy 的值高时，说明系统内核消耗的 CPU 资源多。 |
> | wa   | IO 等待时间百分比。wa 的值高时，说明 IO 等待比较严重，这可能由于磁盘大量运作随机访问造成，也有可能是磁盘出现瓶颈。 |
> | id   | 空闲时间百分比。                                 |

- 每 2 秒显示一次系统内存的统计信息，总共 3 次

```bash
$ vmstat 2 3
```

### 6.2 df

df 命令用于显示磁盘分区上的可使用的磁盘空间。默认显示单位为 KB。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

```bash
$ df
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515056607011-wm)

- 显示所有文件系统的磁盘使用情况

```bash
$ df -a
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515056721557-wm)

- 自动转换单位来显示，提高可读性

```bash
$ df -h
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515056865657-wm)

它还有一些其他的选项：

```bash
-a 或--all：包含全部的文件系统；
--block-size=<区块大小>：以指定的区块大小来显示区块数目；
-h 或--human-readable：以可读性较高的方式来显示信息；
-H 或--si：与-h参数相同，但在计算时是以 1000 Bytes 为换算单位而非 1024 Bytes；
-i 或--inodes：显示 inode 的信息；
-k 或--kilobytes：指定区块大小为 1024 字节；
-l 或--local：仅显示本地端的文件系统；
-m 或--megabytes：指定区块大小为 1048576 字节；
--no-sync：在取得磁盘使用信息前，不要执行 sync 指令，此为预设值；
-P 或--portability：使用 POSIX 的输出格式；
--sync：在取得磁盘使用信息前，先执行 sync 指令；
-t<文件系统类型>或--type=<文件系统类型>：仅显示指定文件系统类型的磁盘信息；
-T 或--print-type：显示文件系统的类型；
-x<文件系统类型>或--exclude-type=<文件系统类型>：不要显示指定文件系统类型的磁盘信息；
--help：显示帮助；
--version：显示版本信息。
```

### 6.3 iotop

iotop 是一个用来监视磁盘 I/O 使用状况的 top 类工具，可监测到哪一个程序使用的磁盘 IO 的信息，在查找具体进程和大量使用磁盘读写进程的时候，这个工具就非常有用。 这个命令只有在 kernel v2.6.20 及以后的版本中才有，python 版本需要 python2.7 及以上版本。由于实验环境的权限限制并不能成功的展示该命令。此处仅做介绍。

磁盘监控操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week6/1-3.mp4
@`

## 7. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

- 内存监控
  - free
- 进程监控
  - ps
  - pstree
- CPU 监控
  - top
  - htop
- 网络监控
  - netstat
  - tcpdump
  - iftop
  - traceroute
  - mtr
- 磁盘监控
  - vmstat
  - df
  - iotop
