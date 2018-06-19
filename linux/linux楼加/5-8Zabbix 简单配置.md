---
show: step
version: 1.0
enable_checker: true
---
# Zabbix 简单配置

## 1. 实验介绍

#### 1.1 实验内容

本节实验主要介绍了 zabbix 的相关配置项，以及在 web 端快速使用 zabbix 来创建用户和创建主机与主机组等操作。

#### 1.2 实验知识点

+ Zabbix 的配置项

+ 快速使用 Zabbix

+ 创建主机和主机组

+ 资产清单管理

+ 批量更新操作

#### 1.3 推荐阅读

[zabbix 官方手册](https://www.zabbix.com/documentation)

## 2. Zabbix 的配置项

在上一节实验中，我们认识了 zabbix 的架构框架和原理。这里我们对架构中的逻辑结构再做进一步的解释说明。

zabbix 逻辑架构如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514860031396.png/wm)
*（图片来自百度图片）*

**原理解释说明：**

作为监控的主机端主要任务是数据的收集，而 zabbix poller(轮询) 正是基于拉取数据机制的一个 server 。数据拉取的方式包括了 `SNMP` 、`zabbix agent` 以及 `internal`、`JMX`、`Windows`、`Various` 等。（详细可以参考[ zabbix 官网](http://www.zabbix.org/wiki/Main_Page)）

其中，基于 zabbix agent 监控的 `poller` 方式在数据采集中通常是在 agent 中的一个 `items 监控项`来完成数据的采集。在 `items` 上定义了一个 `triggers` 即阈值，一旦超出了这个定义的阈值，就会采取一个行动 `action`，从而会发送一个警告的 `email`（或通过其他通讯方式：SMS（短信服务）、Jabber（Linux 即时通讯服务） 等。在触发阈值 `trigger` 时，往往就会发生一个事件 `event` ，也就是事件 `event` 是 `action` 采取行动时判断的条件来源。

我们知道 items 是在 host 主机上，也就是在 agent 上，并且多个 host 可以归为一个 `host groups` ，即多个主机就会有多个 items ，那么如何快速部署主机来监控这些 items 呢？这时就引进了模板 `template` ，其中模板包含了：`items`、`triggers` 和 `graphs`（数据的图形化显示）。这样模板就可以一次性运用到主机上去了。

最后还可以手动调整主机 host 或 host groups 处于维护状态（ `maintenance`）来防止维护阶段发生超过阈值反复报警等状况 。

下面是 zabbix 在架构框架中会涉及到如下的配置组件。（配置项较多也很重要，但是只要理解了原理再来看就会比较容易理解了）

|配置项|说明|
|-|-|
|主机 (`host`)| 监控的网络设备，用 IP 或域名表示
|主机组 (`host group`)|主机的逻辑组，它包含主机和模板。一个主机组里的主机和模板之间并没有任何直接的关联。通常在给不同用户组的主机分配权限时候使用主机组。
|监控项 (`item`)|接收的主机的特定数据，一个度量数据。
|触发器 (`trigger`)|用于定义问题阈值和**评估**监控项接收到的数据的逻辑表达式。当接收到的数据高于阈值时，触发器从 `OK` 变成 `Problem` 状态。当接收到的数据低于阈值时，触发器保留/返回一个 `OK` 的状态
|事件 (`event`)|单次发生的需要注意的事情
|异常 (`problem`)|处在**异常状态**的触发器
|动作 (`action`) |对事件做出反应的预定义的操作。一个动作由操作(例如发出通知)和条件(当时操作正在发生)组成
|升级 (`escalation`) |在动作内执行操作的自定义场景
|媒介 (`media`)|发送告警通知的手段，警告通知的途径
|通知 (`notification`)| 利用已选择的媒体途径把跟事件相关的信息发送给用户
|远程命令 (`remote command`)|预定义好的，满足一些条件的情况下，可以在被监控主机上自动执行的命令
|模版 (`template`)|一组被应用到一个或多个主机上的实体（监控项，触发器，图形，聚合图形，应用，LLD，Web 场景）的集合。主要功能就是加快对主机监控任务的实施，以及使监控任务的批量修改更简单。模版是直接关联到每台单独的主机上。
|应用 (`application`)|一组监控项组成的逻辑分组
|web 场景 (`web scenario`) |利用一个或多个 HTTP 请求来检查网站的可用性
|前端 (`frontend`) |Zabbix 提供的 web 界面
|`Zabbix API` |允许使用 JSON RPC 协议来创建、更新和获取 Zabbix 对象信息或者执行任何其他的自定义的任务
|`Zabbix server`|实现监控的核心程序，主要功能是与 Zabbix proxies 和 Agents 进行交互、触发器计算、发送告警通知，并将数据集中保存
|`Zabbix agent`|部署在监控对象上的，能够主动监控本地资源和应用的程序
|`Zabbix proxy`|帮助 Zabbix Server 收集数据，分担 Zabbix Server 的负载的程序

## 3. 快速使用 Zabbix

下面我们将会学习快速使用 Zabbix。

### 3.1 登录 zabbix

根据上一个实验的学习，确保 zabbix 安装完成后，通过浏览器访问 `http://localhost/zabbix` 网页，进入到如图所示的 zabbix 登录界面

这时输入默认的管理员用户名和密码（默认为：Admin/zabbix）进入到监控界面。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4087timestamp1511864667072.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1timestamp1514531473831.png/wm)

