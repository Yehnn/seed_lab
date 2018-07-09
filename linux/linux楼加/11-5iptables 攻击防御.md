---
show: step
version: 1.0
enable_checker: true
---
# iptables 攻击防御

## 1 实验介绍

#### 注意

iptables 及 SELinux 的实验将使用 CentOS 云主机进行。不同于实验楼先前实验中的 Docker 容器，CentOS 云主机停止实验之后的回收时间比较长（大概10分钟），所以希望大家尽可能避免频繁的开启和停止实验，否则会造成资源的浪费从而造成楼+同学的“资源不足”的报错，如果遇到“资源不足”的情况可以稍等几分钟再次尝试。

#### 1.1 实验内容

前面一节实验我们认识了 iptables ，本节实验将继续学习 iptables 的一些实际场景运用，其中包括了：DDOS 和 SYN 攻击的认识，以及在网络开放之后可能带来的攻击和如何去防御。

#### 1.2 实验知识点

+ DDOS
+ SYN 攻击
+ CC 攻击

#### 1.3 推荐阅读

+ [DDOS wiki](https://en.wikipedia.org/wiki/Denial-of-service_attack#.28S.29SYN_flood/)

+ [TCP](http://alpha.tmit.bme.hu/meresek/lantcp_eng.htm)

## 2 DDOS

下面我们将会学习DDOS。

### 2.1 DDOS 概述

早期的拒绝服务（`DOS，denial-of-service`）攻击，试图消耗一台机器或网络的资源，通常用 `flood` 攻击有针对性的尝试对目标机器或资源的多余请求，使得系统过载，因而阻止了一些或所有的合法请求被满足。

后来随着硬件和网络的发展成为了分布式拒绝服务(`DDoS，Distributed Denial of Service`)攻击，主要是借助于客户/服务器技术，将多个计算机联合起来作为攻击平台，对一个或多个目标发动 DDoS 攻击，从而成倍地提高拒绝服务攻击的威力。

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081470364868978-wm)

（图片来源于：<https://en.wikipedia.org/wiki/Denial-of-service_attack#.28S.29SYN_flood>）

可以用这样一个例子来理解下 DDOS 攻击：

1. 有一个能够容纳 100 人就餐的饭店，有一天对面竞争商家雇佣了 200 人来这个饭店只坐着不消费，结果就是该饭店无法正常营业。（`DDOS 攻击`）

2. 这个时候老板就叫了一帮人来把这 200 个人赶了出去，饭店又恢复正常。（`进行 DDOS 防御`）

3. 竞争商家不服输，再次派出了 2000 个道上的人来逐批次来捣乱，饭店再次无法营业。（`增加 DDOS 流量，改变攻击方式`）

4. 饭店老板不好得罪就将营业规模扩大，这样就可以同时容纳 5000 人就餐，这时就算 2000 人同时来捣乱饭店也还可以继续营业。（`增强防御`）

### 2.2 DDOS 攻击

#### 2.2.1 攻击方式

+ **Ping flood** :攻击者发送压倒性的数量的 ping 包给受害者。

+ **Ping of Death**：攻击者发送修改后的 ping 数据包，如添加分包使前后的逻辑不正确，或者加长数据包使其超过 IP 报文的限制。

+ **Teardrop attacks**：向目标机器发送损坏的 IP 包，诸如重叠的包或过大的包载荷。

+ **UDP flood**：利用大量 UDP 小包冲击服务器。

+ **SYN flood**：利用 TCP 的连接过程，来消耗系统资源。

+ **CC（challenge Collapsar）**：通过构造有针对性的、最大消耗服务器端资源的业务请求，让服务器“劳累过度”而停止服务等。

#### 2.2.2 攻击现象

+ 被攻击主机上有大量等待的 TCP 连接

+ 网络中充斥着大量无用的数据包

+ 源地址为假，制造高流量无用数据，造成网络拥塞，使受害主机无法正常和外界通讯

+ 利用受害主机提供的传输协议上的缺陷反复高速的发出特定的服务请求，使主机无法处理所有正常请求

+ 严重时会造成服务器频繁死机，重启

+ 网络拥塞，访问网页速度缓慢

+ 服务器 CPU 长期处于满负荷状态

#### 2.2.3 攻击防护措施

+ 主机防护措施

    + 关闭不必要的服务及端口
    + 同一时间段内限制打开的 syn 半连接数量
    + 缩短 syn 半连接的超时时间
    + 及时安装修补系统补丁

+ 网络防护措施

    + 禁止对主机非开放服务的访问
    + 限制同时打开的 syn 最大连接数
    + 限制特定的 IP 地址的访问
    + 启用防火墙的防 DDoS 的属性
    + 严格限制对外开放的服务器的向外访问

#### 2.2.4 历史攻击案例

+ 2000 年 2 月，包括雅虎、CNN、亚马逊、eBay、ZDNet，以及 E*Trade 和 Datek 等网站均遭受到了 DDOS 攻击，并致使部分网站瘫痪。
+ 2007 年 5 月，爱沙尼亚三周内遭遇三轮 DDOS 攻击,总统府、议会、几乎全部政府部门、主要政党、主要媒体和2家大银行和通讯公司的网站均陷入瘫痪，为此北约顶级反网络恐怖主义专家前往该国救援。
+ 2009 年 519 断网事件导致南方六省运营商服务器全部崩溃，电信在南方六省的网络基本瘫痪。2009年 7 月，韩国主要网站三天内遭遇三轮猛烈的 DDOS 攻击，韩国宣布提前成立网络司令部。
+ 全球三大游戏平台：暴雪战网、Valve Steam 和 EA Origin 遭到大规模 DDoS 攻击，致使大批玩家无法登录与进行游戏。随后名为 DERP 的黑客组织声称对此次大规模的 DDoS 攻击行动负责。

### 2.3 防御 DDOS 攻击脚本实例

这里分享一个用 **shell 脚本**来定时处理防止服务器受到 DDOS 的攻击的方法，避免人工添加的麻烦。(脚本参考[散尽浮华博客](https://www.cnblogs.com/kevingrace/p/6756515.html))

```
vim dropip.sh # 自动提取攻击 ip
```

```bash
#!/bin/bash
netstat -na | awk '/ESTABLISHED/{split($5,T,":");print T[1]}' | sort | grep -v -E '192.168|127.0' | uniq -c | sort -rn | head -10 | awk '{if ($2!=null && $1>4) {print $2}}' > /var/log/rejectip

for i in $(cat /var/log/rejectip)
do
    rep=$(iptables-save | grep $i)
    if [[ -z $rep ]];then
        /sbin/iptables -A INPUT -s $i -j DROP
        echo "$i kill at `date`">>/var/log/ddos-ip
    fi
done
```

脚本的执行过程：

- 首先通过 `netstat -na` 查看所有的连接
- 然后通过 `awk` 提取所有已经成功建立连接的远程 IP 地址（也就是疑似攻击者的 IP）
    + awk 只对拥有 `ESTABLISHED` 关键字的行做处理
    + split 指定处理第五列（也就是远程 IP 的列），将处理的数据存放入 T 数组中，指定以 `:` 来分隔
- 接着通过 `sort` 排序，此时会将相同的 IP 放在一起（因为 uniq 统计的时候是按行处理的，分开的相同行处理不到）
- 我们将本地的连接剔除统计
- 随后通过 `uniq` 命令对相同的 IP 做统计，统计出其出现的次数
- 再通过 `sort` 命令做反向的排序（ `-r` 参数出现次数高的排前，低的排后）
- 使用 `head` 命令来读取前十个 IP 地址
- 随后我们再次使用 `awk` 过滤出连接数超过 4 次的 IP 地址
- 然后将过滤出来的 IP 地址重定向至 `/var/log/rejectip` 中
- 使用 for 循环读取文本中的 IP 地址
  + 首先检查这个 IP 地址是否已经加入了 iptables 的规则中
    + 若是已经加入则不处理
    + 若是还未加入则使用 iptables 将其 drop 掉并记录至日志中

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4031timestamp1511168578517.png/wm)

**添加脚本的执行权限**

```
$ sudo chmod +x dropip.sh
```

**添加计划任务，每分钟执行一次**(注意记得切换成 root 添加该 crontab)

```
$ crontab -e
*/1 * * * * /home/shiyanlou/dropip.sh
```

查看到执行任务命令之后在日志中的信息反馈

```
sudo tail /var/log/cron
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513392273067.png/wm)

DDoS 及防御操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week3/7-1.mp4
@`

## 3 SYN 攻击

下面我们将会学习 SYN 攻击。

### 3.1 SYN 原理

`SYN` 攻击属于 DDOS 攻击的一种，主要利用了 TCP 协议缺陷，**通过发送大量的半连接请求，耗费服务器 CPU 和内存资源**。SYN 攻击除了能影响主机外，还可能危害路由器，防火墙等网络系统，事实上 SYN 攻击并不管目标是什么系统，只要这些系统打开 TCP 服务就可以实施。

SYN 攻击是最常见又最容易被利用的一种攻击手法，在大部分黑客攻击事件中，他们就是利用这种简单的攻击手法进行的，而 SYN 攻击主要就是**依靠 TCP 的三次握手**。

*（如果忘记了三次握手概念下前简单回顾一下）*

#### 3.1.1 回顾 TCP 三次握手

在 TCP/IP 协议中，TCP 协议提供可靠的连接服务，采用三次握手建立一个连接。

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081470292587273-wm)
（该图片来源于<http://alpha.tmit.bme.hu/meresek/twh_small.jpg>）

**第一次握手**：建立连接时，客户端发送 SYN 包(syn = j)到服务器，并进入 SYN_SEND 状态，等待服务器确认。

**第二次握手**：服务器收到 syn 包，必须确认客户的 SYN（ack = j+1），同时自己也发送一个 SYN 包（syn=k），即 SYN+ACK 包，此时服务器进入 SYN_RECV 状态。

**第三次握手**：客户端收到服务器的 SYN＋ACK 包，向服务器发送确认包 ACK(ack=k+1)，此包发送完毕，客户端和服务器进入 ESTABLISHED 状态，完成三次握手。

完成三次握手，客户端与服务器开始传送数据。

#### 3.1.2 SYN flood

上面是一次完整的 TCP 连接过程。
如果客户端发送 SYN 的请求连接信号，服务器从关闭状态变为监听，同时响应了请求，再发送回确认信息以及请求信号，并且将该信息加入到未连接队列中，这样就变成了 `SYN_RCVD `状态。
如果客户端不做处理，并不予以响应，那么服务器就会一直等待，当等待超时才会将此信息从队列中移除，一旦发生这种情况就会造成资源的耗费，服务器建立连接时 TCP 的状态变化过程是：**LISTEN -> SYN RECV**，这种连接通常称之为半连接 。当这样的不正常连接过多就会造成正常用户无法连接，而大量这种连续的不正常行为就被称为 SYN flood （**SYN 洪水攻击**）。如下图：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081470294625485-wm)

