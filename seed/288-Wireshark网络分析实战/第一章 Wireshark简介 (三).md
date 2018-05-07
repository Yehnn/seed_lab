---
show: step
verison: 0.1
enable_checker: true
---

#第一章 Wireshark简介 (三) 

本节涵盖以下内容：
- 保存、打印及导出数据；

- 配置用户界面（点击EDIT菜单的Preferences菜单项，会弹出Preferences窗口。所谓配置用户界面，就是配置该窗口中User Interface配置选项里的内容）；

- 配置协议参数（即配置Preferences窗口中Protocol配置选项里的内容）。

##1.7  数据文件的保存、打印及导出

**注意：进入实验需要等待一点时间才会出现界面，弹窗提示直接选择 `use default config` 按钮。** 

本节将讨论Wireshark软件中的文件操作，包括数据文件的保存、打印及导出等。

###1.7.1  准备工作   

运行Wireshark软件，点击主工具条上的抓包按钮，开始抓包（或打开一个已经保存的抓包文件）。

###1.7.2  操作方法   

在Wireshark主抓包窗口下，既可把抓来的所有数据都保存进一个文件，也能以不同的格式或文件类型导出自己所需要的数据。
现在，来讲解一下如何执行这些操作。

**数据保存操作**
要把抓包文件中的所有数据包都保存进一个文件（或将现有抓包文件另存为一新文件），请按以下步骤行事。
>1．点击**File**菜单里的Save菜单项（或按Ctrl+S键），在弹出窗口的“文件名”输入栏内输入有待保存的文件名。
>2．点击**File**菜单里的Save as菜单项（或按Shift +Ctrl +S键），在弹出窗口的“文件名”输入栏内输入有待保存的抓包文件的新名称。