### 3.2 添加用户

1.顺序点击屏幕上的 `Administration → Users`。就会出现如下所示的界面，然后点击 `Create user`。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4087timestamp1511864708419.png/wm)

注：初始化的 zabbix 只有两个用户定义：

+ **Admin**：是 zabbix 的超级用户，拥有完整的权限。
+ **guest**：是一个默认的用户，如果没有登录，将会以 guest 身份访问 zabbix ，其中 guest 在 zabbix 对象上没有权限。

2.点击创建用户后，出现如下界面，需要将你创建的用户添加到一个现有的用户组中，例：添加到 `Zabbix administrators` 用户组中。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948369558.png/wm)

3.为用户定义媒体，即通知的传送方式。点击我们刚才创建的用户名，在进入的页面中点击：`Media` 选项，添加一个用户电子邮箱地址，定义相应时间段等选项，可以默认选项。最后点击添加（`add`）完成此步操作。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948406432.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948412038.png/wm)

4.添加权限

点击：`Permissions` 标签栏，默认创建的 `zabbix user` 用户是不具有权限的，为了后面我们的操作这里我们将用户的权限提升到 `Zabbix Super Admin` 超级管理员，这样就具有了读写的权限了。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948473744.png/wm)

然后点击 `update` 按钮。

同时新建的用户默认是没有权限访问主机的，所以这里也可以赋予用户访问主机的权限。（在 Zabbix 中，对主机的访问权限是分配给用户组，而不是单个用户。）

因此，点击：`Administration → User groups`，选择之前设置的用户组 `Zabbix administrators`。然后转到 `Permissions` 选项中，单击 `Select` 按钮，选择访问 `Linux servers` 组，最后更新操作即可。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948527447.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948532821.png/wm)

完成设置之后点击更新即可。

这时，转到 `Users` 页面就可以看到用户表单中新增了 `User1` 用户。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948683982.png/wm)

因为此时是 Adimin 管理员身份登录，所以我们创建的 User1 和 guest 都是不在线状态。此时，User1 的权限被修改成了超级管理员了。

完成设置以后就可以尝试用新用户进行登录。点击右上角的退出按钮，进入登录界面，用刚刚创建的用户和密码登录。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948751433.png/wm)

## 4. 创建主机和主机组

zabbix 中的主机是监控的网络实体，它可以是物理服务器，网络交换机，虚拟机或某种应用程序。

### 4.1 添加主机

1.现在我们需要在 zabbix 前端进行主机配置，首先点击：`Configuration → Hosts`，可以看到如下界面，然后点击右侧 `create host` 创建一个新主机。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948824627.png/wm)

在 Host 选项下填写相应的参数信息。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948878850.png/wm)

**Host 参数说明：**

