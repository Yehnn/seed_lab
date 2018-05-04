#第一章 Wireshark简介 (一) 

---
  > **本节涵盖以下内容**：
  > > 
  > - 安置Wireshark（主机/程序）；
  > - 开始抓包；

---

##1.1  Wireshark简介

本书的前言曾提到过网络排障以及内置于Wireshark能帮助排障的各种工具。一旦决定动用Wireshark协议分析软件，在使用之前，则有必要先确定该软件在网络中的部署（或安装）位置。除此之外，还得对该软件做一些基本的配置，至少应让其界面看起来更为友好。

用Wireshark执行基本的抓包操作，配置起来并不麻烦，但是该软件也包含了很多高级配置选项，可用来应对某些特殊情况。这样的特殊情况包括令Wireshark在某条链路上持续抓取数据包的同时，将抓包文件切分为多个较小的文件；让Wireshark在抓包主窗口的数据包列表区域只显示（发包/收包）主机（或设备）的名称而非IP地址等。本章会介绍如何在Wireshark中配置这些高级选项，以应对上述特殊情况。

---
##1.2 安置Wireshark（程序或主机）


看到了网络故障的表象，决定通过Wireshark抓包来查明故障原委之前，应确定Wireshark（程序或主机）的（安装或部署）位置。为此，需弄到一张精确的网络拓扑图（至少也得弄清楚故障所波及的那部分网络的拓扑结构），如图所示。

安置Wireshark的原理非常简单。

- 首先，应圈定要抓取哪些（哪台）设备发出的流量；
- 其次，要把安装了Wireshark的主机（笔记本）连接到受监控设备所连交换机；
- 最后，开启交换机的端口镜像（或端口监控）功能，把受监控设备发出的流量“重定向”给Wireshark主机。

