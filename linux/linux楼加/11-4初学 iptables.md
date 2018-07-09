---
show: step
version: 1.0
enable_checker: true
---
# 初学 iptables

## 注意

iptables 及 SELinux 的实验将使用 CentOS 云主机进行。不同于实验楼先前实验中的 Docker 容器，CentOS 云主机停止实验之后的回收时间比较长（大概10分钟），所以希望大家尽可能避免频繁的开启和停止实验，否则会造成资源的浪费从而造成楼+同学的“资源不足”的报错，如果遇到“资源不足”的情况可以稍等几分钟再次尝试。

## 1 实验介绍

#### 1.1 实验内容

在 `Linux` 安全系统中 `iptables` 十分重要，在往后发展过程中其地位也显得越来越重要。本实验将带大家初识 `iptables`。逐步讲解 `iptables` 的发展、基础知识和基本语法以及在实际中的运用。

#### 1.2 实验知识点

+ `iptables` 的发展
+ `iptables` 的基础知识
+ `iptables` 的基本语法
+ `iptables` 的运用

## 2 `iptables` 的发展

#### 2.1 发展过程

1. `Linux` 内核 1.x：`ipfirewall`
2. `Linux` 内核 2.0：`ipfwadm`
3. `Linux` 内核 2.2：`ipchains`
4. `Linux` 内核 2.4：`iptables`

**`Linux` 内核1.x 时代**，是一个作者从 `freeBSD` 上移植过来的，能够工作在内核当中的，对数据包进行检测的一款简易访问控制工具。

**`Ipfwadm`** 是管理 Linux 内核并提供 `IP accounting` 和防火墙服务的工具。

**`Ipchains` 相较于 `ipfwadm` 的优势：**

+ `Qos（Quality of service）`的支持

+ `ipchains` 是树形结构的链，而 `ipfwadm` 是线性的结构，树形结构能够在自己拥有链的情况下再去接受其他跳转过来的链

+ `ipchains` 比 `ipfwadm` 在配置上更加的灵活

+ `ipchains` 可以明确地过滤任何的 `IP` 协议，不仅仅是 `TCP`、`UDP`、`ICMP`

**`iptables` 相较于 `ipchains` 的优势：**

+ 可以追踪有状态的 `IPV4` 的协议与应用
+ 可以追踪有状态的 `IPV6` 的协议
+ 能够做到 `NAT` 的一对多与多对多
+ 内建的 `PORTFW` 功能

#### 2.2 `Netfilter/Iptables`

通常我们所说的 `Linux` 防火墙是指 `Linux` 内核集成的 `IP` 信息包过滤系统—— **`Netfilter`/`iptables`** 。

+ Netfilter：主要是由内核模块来实现，工作在内核空间，是 `Linux` 核心中的一个通用架构。在这个框架上实现包过滤、NAT（网络地址转换）等模块功能，采用这样的模块化设计，使其拥有更好的扩展性和较强的灵活性。

+ Iptables：主要是一个上层的操作工具，工作在用户空间，提供一系列的表（tables）、每个表中有若干链（chain），每条链中由一条或多条规则（rule）组成，这些表包含内核用来控制信息包过滤处理的规则集。但真正执行过滤规则的是 `Netfilter` ，而 `iptables` 则是工作在其之上，作为用户的一个编写规则的工具。

如图所示：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081470275678415-wm)
（此图来源于：[维基百科](https://en.wikipedia.org/wiki/`Netfilter`)）

## 3 `iptables` 的基础知识

下面我们将会学习`iptables` 的基础知识。

### 3.1 结构图

`Netfilter` 所设置的规则是存放在内核中的，而 `iptables` 通过 `Netfilter` 放出的内核接口对存放在内核中的 `Netfilter` 配置表进行修改。

我们知道 `iptables` 是由一系列的表（`tables`）、每个表中有若干链（`chain`）以及每条链中由一条或多条规则（`rule`）组成的，最常用到的就是**三表五链**。

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081470275751536-wm)
此图片来源于<http://byrev.space/free/wp-content/uploads/sites/3/iptables-linux.png>