|属性|描述|
|-|-|
|`Host name`|输入一个唯一的主机名。允许有字母、空格、圆点、破折号和下划线。**注意：由于 Zabbix agent 运行在你所配置的那台主机上，所以此 agent 配置文件的参数 Hostname 必须和这里输入的主机名是一致的。**
|`Visible name`|显示名称。如果设置了这个名称，它将会在列表、拓扑图等地方显示。此属性支持 UTF-8 。
|`Groups`|选择主机所属主机组。**一个主机必须至少属于一个主机组。**
|`New group`|可以创建一个新的组并和主机关联。如果为空表示忽略。
|`Interfaces`|支持这几种主机接口类型: Agent, SNMP, JMX 和 IPMI。要增加一个新接口，在 Interfaces 区域点击 Add ，输入 IP/DNS, Connect to 和 Port 信息。
|`IP address`|主机的 IP 地址（可选）。
|`DNS name`|主机的 DNS 名称（可选）。
|`Connect to`|点击对应的按钮告诉 Zabbix 服务器采用哪种模式从代理端获取数据。
|`Port`|TCP/UDP 端口。默认端口：Zabbix agent 10050, SNMP agent 161 , JMX 12345 ， IPMI 623
|`Default`|选择单选按钮设置默认接口
|`Description`|填写主机描述。
|`Monitored by proxy`|主机可以被 Zabbix 服务器或者 Zabbix 代理服务器监控。`no proxy`：Zabbix 服务器监控主机 Proxy
|`Enabled`|选中此项表示激活主机，准备接受监控。如果没选中，表示主机未激活，不能被监控。

**Templates**

允许将 templates 链接到主机。所有实体（监控项, 触发器, 图表和应用集）将从模板继承。

*后面实验会着重讲解！*

**IPMI**

IPMI（Intelligent Platform Management Interface，智能平台管理接口）是一套针对自主计算机子系统的计算机接口规范，可独立于主机系统的CPU，固件（BIOS 或 [UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface)）和操作系统并提供管理和监视功能。 IPMI 定义了一系列用于系统管理员以及用于计算机系统的带外管理和监视其操作的接口。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948915431.png/wm)

这里所包含 IPMI 的管理属性如下：

|参数|描述
|-|-|
|`Authentication algorithm`|选择认证算法
|`Privilege level`|选择权限级别
|`Username`|认证用户名
|`Password`|认证用户密码

**Macros**

允许定义主机级别的`用户宏`。
点击：`Macros`，选择 `Inherited and host macros` 选项，可以查看模板的宏以及全局宏。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4087timestamp1511865078101.png/wm)

还可以在主机级别编辑一个**模板/全局宏**，有效地创建主机上宏的副本。

**Host inventory**

允许为主机手工输入库存信息。也可以选择启用自动库存量, 或者禁用此主机的库存量。

**Encryption**

允许请求与主机的加密链接，默认选择无加密。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948948298.png/wm)

2.查看主机列表

点击：`Configuration → Host`。

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515395060785-wm)

可以看到状态是 `disabled` ，点击 disabled 然后点击确定，就手动启动了 zabbix server。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514948977385.png/wm)

+ 如果 `Availability` 列中的 `ZBX` 图标显示的是红色，则说明通信存在一些错误，把鼠标光标移到该位置就可以查看到具体的错误消息。
+ 如果图标是灰色的，则说明目前没有状态更新。可以通过检查 Zabbix 服务器是否正在运行，并在稍后尝试刷新页面。
+ 如果显示的是绿色的，就表示成功的监控了这台客户端（如图中的 zabbix server 这台主机）。

注：因为我们目前未做任何操作，只是创建了主机，因此此时显示的是灰色的。

3.如何查看监控数据

主机添加完成后，可以通过下面的途径进行查到最新的数据情况。

点击：`Monitoring → Latest data` ，然后点击 `Apply`，可以选择查看主机的最近数据。因为我们新建的主机还没有数据，不过可以看看 `zabbix server` 这台主机的数据信息。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514949072722.png/wm)

再点击：``Monitoring → Graphs`，就可以选择查看主机中的各类实体项的具体图像信息。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514949100330.png/wm)

Zabbix 添加主机操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week6/8-1.mp4
@`

### 4.2 创建并配置主机组

配置完主机之后，我们来创建一个自定义的主机组。在 zabbix 页面中，首先，点击：`Configuration → Host groups`，可以观察到如下图所示的默认的主机组。可以看到在创建主机的时候我们将自定义的主机选择到了 `Linux servers` 这个主机组下面。然后点击右上方：`create host group` 按钮创建新的主机组。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514949137201.png/wm)

填写相应参数。

**参数说明**

