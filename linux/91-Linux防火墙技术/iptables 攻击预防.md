---
show: step
version: 0.1
enable_checker: true
---

# iptables 攻击预防

## 一、实验介绍

在了解了 iptables 的结构与用法之后我们需要学会在特定的场景去应用他们，而不再是简单的纸上谈兵。本实验将带大家了解网络开放之后可能带来的攻击以及如何去预防

### 实验涉及的知识点

- DDOS 攻击
- SYN 攻击

## 二、DDOS 攻击

在计算中，拒绝服务（ DOS，denial-of-service ）攻击是试图消耗一台机器或网络的资源，例如去暂时或无限期中断甚至去暂停服务的主机连接到互联网，使其无法给予真正需求的用户。拒绝服务通常是通过 flood 攻击有针对性的尝试对目标机器或资源的多余请求，以使得系统过载，并防止一些或所有合法的请求。因为早期的服务器的性能和网络刚发展，性能并不高，所以由一台主机去不断地请求服务，就可以导致服务器应付不过来

后来硬件与网络飞速的发展，一台主机发动再多的请求，服务器也能够从容地应对，既然一台不够，那就成千上万台的来，所以有了分布式拒绝服务（DDoS，distributed denial-of-service）。

一个分布式拒绝服务（DDoS，distributed denial-of-service）攻击源往往不止一个，又往往成千上万个，并且都是真实唯一的IP地址。

