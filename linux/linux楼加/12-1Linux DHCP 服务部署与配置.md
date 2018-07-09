---
show: step
version: 1.0
enable_checker: true
---
# Linux DHCP 服务部署与配置

## 1. 实验介绍

#### 1.1 实验内容

传统的方法 IP 地址是手动设置，只有在配置了正确的 IP 地址与网关等信息之后我们才能上网，而 DHCP 服务可以自动的分配 IP 与相关的网络参数给用户端，来提供用户端自动完成相关信息的配置。这样大大提高了网络配置的效率，那这一节我们就来一起学习一下 DHCP 服务器原理以及如何搭建 DHCP 服务器。

#### 1.2 实验知识点

+ DHCP 原理
+ DHCP 服务器搭建

#### 1.3 推荐阅读

+ [ubuntu 社区文档](https://help.ubuntu.com/community/isc-dhcp-server)
+ [官方文档](https://www.isc.org/wp-content/uploads/2017/08/dhcp41conf.html)

## 2. DHCP 原理

#### 2.1 DHCP 概述

DHCP (Dynamic Host Configuration Protocol)，动态主机配置协议是应用层上一种客户端/服务器协议，用于**动态分配 IP 地址**与网关等参数到 `DHCP` 客户端。

DHCP 主要是为计算机自动提供 `IP 地址`、`子网掩码`和`网关`。网络管理员会分配某个范围的 IP 地址来分发给局域网上的客户机，当设备接入这个局域网时，会向 DHCP 服务器请求一个 IP 地址。然后 DHCP 服务器为每个请求的设备分配一个地址，直到分配完该范围内的所有 IP 地址为止。已经分配的 IP 地址必须定时地延长借用期，这个延期的过程称做 `leasing`。

#### 2.2 DHCP 特点

+ DHCP 最大的功能就是**动态分配**，除了 IP 地址，DHCP 还为客户端提供其他的配置信息，比如子网掩码、网关等，客户端自动配置后就可以连接到网络。
+ DHCP 可以确保同一时刻一个 IP 地址只分配给一个用户。
+ DHCP 可以为特定的用户分配固定的 IP 地址。

#### 2.3 DHCP 工作流程

用户端主机通过 UDP 广播的形式向本地网络中所有设备发送请求数据包，当 DHCP 服务器收到请求后根据自身的配置信息将分配给客户端的信息发送给客户端主机，其中的租赁期是有限的。一旦租赁期到期，服务端会回收相关的 IP 地址，所以客户端在租凭过半的时候会向服务器发送更新租约的请求，从而继续使用相关的 IP 地址。

如下图所示为通过 DHCP 获取 IP 的过程：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1862timestamp1509930373342.png/wm)

1.DHCP 发现阶段

客户端以广播方式（因为 DHCP 服务器的 IP 地址对于客户端来说是未知的）发送 DHCP discover 信息来查找 DHCP 服务器。

2.DHCP 提供阶段

在网络中接收到 DHCP discover 数据包的 DHCP 服务器都会做出响应。它从尚未出租的  IP 地址池中挑选一个分配给 DHCP 客户端，并且发送一个包含要分配的 IP 地址以及一些其他配置信息的 DHCP offer 数据包给客户端。

3.DHCP 选择阶段

如果有多台 DHCP 服务器向 DHCP 客户端发送 DHCP offer 数据包，则 DHCP 客户端只接受第 1 个收到的 DHCP offer 信息（**即先到先得**）。然后它就以广播方式回复一个 DHCP request 数据包，该数据包中包含向它所选定的 DHCP 服务器的 IP 地址与被分配的 IP 地址信息。之所以要以广播方式回答，是为了通知所有 DHCP 服务器。

4.DHCP 确认阶段

当 DHCP 服务器收到 DHCP 客户端回答的 DHCP request 数据包之后，会确认 request 数据包中的 DHCP 服务端 IP 地址是否与自己的相同，若是不同则删除自己记录中的分配记录，所以除 DHCP 客户端选中的服务器外，其他的 DHCP 服务器都将收回曾提供的 IP 地址。若是相同它会向 DHCP 客户端发送一个包含其所提供的 IP 地址和租凭时间等其他信息的 DHCP ACK 数据包，告诉 DHCP 客户端可以使用该 IP 地址。

5.DHCP 请求阶段（DHCP information）

DHCP 客户端在接收到服务端的 ACK 数据包之后，检查分配的 IP 地址是否能够使用，若是能够使用则成功获得 IP 地址，并会根据租凭信息中的租约时间自动更新、延续租凭时间，若是 IP 地址不能够使用，则会发送 DHCP Decline 的数据包告知 DHCP 禁用该 IP 地址，然后重新申请 IP 地址。或者是获取更多的其他信息，如代理等信息。

6.DHCP 更新租约

DHCP 服务器向 DHCP 客户端出租的 IP 地址一般都有一个租借期限，期满后 DHCP 服务器便会收回该 IP 地址。如果 DHCP 客户端要延长其 IP 租约，则必须向 DHCP server 端发送更新其 IP 租约的请求数据包。DHCP 客户端在 IP 租约期限过一半时，DHCP 客户端都会自动向 DHCP 服务器发送更新其 IP 租约的信息。

## 3. DHCP 服务器搭建

下面我们将会学习 DHCP 服务器搭建。

### 3.1 安装 DHCP 服务器软件

DHCP 服务器的安装相对比较容易，只需要安装 DHCP 软件即可。从上期实验中大家开始接触 centos，其实和 ubuntu 的区别并不是太大，只是在安装的方式、包名、配置文件位置等的一些细小区别而已。

在实验中我们使用 centos，其安装 DHCP 的方式为：

```bash
sudo yum install dhcp
```

若环境是 `ubuntu`，则我们安装的软件包是 `isc-dhcp-server`，这个软件包在 ubuntu 12.04 LTS 之前的版本被称为 `dhcp3-server`，输入如下命令进行 DHCP 软件的安装：

```bash
$ sudo apt-get update
$ sudo apt-get install -y isc-dhcp-server
```

### 3.2 配置文件解析

#### 3.2.1 DHCP 配置文件

在对 DHCP Server 做一些定制化的配置时，我们主要会修改这样两个配置文件：

- `/etc/dhcp/dhcpd.conf`：DHCP 服务端为客户端分发的参数配置
- `/etc/sysconfig/dhcpd`：DHCP 服务启动时的一些特殊参数

>注意在 ubuntu 中该文件位于：
>- `/etc/default/isc-dhcp-server`：DHCP 服务启动时候的一些特殊参数

其中 `/etc/dhcp/dhcpd.conf` 配置文件中配置项主要的配置依赖于：

+ parameters：指定怎么执行任务(如多长的租约提供)、是否执行任务（是否给默认客户端提供 IP），以及为 DHCP 客户端提供网络配置选项（例如提供网关）。

+ declarations：描述一个网络拓扑（包含 shared-network、子网等信息），会提供的地址等等的一些信息

当然还有一些其他的参数，例如配置项以 option 进行修饰，带表达式的配置选项，黑白名单的配置等等，这样的一些高级配置选项这里没有办法做到一一讲解，接下来会通过一个简单的搭建例子带领大家了解 DHCP 是如何搭建，对于新的需求、需要用到的高级配置可以通过推荐阅读中的官方文档查看。

其中 `/etc/sysconfig/dhcpd` 中配置 DHCP Server 端的一些启动配置选项，例如启动时读取配置文件的位置，相关的服务启动监听在哪一张网卡上等等，一般其他的配置选项保持默认，需要自定义的是服务所监听的网卡：

```bash
DHCPDARGS=eth0
```

但现在这个文件逐渐被废弃，dhcp 服务会根据我们配置的子网所对应的网络接口自动启动在相应的接口上。

在 ubuntu 中的 `/etc/default/isc-dhcp-server` 是这样配置：

>
>```bash
>INTERFACES="eth0"
>```
>

细心的同学会注意到这里用的是 INTERFACES，所以这里其实可以配置多张网卡，不同的网卡提供不同子网的地址，用空格分隔即可。

#### 3.2.2 配置文件示例

说了再多也不如一个例子来的直接：

1.配置 `/etc/dhcp/dhcpd.conf` 文件:

我们通过 vim 修改 `/etc/dhcp/dhcpd.conf` 文件，默认没有任何配置，我们只需要配置 subnet 。如果需要配置更多参数，可以查看安装的示例配置文件  `/usr/share/doc/dhcp-4.2.5/dhcpd.conf.example` 。

```
$ sudo vim /etc/dhcp/dhcpd.conf
```

当前的环境有 eth0、eth1 两张网卡，我准备将服务放在 eth0 网卡上，通过 `ifconfig eth0` 命令我们可以看到当前的 IP 地址，从而判断当前的网段:

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4272timestamp1514136147048.png/wm)