`iptables` 的结构主要是由 `tables`、`chain` 和 `target` 组成。

#### 3.1.1 **tables**

| tables   | 说明                                 | 支持的链                        |
| -------- | ------------------------------------ | ------------------------------- |
| `filter` | 默认的表，包含过滤规则               | INPUT、FORWARD、OUTPUT          |
| `NAT`    | 包含源、目的地址和端口转换使用的规则 | PREROUTING、OUTPUT、POSTROUTING |
| `mangle` | 设置特殊的数据包路由标志的规则       | 五个链都可以                    |
| `raw `   | 实现数据跟踪，是新增的表             | PREROUTING、OUTPUT              |

以上这些表具有一定的优先级顺序：

> filter < NAT < mangle < raw

#### 3.1.2 **chains**

每个表中的 `chains` 不尽相同，而 `chains` 是 `Netfilter` 框架中制定来对数据包的 `Hook Point`，其中 Hook Point 会在一个数据包通过网卡流经系统内核相应位置时对数据包的流向做出一定的修改，在系统上主要存在 `5` 个 `Hook Point` 的挂载点，如下。

| chains        | describe                                                     |
| ------------- | ------------------------------------------------------------ |
| `PREROUTING`  | 用于路由判断前的规则，如修改目的地址（DNAT）                 |
| `INPUT`       | 数据包通过路由计算判断为本地的 `Linux` 系统则通过此链的检查  |
| `OUTPUT`      | 用来针对所有本地生成的包                                     |
| `FORWARD`     | 两个网络连接时，用于传递数据包，两个网络间的数据包必须流经该防火墙 |
| `POSTROUTING` | 用于路由判断后的规则，如修改源地址（SNAT）                   |

#### 3.1.3 **target**

`target` 中的规则大部分是通用的，当然也有部分是特定使用的，这里列举常用的几种规则，更多的规则大家可以通过使用 `man` 来查看。

| target       | describe                                            |
| ------------ | --------------------------------------------------- |
| `ACCEPT`     | 满足匹配条件就接受数据包                            |
| `DROP`       | 满足匹配条件就丢弃数据包                            |
| `REJECT`     | 和 DROP 相似，拦截数据包，并返回错误信息            |
| `SNAT`       | 源网络地址转换                                      |
| `DNAT`       | 目的网络地址转换                                    |
| `MASQUERADE` | 和 SNAT 的作用相同，区别在于它不需要指定 –to-source |
| `REDIRECT`   | 满足匹配条件就将转发数据包到另一个端口              |
| `MIRROR`     | 颠倒 `IP` 头部中的源目的地址，然后再转发包          |

### 3.2 表（tables）

#### 3.2.1 filter 表

filter 表主要用于对数据包的过滤，是默认的一个表，如果没有指定哪个表，iptables 就默认使用 filter 表来执行所有命令。它是包含了真正的过滤规则。规则链主要是：`INPUT、FORWARD、OUTPUT`。

> filter 表在内核中是调用 `iptables_filter` 这个模块（模块间相互独立），该模块的初始化在 `net/ipv4/netfilter/iptable_filter.c –>iptable_filter_init` 中。

可以大致查看一下
```
static struct nf_hook_ops ipt_ops[] __read_mostly = {
    # 这是注册的 INPUT 的链
    {
        .hook        = ipt_local_in_hook,
        .owner        = THIS_MODULE,
        .pf        = NFPROTO_IPV4,
        .hooknum    = NF_INET_LOCAL_IN,
        .priority    = NF_IP_PRI_FILTER,
    },
    # 这是注册的 FORWARD 的链
    {
        .hook        = ipt_hook,
        .owner        = THIS_MODULE,
        .pf        = NFPROTO_IPV4,
        .hooknum    = NF_INET_FORWARD,
        .priority    = NF_IP_PRI_FILTER,
    },
    # 这里注册的是 OUTPUT 的链
    {
        .hook        = ipt_local_out_hook,
        .owner        = THIS_MODULE,
        .pf        = NFPROTO_IPV4,
        .hooknum    = NF_INET_LOCAL_OUT,
        .priority    = NF_IP_PRI_FILTER,
    },
};
```