### 3.2 SYN 模拟攻击实例

SYN flood 是攻击者利用 TCP 在建立连接时使用 SYN 的请求发送，那么就可以利用 iptables 做出相应的限制，来防止一个 ip 发出过多的请求。

**1.使用 apache 的 `ab` 工具来模拟高并发的请求**

```
# 安装 httpd Web 服务器
sudo yum install httpd
# 重启 httpd 服务，提供访问
sudo service httpd restart
```

如果你是接着上一个“初学 iptables ”实验的话，需要清除之前定义的一个规则（将 80 端口的访问数据包都丢掉）。因为在下面的实验中会对 80 端口进行访问，所以需要执行下面的命令来恢复到默认状态，避免出错。

```
# 规则链中已有的条目
sudo iptables -F
```

**2.查看当前连接数**

```
sudo su
netstat -an | grep SYN
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4031timestamp1511168738499.png/wm)

**3.使用压力测试**

使用 ab 工具来模拟高并发的请求。

```
ab -n 10000000 -c 600 http://localhost/index.html

# 这是模拟发送 1000 万个请求，同时并发量为 600 个，访问本地的 apache 首页
# 这样的访问并没有查询数据库这么消耗资源，这仅仅只是模拟
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4031timestamp1511168755599.png/wm)

**打开一个新的终端！**

