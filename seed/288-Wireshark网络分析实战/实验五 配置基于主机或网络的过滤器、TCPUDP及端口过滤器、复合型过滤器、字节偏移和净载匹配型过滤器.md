---
show: step
version: 0.1
enable_checker: true
---

#第二章 抓包过滤器的用法（二）

##一、课程简介

本节涵盖以下内容：

- 配置基于主机或网络的过滤器；

- 配置TCP/UDP及端口过滤器；

- 配置复合型过滤器；

- 配置字节偏移和净载匹配型过滤器。

##二、配置主机和网络过滤器

所谓主机和网络过滤器，是指基于IP地址的第三层过滤器，本章会介绍此类过滤器的使用及配置方法。

###2.1 配置准备   

以下所列为一些简单的第三层过滤器。
> - ip或ipv6：让Wireshark只抓取IPv4或IPv6流量。
> - host <host>：让Wireshark只抓取源于或发往由标识符host所指定的主机名或IP地址的IP流量。
> - dst host <host>：让Wireshark只抓取发往由标识符host所指定的主机名或IP地址的IP流量。
> - src host <host>：让Wireshark只抓取源于由标识符host所指定的主机名或IP地址的IP流量。


>**注 意**	
>
> 通过标识符host，既可以指定IP地址，也可以指定与某个IP地址相关联的主机名称。比如，抓包过滤器host www.packtpub.com 一经配置，Wireshark就只会抓取发往或源于Packt网站的流量了，即所抓数据包的源或目的IP地址（在某种Hostname-to-IP address解析机制里）跟主机名称 www.packtpub.com 相绑定。	 


> - gateway <host>：让Wireshark只抓取穿host而过的流量。标识符gateway所指定的host必须为主机名称，且必须同时在某种Hostname-to-IP address解析机制（比如，主机名文件、DNS或NIS等）以及Hostname-to-Ethernet address解析机制（比如，/etc/ethers文件等）里登记在案。也就是说，该过滤器一经配置，Wireshark所抓流量的源或目的MAC地址一定为标识符gateway所指定的host的MAC地址，但源或目的IP地址绝不会是标识符gateway所指定的host的IP地址。
> - net <net>：让Wireshark只抓取源于或发往由标识符net所标识的IPv4/IPv6网络号的流量。
> - dst net <net>：让Wireshark只抓取发往由标识符net所标识的IPv4/IPv6网络号的流量。
> - src net <net>：让Wireshark只抓取源于由标识符net所标识的IPv4/IPv6网络号的流量。
> - net <net> mask <netmask>：让Wireshark只抓取源于或发往由标识符net和mask共同指明的IPv4网络号的流量（对IPv6流量无效）。
> - dst net <net> mask <netmask>：让Wireshark只抓取发往由标识符net和mask共同指明的IPv4网络号的流量（对IPv6流量无效）。
> - src net <net> mask <netmask>：让Wireshark只抓取源于由标识符net和mask共同指明的IPv4网络号的流量（对IPv6流量无效）。
> - net <net>/<len>：让Wireshark只抓取源于或发往由标识符net指明的IPv4网络号的流量。
> - dst net <net>/<len>：让Wireshark只抓取发往由标识符net指明的IPv4网络号的流量。
> - src net <net>/<len>：让Wireshark只抓取源于由标识符net指明的IPv4网络号的流量。
> - broadcast：让Wireshark只抓取IP广播包。
> - multicast：让Wireshark只抓取IP多播包。
> - ip proto <protocol code>：让Wireshark只抓取IP包头的协议类型字段值等于特定值（等于由标识符proto所指明的protocol code［协议代码］值）的数据包。IP数据包种类繁多，随IP包头的协议类型字段值而异，比如，TCP数据包（协议类型字段值为6）、UDP数据包（协议类型字段值为17）和ICMP数据包（协议类型字段值等于1）等。
> - ip6 proto <protocol>：让Wireshark只抓取IPv6（主）包头中下一个包头字段值等于特定值（等于由标识符proto所指明的protocol值）的IPv6数据包。请注意，无法使用该原词根据IPv6扩展包头链中的相关字段值来执行过滤。