从图中我们可以看到当前的 IP 地址是 `10.29.113.73`，所以只要我的 IP 在我设置的子网网段中就没有问题，由此得来以下配置:

```bash
subnet 10.29.113.0 netmask 255.255.255.0 {
    # 分配的地址段
    range 10.29.113.1 10.29.113.100;

    # 配置的网关
    option routers 10.29.113.1;
}
```

dhcp-server 不像我们在配置路由器或者 switch 中的 dhcp 一样可以使用 exclude 来排除一些正在使用的 IP 地址，所以解决方法便是若中间的 IP 地址被占用就多个 range 的 IP 地址区间段。

网关的配置是可选的，也就是可以不用配置，与此类似的还有子网掩码、广播地址、ntp 服务器、域名解析服务器等等。

实际操作时需要以当前主机 eth0 网卡的 ip 和 netmask 为准。这里一定要注意 subnet 号要计算正确，使用从 ifconfig 命令获取到的 eth0 的 netmask 去跟 eth0 的 ip 进行按位与计算，即可得到 subnet 号。否则启动会失败。

当然我们可以在这里面配置静态地址，例如根据某个 MAC 地址，为其安排固定的 IP 地址：

```bash
subnet 10.29.113.0 netmask 255.255.255.0 {
    # 分配的地址段
    range 10.29.113.1 10.29.113.100;

    # 配置的网关
    option routers 10.29.113.1;

    host shiyanlou-1 {
            hardware ethernet 02:42:c0:a8:00:03;
            fixed-address 10.29.113.2;
    }
}
```

