---
show: step
version: 1.0
enable_checker: true
---
# iptables 脚本实例

**注意**

**实验楼的桌面就是用 VNC 在配置的时候要注意，建议切换到字符模式进行配置，避免桌面中断造成的影响。**
iptables 及 SELinux 的实验将使用 CentOS 云主机进行。不同于实验楼先前实验中的 Docker 容器，CentOS 云主机停止实验之后的回收时间比较长（大概10分钟），所以希望大家尽可能避免频繁的开启和停止实验，否则会造成资源的浪费从而造成楼+同学的“资源不足”的报错，如果遇到“资源不足”的情况可以稍等几分钟再次尝试。

## 1 实验介绍

#### 1.1 实验内容

本节实验主要是对 iptables 脚本编写的实践。主要讲解 iptables 编写脚本的默认规则，以及编写流程化的步骤和 iptables 脚本的实例。

#### 1.2 实验知识点

+ 脚本默认规则
+ 如何编写脚本
+ 脚本编写实例

## 2 Bash 脚本中使用 iptables

下面我们将会学习 Bash 脚本中使用 iptables。

### 2.1 iptables 规则设置

1. 默认规则

这些是由 iptables-save 命令获得的默认的规则，表示默认 INPUT 等 chain 都是 ACCEPT 的状态：

```
:INPUT ACCEPT [0:0]
# 表示 INPUT 链默认规则是 ACCEPT
:FORWARD ACCEPT [0:0]
# 表示 FORWARD 链默认规则是 ACCEPT
:OUTPUT ACCEPT [0:0]
# 表示 OUTPUT 表默认规则是 ACCEPT
```

> 注：[0:0]表示：[包计数器:字节计数器]

2.清除规则

```
# 清除规则链中已有的条目
sudo iptables -F
# 清除用户自定义的空链
sudo iptables -X
# 清空链和链中默认规则的计数器
sudo iptables -Z
```

3.禁止其他机器向该主机发送任何连接请求，即此时该主机不提供任何服务。

```
-A INPUT -m state --state NEW -j DROP
```

4.允许所有建立连接或相关的数据通过。（ESTABLISHED：已建立的连接状态。RELATED：该数据包与本机发出的数据包有关。）

```
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

5.查看主机上 iptables 的规则

```
$ sudo iptables -L
```

6.查看主机上已经生效的规则

```
$ sudo iptables-save
```

### 2.2 测试实例

根据前面我们学习到的默认规则可以简单编写一个 iptables 脚本，作为主机上一个默认的防火墙。

创建一个脚本文件

```
$ touch test.sh
$ vim test.sh
```

脚本如下：

```bash
#!/bin/bash

# 清除规则
iptables -F
iptables -X
iptables -Z

# 设置链的默认策略
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT


iptables -A INPUT -m state --state NEW -j DROP

iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

保存退出，设置可执行权限，运行脚本，最后查看 iptables 规则

```
sudo chmod +x test.sh
sudo ./test.sh
sudo iptables -nv -L
```

执行脚本后，此主机将拒绝新建数据的连接。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4045timestamp1511317945014.png/wm)

可以用  `telnet` 命令来验证一下。

```
telnet localhost 22
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4045timestamp1511319161024.png/wm)

iptables 默认规则操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week3/8-1.mp4
@`

## 3 如何编写脚本

体验了上面一个比较简单的脚本编写后，这里给大家讲解一下如何流程化的编写好一个 iptables 脚本，主要的步骤如下。

### 3.1 清除表的所有规则

在制定规则前往往需要删除现有的规则。

```
# 清除规则链中已有的条目
sudo iptables -F
# 清除用户自定义的空链
sudo iptables -X
# 清空链和链中默认规则的计数器
sudo iptables -Z
```

### 3.2 载入模块

编写脚本的时候没有用服务的方式来启动 iptables 时，就需要我们手动加载一些模块。
常用模块有：