>**注 意**	
>
>在IPv6包头中，有一个名为“下一个包头”的字段，用来指明本包头后跟随了哪一种可选扩展包头。IPv6数据包可以形成扩展包头层层嵌套的局面。对当前版本的Wireshark而言，其抓包过滤器不支持基于IPv6扩展包头链中的相关字段值来执行过滤。	 

> - icmp[icmptype]==<identifier>：让Wireshark只抓取特定类型[icmptype]的ICMP数据包， <identifier>表示的是ICMP头部中的类型字段值，比如，0（ICMP echo reply数据包）或8（ICMP echo request数据包）等。

###2.2 配置方法   

现在，根据上一节的内容，举几个抓包过滤器的配置实例。
>- 1．要让Wireshark只抓取源于或发往主机10.10.10.1的所有流量，抓包过滤器应如此配置：host 10.10.10.1。
- 2．要让Wireshark只抓取源于或发往主机 www.cnn.com 的所有流量，抓包过滤器应如此配置：host www.cnn.com。
- 3．要让Wireshark只抓取发往主机10.10.10.1的所有流量（即目的IP地址为10.10.10.1的数据包），抓包过滤器应如此配置：dest host 10.10.10.1。
- 4．要让Wireshark只抓取源自主机10.10.10.1的所有流量（即源IP地址为10.10.10.1的数据包），抓包过滤器应如此配置：src host 10.10.10.1。
- 5．要让Wireshark只抓取源于或发往IP网络192.168.1.0/24的所有流量，抓包过滤器应如此配置：net 192.168.1或net 192.168.1.0 mask 255.255.255.0 或net 192.168.1.0/24。
- 6．要让Wireshark只抓取单播流量，抓包过滤器应如此配置：not broadcast或not multicast。
- 7．要让Wireshark只抓取源于或发往IPv6网络2001::/16的IPv6数据包，抓包过滤器应如此配置：net 2001::/16。
- 8．要让Wireshar只抓取源于或发往IPv6主机2001::1的所有流量，抓包过滤器应如此配置：host 2001::1。
- 9．要让Wireshark只抓取ICMP流量，抓包过滤器应如此配置：ip proto 1。
- 10．要让Wireshark只抓取ICMP echo request流量，抓包过滤器应如此配置：icmp
  [icmptype]==icmp-echo或icmp[icmptype]==8。在以上两个过滤器中， icmp-echo和8分别表示ICMP echo request数据包的名称和类型（即ICMP数据包的ICMP头部中的类型字段值和与之对应的名称）。

###2.3 幕后原理   

配置主机过滤器时，若根据主机名执行过滤，则Wireshark会（通过某种名字解析服务）把用户输入的主机名转换为IP地址，并抓取与这一IP地址相对应的流量。比方说，若所配抓包过滤器为host www.cnn.com ，Wireshark会通过某种名字解析服务（多半为DNS）将其转换为某个IP地址，并抓取源于或发往这一IP地址的所有数据包。请注意，在此情形，倘若CNN Web站点将（访问它的）流量转发给设有另一IP地址的其他Web站点，Wireshark也只会抓取IP地址为前者的数据包。

###2.4 拾遗补缺   

以下所列为一些常用的抓包过滤器。
>-  ip multicast：用来抓取IP多播数据包。
>-  ip broadcast：用来抓取IP广播数据包。
>-  ip[2:2] == <number>：用来抓取特定长度的IP数据包（number表示IP包头中的IP包总长度字段值）。
>-  ip[8] == <number>：用来抓取具有特定TTL（生存时间）的IP数据包（number表示IP包头中的TTL字段值）。
>-  ip[9] == <number>：用来抓取特定协议类型的IP数据包（number表示IP包头中的协议类型字段值）。
>-  ip[12:4] ==1 ip[16:4]）： 表示数据包的源和目的IP地址相同。

