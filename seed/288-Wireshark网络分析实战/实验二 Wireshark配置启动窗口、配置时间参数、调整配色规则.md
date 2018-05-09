---
show: step
version: 0.1
eneble_checker: true
---

#第一章 Wireshark简介 (二) 

##一、实验简介

本节涵盖以下内容：

- 配置启动窗口；

- 配置时间参数；

- 调整配色规则；

##二、配置启动窗口


本节会介绍与Wireshark启动窗口有关的基本配置，同时会介绍抓包主窗口、文件格式以及可视选项的配置。

###2.1  准备工作   

启动Wireshark软件，首先映入眼帘的就是启动窗口。可在此窗口中调整以下各项配置参数，来满足抓包需求：
>  工具条配置；
>
>  抓包主窗口配置；
>
>
>  时间格式；
>
>  名字解析；
>
>  所抓数据包的配色；
>
>  抓包时是否自动滚屏；
>
>  字体大小；
>
>  主窗口数据包属性栏的配置；
>
>  配色规则。

先来熟悉一下Wireshark启动窗口内几个常用的工具条（栏），如图所示。
![2-2.1-1](https://dn-anything-about-doc.qbox.me/userid2418labid905time1429413176904?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


本节将重点介绍下列工具条的结构及用法：
>  主工具条（Main Toolbar）；
>
>  显示过滤器工具条（Filter Toolbar）；
>
>  状态栏（Status Toolbar）。

**主工具条**

主工具条上各个（组）按钮的用途如图所示。

主工具条最左边一组5个按钮都与抓包操作有关，其余按钮分别涉及文件操作、数据包选择操作、字体缩放操作、配色及自动滚屏、抓包/显示过滤器的调整及应用、帮助等。
 ![2-2.1-2](https://dn-anything-about-doc.qbox.me/userid2418labid905time1429413406185?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


**显示过滤器工具条**

显示过滤器工具条上有一个输入栏和4个按钮，如图所示。
 ![2-2.1-3](https://dn-anything-about-doc.qbox.me/userid2418labid905time1429413470441?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


**状态栏**

在Wireshark主窗口的最底部，有一个状态栏，分5个区域，如图所示。
通过图示的Wireshark主窗口底部状态栏，可以：
>观察到专家系统中的错误 ；

 ![2-2.1-4](https://dn-anything-about-doc.qbox.me/userid2418labid905time1429413583891?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


>选择为抓包文件添加注释信息；
>
>观察到抓包文件的名称（在抓包期间，抓包文件名由Wireshark软件临时分配）；
>
>获知抓包文件中包含的数据包的数量、Wireshark实际显示出的数据包的数量，以及人为打上标记的数据包的数量。

###2.2  配置方法   

本节会按部就班地指导读者配置Wireshark启动窗口和抓包主窗口。

**定制工具条**

对于一般情况下的抓包，根本无需调整与Wireshark工具条有关的任何配置。但若要抓取无线网络中的数据（即要让Wireshark主机抓到无线网络里其他主机的无线网卡收发的数据），则需要在Wireshark启动窗口内激活wireless工具栏。为此，请点击启动窗口中的View菜单，并选择Wireless Toolbar菜单项，如图所示。
 ![2-2.2-1](https://dn-anything-about-doc.qbox.me/userid2418labid905time1429413693260?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


**定制抓包主窗口**

可按图所示来配置Wireshark，定制其抓包主窗口的界面。
 ![2-2.2-2](https://dn-anything-about-doc.qbox.me/userid2418labid905time1429413758020?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


一般而言，无需对Wireshark抓包主窗口的界面做任何调整。但在某些情况下，也有可能需要取消勾选View菜单中的Packet Bytes菜单项，把抓包主窗口的空间都留给“数据包列表”（对应于View菜单中的Packet List菜单项）和“数据包内容”区域（对应于View菜单中的Packet Details菜单项）。

**名字解析**

在Wireshark软件里，名字解析功能一经启用，数据包中的L2（MAC）/L3（IP）地址以及第4层（UDP/TCP）端口号将会分别以有实际意义的名称示人，如图所示。
 ![2-2.2-3](https://dn-anything-about-doc.qbox.me/userid2418labid905time1429413870561?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


由图中画线的地方可知，数据包所含MAC地址、IP地址以及TCP端口号都以名称进行相应的替换。

**为数据包着色**

通常，在使用Wireshark抓包时，应为抓取到的网络中的正常流量建立一个（视觉上的）基线模板。这样一来，便可以一边抓包，一边通过抓包主窗口显示出的数据包的色差，来发现潜在的令人生疑的以太网、IP或TCP流量。

要让Wireshark体现出这样的色差，请在抓包主窗口的数据包列表区域，选择一个深受怀疑或需要着色的数据包，同时点右键，在右键菜单Colorize Conversation下点选Ethernet、IP或TCP/UDP（TCP和UDP只有一项可选，视数据包的第四层类型而定）子菜单项名下的各种颜色（color）菜单项。如此操作，会让该数据包所归属的（Ethernet、IP、UDP或TCP）会话中的所有其他数据包都以相同的颜色示人。

现举一个给抓包文件中的数据包上色的例子，如图所示，作者将归属传输层安全（Transport Layer Security，TLS）会话的所有数据包以另外一种颜色示人。
 ![2-2.2-4](https://dn-anything-about-doc.qbox.me/userid2418labid905time1429413982283?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


要取消配色规则，请按下列步骤行事。
>1．点击View菜单。
>
>2．选择底部的菜单项Reset Coloring 1-10。

**抓包时实时自动滚屏的配置**

要配置Wireshark使其在抓包时自动滚屏，请按以下步骤行事。
>1．点View菜单。
>
>2．选择中间的Auto Scroll in Live Capture菜单项。

**字体缩放**

要缩放Wireshark抓包主窗口的字体，请按以下步骤行事。
>1．点View菜单。
>
>2．放大字体，请点中间的菜单项Zoom In或按Crtl+“+”键。
>
>3．缩小字体，请点中间的菜单项Zoom Out或按Crtl+“-”键。

##三、配置时间参数

本节将介绍如何配置数据包的时间显示格式。对时间显示格式的调整，会在Wireshark抓包主窗口数据包列表区域的time列的内容里反映出来。在某些情况下，有必要让Wireshark以多种时间格式来显示数据包。比方说，在观察隶属同一连接的所有TCP数据包时，每个数据包的发送间隔时间是应该关注的重点；当所要观察的数据包抓取自多个来源时，则最应关注每个数据包的确切抓取时间。

###3.1 准备工作   

要配置Wireshark抓包主窗口数据包列表区域中的数据包的时间显示格式，请进入View菜单，选择Time Display Format菜单项，右边会出现图示的子菜单。

![2-3.1-1](https://dn-simplecloud.shiyanlou.com/5962221525744213852-wm)


###3.2  配置方法   

上一节图中所示Time Display Format菜单项的上半部分子菜单包含以下子菜单项。

>**Date and Time of Day和Time of Day**：当通过Wireshark抓包来帮助排除网络故障，且故障发生的时间也是定位故障的重要依据时（比如，已获悉了故障发生的精确时间，且还想知道相同时间网络中发生的其他事件时），就应该根据具体情况，选择这两个子菜单项之一。
>
>**Seconds Since Epoch**（自Epoch以来的秒数）：Epoch是指通用协调时间（格林威治标准时间的前称）的1970年1月1日早晨0点。这也是UNIX系统问世的大致时间。
>
>**Seconds Since Beginning of Capture**（自开始抓包以来的秒数）：此乃Wireshark默认选项。
>
>**Seconds Since Previous Captured Packet**（自抓到上一个数据包以来的秒数）：这也是一个常用选项，此菜单项一经点选，数据包列表区域的time列将显示每个数据包的抓取时间差。当监控时间敏感型数据包（比如，TCP流量、实时视频流量、VoIP语音流量）时，就应该点选该子菜单项，因为此类数据包的发送时间间隔对用户体验有至关重要的影响。
>
>**UTC Date and Time of Day和UTC Time of Day**：提供UTC时间。

可使用快捷键Ctrl + Alt + 任意数字键来调整上述时间格式选项。

Time Display Format菜单项的下半部分子菜单提供的是时间精度选项，若无时间精度方面的需求，则应保持默认设置。

###3.3  幕后原理   

为抓到的数据留时间“烙印”时，Wireshark依据的是操作系统的时间。在默认情况下，生效的是Seconds Since Beginning of Capture子菜单项功能。

##四、定义配色规则

Wireshark会根据事先定义的配色规则，用不同的颜色来“分门别类”地显示抓包文件中的数据。合理的定义配色规则，让不同协议的数据包以不同的颜色示人（或让不同状态下的同一种协议的数据包呈现出多种颜色），能在排除网络故障时帮上大忙。

Wireshark支持基于各种过滤条件，来配置新的配色规则。这样一来，就能够针对不同的场景，定制不同的配色方案，同时还能以不同的模板来保存。也就是说，网管人员可在解决TCP故障时，启用配色规则A；解决SIP和IP语音故障时，启用配色规则B。
 >注 意	可以通过定义模板（profile）的方式，来保存针对Wireshark软件自身的配置（比如，事先配置的配色规则和显示过滤器等）。要如此行事，请点击EDIT菜单下的Configuration Profiles菜单项。	 

###4.1  准备工作   

要定义配色规则，请按以下步骤行事。
>1．选择View菜单。
>
>2．点击底部的Coloring Rules菜单项，Coloring Rules窗口会立刻弹出，如图所示。

 ![2-4.1-1](https://dn-simplecloud.shiyanlou.com/5962221525744148353-wm)


###4.2  配置方法   

现在，来看一下如何定义一条新的配色规则。

点击Coloring Rules窗口中的 `新建` 按钮，Edit Color Filter窗口会立刻弹出，如图所示。
![2-4.2-1](https://dn-simplecloud.shiyanlou.com/5962221525744314454-wm)


新的配色规则就在此窗口内配置，请按以下步骤行事。
>1．在**Name**输入栏内填入这条规则的名称。譬如，要想专为NTP协议数据包定制配色规则，那就在该输入栏内填入NTP。
>
>2．在**String**字段内填入显示过滤表达式，指明本配色规则对哪些数据包生效。也可以点击**Expression**按钮，根据Wireshark预制的显示过滤参数来构造显示过滤表达式。欲知更多与显示过滤器和显示过滤表达式有关的内容，请阅读第3章。
>
>3．点击**Foreground Color**按钮，为本配色规则选择一款前景色。此款颜色将成为受本配色规则约束的数据包在抓包主窗口的数据包列表区域里的前景色。
>
>4．点击**Background Color**按钮，为本配色规则选择一款背景色。此款颜色将成为受本配色规则约束的数据包在抓包主窗口的数据包列表区域里的背景色。
>
>5．要是想修改现有的配色规则，请点击**Edit**按钮。还可以点击**Import**按钮，导入现成的配色方案，或点击**Export**按钮导出当前的配色方案。


> 注 意	Coloring Rules窗口中配色规则的排放次序是有讲究的。请务必确保配色规则的排放次序与配色方案的执行次序相匹配。比方说，作用于应用层协议数据包的配色规则应置于作用于TCP/UDP数据包的配色规则之前，只有如此，方能避免Wireshark为应用层协议数据包干扰TCP/UDP数据包的颜色。	 


###4.3  幕后原理   

Wireshark软件中的许多操作都与显示过滤器紧密关联，定义配色规则也是如此，因为受配色规则“约束”的数据包都是经过预定义的显示过滤器过滤的数据包。

###4.4  拾遗补缺   

可从http://wiki.wireshark.org/ColoringRules 下载到很多经典的Wireshark数据包配色方案，在Internet上也能搜到许多其他的配色方案示例。

 版权声明
>
> Copyright © Packt Publishing 2013. First published in the English language under the title Network Analysis Using Wireshark Cookbook.
>
> All Rights Reserved.
>
> 本书由英国Packt Publishing公司授权人民邮电出版社出版。未经出版者书面许可，对本书的任何部分不得以任何方式或任何手段复制和传播。
>
> 版权所有，侵权必究。