若要保存抓包文件（或已抓数据包）中的部分数据（比如，经过显示过滤器过滤的数据），请按以下步骤行事。
>1．点击**File**菜单里的**Export Specified Packets**菜单项，**Export Specified Packets**窗口会立刻弹出，如图1.22所示。
>![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid906time1429415908235?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
>图1.22
>
>2．可在**Export Specified Packets**窗口的左下角区域，点击相应的单选框，来选择文件的导出方式。
>
>3．要把抓包文件中的所有数据包或所有已抓数据包作为一个文件导出，请同时选择**All packets和Captured**单选框，再点**Save**按钮。
>
>4．要把抓包文件（或已抓数据包）中经过显示过滤器过滤的数据包作为一个文件导出，请同时选择**All packets和Displayed**单选框，再点**Save**按钮。
>
>5．要把已选中的数据包（即在数据包列表区域中用鼠标点选的数据包）作为一个文件导出，请选择**Selected packet**单选框，再点**Save**按钮。
>
>6．要把所有带标记的数据包（给数据包打标的方法是：先在“数据包列表”区域选中一个数据包，点击右键，在右键菜单中选择**Mark packet toggle**菜单项）作为一个文件导出，请选择**Marked packet**单选框，再点**Save**按钮。
>
>7．要把“数据包列表”区域中位列2个带标记的数据包之间的所有数据包作为一个文件导出，请选择**First to last marked**单选框，再点**Save**按钮。
>
>8．要把抓包文件中编号（详见“数据包列表”区域里的“No.”列）连续的那部分数据包作为一个文件导出，请选择**Range**单选框，并在其后的输入栏内填写数据包的编号范围，再点**Save**按钮。
>
>9．导出抓包文件时，要是希望放弃其中的某些数据包，请先在“数据包列表”区域里选中那些数据包，单击右键，选择右键菜单里的**Ignore packet toggle**菜单项；然后，选择**Export Specified Packets**窗口中的**Remove Ignored Packets**复选框，再点**Save**按钮。

再次重申，上述“存盘”操作既可以基于整个抓包文件中的所有数据包来进行，也可以基于抓包文件中经过显示过滤器过滤的数据包来进行。

**保存数据的格式选取**
Wireshark支持将抓到的数据以不同的格式来保存，好让各种其他工具做进一步的分析。
可让Wireshark将抓取到的数据存为以下格式。
>**纯文本格式（*.txt）**：存为纯文本ASCII文件格式。
>
> **PostScript（*.pst）**：存为PostScript文件格式。
>
>**逗号分割文件格式**（Comma Separated Values）（*.csv）: 存为逗号分割文件格式。这种格式的文件可为电子表格程序（比如，Microsoft Excel）所用。
>
>**C语言数组格式**（*.c）：把数据包的内容以C语言数组的格式保存，可以很方便地将这种文件格式导入C程序。
>
>**PSML-XML格式**（*.psml）：存为PSML文件格式。PSML是一种基于XML的文件格式，只能保存数据包的汇总信息。欲了解与此文件格式有关的详细信息，请见：http://www.nbee.org/doku.php?id=netpdl:psml_specification。
>
>**PDML–XML格式**（*.PDML）：存为PDML文件格式。PSML也是一种基于XML的文件格式，但能保存数据包的详细信息。欲了解与此文件格式有关的详细信息，请见：http://www.nbee.org/doku.php?id=netpdl:pdml_specification。

要以不同的文件格式来保存Wireshark抓取到的数据，请点击File菜单的Export Packet Dissections菜单项下相应的子菜单项。一旦点击，Export File窗口会立刻弹出，如图1.23所示。
 ![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid906time1429416283737?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
图1.23

在图1.23所示的Export File窗口的下半部分，用方框圈定了2个区域。左侧区域中的单选或复选框用来完成待存数据的筛选操作，这在上一节已做过深入探讨。右侧区域中的复选框和下拉菜单则用来确定应以何种“精细程度”，来保存抓取到的数据包（“精细程度”分为三种：数据包列表、数据包的详细结构以及数据包的内容，这分别对应于在抓包主窗口中出现的那三个区域里的内容）。

**如何打印数据**
要想打印数据，请点击File菜单里的Print菜单项，Print窗口会立刻弹出，如图1.24所示。
可在Print窗口中做如下选择。
>  在窗口的上半部分区域，可选择有待打印的文件格式。
>
>  在窗口的左下区域，可选择有待打印的数据包（操作方法类似于文件保存）。
>
>  在窗口的右下区域，可选择有待打印的数据包的具体内容，可通过其中的复选框来选择打印抓包主窗口中以下区域的内容：
>
>  数据包列表区域（Packet Summary pane）；
>
>  数据包结构区域（Packet Details pane）；
>
>  数据包内容区域（Packet Byte pane）。
>  ![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid906time1429416379690?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
>  图1.24

###1.7.3  幕后原理   

Wireshark支持以文本格式或postscript格式来打印数据（以后一种格式打印时，打印机应为postscript感知的打印机），同时支持将数据打印至一个文件。选妥了Print窗口中的各个选项，点击Print按钮之后，会弹出操作系统自带的常规“打印”窗口，可在其中选择具体的打印机来打印。

##1.8  通过Edit菜单中的Preferences菜单项，来配置Wireshark主界面

借助于Edit菜单中的Preferences菜单项，能控制Wireshark软件的主界面及软件自身的诸多参数，包括数据包的呈现方式、抓包文件的默认存盘位置，以及用来抓取数据包的网卡等。
本章将介绍Wireshark主界面及软件自身的常用参数的配置，熟知这些参数的配置，可帮助我们更好地应对不同的抓包场景。

###1.8.1  配置准备   

要配置Wireshark的用户界面（主界面），请点击Edit菜单中的Preferences菜单项，Preferences窗口会立刻弹出，如图1.25所示。

 ![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid906time1429416452237?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
图1.25

在本章接下来的内容中，会介绍Preferences窗口中以下参数的配置：
>  Columns（数据包属性列）；
>  Capture（抓包方式）；
>  Name Resolution（名字解析）。

###1.8.2  配置方法   

本节会介绍如何调整事关Wireshark软件自身的配置参数。

**调整及添加数据包属性列**

在默认情况下，显现在抓包主窗口的数据包列表区域里的数据包属性列有：No.（编号）、Time（抓取时间）、Source（源地址）、Destination（目的地址）、Protocol（协议类型）、Length（长度）以及Info（信息），可通过图1.26所示的Preferences窗口来调整出现在数据包列表区域里的数据包属性列。
![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid906time1429416528423?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
图1.26

要给数据包列表区域添加一新列，可通过以下两个途径。
>1．从Preferences窗口下部的Field type下拉菜单栏里，选择预先定义好的属性列。这些预先定义好的属性列包括time delta、 IP DSCP value、src port和dest port等。
>
>2．点击Add按钮，从Preferences窗口下部的Field type下拉菜单栏里，选择Custom菜单项，定制数据包属性列。此时，可在图1.26所示的Field name输入栏内输入可在显示过滤器中露面的任一参数，然后点击OK按钮，将这一新列添加至抓包主窗口。
>之后，在抓包主窗口选中该列，点击右键，选择右键菜单中的Edit Column Details菜单项，便可以为新添加的这一新列改名了。

下面举几个用Custom菜单项定制数据包属性列的例子。
>1．要想在抓包主窗口中新增一列，以便观看TCP数据包的TCP窗口大小，需在Field name输入栏内输入显示过滤器参数tcp.window_size。
>
>2．要想在抓包主窗口中新增一列，以便观看每个IP数据包包头中的TTL字段值，需在Field name输入栏内输入显示过滤器参数ip.ttl。
>
>3．要想在抓包主窗口中新增一列，以便观看每个RTP数据包中marker位置1的实例，需在Field name输入栏内输入显示过滤器参数rtp.marker。
>
>4．在分析网络故障时，酌情使用Custom菜单项定制数据包属性列，可加快定位故障的原因，与此有关的内容本书后文再叙。

**调整跟执行抓包任务有关的配置**
执行抓包任务之前，可调整涉及抓包的配置。为此，请在Preferences窗口中点击Capture选项，如图1.27所示。
 ![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid906time1429416650084?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
图1.27

要想更改默认用来抓取数据包的网卡，请点击Edit按钮，在弹出的Interface Option窗口中选择相应的网卡，再点OK按钮即可（重启Wireshark软件之后才能生效）。当然，这里更改的只是默认配置，可在每次重新开始抓包之前，更换用来接收数据包的网卡。

**调整名字解析**
Wireshark支持以下三个层级的名字解析（见图1.28）。
>  在第二层（L2）：Wireshark可把MAC地址的前半部分解析并显示为网卡芯片制造商的名称或ID。比方说，可把一个MAC地址的前三个字节14:da:e9解析并显示为AsusTeckC （ASUSTeK Computer Inc）。
>  在第三层（L3）：Wireshark可把IP地址解析并显示为DNS名称，比如可把157.166.226.46这一IP地址，解析并显示为www.edition.cnn.com。
>  在第四层（L4）：Wireshark可把TCP/UDP端口号解析并显示为应用程序（服务）名称，比如可把TCP 80端口解析并显示为HTTP，把UDP 53端口解析并显示为DNS。

 ![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid906time1429416717913?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
图1.28

>**注 意**	在第四层，只有接收由客户端发起的会话建立请求的TCP/UDP目的端口号才有做名字解析的意义。客户端用来发起会话建立请求的TCP/UDP源端口号是一个随机端口号（大于1024），因此并没有转换成应用程序（服务）名的必要。	 


在默认情况下，Wireshark只会针对第二层MAC地址和第四层TCP/UDP端口号做名字解析。开启IP地址的名字解析功能之前要三思而后行，因为这会让Wireshark委托OS发出大量DNS查询消息，从而拖慢Wireshark的运行速度。

###1.8.3  幕后原理   

原理非常简单，只是调整Wireshark某些菜单项的配置而已。要修改Wireshark主界面，除了本节介绍的配置选项之外，还有其他配置选项可供选择。更多详情请参考www.wireshark.org上的Wireshark配置手册。

##1.9  配置Preferences窗口中的Protocol选项

借助于Preferences窗口中的Protocol选项，可调整Wireshark对相关协议流量的抓取和呈现方式。本节会介绍如何借助Preferences窗口中的Protocol配置选项，来调整Wireshark对常见协议流量的抓取和呈现方式。

###1.9.1  配置准备   

>1．点击Edit菜单中的Preferences菜单项，Preferences窗口会立刻弹出，如图1.29所示。
>![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid906time1429416839226?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
>图1.29
>
>2．在Preferences窗口的左边找到Protocols配置选项，点其左边的“+”号，会出现一份“协议”列表。在这份列表里包含了诸多常用或不常用协议。此处只讨论某些常用协议的配置，对相关协议的介绍详见7～14章。

###1.9.2  配置方法   

本节将介绍以下“基本”协议的配置方法（“基本”意指常用，并不是指简单）：
>  IPv4/IPv6；
>  TCP/UDP。

**配置Protocol选项里的IPv4和IPv6协议**
在Preferences窗口的Protocols选项里选中IPv4或IPv6协议时，窗口的右边会出现若干子选项，如图1.30所示。
 ![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid906time1429416919377?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
图1.30

下面是对IPv4协议下若干子选项的解释。
> **Decode IPv4 ToS field as DiffServ Field**：制定IPv4协议标准之初，为能在IPv4网络中保证服务质量，在IPv4包头中设立了一个叫做“服务类型（ToS）”的字段。后来，IETF又制定了一套IPv4服务质量的新标准（区分服务，DiffServ），打的也是IPv4包头中原ToS字段的主意，只是对其中各个位的置位方式有了新的定义。若未勾选与Decode IPv4 ToS field as DiffServ Field子选项相关联的复选框，Wireshark便会按老的IPv4服务质量标准，来解析所抓IPv4数据包包头中的ToS字段。
>
>  **Enable GeoIP lookups**：GeoIP是一个数据库，Wireshark可根据该数据库里的内容，来呈现（其所抓数据包IP包头中源和目的）IP地址所归属的地理位置。若勾选与Enable GeoIP lookups子选项相关联的复选框，Wireshark便会针对所抓IPv4和IPv6数据包的IP地址来呈现其所归属的地理位置。该子选项功能涉及名字解析，一旦开启，便会拖慢Wireshark实时运行速率。

**配置  Protocol选项里的UDP和TCP协议**
UDP协议非常简单，因此Protocols选项里UDP协议名下可配置的子选项并不多；而TCP协议则很是复杂，Protocols选项里TCP协议名下可配置的参数（子选项）也要多于UDP协议（见图1.31）。
更改TCP协议名下的各个参数，其实也就是调整Wireshark对TCP数据包的解析方式。
 ![图片描述信息](https://dn-anything-about-doc.qbox.me/userid2418labid906time1429416991613?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
图1.31

> **Validate the TCP checksum if possible**：有时，Wireshark会抓到超多校验和错误（checksum errors）的数据包，这要归因于在抓包主机的网卡上开启的TCP Checksum offloading（TCP校验和下放）功能。该功能一开，便会导致Wireshark将抓到的本机生成的数据包显示为“checksum errors”（具体原因后文再表）。因此，若Wireshark抓到了超多校验和错误的数据包，则有必要先取消勾选与Validate the TCP checksum if possible子选项关联的复选框，再去验证是否真的存在校验和问题。
>
>  **Analyze TCP Sequence numbers**：要让Wireshark对TCP数据包做详尽分析，就必须勾选与Analyze TCP Sequence numbers参数相关联的复选框，因为“TCP Sequence numbers（TCP序列号）”是TCP最重要的特性之一。
>
>  **Relative Sequence Numbers**：主机在建立TCP连接时，会启用一个随机的序列号，并将其值存入相互交换的第一个报文段的TCP头部的序列号字段 。只要勾选了与Relative Sequence Numbers参数相关联的复选框，Wireshark就会把（一股TCP数据流中的）第一个TCP报文段的（TCP头部的）序列号字段值显示为0，后续TCP报文段的序列号字段值将依次递增，从而隐藏了真实的序列号字段值。在大多数情况下，都应该让Wireshark显示TCP报文段的相对序列号（relative number），以方便网管人员查看。
>
>  **Calculate conversations timestamps**：与该参数相关联的复选框一经勾选，在抓包主窗口的数据包结构区域中，只要是TCP数据包，就会在transmission control protocol树下多出一个Timestamps结构，点击其前面的“+”号，就能看到Wireshark记录的该TCP数据包在本股TCP数据流中的相关时间戳（Timestamps）参数。让Wireshark显示TCP数据包的Timestamps信息，将有助于排查当时间敏感型TCP应用程序故障。

###1.9.3  幕后原理   

通过修改Preferences窗口中Protocols选项下相关协议的参数，便能开启或禁用Wireshark软件对相应协议流量的某些分析功能。需要注意的是，为了保证Wireshark软件的运行速度，应尽量禁用不必要的分析功能。

###1.9.4  进阶阅读   

欲知更多与GeoIP有关的信息，请访问http://wiki.wireshark.org/HowToUseGeoIP。



 版权声明
>
> Copyright © Packt Publishing 2013. First published in the English language under the title Network Analysis Using Wireshark Cookbook.
>
> All Rights Reserved.
>
> 本书由英国Packt Publishing公司授权人民邮电出版社出版。未经出版者书面许可，对本书的任何部分不得以任何方式或任何手段复制和传播。
>
> 版权所有，侵权必究。

