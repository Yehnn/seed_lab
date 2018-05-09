---
show: step
version: 0.1
enable_checker: true
---

#第三章 显示过滤器的用法（二）

##一、实验简介

本节涵盖以下主题：

- 配置TCP/UDP过滤器；

- 配置特定协议类型的过滤器；

- 配置字节偏移型过滤器；

- 配置显示过滤器宏。

##二、配置TCP/UDP过滤器

TCP和UDP是IP协议族中的两种主要协议，都可供驻留在不同主机上的应用程序“互通有无”。只要在某台主机上执行了某款网络应用程序的客户端程序，便拉开了从某个TCP/UDP源端口（具体的端口号为操作系统随机选择，但通常都高于1024）向（早已监听多时的）该应用程序服务器端目的TCP/UDP端口（端口号一般为提前预设或已登记在案的固定端口号）建立TCP/UDP会话的序幕。上述源端口号和目的端口号，外加客户端主机和服务器端主机的IP地址，可唯一地标识某特定主机与服务器之间运行此款应用程序所建立的TCP/UDP会话。TCP和UDP头部自然也会包含源端口号字段和目的端口号字段。

TCP和UDP头部还包含其他字段。UDP头部的结构非常简单，而TCP头部的结构要复杂许多。因此，在配置显示过滤器时，TCP过滤器所涉及的参数也会多得多。
本节会介绍各种类型的TCP/UDP显示过滤器的配置方法。

###2.1 配置准备

配置显示过滤器之前，需明确应让Wireshark从抓包文件里筛选出哪些数据包，并基于此来精确编制显示过滤语句。

要想根据TCP/UDP端口号来筛选数据包，可用以下显示过滤器。
>  tcp.port == <value> 或 udp.port == <value>：让Wireshark在显示数据包时，根据指定的TCP/UDP源、目端口号来筛选。
>
>  tcp.dstport == <value>或 udp.dstport == <value>：让Wireshark在显示数据包时，根据指定的TCP/UDP目的端口号来筛选。
>
>  tcp.srcport == <value> 或 udp.srcport == <value>：让Wireshark在显示数据包时，根据指定的TCP/UDP源端口号来筛选。

UDP头部的结构非常简单，只包含源/目端口号字段、数据包长度字段，以及校验和字段。因此，对UDP数据包而言，最重要的特征就是源、目端口号。

TCP头部则截然不同。因为TCP是一种面向连接的协议，内置有可靠的传输机制，所以TCP头部要比UDP头部复杂得多。不过，Wireshark完全能够理解TCP所具备的面向连接以及可靠性保证等机制。Wireshark提供了tcp.flags、tcp.analysis等诸多功能强大的涉及TCP的显示过滤参数，只要运用得当，发现并解决TCP性能问题（比如，TCP重传、重复确认、零窗口等问题）或运作问题（TCP半开连接、会话重置等问题）自然不在话下。

以下所列为实战中常用的有关TCP的显示过滤参数。

 **tcp.analysis**：可用该参数来作为分析与TCP重传、重复确认、窗口大小有关的网络性能问题的参照物。在这一过滤参数名下，还包含多个子参数（可在Filter输入栏内，借助自动补齐特性，来获取该参数名下完整的子参数列表），如下所列。
>  tcp.analysis.retransmission：用来让Wireshark显示重传的TCP数据包。
>
>  tcp.analysis.duplicate_ack：用来让Wireshark显示确认多次的TCP数据包。
>
>  tcp.analysis.zero_window：用来让Wireshark显示含零窗口通告信息的TCP数据包（TCP会话一端的主机通过此类TCP数据包，向对端主机报告：本机TCP窗口为0，请贵机停止通过该会话发送数据）。

> **注 意**
>
>  Wireshark在调用tcp.analysis参数筛选数据包时，并不会检查数据包的TCP头部，所依据的是该软件自带的expert system对TCP传输机制的分析和理解。	 


 **tcp.flags**：该参数一经调用，Wireshark就会检查数据包TCP头部中各标记位的置位情况。以下所列为该参数名下的几个子参数。

>  tcp.flags.syn == 1：用来让Wireshark显示SYN标记位置1的TCP数据包。
>
>  tcp.flags.reset == 1：用来让Wireshark显示RST标记位置1的TCP数据包。
>
>  tcp.flags.fin == 1：用来让Wireshark显示FIN标记位置1的TCP数据包。