![图1.1](https://dn-anything-about-doc.qbox.me/userid17579labid892time1429251034406?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

可利用Wireshark监控LAN端口、WAN端口、服务器/路由器端口或连接到网络的任何其他设备的流量。

如图所示，利用Wireshark软件（安装在交换机左边的PC上）外加交换机的端口镜像（也叫做端口监控，需在交换机上激活该特性，流量镜像的方向已在图中标出）功能，便可以监控到进、出服务器S2的所有流量。当然，也可以在服务器S2上直接安装Wireshark，如此行事，便能直接在服务器S2上监控进、出该服务器的流量了。

某些厂商的交换机还支持以下流量监控特性。

- 监控整个VLAN的流量：即监控整个VLAN（服务器VLAN或语音VLAN）的流量。可借助该特性，在指定的某一具体VLAN内进行流量监控。
- “多源归一”的流量监控方式：以图1.1为例，借助该特性，可让Wireshark主机同时监控到服务器S1和S2的流量。
- 方向选择：可选择监控入站流量、出站流量或同时监控出、入站流量。

###1.2.1  准备工作   

使用Wireshark抓包之前，请先访问Wireshark官网，下载并安装最新版本的Wireshark。

Wireshark软件的后续更新会发布在其官网[http://www.wireshark.org](http://www.wireshark.org)的Download页面下，其最新的稳定版本也可以从该页面下载。

每个Wireshark Windows安装包都会自带WinPcap驱动程序的最新稳定版本，WinPcap驱动程序为实时抓包所必不可缺。用于抓包的WinPcap驱动程序为UNIX Libpcap库的Windows版本。

###1.2.2  操作方法   

现以下图这一典型网络为例，来简单分析一下该网络的架构、网络中设备的部署及运作方式、Wireshark的安置方法，以及如何按需配置网络设备。

![图1. 2](https://dn-anything-about-doc.qbox.me/userid17579labid892time1429251371449?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

请读者仔细研究一下图所示的简单而又常见的网络拓扑结构。

**服务器流量监控**

像服务器流量监控这样的需求，在实战中非常常见。要想监控到某台服务器（收/发）的流量，既可以在交换机上针对连接服务器的端口配置端口镜像（如图中的编号①所示），将流量“重定向”至Wireshark主机，也可以在服务器上直接安装Wireshark。

**路由器流量监控**

要想监控进、出路由器的流量，监控其LAN端口（如图中的编号②和⑥所示）或WAN端口（如图中的编号⑤所示）都可以办到。

路由器LAN端口的流量监控起来比较简单，只要在交换机上配置端口镜像，把与路由器LAN口相连的端口的流量“重定向”至连接Wireshark主机的端口。要想监控路由器WAN口的流量，则要在路由器WAN口和SP（服务提供商）网络之间部署一台交换机，在这台交换机上配置端口镜像，如图所示。

![1.3](https://dn-anything-about-doc.qbox.me/userid17579labid892time1429251482540?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

在SP网络与路由器WAN口之间部署一台交换机，是一项会导致断网的操作。不过，真要如此行事的话，网络中断的时间最多也就一两分钟。

监控路由器的流量时，有一点请务必留意：发往路由器的数据包并不一定都会得到转发。有些数据包或许会在途中“走失”，而路由器既有可能会因缓存溢出而对部分数据包“忍痛割爱”，也有可能会把某些数据包从接收端口“原路送回”。

执行上述流量监控任务时，可能会用到以下两种设备。

>**TAP**：可在受监控链路上用一种叫做分路器（Test Access Point ，TAP）的设备来取代图中的交换机，这是一种简单的“三通”（三端口）设备，执行流量监控时，其所起作用跟交换机相同。与交换机相比，TAP不但便宜而且使用方便。此外，TAP还会把错包原样传递给Wireshark，而LAN交换机则会把错包完全丢弃。交换机不但价格高昂，而且还需要花时间来配置，当然它所支持的监控功能也更多（比如，一般的LAN交换机都支持简单网络管理协议[SNMP]）。排除网络故障时，最好能用可网管交换机，哪怕是功能没那么丰富的可网管交换机也好。
>**HUB**：可在受监控的链路上用一台HUB来取代图中的交换机。HUB属于半双工设备，藉此设备，路由器和SP设备之间穿行的每一个数据包都能被Wireshark主机“看”的一清二楚。使用HUB最大的坏处是，会显著加剧流量的延迟，从而对流量采集产生影响。如今，监控1Gbit/s端口的流量可谓是家常便饭，在这种情况下使用HUB，将会使速率骤降至100Mbit/s，这会对抓包产生严重影响。所以说，在抓包时一般都不用HUB。

**防火墙流量监控**

防火墙流量监控的手段有两种，一种是监控防火墙内口（如图中的编号③所示）的流量，另外一种是监控防火墙外口（如图中的编号④所示）的流量。若监控防火墙内口，则可以“观看”到内网用户发起的所有访问Internet的流量，其源IP地址均为分配给内网用户的内部IP地址；若监控防火墙外口，则能“观看”到的所有（经过防火墙放行的）访问Internet的流量，这些流量的源IP地址均为外部IP地址（拜NAT所赐，分配给内网用户的内部IP地址被转换成了外部IP地址）；而由内网用户发起，但防火墙未予放行的流量，监控防火墙外口是观察不到的[① 译者注：原文是“On the internal port you will see all the internal addresses and all traffic initiated by the users working in the internal network, while on the external port you will see the external addresses that we go out with (translated by NAT from the internal addresses); you will not see requests from the internal network that were blocked by the firewall”。原文较差，译文酌改。]。若有人（通过Internet）发动对防火墙（或内网）的攻击，要想“观察”到攻击流量，观测点也只能是防火墙外口。

###1.2.3  幕后原理   

要想弄清端口镜像（端口监控）的运作原理，需先理解LAN交换机的运作方式。LAN交换机执行数据包转发任务时的“举动”如下所列。

>1．LAN交换机会“坚持不懈”地学习接入本机的所有设备的MAC地址。
>2．收到发往某MAC地址的数据帧时，LAN交换机只会将其从学得此MAC地址的端口外发。
>3．收到广播帧时，交换机会从除接收端口以外的所有端口外发。
>4．收到多播帧时，若未启用Cisco组管理协议（Cisco Group Management Protocol，CGMP）或Internet组管理协议（Internet Group Management Protocol，IGMP）监听特性，LAN交换机会从除接收端口以外的所有端口外发；若启用了以上两种特性之一，LAN交换机将会通过连接了相应多播接收主机的端口，外发多播帧。
>5．收到目的MAC地址未知的数据帧时（这种情况比较罕见），交换机会从除接收端口以外的所有端口外发。

综上所述，在LAN交换机上配置端口镜像去监控某个端口时，可“采集”到进、出该端口的所有流量。若只是将一台安装了Wireshark的笔记本接入LAN交换机，未在交换机上开启端口镜像功能，则只能抓到流入或流出该笔记本的所有单播流量，以及同一VLAN里的多播及广播流量。

###1.2.4  拾遗补缺   

使用Wireshark抓包时，还需提防几种特殊情况。

其中的一个特殊情况是，抓取整个VLAN的流量（VLAN流量监控）。在基于VLAN执行抓包任务时，有几个重要事项需要铭记。第一个要注意的地方是，Wireshark主机只能采集到与其直连的交换机承载的同一VLAN的流量。比方说，在一个交换式网络（LAN）内，有多台交换机都拥有隶属于VLAN 10的端口，要是只让Wireshark主机直连某台接入层交换机，那必然采集不到VLAN 10内其他接入层交换机上的主机访问直连核心层交换机的服务器的流量。
![1.4](https://dn-anything-about-doc.qbox.me/userid17579labid892time1429251824628?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

请看上图所示的网络，用户一般会分布在各个楼层，跟所在楼层的接入层交换机相连。各台接入层交换机会跟一台或两台（为了冗余）核心层交换机相连。Wireshark主机要想抓全某个VLAN的流量，必须与承载此VLAN流量的交换机直接相连，才能采集到相应VLAN的流量。因此，要想抓全VLAN 10的流量，Wireshark主机必须直连核心层交换机。

在上图中，若Wireshark主机直连SW2，且在SW2上激活了相关端口镜像功能，开始监控VLAN 30的流量，则其只能抓取到进、出SW2 P2、P4、P5端口的流量，以及由SW2承载的同一VLAN的流量。该Wireshark主机绝不可能采集到SW3和SW1之间来回穿行的VLAN 30的流量。

基于整个VLAN来实施抓包任务时，可能会抓到重复的数据包，是另外一个需要注意的地方。之所以会出现这种情况，是因为启用端口镜像时，对于在不同交换机端口之间交换的同一VLAN的流量，Wireshark主机会在流量接收端口的流入（input）方向及流量发送端口的流出（output）方向分别抓取一遍。

如下图所示，在交换机上已激活了端口镜像功能，对VLAN 30的流量实施监控。对于服务器S4向S2发送的数据包，当其（从连接S4的交换机端口）流入VLAN 30时，Wireshark主机将抓取一次；当其从（从连接S2的交换机端口）流出VLAN 30时，Wireshark主机会再抓取一次。这么一来，便采集到了重复的流量。
![1.5](https://dn-anything-about-doc.qbox.me/userid17579labid892time1429251920526?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
###1.2.5  进阶阅读   

欲深入了解端口镜像相关信息，请参阅各网络设备厂商提供的操作手册。有些厂商也把端口镜像称为“端口监控”或SPAN（Switched Port Analyzer）（Cisco公司）。

某些厂商的交换机支持远程流量监控（能让直连本地交换机的Wireshark主机采集到远程交换机端口的流量）以及高级过滤功能（比如，在把流量重定向给Wireshark主机的同时，过滤掉具有特定MAC地址的主机发出的流量）。还有些高端交换机本身就具备抓取并分析数据包的功能。某些交换机还能支持虚拟端口（例如，聚合端口或以太网通道端口）的流量监控。有关详情，请阅读交换机的随机文档。

---
##1.3  开始抓包

本节首先将介绍如何启动Wireshark，然后会讲解布放好Wireshark之后，如何对其进行配置，以应对不同的抓包场景。

###1.3.1  准备工作   

安装过Wireshark之后，需点击桌面→开始→程序菜单或快速启动栏上相应的图标，运行该数据包分析软件。

Wireshark一旦运行，便会弹出图所示的窗口（Wireshark1.10.2运行窗口）。

![1.6](https://dn-anything-about-doc.qbox.me/userid17579labid892time1429251982108?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

###1.3.2  操作方法   

要想让Wireshark软件能抓到数据包，有以下三种途径：点击Capture菜单下的相关菜单项；点击快速启动工具栏里的绿色图标；点击Wireshark主窗口左侧居中的Start区域里的相关选项，如图1.6所示。此外，在抓包之前，还可对Wireshark的某抓包选项进行配置。

**如何选择实际用来抓包的网卡**

若只是点击图所示Wireshark快速启动工具栏里的绿色图标（正数第三个图标），Wireshark在抓包时，实际使用的网卡将会是该软件默认指定的网卡（如何更改这一默认配置，详见3.3节）。
![1.7](https://dn-anything-about-doc.qbox.me/userid17579labid892time1429252034888?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

要选择Wireshark抓包时实际使用的网卡，请点击快速启动栏里左边第一个图标（List the available capture interfaces图标），Wireshark Capture Interfaces窗口会立刻弹出，如图所示。
![1.8](https://dn-anything-about-doc.qbox.me/userid17579labid892time1429252146236?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

要想得知哪块网卡为有效网卡，最佳途径是观察其是否能够收发流量。通过上图，可以得知Wireshark感知到的各块网卡正在收、发的数据包的个数（Packets列）及速率（Packets/s列）。

若Wireshark的版本不低于1.10.2，则可以选择一块以上的网卡来同时抓包。如此行事的好处是，只要Wireshark主机配有多块网卡，便可同时监控多个服务器端口、多个路由器（或其他网络设备）端口的流量。下图所示为这样的一个应用场景。
![1.9](https://dn-anything-about-doc.qbox.me/userid17579labid892time1429252180233?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

**如何配置实际用来抓包的网卡**
要想对实际用来抓包的网卡做进一步的配置，请点击Capture菜单中的Options菜单项，Wireshark Capture Options窗口会立刻弹出，如下图所示。
![1.10](https://dn-anything-about-doc.qbox.me/userid17579labid892time1429252226333?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
在上图所示的Wireshark Capture Options窗口中，可配置以下参数。

>1．在Wireshark Capture Options窗口的上半部分区域，可以点选实际用来抓包的网卡。
>
>2．在Wireshark Capture Options窗口的左中区域有一个Use promiscuous mode on all interfaces复选框。选中时，会让Wireshark主机抓取交换机（端口镜像功能）重定向给自己的所有数据包，哪怕数据包的目的（MAC/IP）地址不是本机地址；否则，Wireshark主机只能抓取到目的（MAC/IP）地址为本机地址的数据包，外加广播及多播数据包。
>
>3．在某些情况下，选中该复选框后，Wireshark将不会从无线网卡抓包。因此，若选用无线网卡抓包，且一无所获时，请取消勾选该复选框。
>
>4．在Use promiscuous mode on all interfaces复选框下有Capture Files字样，其后有个files输入栏，可在栏内输入一个文件名，然后再点选use multiple files复选框。这么一点，Wireshark不但会把所抓数据保存在由其命名的文件内（其系统路径可由用户指定），而且还可根据特定的需求，以多个文件的形式存储。当以多个文件的形式存储时，在同一目录下，Wireshark会自动在原始文件名后添加后缀“_xxxxx_具体时间”来加以区分。若所要抓取的数据较多，Wireshark的这一功能便非常有用。譬如，在网卡收到的流量较高，或需要长期抓取数据的情况下，就可以利用这一多文件存储功能，基于特定的时间间隔（点选第二个next file every复选框）或希望保存的每个抓包文件的大小（点选第一个next file every复选框），让Wireshark另行打开一个新的文件来保存所抓取的数据。

>5．在Wireshark Capture Options窗口的左下区域，有Stop Capture Automatically字样。可点选其名下的三个复选框，让Wireshark根据抓包时长、所保存的抓包文件的大小或所抓取的数据包的数量，来决定是否停止抓包任务。

>6．在Wireshark Capture Options窗口的右中区域，有Display Options字样。可点选其名下的三个复选框，来配置Wireshark抓包主窗口的显示选项。点选Update list of packets in real time复选框，Wireshark抓包主窗口将会实时显示抓取到的所有数据包；点选Automatically scroll during live capture复选框，Wireshark抓包主窗口会在实时显示数据包时自动滚屏；点选Hide capture info dialog复选框，Wireshark将不再弹出与实际用来抓包的网卡相关联的流量统计窗口。一般而言，无需改变Wireshark软件的上述任何一项默认配置。

>7．在Wireshark Capture Options窗口的右下区域，有name resolution字样。可点选其名下的4个复选框，来调整与名字解析有关的配置。点选前三个复选框，就会让Wireshark在显示数据时，解析出与MAC地址、IP地址以及第四层协议端口号相对应的名称（比如，MAC地址所隶属的厂商名、与IP地址相对应的主机名或域名、与TCP/UDP端口号相对应的应用程序名等）；点选最后一个复选框Use external network name resolver，Wireshark便会调用由操作系统指明的名字解析程序（比如，DNS解析程序），来解析上述名称。

###1.3.3  幕后原理   

Wireshark的抓包原理非常简单。把Wireshark主机上的网卡接入有线或无线网络开始抓包时，介于有线（或无线）网卡和抓包引擎之间的软件驱动程序便会参与其中。在Windows和UNIX平台上，这一软件驱动程序分别叫做WinPcap和Libcap驱动程序；对于无线网卡，行使抓包任务的软件驱动程序名为AirPacP驱动程序。

###1.3.4  拾遗补缺   

若（数据包的收、发）时间是一个重要因素，且还要让Wireshark主机从一块以上的网卡抓包，则Wireshark主机就必须与抓包对象（受监控主机或服务器）同步时间，可利用NTP（网络时间协议）让Wireshark主机/抓包对象与某个中心时钟源同步时间。

当网管人员既需观察Wireshark抓包文件，也需检查抓包对象所生成的日志记录，以求寻得排障线索时，Wireshark主机与抓包对象的系统时钟是否同步将会变得无比重要。比方说，Wireshark抓包文件显示的发生TCP重传的时间点，与受监控服务器（生成的）日志显示的发生应用程序报错的时间点相吻合，则可以判断TCP重传是拜服务器（上运行的应用程序）所赐，与网络无关。

Wireshark软件所采用的时间取自操作系统（Windows、Linux等）的系统时钟。至于不同OS中NTP的配置方法，请参考相关操作系统配置手册。

以下所列为在Microsoft Windows 7操作系统内配置时间同步的方法。

>1．单击任务栏最右边的时间区域，会出现时间窗口。
>
>2．在时间窗口中点击“更改日期和时间设置”，会弹出“日期和时间”窗口。
>
>3．在“日期和时间”窗口中，点击“Internet时间”标签，再点击“更改设置”，会弹出“Internet时间设置”窗口。
>
>4．在“Internet时间设置”窗口中，选中“与Internet时间服务器同步”复选框，在“服务器”后的输入栏内输入时间服务器（NTP）的IP地址，再点确定按钮。
> **注 意**
> 	在Microsoft Windows 7及后续版本的操作系统中，默认包含了几个时间服务器（格式为域名）。可选择一个时间服务器，让网络内的所有主机都与其同步时间。	


RFC 1059（NTPv1）是定义NTP的第一份标准文档，RFC 1119（NTPv2）则是第二份；目前常用的NTPv3和v4则分别定义于RFC 1305和RFC 5905。

NTP服务器IP地址表可从多处下载，比如http://support.ntp.org/bin/view/Servers/StratumOneTimeServers和http://wpollock.com/AUnix2/NTPstratum1PublicServers.htm。

###1.3.5  进阶阅读   

可浏览以下站点，来了解与PACP驱动程序有关的信息。

 -  WinPcap：[http://www.winpcap.org](http://www.winpcap.org)
 -  LibPcap：[http://www.tcpdump.org](http://www.tcpdump.org)

> 版权声明
>
> Copyright © Packt Publishing 2013. First published in the English language under the title Network Analysis Using Wireshark Cookbook.
>
> All Rights Reserved.
>
> 本书由英国Packt Publishing公司授权人民邮电出版社出版。未经出版者书面许可，对本书的任何部分不得以任何方式或任何手段复制和传播。
>
> 版权所有，侵权必究。