最终的调用代码如下：

```
/* Returns one of the generic firewall policies, like NF_ACCEPT. */
unsigned int
ipt_do_table(struct sk_buff *skb,
         unsigned int hook,
         const struct net_device *in,
         const struct net_device *out,
         struct xt_table *table)
```

以上只是说明了 filter 在初始化时注册的三个 Hook point ，本实验不做深入研究，若有兴趣的同学可以在 [ netfilter 官网](http://www.netfilter.org/)下载源码进行修改研究。

#### 3.2.2 NAT 表

**NAT 表**（Network Address Translation，网络地址转换）：就是一种将内部网络的 `IP` 地址转换为合法的公网 `IP` 地址的技术。主要是修改数据包报头的 `IP` 地址、端口号等信息，实现数据包的伪装、平衡负载、端口转发以及透明代理。

**规则链**主要是：PREROUTING 链、OUTPUT 链、POSTROUTING 链。

NAT 主要有三种类型：

+ **静态 NAT**（static NAT）：IP 地址在转换时是一对一的关系。

+ **动态 NAT**（dynamic NAT 或者叫 pooled NAT）： `IP` 地址在转换时是多对多的关系。

+ **NAPT**（Network Address Port Translation）：网络地址端口转换在 `IP` 地址的层面上是多对一的关系。

NAPT 是使用较为普遍的转换方式，还可以细分为：

+ **源 NAT**（Source NAT，SNAT）：修改数据包的源地址。源 NAT 改变数据流的第一个数据包的来源地址，数据包伪装就是一个 SNAT 的例子。

> SNAT：若数据包是被送往 POSTROUTING 链的，同时匹配了规则，则执行 SNAT 或 MASQUERADE 目标。系统在决定了数据包的路由之后就执行该链中的规则。

+ **目的 NAT**（DNAT，Destination NAT）：修改数据包的目的地址。它是改变第一个数据包的目的地址，如平衡负载、端口转发和透明代理就是属于 DNAT。

> DNAT：若数据包是被送往 PREROUTING 链，并匹配了规则，则执行 DNAT 或 REDIRECT 目标。为了使数据包得到正确路由，必须在路由之前进行 DNAT。

**工作原理**

> Nat 的初始化工作和 filter 基本一样。Nat 的 ipv4 部分在 `Iptables_nat.c` 、Core 部分在 `nf_nat_core.c`，不同的就是表不一样。

先看一下 `IP` 包的结构，如下图：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4030timestamp1510826570747.png/wm)

在 `IP` 数据包中，都会有源地址（Source ip address）和目的地址（Destination ip address）两个字段，数据包经过的路由器就是根据这两个字段来判定数据包由什么地方发过来，该发往何处。

iptables 中的 DNAT 和 SNAT 的原理和这个类似，下图是可靠数据包在 `iptables` 中经过的链（chains）：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4030timestamp1510826582700.png/wm)

如图所示，图中的菱形部分就是对数据包进行判定转发的地方。如果目的地址是本机地址，数据被转交给 INPUT 链，反之转交给 FORWARD 链。

**举例**

例如，做 DNAT 就是在 PREROUTING 链中，把访问 192.168.42.1 的访问转发到 192.168.0.2 上：

```
iptables -t nat -A PREROUTING -d 192.168.42.1 -j DNAT --to-destination 192.168.0.2
```

而 SNAT 是在数据包流出这台机器之前在 POSTROUTING 链进行操作。

```
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to-source 192.168.42.3
```

这个语句就是告诉系统把即将要流出本机的数据的 `source ip address` 修改成为 192.168.42.3。这样，数据包在达到目的机器以后，目的机器会将包返回到 192.168.42.3 也就是本机。如果不做这个操作，那么你的数据包在传递的过程中，reply 的包肯定会丢失。