本章的最后一节会对上述过滤器的语法做进一步的解释。下一节的图片揭示了上述过滤器的基本原理，中括号内的数字用来确定抓包过滤器所要关注的相关协议头部（图中所示为IP包头，还可以关注TCP、UDP或其他协议头部）的内容，第一个数字指明了抓包过滤器应从协议头部的第几个字节开始关注，第二个数字则定义了所要关注的字节数。

###2.5 进阶阅读   

欲知更多与Wireshark抓包过滤器有关的内容，请访问tcpdump手册页的主页：http://www.tcpdump.org/tcpdump_man.html
![5-2.5-1](https://dn-anything-about-doc.qbox.me/userid2418labid908time1429418887472?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10) 


##三、配置TCP/UDP及端口过滤器

本章会介绍在使用Wireshark抓包时，如何根据第四层协议TCP/UDP的端口号来实施过滤，同时会介绍这种抓包过滤器的用法。

###3.1 配置准备   

以下所列为几种基本的第四层抓包过滤器。
>-  port <port>：当根据第四层协议（如TCP或UDP）来实施抓包过滤时，该过滤器一经应用，Wireshark所抓数据包的（第四层协议的）源或目的端口号将匹配标识符port所指明的端口号。
>-  dst port <port>：当根据第四层协议（如TCP或UDP）来实施抓包过滤时，该过滤器一经应用，Wireshark所抓数据包的（第四层协议的）目的端口号将匹配标识符port所指明的端口号。
>-  src port <port>：当根据第四层协议（如TCP或UDP）来实施抓包过滤时，该过滤器一经应用，Wireshark所抓数据包的（第四层协议的）源端口号将匹配标识符port所指明的端口号。

以下所列为几种根据端口范围来执行过滤的第四层抓包过滤器。
>-  tcp portrange <p1>-<p2> 或udp portrange <p1>-<p2>：用来抓取（源或目的）端口范围介于p1和p2之间的TCP或UDP数据包。
>-  tcp src portrange <p1>-<p2> 或 udp src portrange <p1>-<p2>：用来抓取源端口范围介于p1和p2之间的TCP或UDP数据包。
>-  tcp dst portrange <p1>-<p2> 或udp src portrange <p1>-<p2>：用来抓取目的端口范围介于p1和p2之间的TCP或UDP数据包。

###3.2 配置方法   

现在，根据上一节的内容，举几个抓包过滤器的配置实例。
>1．让Wireshark只抓目的端口号为80的数据包（HTTP流量），抓包过滤器应如此配置：dst port 80或dst port http。
>
>2．让Wireshark只抓源或目的端口号为5060的数据包（SIP流量），抓包过滤器应如此配置：port 5060。
>
>3．让Wireshark只抓所有TCP连接中用来发起（SYN标记位置1）连接或终止连接（FIN标记位置1）的数据包（TCP连接属于全双工连接，客户端与服务器之间会建立双向连接。也就是说，建立TCP连接时，客户端向服务器发起连接之后，服务器也会向客户端发起连接，终止连接亦然），抓包过滤器应如此配置：tcp [tcpflags] & (tcp-syn | tcp-fin) != 0。


>**注 意**	
>
>请注意，在tcp [tcpflags] & (tcp-syn | tcp-fin) != 0中，使用的是位与运算，并非逻辑与运算。例如，010 or 101等于111，不等于000。	 

>4．让Wireshark只抓所有RST标记位置1的TCP数据包，抓包过滤器应如此配置：tcp [tcpflags]& (tcp-rst) != 0。
>
>5．要想让Wireshark只抓取特定长度的数据包，抓包过滤器的写法有以下两种 。
>
>less <length>：让Wireshark 只抓取不长于标识符less所指定的长度的数据包，其等价写法为：len <= <length>。
>
>greater <length>：让Wireshark只抓取不短于标识符greater所指定的长度的数据包，其等价写法为：<len >= <length>。
>
>6．让Wireshark只抓源或目的端口范围在2000到2500之间的TCP数据包，抓包过滤器的写法为：tcp portrange 2000-2500。
>
>7．让Wireshark只抓源或目的端口范围在5000到6000之间的UDP数据包，抓包过滤器的写法为：udp portrange 5000-6000。

有些应用程序在运行时可能需要关联某段连续（而非某个具体）的TCP或UDP端口号，若要抓取涉及此类应用程序的流量，则可以根据端口范围来配置抓包过滤器。

###3.3 幕后原理   

第四层协议（主要是指TCP或UDP）属于互连末端应用程序的协议。一端节点（比如，Web客户端）向另一端节点（比如，Web服务器）发出连接建立请求时，最常见的举动就是发送（第四层协议）报文。运行在两个末端节点之上，用来发起或接受连接的进程的代号称为（第四层）端口号。第9章会对此展开深入探讨。

对TCP和UDP而言，端口号就是用来标识应用程序的代号。这两种第四层协议之间的差别在于，前者属于面向连接的可靠协议，而后者则是无连接（即不建立连接）的不可靠协议。还有一种名叫流控传输协议（Stream Control Transport Protocol，SCTP）的第四层协议，这是一种高级版本的TCP协议，也使用端口号。

TCP头部设有若干个标记位，这些标记位的主要作用是建立、维护及拆除连接。当（TCP报文段的）发送方将其中某一标记位置1时，其意在（向TCP报文段的接收方）传递某种信号。以下所列为TCP头部中几种常用的标记位。
>-  SYN：用来表示打开连接。
>-  FIN：用来表示拆除连接。
>-  ACK：用来确认（通过TCP连接）收到的数据。
>-  R-ST：用来表示立刻拆除连接。
>-  PSH：用来表示应将数据提交给末端应用程序（进程）处理。

利用（第四层）抓包过滤器，既可以让Wireshark只抓取某特定应用程序生成或接收的流量，也能够筛选出开启了某个标记位的TCP流量。

###3.4 拾遗补缺   

下列第四层抓包过滤器可供读者在某些反常情况下（比如，网络遭到攻击时）使用。
> - tcp[13] & 0x00 = 0：用来抓取所有标记位都未置1的TCP流量（在怀疑遭遇了空扫描[null scan]攻击时使用）。
> - tcp[13] & 0x01 = 1：用来抓取FIN位置1，但ACK位置0的TCP流量。
> - tcp[13] & 0x03 = 3：用来抓取SYN和FIN位同时置1的TCP流量。
> - tcp[13] & 0x05 = 5：用来抓取RST和FIN位同时置1的TCP流量。
> - tcp[13] & 0x06 = 6：用来抓取SYN和RST位同时置1的TCP流量。
> - tcp[13] & 0x08 = 8：用来抓取PSH位置1，但ACK位置0的TCP流量。

下图揭示了上述TCP抓包过滤器的幕后原理。由图中所示TCP头部的格式可知，上述TCP抓包过滤器中，tcp[13]所含数字13指代的是TCP头部中的标记字段（自TCP头部的起始处偏移13个字节），而“=”后面的数字1、3、5等则表示的是标记字段中各TCP标记位的置位情况。
![5-3.4-1](https://dn-anything-about-doc.qbox.me/userid2418labid908time1429419056794?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10) 


###3.5 进阶阅读   

第9章会详细介绍TCP和UDP这两种第四层协议。

##四、配置复合型过滤器

复合型过滤器也叫结构化过滤器，由多个过滤条件构成，过滤条件之间通过not、and或or之类的操作符来进行关联。

###4.1 配置准备   

结构化抓包过滤器的格式如下所示。

[not] primitive [and | or [not] primitive ...]

以下所列为创建Wireshark抓包过滤器时经常用到的操作符。
>  !或not
>
>  &&或and
>
>  || 或or

###4.2 配置方法   

编写结构化抓包过滤器也很简单，只需根据本章前几节的内容“拼接”好满足需求的一个个“条件”即可。

下面给出一些经常会用到的结构化抓包过滤器。
>- 1．要让Wireshark只抓单播数据包，抓包过滤器应如此配置：not broadcast and not multicast。
- 2．要让Wireshark只抓往来于 www.youtube.com 站点的HTTP流量，抓包过滤器应如此配置：host www.youtube.com and port 80。
- 3． 要让Wireshark只抓往来于主机192.180.1.1的Telnet流量，抓包过滤器应如此配置：tcp port 23 and host 192.180.1.1。
- 4．要让Wireshark抓取所有Telnet流量，但由主机192.168.1.1发起的除外，抓包过滤器应如此配置：tcp port 23 and not src host 192.168.1.1。

###4.3 幕后原理   

再举一个复杂的结构化抓包过滤器示例。

要让Wireshark抓取所有TCP源端口范围为5000～6000的Telnet流量（即源端口范围为5000～6000，目的端口号为23的TCP流量），抓包过滤器应如此配置：tcp dst port 23 and tcp src portrange 5000-6000。

###4.4 拾遗补缺   

最后再举几个比较有意思的结构化抓包过滤器，其具体涵义由读者自行分析。
>  host www.mywebsite.com and not (port 80 or port 23)
>
>  host 192.168.0.50 and not tcp port 80
>
>  host 10.0.0.1 and not host 10.0.0.2

###4.5 进阶阅读   

欲掌握更多与结构化抓包过滤器有关的内容及示例，请访问以下链接。
>  http://www.packetlevel.ch/html/tcpdumpf.html
>
>  http://www.packetlevel.ch/html/txt/tcpdump.filters

##五、配置字节偏移和净载匹配型过滤器

就过滤功能而言，字节偏移和净载匹配型过滤器要更加灵活，网管人员可凭籍该工具来配置自定义型抓包过滤器（自定义型过滤器是指所含字段为非Wireshark解析器预定义的过滤器，可针对私有协议流量实施过滤）。只要网管人员熟悉所接触的网络协议，且对协议数据包的结构摸得门清，就能针对包中所含特定字符串定制特殊的抓包过滤器，让Wireshark在抓包时根据这一过滤器来筛选流量。本章会讲解如何配置这种特殊类型的抓包过滤器，同时还会列举几个在实战中可能会经常用到的配置示例。

###5.1 配置准备   

要配置字节偏移和净载匹配型抓包过滤器，请运行Wireshark软件。

###5.2 配置方法   

>1．字节偏移和净载匹配型抓包过滤器一经应用，Wireshark便会用其中所含字符串与所抓数据包的相关协议头部中的某些字段值进行比对，并根据比对结果实施过滤。这种过滤器的格式如下所示。
>proto [Offset: bytes]

 >有了这种过滤器，便可以让Wireshark在抓包时，根据IP、UDP、TCP等协议头部中的某些字段值来实施过滤。

>2．要想针对IP层来实施过滤，字节偏移和净载匹配型抓包过滤器的格式如下所示。
>
>ip [Offset:Bytes]
>
>3．要想根据第四层协议头部中的某些字段值，乃至应用程序的某些特征来实施过滤（比如，针对UDP、TCP头部中的某些字段值，或FTP、HTTP流量的某些特征来实施过滤），最常用的字节偏移和净载匹配型抓包过滤器有以下两种。
>
>tcp[Offset:Bytes]或udp[Offset:Bytes]

###5.3 幕后原理   

下面给出了字节偏移和净载匹配型抓包过滤器的常规写法。

proto （协议类型，如IP、UDP、TCP等）[从协议头部的开始所偏移的字节数:抓包过滤器所要检查的字节数]

下面举几个常用的字节偏移和净载匹配型抓包过滤器示例。
>1．要让Wireshark只抓目的端口范围为50～100的TCP数据包，抓包过滤器应如此配置：tcp[2:2] > 50 and tcp[2:2] < 100，如图2.9所示。
>
>中括号内的第一个数字2表示：抓包过滤器应从（Wireshark主机网卡所收数据包的）TCP头部的第2个字节起开始检查；第二个数字2则指明了检查范围为2字节长，即只检查TCP头部中目的端口号字段值。数字50和100则划定了端口范围（确定了TCP头部中目的端口号字段值的范围）。

 ![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid908time1429419231999?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
图2.9

>2．要让Wireshark只抓窗口大小字段值低于8192的TCP数据包，抓包过滤器应如此配置：tcp[14:2] < 8192，如图2.10所示。
>![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid908time1429419285240?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
>图2.10

中括号内的第一个数字14表示：抓包过滤器应从（Wireshark主机网卡所收TCP数据包的）TCP头部的第14个字节起开始检查；第二个数字2则指明了检查范围为2字节长，即只检查TCP头部中窗口大小字段值；< 8192则指明了检查条件。

有一款Wireshark字节偏移和净载匹配型抓包过滤器生成工具，非常好用，请见链接：http://www.wireshark.org/tools/string-cf.html。

###5.4 拾遗补缺   

下面再给几个刊载于tcpdump手册页中的字节偏移和净载匹配型抓包过滤器示例。
>1．要让tcpdump（或Wireshark）只抓取TCP源或目的端口号均为80的HTTP流量（其实是抓取源或目的端口号均为80，且只包含实际HTTP数据的TCP流量。也就是说，在这批数据包的TCP头部的SYN位、FIN位或ACK位中，有且只有1位置1），抓包过滤器应如此配置：tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) -((tcp[12]&0xf0)>>2)) != 0)。
>
>2．要让tcpdump（或Wireshark）抓取各条TCP会话中的首、尾2个数据包，且这些数据包的源和目的IP地址均不隶属于抓包主机所在IP子网，抓包过滤器应如此配置：tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not net <local-subnet>。请牢记，TCP连接为全双工，此过滤器一配，对于每条TCP连接，Wireshark都会抓到4个数据包，即建立连接三次握手时，客户端和服务器之间交换的第一个数据包，外加关闭连接四次握手时两者间互发的最后一个数据包。

>3．要让tcpdump（或Wireshark）抓取以非以太网封装方式发送的IP多播或广播数据包，抓包过滤器应如此配置：ether[0] & 1 = 0 and ip[16] >= 224。

>4．要让tcpdump（或Wireshark）抓取所有类型的ICMP数据包，但ICMP echo reply和echo request数据包除外（即抓取所有ICMP流量，但由IP ping程序生成的流量除外），抓包过滤器应如此配置：icmp[icmptype] != icmp-echo and icmp[icmptype] !
>= icmp-echoreply。请注意，并不是只有执行 ping命令才能生成ICMP echo reply和echo request数据包，执行traceroute等操作也能生成这两种类型的数据包。

###5.5 进阶阅读   

>-  Wireshark官网提供了一款Wireshark抓包过滤器生成工具，链接为http://www.wireshark.org/tools/string-cf.html
>  虽然该工具生成的抓包过滤器未必总能有效，但用它来练练手还是不错的。

>-  读者还可以看一下这篇博文，链接为http://www.packetlevel.ch/html/txt/byte_offsets.txt

版权声明
>
> Copyright © Packt Publishing 2013. First published in the English language under the title Network Analysis Using Wireshark Cookbook.
>
> All Rights Reserved.
>
> 本书由英国Packt Publishing公司授权人民邮电出版社出版。未经出版者书面许可，对本书的任何部分不得以任何方式或任何手段复制和传播。
>
> 版权所有，侵权必究。