这样配置之后当 DHCP 服务器收到这个 MAC 地址发送来的请求数据包，就只会给他分发这个 IP 地址，不会分发其他的 IP 地址了。

### 3.3 启动 DHCP 服务器

通过这样的命令启动 DHCP 后台程序：

```bash
$ sudo systemctl start dhcpd
```

若是本来 DHCP 后台程序处于活动中，便需要通过这样的命令重启 DHCP 来使得配置生效：

```bash
$ sudo systemctl restart dhcpd
```

若是启动失败我们可以在 `/var/log/message` 中查看到报错日志.

例如我的 IP 地址是 `192.168.0.3` 但是子网部分我配置的是 `192.168.1.0` 网段，我重启服务的时候:

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1862timestamp1510037902402.png/wm)

例如我的配置文件中少写了一个分号来结尾：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1862timestamp1510038097946.png/wm)

通过这样的方式可以在启动失败的情况下知道如何去调试，和错误的位置，以及如何去修正相关的错误。

这样便是成功启动的状态：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1862timestamp1510038290513.png/wm)

图中的 DHCP stop 失败（fail）是因为使用的是 restart 操作，因为之前还没有启动成功，所以 stop 就会失败。

### 3.4 启动检验

除了在启动服务的时候判断是否成功启动，我们还可以通过查看端口的方式得知是否能够成功启动：

```bash
netstat -lunat|grep 67
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1862timestamp1510038480589.png/wm)

因为 dhcp 默认使用的是 udp 的 67 端口，所以这里 grep 的是 67 端口，当然若是修改了服务的端口，就直接 grep 其对应的端口即可。

我们还可以通过查看当前运行中的进程可以知道 dhcp 有没有正常的启动：

```bash
ps -ef |grep dhcp
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1862timestamp1510038772834.png/wm)

还有一种情况就是你查看到端口，进程也存在，但是客户端无论如何都连接不上，这种情况可能是防火墙的问题了。

若是在本地没有问题，但是在远程失败，你可以检查一下 ufw 或者 iptables 或者你所配置的安全组，亦或者是在 dhcp 配置文件中配置的 allow 与 deny 黑白名单。

配置并启动 DHCP 服务器操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week4/1-1.mp4
@`

### 3.5 配置 DHCP 客户端

以下部分并不能在实验环境中得到验证，建议大家在本地启动两个虚拟机尝试。

若是远程的客户端需要使用 dhcp 来获取 IP 地址，只需要修改 interfaces 文件：

若是 centos 的用户则这样操作：

```bash
$ sudo vim /etc/sysconfig/network-scripts/ifcfg-eth0
```

修改成：

```bash
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=dhcp
```

若是 ubuntu 的用户则这样操作：

```bash
$ sudo vim /etc/network/interfaces
```

配置成 DHCP 的配置方式即可：

```bash
auto  eth0
iface eth0 inet dhcp
```

保存并退出，重启网络服务

```bash
$ sudo service networking restart 

# 或者重启网卡

$ ifdown eth0
$ ifup eth0
```

配置 DHCP 客户端介绍视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week4/1-2.mp4
@`

## 4. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

+ DHCP 原理
+ DHCP 服务器搭建

本实验只是带领搭建配置了最简单的 DHCP，还有很多的功能、配置选项我们并没有用到，若是有相关需求的同学可以查看推荐阅读中的官方文档。