![4-2](https://dn-simplecloud.shiyanlou.com/1135081470364868978-wm)

（图片来源于：<https://en.wikipedia.org/wiki/Denial-of-service_attack#.28S.29SYN_flood>）

它类似于一群人拥挤的堵在一家商店里面，就连门口都是人，而且这些人什么也不干，不买东西，无所事事，而真正需要买东西的人却堵在门口，进不去，这样便扰乱了商店或者企业的正常的运作。DDoS攻击的规模仍在不断增长

常见的可能受到 DDOS 攻击体现的症状有：

- 网络堵塞，打开网站的速度异常的缓慢，或者平时都能打开，却打不开了
- 服务器的 CPU 长期处于满负荷
- 服务器的频繁死机，重启

而常见的 DDOS 攻击有以下几种

- Ping flood :发送ping包的攻击者压倒性的数量给受害者
- Ping of Death：攻击者发送修改后的 ping 数据包，如添加分包使前后的逻辑不正确，或者加长数据包使其超过 IP 报文的限制
- Teardrop attacks：向目标机器发送损坏的IP包，诸如重叠的包或过大的包载荷。
- UDP flood：利用大量UDP小包冲击服务器
- SYN flood：利用 TCP 的连接过程，来消耗系统资源。
- CC（challenge Collapsar）：通过构造有针对性的、对最为消耗服务器端资源的业务请求，让服务器“劳累过度”而停止服务等等

还有很多的 DDOS 的攻击，只有针对具体的情况来做具体的分析。他们的目的都是消耗机器的资源，让其不能为用户提供正常的服务

## 三、SYN 攻击

据统计，在所有黑客攻击事件中，SYN攻击是最常见又最容易被利用的一种攻击手法。在2000年时 YAHOO 的网站遭受的攻击，就是黑客利用的就是简单而有效的 SYN 攻击，而 SYN 攻击依靠的就是 TCP 三次握手

>**TCP**( Transmission Control Protocol 传输控制协议）是一个 Internet 协议套件的核心协议。它起源于最初的网络实现，它补充了互联网协议（IP,网络层的 IP 协议，IPv4、IPv6）。因此，整个套件通常被称为 TCP/IP。TCP 提供可靠的，有序的，和错误检查流在 IP 网络上的主机通信的运行的应用程序之间传递数据。主要的互联网应用，如万维网（www）、电子邮件(smtp,pop)、远程管理(telnet)和文件传输(ftp)都是依靠 TCP。【注释1】

TCP是面向连接的，可靠的进程通信的协议，提供全双工服务，即数据可在同一时间双向传输，也正是因为 TCP 的可靠连接，所以广泛应用在大多数的应用层协议，而 TCP 在建立一个连接，需要客户端与服务器端发送3个数据包，这个过程叫 Three-way Handshake（三次握手）。

>在这之前，服务器必须先绑定到一个端口并监听一个端口，以打开它的连接：这被称为被动打开。一旦被动打开，客户端可以启动一个主动打开,建立连接

![4-3-1](https://dn-simplecloud.shiyanlou.com/1135081470292587273-wm)
（该图片来源于<http://alpha.tmit.bme.hu/meresek/twh_small.jpg>）

1. 由客户端使用一个随机的端口号，向服务器端特定的端口号发送 SYN 建立连接的请求，并将 TCP 的SYN 控制位置为1。SYN（synchronous）是TCP/IP建立连接时使用的握手信号，客户端将该段的序列号设置为一个随机值。
2. 服务器端收到了客户端的请求，会向客户端发送一个确认的信息，表示已收到请求，这是的数据包里会将 ACK 设置为客户端请求序列号+1，同时服务器端还会向客户端发送一个 SYN 建立连接的请求，SYN 的序列号为另外一个随机的值
3. 客户端在收到了服务器端的确认型号以及请求信号之后，也会向服务器端发送一个确认信号，确认信号的序列号为服务器端发送来的请求序列号+1。在这一点上，客户端和服务器都已收到了连接的确认。步骤1，2建立一个方向的连接参数（序列号），它是公认的。步骤2、3建立了另一个方向的连接参数（序列号），并被确认。有了这些，便建立了一个全双工通信。

这样是一次完整的 TCP 连接过程，而若是我的客户端发送了 SYN 的请求连接的信号，然后服务器从关闭变成监听状态，然后响应了我的请求，发送来了确认信息以及他的请求信号，并将该信息加入未连接的队列中，变成SYN_RCVD的状态

若是客户端若是不处理，不给予响应，那么服务器便会一直的等待，只有等待超过了一定的时间，才会将此信息从队列中移除，而当有很多这样的事情发生时是非常耗费资源的。这是整个服务器建立连接时 TCP 状态的变化过程：

CLOSED -> LISTEN -> SYN recv -> ESTABLISHED

若是这样不正常的连接过多，需要正常连接的用户便会连接不上。

![4-3-2](https://dn-simplecloud.shiyanlou.com/1135081470294625485-wm)

这样的不正常的连接行为便称为 SYN flood（SYN 洪水攻击）

既然我们明白了 SYN flood 是 攻击者利用 TCP 在建立连接时使用 SYN 的请求发送，那么我就可以利用iptables 做这样的限制，来防止一个 ip 给我发出过多的请求

```
#使用 limt 对 syn的每秒钟最多允许3个新链接
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s  --limit-burst 3 -j ACCEPT

#或者针对每个客户端作出限制，每个客户端限制并发数为10个，这里的十个只是为了模拟，可以自己酌情考虑
sudo iptables -I INPUT -p tcp --syn --dport 80 -m connlimit --connlimit-above 10 -j REJECT
```

而相对于 SYN flood 来说，CC 攻击就是做一次完整的请求，属于TCP全连接攻击，攻击者使用“合法”的源IP与访问请求，Collapsar 无法判断真假只好放过，但是整个请求非常的消耗资源，如翻阅数据库呀，不停的翻页等等，一次的连接很长，消耗很多的资源的。

若是只有一两个，上百个这样的请求没问题，但是成千上万台做这样的请求。就会导致服务器的连接数超过上限，使得网页出现service unavailable提示，服务器 CPU 占用率很高，网络连接状态：netstat –na,若观察到大量的ESTABLISHED的连接状态 单个IP高达几十条甚至上百条，外部无法打开网站,软重启后短期内恢复正常,几分钟后又无法访问。

我们可以使用这样的方式来测试一下，我们可以使用 apache 的 ab 工具来模拟高并发的请求

```
sudo apt-get update
sudo apt-get install apache2-utils

#开启我们的 apache2 服务，提供访问
sudo service apache2 start

#这意思是模拟发送1000万个请求，同时并发量为600个，访问本地的 apache 首页
#当然这样的访问并没有查询数据库这么消耗资源，这也仅仅只是模拟嘛
ab -n 10000000 -c 600 http://localhost/index.html 
```

首先我们来看一下当前的连接数：

![4-3-3](https://dn-simplecloud.shiyanlou.com/1135081470304766372-wm)

使用我们的压力测试：

![4-3-4](https://dn-simplecloud.shiyanlou.com/1135081470304927840-wm)

现在再来看看我们的连接数量：

```
netstat -an | grep SYN

netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

![4-3-5](https://dn-simplecloud.shiyanlou.com/1135081470304955481-wm)

从图中我们可以看到 SYN 的连接爆发式的增长了

然后我们增加了 iptables 之后，当我们发送了10 包之后，便无法发送了

![4-3-6](https://dn-simplecloud.shiyanlou.com/1135081470306141781-wm)

当然对于 CC 攻击这样的类型，我们不仅从 iptables 来防范他，还可以修改 TCP 的一些配置如

```
sudo vim /etc/sysctl.conf

#表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
net.ipv4.tcp_syncookies = 1 

#表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_reuse = 1 

#表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_tw_recycle = 1 

#表示开启TCP连接的最大数量的限制
net.ipv4.tcp_max_tw_buckets = 5000
```

`tcp_cookies` 可以很巧妙的来减少 `SYN` 的工具了，当启用 `tcp_syncookies` 时，内核生成一个特定的值，而不把客户的连接放到半连接的队列里。当客户端提交第三次握手的 `ACK` 包时，`linux` 内核取出 `n` 值，进行校验，如果通过，则认为这个是一个合法的连接。


## 四、实验总结

通过本实验我们了解了 iptables 的一些高级用法，以及一些网络上的攻击方式。

## 五、参考资料

[1][wikipedia_DDOS](https://en.wikipedia.org/wiki/Denial-of-service_attack#.28S.29SYN_flood)

[2][维基百科](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)

[3][alpha](http://alpha.tmit.bme.hu/meresek/lantcp_eng.htm)