**4.再次查看连接数**

```
netstat -an | grep SYN
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4031timestamp1511168805870.png/wm)

**5.增加 iptables 规则，并发送 10 个包**

```
#使用 limit 对 syn 的每秒钟最多允许 1 个新链接，接收第三个数据包的时候触发
iptables -A INPUT -p tcp --syn -m limit --limit 1/s    --limit-burst 3 -j ACCEPT

#或者针对每个客户端做出限制，每个客户端限制并发数为 10 个，这里的十个只是为了模拟，可以自己酌情考虑
iptables -I INPUT -p tcp --syn --dport 80 -m connlimit --connlimit-above 10 -j REJECT

ab -n 10000000 -c 600 http://localhost/index.html
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971522310957457-wm)

可以看到发送包后，无法再发送了。

### 3.3 SYN 防范配置文件

对于这样类型的攻击，不仅可以从 iptables 来防范，还可以**修改 TCP 的一些配置**来进行防范。

```
vim /etc/sysctl.conf

# 表示开启 SYN Cookies。当出现 SYN 等待队列溢出时，启用 cookies 来处理，可防范少量 SYN 攻击，默认为 0，表示关闭
net.ipv4.tcp_syncookies = 1

# 表示开启重用。允许将 TIME-WAIT sockets 重新用于新的 TCP 连接，默认为 0，表示关闭；
net.ipv4.tcp_tw_reuse = 1

# 表示开启 TCP 连接中 TIME-WAIT sockets 的快速回收，默认为 0，表示关闭。
net.ipv4.tcp_tw_recycle = 1

# 表示开启 TCP 连接的最大数量的限制
net.ipv4.tcp_max_tw_buckets = 5000
```