*后面会详细讲解 `iptables` 的语法结构*

#### 3.2.3 mangle 表

mangle 表：主要用于修改数据包的 TOS（Type Of Service，服务类型，根据不同的服务质量来选择经过路由的路径）、TTL（Time To Live，生存周期，每经过一个路由器将减 1，mangle 可以修改此值设定 TTL 要被增加的值，这个选项可以使我们的防火墙更加隐蔽，而不被 trace-routes 发现等等）以及为数据包设置 Mark 标记（特殊标记，用来做高级路由，以使不同的包能使用不同的队列要求等等），Qos(Quality Of Service，服务质量)调整以及策略路由等应用，由于 TOS，Qos 类似的方式需要相应的路由设备支持，所以应用并不广泛。（这里不做过多的讲解）

这个表中包含五个规则链：PREROUTING，POSTROUTING，INPUT，OUTPUT，FORWARD。

#### 3.2.4 raw 表

raw 表： 是自 1.2.9 版本以后 `iptables` 新增的表，主要用于决定数据包是否被状态跟踪机制处理。在匹配数据包时，raw 表的规则要优先于其他表。
包含两条规则链：OUTPUT、PREROUTING。

### 3.3 `iptables` 的状态（status）

`iptables` 中数据包被跟踪连接有 4 种不同状态：

+ NEW ：数据包开始一个新连接（重新连接或将连接重定向）
+ RELATED ：数据包是基于某个已经建立的连接而建立的新连接。
+ ESTABLISHED ：只要发送并接到应答，一个数据连接就从 NEW 变为 ESTABLISHED ，而且该状态会继续匹配这个连接的后续数据包。
+ INVALID ：数据包不能被识别属于哪个连接或没有任何状态的，比如内存溢出，收到不知属于哪个连接的 ICMP 错误信息，一般会 DROP 这个状态的任何数据。

## 4 `iptables` 的基本语法

下面我们将会学习`iptables` 的基本语法。

### 4.1 语法格式

```
iptables [-t tables] COMMAD chains CRETIRIA -j target
```

说明：

+ -t table ：4 个 tables：filter、nat、mangle、raw
+ COMMAND：执行操作
+ chain：定义规则在链上的操作
+ CRETIRIA：指定匹配标准
+ -j target：指定进行数据包处理方式

*`iptables` 的语法规则比较详细，在书写时一定要规范，每个参数都要明确！*

### 4.2 常用 COMMAND

| COMMAND | describe                             |
| ------- | ------------------------------------ |
| `-A`    | 新增规则，在当前链的最后新增一个规则 |
| `-I`    | 插入规则，把当前规则插入为第几条     |
| `-R`    | 替换/修改第几条规则                  |
| `-D`    | 删除第几条规则                       |
| `-P`    | 设置默认策略                         |
| `-L`    | 显示指定表和指定链的规则             |

**举例：清除已有 `iptables` 规则**

```
# 清除规则链中已有的条目
$ iptables -F

# 清除用户自定义的空链
$ iptables -X

# 清空链和链中默认规则的计数器
$ iptables -Z
```

### 4.3 CRETIRIA

| CRETIRIA  | describe                 |
| --------- | ------------------------ |
| `-i`      | 数据包进入本机的网络接口 |
| `-o`      | 数据包离开本机的网络接口 |
| `-p`      | 匹配数据包的协议类型     |
| `-s`      | 匹配数据包源 `IP` 地址   |
| `-d`      | 匹配数据包目的 `IP` 地址 |
| `-j`      | 指定跳转的目标           |
| `--sport` | 源端口号                 |
| `--dport` | 目标端口号               |

iptables命令选项输入顺序：

> `iptables` -t 表名 <-A/I/D/R> 规则链名 [规则号] <-i/o 网卡名> -p 协议名 <-s 源IP/源子网> --sport 源端口 <-d 目标IP/目标子网> --dport 目标端口 -j 动作