|参数|描述
|-|-|
|`Group name`|输入唯一的主机组名称。
|`Hosts`|选择主机、组成员。主机组可能有 0 个、1 个或多个主机。

将我们之前创建的主机选择到这个主机组中，点击添加即可。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514949183628.png/wm)

> 补充：`Group name` 在创建一个嵌套的主机组，应该使用 `/` 正斜杠分隔符，例如：`Asia/Shanghai/Zabbix servers`。 即使不存在这两个父主机组(`Asia/Shanghai`) ，也可以创建该组。在这种情况下，创建父主机组取决于使用者，它们不会自动创建。Zabbix 3.2.0 支持主机组的嵌套。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514949272165.png/wm)

可以看到主机组被创建后，我们的主机 `Host_shiyanlou` 属于创建的主机组中。

**权限说明**

+ 创建子主机组到现有的父主机组时，子主机组会继承父主机组的权限。
+ 创建父主机组到现有子主机组时，不用设置父主机的权限。


## 5. Zabbix 资产清单管理

下面我们将会学习 Zabbix 资产清单管理。

### 5.1 概述

`Zabbix` 专门设置了一个资产管理的功能，来处理监控设备和服务器的配置项等资产的管理问题。`Zabbix` 管理页面主要是一个特殊的 `Inventory` 菜单。一开始不会有任何数据，也不能输入任何资产相关的信息。资产信息是在配置主机时人工录入建立的资产数据，或者是通过使用某些自动填充选项完成的录入。

### 5.2 配置清单模式

配置资产清单主要有两种模式：**手动模式**和**自动模式**。

**手动模式**

当创建或配置主机的时候，在清单（Inventory）处选择手动模式，然后输入当前设备的序列号，mac 地址，所在地区，硬件等信息。
如果主机库存信息中包含 `URL`，并以 `http` 或 `https` 开头，则会在 `Inventory` 中呈现可点击的链接。

**自动模式**

如果选择自动模式，部分信息会被自动填充，例如：主机名，系统信息。不过有些其他的信息还是需要手动输入。这里的自动仅仅是把基本的信息给自动获取，大部分还是要手动补充，可以说这算是一个半自动模式。

**资产模式选择**

在主机配置过程中选择资产模式。

根据 `Administration → General → Other` 中的默认主机资产模式设置，选择新主机的默认清单模式。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514949307627.png/wm)

对于通过网络发现或自动注册操作添加的主机，可以设置 ` Default host inventory mode` ，选择**手动或自动模式**。

>  `Manual` ：手动模式。
>
>  `Automatic`：自动模式。

### 5.3 资产信息

点击：`inventory → host` 可以看到如下资产清单的界面，再选择一个主机就可以看到这个 host 的一些基本信息，包括：`host name`、 `agent interfaces` 、`os`、`configuration`等信息。（因为在之前创建主机的时候我们没有填写 `Host_shiyanlou` 的资产信息，所以这里就不会显示。）

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515398647948-wm)

## 6. 批量更新

在实际中可能会遇到要同时更新多台主机配置项的情况，这时批量更新就显得尤为重要。

步骤如下

点击：`Configuration → Hosts` ，勾选需要操作的主机前的复选框，然后点击下方的 `Mass update` 按钮。进入操作界面如下，填写相应参数。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514949405048.png/wm)

**Host 项**

+ `Replace host groups` ：将从任何现有主机组中删除主机，并将其替换为该字段中指定的主机。
+ `Add new or existing host groups`： 允许从现有主机组指定其它主机组，或为主机输入全新的主机组。
+ `Monitored by proxy`：是否选用代理来监控。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514949431880.png/wm)

**Templates 项**

+ `Link templates`：可以批量更新选择链接模板。
+ `Replace`：允许链接新模板，同时取消链接到之前与主机链接的任何模板。
+ `Clear when unlinking`：不仅可以取消链接任何以前链接的模板，还可以删除所有继承自它们的元素（监控项、触发器等）。

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515399917117-wm)

其他选项截图如下：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514949457613.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514949464257.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514949470753.png/wm)

完成修改后，点击更新（update）按钮即可。

## 7. 总结

通过本节实验可以快速的对 Zabbix 的使用有了初步的认识，配置相关的参数，创建主机和主机组以及 Zabbix 资产清单管理，在后面节实验将继续对 Zabbix 的其他组件进行讲解。