> **注 意**
> ​	
> 	可利用tcp.flags 过滤参数，让Wireshark检查IP包的TCP头部中各标记位的置位情况。	 


 **tcp.window_size_value <<value>**：该过滤参数一经调用，Wireshark将会只显示TCP头部中窗口大小字段值低于指定值的数据包。可利用该参数来排除与TCP窗口过小有关的网络性能问题，此类问题有时要拜赐于参与TCP会话的网络设备“反应过慢”。

###2.2 配置方法

先举几个TCP/UDP显示过滤器的配置实例。

>- 要让Wireshark只显示涌向HTTP服务器的所有流量，显示过滤器应如此配置：tcp.dstport == 80。
>- 要让wireshark只显示由IP子网10.0.0.0/24内的主机访问HTTP服务器的所有流量，显示过滤器应如此配置：ip.src==10.0.0.0/24 and tcp.dstport == 80。
>- 要让Wireshark只显示某条特定TCP连接（在抓包文件中编号为16的TCP连接）中发生重传的所有TCP数据包，显示过滤器应如此配置：tcp.stream eq 16 && tcp.analysis.retransmission。

要想让Wireshark只显示某条TCP连接从建立到终结，会话双方生成的所有数据包，请在抓包主窗口选择一个隶属于该TCP连接（也叫TCP Stream[TCP流]）的TCP数据包，同时点右键，在右键菜单中选择Follow TCP Stream菜单项。一条TCP Stream就是TCP会话双方从建立连接到终止连接那段时间内交换的所有数据包。只要点击过Follow TCP Stream菜单项，在Filter输入栏内会自动出现tcp.stream eq  <value>的字样。这里的value就是Wireshark在抓包文件中为这条TCP连接分配的标识号。对于前例，过滤参数中所引用的标识号为16，可为任意数字（在所有抓包文件中，该标识号从1开始分配）。

导致TCP重传的原因有很多，本书第9章会对此展开深入讨论。

>**注 意**
>
> 当使用Wireshark分析TCP重传、重复确认以及其他影响网络性能的现象的原因时，应（借助于tcp.analysis过滤参数及Follow TCP Stream菜单项）把上述现象与具体的TCP连接建立起关联。	 

再举几个与TCP/UDP有关的显示过滤器配置实例。
>-  要让Wireshark只显示某条特定TCP连接中出现窗口问题的TCP数据包，显示过滤器应如此配置。
>  1. tcp.stream eq 0 && (tcp.analysis.window_full || tcp.
>     analysis.zero_window)
>  2. tcp.stream eq 0 and (tcp.analysis.window_full or tcp.
>     analysis.zero_window)
>-  要让Wireshark只显示IP地址为10.0.0.5的主机访问DNS服务器的流量，显示过滤器应如此配置：ip.src == 10.0.0.5 && udp.port == 53。
>-  要让Wireshark只显示包含某指定字符串（区分大小写）的TCP数据包（比如，在百度中搜索关键字“Windows”），显示过滤器应如此配置：tcp contains " Windows "。
>-  要让Wireshark只显示由IP地址为10.0.0.3的主机生成的所有TCP重传数据包，显示过滤器应如此配置：ip.src ==10.0.0.3 and tcp.analysis.retransmission。
>-  要让Wireshark只显示涌向HTTP服务器的所有流量，显示过滤器应如此配置：tcp.dstport == 80。
>-  要让Wireshark只显示由指定主机建立TCP连接时生成的所有数据包（若某台主机在执行某种形式的TCP端口扫描，或某台主机感染了蠕虫病毒时，就会批量生成此类数据包），显示过滤器应如此配置：ip.src==10.0.0.5 && tcp.flags.syn==1 && tcp.flags.ack==0。
>-  要让Wireshark只显示由指定主机发送的包含HTTP cookie的所有数据包，显示过滤器应如此配置：ip.src==10.0.0.3 &&(http.cookie || http.set_cookie)。

###2.3 幕后原理