iptables 基本语法操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week3/6-1.mp4
@`

## 5 `iptables` 的运用

下面我们将会学习`iptables` 的运用。

### 5.1 查看当前 `iptables` 的规则

```
$ sudo iptables -nL
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513390489810.png/wm)

因为该环境下还未设置 iptables 的规则，所以看到的是一个空的。

```
$ sudo iptables -nvL --line
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513390512008.png/wm)

+ 第一列： num 显示了该规则在该链中的顺序位置
+ 第二列： target 显示了该规则所做的行为
+ 第三列： port 匹配的端口
+ 第四列： opt 是 TCP 协议头部中 options 的一部分，并不是重点，我们可以不必关注，有兴趣的也可以通过[维基百科](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Protocol_operation)深入了解
+ 第五列、第六列： source、destination 表示对包中分析得出的数据源地址与数据的目的地址的匹配

### 5.2 实例一

规则：

+ 对所有地址开放本机的 TCP （80，22，1-20）端口
+ 允许所有地址开放本机基于 ICMP 协议的数据包

主机地址：

```
$ ifconfig
```

设置规则：

```
sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT
 
sudo iptables -I INPUT -p tcp --dport 22 -j ACCEPT

sudo iptables -I INPUT -p tcp --dport 1:20 -j ACCEPT
 
sudo iptables -I INPUT -p icmp -j ACCEPT

```

验证查看规则

```
sudo iptables -L
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513390925069.png/wm)

从输出可看到我们添加的规则已经写入了 iptables 的规则中。

### 5.3 实例二

下面这个例子是将所有访问 80 端口的数据包都丢掉。

**实验准备**

1. 实验环境未安装 httpd 服务所以需要安装一下，然后再启动服务

```
sudo yum install httpd -y
sudo service httpd start
```

2. 验证 80 端口的开放性

```
sudo netstat -lunpt | grep 80
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513391240550.png/wm)

可以看到 `80` 端口已经开放。


3. 尝试查看能否访问该页面，若返回值为 200 则为成功访问。这里虽然报 403 forbidden，但是可以看到一些返回信息，80 端口是可以连接的。


```
curl -I localhost
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971522307832009-wm)


**添加规则**

```
# 将所有访问 80 端口的数据包都丢掉
sudo iptables -t filter -I INPUT -p tcp --dport 80 -j DROP

# 只是查看 INPUT 链是否加入了这条规则
sudo iptables -nvL INPUT

# 再来尝试，能否查看到输出信息
curl localhost

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513391412641.png/wm)

从截图可以看到没返回，这是因为 DROP 将所有来访问该页面的数据包丢掉，不给任何返回和响应，而 curl 一直等待着服务端的响应。

若将 DROP 行为改成 REJECT 的话，就会有返回信息说这个端口无法访问。

```
# 删除之前添加的规则，这里没有添加 `-t` 的参数
# 是因为不指明该参数，系统默认为修改指向 filter 表
sudo iptables -D INPUT -p tcp --dport 80 -j DROP

#添加拒绝的规则
sudo iptables -I INPUT -p tcp --dport 80 -j REJECT

#再次验证
curl -I localhost
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513391341572.png/wm)

通过上面的例子可以看到 `iptables` 已经修改就会立即生效。

还可以通过另一种方式来查看已经生效的规则，这种方式查看的是直接插入的规则：

```
$ sudo iptables-save
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513390778343.png/wm)

当然 `iptables-save` 有一个参数 `-t` 可以指定要保存的表，使用重定向可以将输出保存到某个文件中。

```
#在执行该命令之前请先用sudo su 切换到 root 用户

sudo iptables-save -t filter > filter.bak

cat filter.bak
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513390872289.png/wm)

另外，还有一个 iptables-restore 命令可以将保存到文件中的规则重新恢复到系统中。大家可以在实验环境中尝试下。

iptables 运用实例操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week3/6-2.mp4
@`

## 6 总结

通过本节实验的学习，可以大致了解到 `iptables` 的基本概念以及相应的一些运用，由于篇幅的原因，更多深入的知识可以在课后继续探讨研究。