注：当启用 tcp_syncookies 时，内核生成一个特定的值，而不把客户的连接放到半连接的队列里。当客户端提交第三次握手的 ACK 包时，linux 内核取出 n 值，进行校验，如果通过，则认为这个是一个合法的连接。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4031timestamp1511168864579.png/wm)

SYN 攻击及防范操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week3/7-2.mp4
@`

## 4 CC 攻击

下面我们将会学习 CC 攻击。

### 4.1 CC 原理

**CC 攻击(ChallengeCollapsar)**，它的原理就是攻击者控制某些主机不停地发大量数据包给对方服务器造成服务器资源耗尽，一直到宕机崩溃。CC 主要是用来攻击页面，当一个网页访问的人数特别多的时候，打开网页就很慢。而 CC 就是模拟多个用户（多少线程就是多少用户）不停地进行访问那些需要大量数据操作的页面，造成服务器资源的浪费，CPU 长时间处于 100%，一直都有处理不完的连接直至就网络拥塞，正常的访问被中止。

相对于 SYN flood 来说，CC 攻击就是做一次完整的请求，属于 TCP 全连接攻击，攻击者使用“合法”的源 IP 与访问请求，Collapsar 无法判断真假只好放过，但是整个请求非常的消耗资源，如翻阅数据库呀，不停的翻页等等，一次的连接很长，消耗很多的资源的。

### 4.2 CC 攻击防御方法

+ **域名欺骗解析**

发现针对域名的 CC 攻击，可以把被攻击的域名解析到 127.0.0.1 这个地址上。因为 127.0.0.1 是本地回环 IP 是用来进行网络测试的，如果把被攻击的域名解析到这个 IP 上，就可以实现攻击者自己攻击自己的目的。

+ **更改 Web 端口**

Web 服务器通常是通过 80 端口对外提供服务，因此攻击者实施攻击就以默认的 80 端口进行攻击，所以可以通过修改 Web 端口达到防止 CC 攻击的目的。

+ **IIS屏蔽 IP**

通过命令或查看日志来发现 CC 攻击的源 IP，就可以在 IIS 中设置屏蔽该 IP 对 Web 站点的访问，从而达到防范 IIS 攻击的目的。

*（这里 CC 攻击不做重点讲解，感兴趣的可以下来探讨研究）*

## 5 总结

通过本实验，我们可以了解到 iptables 的一些其他用法，以及比较常见的 DDOS 攻击、SYN 攻击和 CC 攻击的用法。