下面两张图片分别示出了IPv4包头和TCP头部的格式，而UDP头部的结构则比较简单，只包括源、目端口号字段、长度字段以及校验和字段，因此不再示出。先来看一下IP包头的结构。
![7-2.3-1](https://dn-anything-about-doc.qbox.me/userid2418labid910time1429422308666?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


下面简单介绍一下IPv4包头中的若干重要字段。
> - 版本字段：表示IP协议的版本号，其值为4。
> - IP包头长度字段：用来指明IP包头的长度，单位为4字节，其值一般为5，最大值为15（考虑了IP包头中含有选项字段的情况）。
> - QoS（服务类型）字段：一般都采用区分服务的置位方式，用来区分不同类型流量的贵贱程度。


 **注 意**
 	
 	在发布于1981年9月的RFC 791中，曾把QoS字段命名为ToS（服务类型）字段，并针对该字段中的每一位定义了一套置位方式。在1998年发布的RFC 2474、RFC 2475，以及后来发布的其他Internet文档中，又围绕该字段另行定义了区分服务标准，以及另外一套置位方式，同时得到了广泛应用。	 


>-  长度字段：表示整个IP包的总长度。
>-  标识符、长度以及分片偏移字段：每个IP包都有一个ID（标识符）。当IP包以分片方式传送时，接收方能凭借这三个字段来进行重组。
>-  生存时间（TTL）字段：该字段的起始值为64、128或256（随发包主机的操作系统而异），数据包在转发过程中，路径沿途的每一台路由器都会将该字段值减一。这是为了防止网络中的数据包形成转发环路。若收到了TTL字段值为1的数据包，路由器在将其值递减为0同时，还会做丢弃处理。
>-  高层协议类型字段：用来指明IP包头所封装的高层协议类型，若其值为6，就表示IP包封装的是TCP报文段；若为1，则表示封装的是ICMP报文。
>-  校验和字段：该字段包含的是IP包的校验和。IP包的发送方会采用某种错误检测机制，针对整个IP包计算一个值，并在发送时将该值填入校验和字段。收到IP包时，接收方也会先用相同的机制计算出一个值，再将该值与IP包的校验和字段值进行比对，若两值不等，则认为IP包在传送时发生了错误。
>-  源、目IP地址字段：顾名思义，这2个字段值分别为IP包的源和目的IP地址。
>-  选项字段：IPv4数据包一般不含该字段。
>  接下来，再来看一下TCP头部的结构。
>  ![7-2.3-2](https://dn-anything-about-doc.qbox.me/userid2418labid910time1429422421715?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


下面来介绍一下TCP头部中的若干重要字段。
>-  源、目端口号字段：这两个字段值，再加上IP包头中的源、目IP地址即能唯一地标识一条TCP连接。
>-  序列号字段：用来统计发送方通过TCP连接交付给接收方的数据的字节数。
>-  确认号字段：该字段指明了（执行确认的）TCP发送方期待接收的下一个TCP数据包中的序列号字段值。本书第9章会对该字段的用途做深入探讨。
>-  头部长度：用来表示TCP头部的长度，同时还能指明TCP头部中是否包含有选项字段。
>-  预留字段：该字段为预留的标记位字段。
>-  8个标记位：作用包括：发起连接（SYN位）、终止连接（FIN位）、重置连接（RST位）、快推数据至应用层（PSH位）。本书第9章会详细介绍这些TCP标记位。
>-  接收方窗口大小字段：用来表示接收方分配给应用程序的进程，用来接收TCP数据的缓存的大小。
>-  校验和字段：用来存储经过校验和计算产生的值，计算范围覆盖TCP头部、数据以及IP头部中的某些字段。
>-  选项字段：包括时间戳选项字段、接收方窗口扩张选项字段及最长报文段大小（MSS）选项字段等。MMS选项字段指明了该字段的通告方（即发出含MMS选项字段的TCP报文段的主机）希望（逆向）接收的TCP净载的最大长度。本书第9章将会对MMS选项字段做进一步的探讨。

###2.4 拾遗补缺

TTL字段是IP包头中非常有用的字段。通过检查该字段，就能弄清IP包所穿越的路由器的台数。在默认情况下，由不同操作系统生成的IP包的TTL字段值都比较固定（只有64、128和256这三种可能），而IP包在Internet上传输所穿路由器的台数最多也不能超过30（在私有网络中，这一数字将会更低）。因此，若一IP包的TTL字段值为120，则其所穿路由器的台数必定为8；若TTL字段值为52，则所穿路由器的台数将会是12。

###2.5 进阶阅读

>  欲进一步了解与TCP/IP协议栈有关的内容，请参阅第9章。

##三、配置协议所独有的显示过滤器

本节会介绍如何针对常用的（应用层）协议（比如，DNS协议、HTTP协议、FTP协议），来配置Wireshark显示过滤器。

本节的目标是要教会读者在排除网络故障时，如何利用显示过滤器来助一臂之力。在随后的章节里，也会出现与排除网络故障有关的内容。

###3.1 配置准备

要配置显示过滤器，只需运行Wireshark软件，无需其他任何准备。

###3.2 配置方法

本节会介绍如何针对若干常用的（应用）层协议，配置Wireshark显示过滤器。

**HTTP显示过滤器**

以下所列为一些实战中常用的HTTP显示过滤器。
>-  要让Wireshark只显示访问某指定主机名的HTTP协议数据包，显示过滤器应如此配置：http.host == <"hostname">。
>-  要让Wireshark只显示包含HTTP GET方法的HTTP协议数据包，显示过滤器应如此配置：http.request.method =="GET"。
>-  要让Wireshark只显示HTTP客户端发起的包含指定URI请求的HTTP协议数据包，显示过滤器应如此配置：http.request.uri == <"Full request URI">。譬如，http.request.uri == "/v2/rating/mail.google.com"。
>-  要让Wireshark只显示HTTP客户端发起的包含某指定字符串的URI请求的HTTP协议数据包，显示过滤器应如此配置：http.request.uri contains "URI String"。譬如，http.request.uri contains "mail.google.com"（只显示包含字符串“mail.google.com”的URI请求的HTTP协议数据包）。
>-  要让Wireshark只显示网络中传播的所有包含cookie请求的HTTP协议数据包（请注意，cookie总是从HTTP客户端发往HTTP服务器），显示过滤器应如此配置：http.cookie。
>-  要让Wireshark只显示所有包含由HTTP服务器发送给HTTP客户端的cookie set命令的HTTP协议数据包，显示过滤器应如此配置：http.set_cookie。
>-  要让Wireshark只显示所有由Google HTTP服务器发送给本地HTTP客户端，且包含cookie set命令的HTTP协议数据包，显示过滤器应如此配置：(http.set_ cookie) && (http contains "google")。
>-  要让Wireshark只显示包含ZIP文件的HTTP数据包，显示过滤器应如此配置：http matches "\.zip" && http.request.method == "GET"。 

**DNS显示过滤器**

此处来举几个DNS显示过滤器示例。

>- 要让Wireshark只显示所有DNS查询和DNS响应数据包，显示过滤器应如此配置。
>   dns.flags.response == 0 （DNS查询）
>    dns.flags.response == 1 （DNS响应）
>- 要让Wireshark只显示所有anser count字段值大于或等于4的DNS响应数据包 ，显示过滤器应如此配置：dns.count.answers >= 4。

**FTP显示过滤器**

以下所列为实战中常用的FTP显示过滤器。

>-  要让Wireshark只显示所有包含特定的FTP请求命令的FTP数据包，显示过滤器应如此配置：ftp.request.command == <"requestedcommand">。
>-  要让Wireshark只显示所有通过TCP端口21传送的包含FTP命令的FTP数据包，显示过滤器应如此配置：ftp。要让Wireshark只显示所有从TCP端口20或从其他端口发出的包含实际FTP数据的FTP数据包，显示过滤器应如此配置：ftp-data 。

###3.3 幕后原理

Wireshark显示过滤语句所采用的正则表达式的语法，等同于Perl语言所采用的正则表达式。

以下所列为正则表达式中元字符的含义。

>-  ^：用来匹配行的开头。
>-  $：用来匹配行的结尾。
>-  |：用来表示二者任选其一。
>-  ()：起分组的作用。
>-  *：匹配0次或多次前一模式（字符）。
>-  +：匹配1次或多次前一模式（字符）。
>-  ?：匹配0次或1次前一模式（字符）。
>-  {n}：精确匹配n次前一模式（字符）。
>-  {n,}：匹配至少n次前一模式（字符）。
>-  {n,m}：匹配既不能低于n次也不能高于m次前一模式（字符）。

可利用上述元字符来配置非常复杂的显示过滤器，下面例举若干示例。
>-  要让Wireshark只显示“携带”请求下载ZIP文件的GET命令的HTTP请求数据包，显示过滤器应如此配置：http.request.method == "GET" && http matches "\.zip" && !(http.accept_encoding == "gzip, deflate") 。
>-  要让Wireshark只显示发往域名以.com结尾的Web站点的HTTP数据包，显示过滤器应如此配置：http.host matches "\.com$"。

###3.4 进阶阅读

>  Perl语言所支持的正则表达式的语法列表及手册请见 http://www.pcre.org/ 和 http://perldoc.perl.org/perlre.html

##四、配置字节偏移型过滤器

字节偏移型显示过滤器的通用格式为Protocols[x:y] == <value>。这种过滤器实际上就是先通过x来定位到数据包协议头部中的某个字段（即该字段位于协议头部起始处第x个字节），并检查接下来y个字节的值是否等于value。Wireshark会根据检查结果来显示抓包文件中的相关数据。

这种过滤器的应用场合非常广泛，只要熟知各种协议头部的格式，对其中各字段的位置及长度了然于胸，就能随心所欲地使用它在抓包文件中筛选出自己想看的数据包。

###4.1 配置准备

除了要运行Wireshark软件，打开抓包文件以外，本节无需任何准备工作。字节偏移型显示过滤器的通用格式为：

Protocols[x:y] == <value>

其中，x指明了显示过滤器检查协议头部的位置（应从协议头部开始处的第几个字节开始检查），y则表示显过滤器所要检查的字节数。

###4.2 配置方法

先举几个字节偏移型显示过滤器的例子，如下所示。
>-  要让Wireshark只显示（在以太网内传送的）IPv4多播数据包，字节偏移型显示过滤器应如此配置：eth.dst[0:3] == 01:00:5e（RFC 1112第 6.4节规定，IPv4多播数据包在以太网内传送时，其以太网帧的多播目的MAC地址一定不出MAC地址空间01-00-5E-00-00-00 ~ 01-00-5E-FF-FF-FF）。
>-  要让Wireshark只显示（在以太网内传送的）IPv4多播数据包，字节偏移型显示过滤器应如此配置：eth.dst[0:2] == 33:33:00（RFC 2464第7节规定，IPv6多播数据包在以太网内传送时，其以太网帧的多播目的MAC地址一定是以33-33打头）。

###4.3 幕后原理

网管人员只要熟知各种协议报文结构，便可利用字节偏移型显示过滤器，直接根据数据包协议头部的第某某字节到某某字节的内容，在Wireshark抓包文件中做一番筛选。对于上一节所举的Wireshark字节偏移型示例，就必须熟悉以太网帧的结构。

##五、配置显示过滤器宏

配置显示过滤器宏，是创建复杂的显示过滤器的便捷通道，可以一次配置，多次使用。

###5.1 配置准备

要配置显示过滤器宏，进入Analyze菜单，选择Display Filter Macros菜单项，在弹出窗口中点击 新建 按钮，如图所示。
![7-5.1-1](https://dn-simplecloud.shiyanlou.com/5962221525746114094-wm)


###5.2 配置方法

1．配置显示过滤器宏：要先在Name文本框内输入一个名称，这也就是显示过滤器宏的名称，再朝Text文本框里输入需多次使用的显示过滤语句，输入完毕后点OK按钮。

2．调用显示过滤器宏：在抓包主窗口的Filter输入栏内输入宏调用语句\$(macro_name:parameter1;paramater2;parameter3 …)。

3．现举例加以说明。配置一个名叫test01的显示过滤器宏，其作用是让Wireshark只显示指定源IP地址和指定目的端口号的TCP数据包。

4．配置显示过滤器宏：在Name文本框内输入test01，作为显示过滤器宏的名称；朝Text文本框内输入ip.src==\$1 && tcp.dst port==\$2。其中，\$1和\$2用来取代传递给显示过滤器宏的参数。然后，点OK按钮。最后，在Display Filter macros窗口内再点一次OK按钮，保存这一显示过滤器宏。

5．调用显示过滤器宏test01：若要让Wireshark只显示所有源IP地址为10.0.0.4，目的端口号为80的数据包，则需在抓包主窗口的Filter输入栏内输入${test01:10.0.0.4；80}。其中，10.0.0.4和80分别表示要传递给宏test01的IP地址参数和TCP端口号参数。

###5.3 幕后原理

显示过滤器宏的运作原理非常简单：先用符号“$”加编号作为显示过滤器的位置参数；当随后在Filter输入栏内调用显示过滤器宏时，相关显示过滤参数会按编号的顺序传递进来。

版权声明
>
> Copyright © Packt Publishing 2013. First published in the English language under the title Network Analysis Using Wireshark Cookbook.
>
> All Rights Reserved.
>
> 本书由英国Packt Publishing公司授权人民邮电出版社出版。未经出版者书面许可，对本书的任何部分不得以任何方式或任何手段复制和传播。
>
> 版权所有，侵权必究。

**本课程是人民邮电出版社授权课程，授权内容已更新完毕，如想继续学习该书，请购买书籍后到实验楼环境中学习。**