```
sudo modprobe ip_tables # 启动 iptables
sudo modprobe iptable_nat # 使用 nat 表时载入
sudo modprobe ip_nat_ftp # 在出去的包做了伪装以后，就必须加载该模块，否则防火墙无法知道返回的包该转发到哪里
sudo modprobe ip_conntrack_ftp # 使防火墙能够识别 FTP 某类特殊的返回包
sudo modprobe ip_conntrack # 使防火墙具有连接跟踪能力
sudo modprobe ipt_MASQUERADE # 数据包伪装的作用
```

可以通过 `lsmod` 命令来查看加载的模块：

```
lsmod | grep ip
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4045timestamp1511260639177.png/wm)

### 3.3 设置默认策略

一般情况下搭建防火墙，都会建议拒绝一切数据包接入，往往会把链表的规则都设置为 `DROP`，实际中也可以根据需要按照如下进行配置。

```
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD DROP
```
打开 lo 回环，允许 127.0.0.1 这个虚拟网卡上的所有网络流入和流出流量

```
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT
```

补充：还可以添加前面讲到的默认策略。

```
sudo iptables -A INPUT -m state --state NEW -j DROP

sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

### 3.4 根据需求制定各项规则

默认规则制定完成以后，就可以根据实际需求来建立防火墙的相应规则。

例如 1： 允许服务器 `ping` 对方主机而不允许对方主机 `ping `服务器。

```
sudo iptables -A INPUT -p icmp -–icmp-type 8 -j DROP
sudo iptables -A OUTPUT -p icmp --icmp-type 8 -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type 0 -j ACCEPT
```

例如 2：开放主机对 `dns` 的访问.

```
sudo iptables -A INPUT -p udp --sport 53 -j ACCEPT
sudo iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
```

### 3.5 更改权限并执行

主要是对保存的脚本加上`x`可执行权限。

```
sudo chmod +x test.sh
```

以上 `5` 步就是编写 iptables 的大致步骤，某些部分可以根据实际情况进行删减。

iptables 脚本编写实践操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week3/8-2.mp4
@`

## 4 脚本编写实例

了解 iptables 默认策略等知识以及流程化编写 iptables 脚本的步骤，下面我们就来实践一下吧！

### 4.1 实例一

首先，我们制定一个主机上的安全措施的规则，主要是**防止 ping flooding 的发生**。

```
$ touch test1.sh
$ vim test1.sh
```

```bash
#!/bin/bash

iptables -F
iptables -F -t nat
iptables -X

iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

modprobe ip_conntrack
modprobe iptable_nat

iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 这条规则主要是防止 ping 泛洪，并且限制每秒的 ping 包不超过 10 个
iptables -A FORWARD -p icmp --icmp-type echo-request -m limit --limit 1/s --limit-burst 10 -j ACCEPT
```

> 说明：
> limit: 速率限制
> limit-burst: 设置默认阀值

```bash
chmod +x test1.sh
sudo ./test1.sh
sudo iptables -nv -L
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4045timestamp1511260725806.png/wm)

### 4.2 实例二

本实例主要制定一些我们常见的端口服务的规则。

```
$ touch test2.sh
$ vim test2.sh
```

```bash
#!/bin/bash

# 清除规则
iptables -F
iptables -X
iptables -Z

# 设定策略
iptables -P   INPUT DROP
iptables -P  OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

# 设置默认规则，打开环回
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# SSH 的端口，实现远程
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# FTP 服务器，开启 21 端口
iptables -A INPUT -p tcp --dport 21 -j ACCEPT
iptables -A INPUT -p tcp --dport 10000:20000 -j ACCEPT

# WEB 服务器,开启 80 端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

#减少不安全的端口连接
iptables -A OUTPUT -p tcp --sport 31337 -j DROP
iptables -A OUTPUT -p tcp --dport 31337 -j DROP
```

```
chmod +x test2.sh
sudo ./test2.sh
sudo iptables -nv -L
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4045timestamp1511260804831.png/wm)


## 5 总结

通过本节实验，可以加深对 iptables 的认识和了解，从实践中再次熟悉 iptables 的作用以及其重要性，由于 iptables 的语法较为复杂并且还作为防火墙中的一个重点，希望大家能够将基础夯实，一步一脚印。