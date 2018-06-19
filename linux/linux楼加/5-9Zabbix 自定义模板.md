---
show: step
version: 1.0
enable_checker: true
---
# Zabbix 自定义模板

## 1. 实验介绍

#### 1.1 实验内容

本节实验主要讲解了 Zabbix 配置项中的模板。主要涉及到了如何创建自定义的模板以及模板中的实体项，主机如何去链接模板，模板和模板间如何进行嵌套等详细操作。

#### 1.2 实验知识点

+ Zabbix 模板

+ 创建自定义模板

+ 主机链接模板

+ 模板间的嵌套

#### 1.3 推荐阅读

[Zabbix 官方模板](http://www.zabbix.org/wiki/Zabbix_Templates/Official_Templates)

## 2. Zabbix 模板

#### 2.1 概述

`Templates` （模板）就是一组可以运用于多个主机的实体。其中的实体包括：**items、triggers、graphs、screens、applications 和 web scenarios 等。**

创建这些实体后，在后续的主机只需要套用这个模板，主机就可以监控模板里面所配置的监控项目。

#### 2.2 使用模板原因

在 Zabbix 逻辑结构图中我们了解到了 `Templates` 模板在逻辑结构中的原理。而之所以引进模板的原因是为什么呢？我们可以做这样一个比喻，把模板看作是“四大发明”中的印刷术，传统的手工抄写费时费力，就像如果我们手工配置 100 台主机的 items 一样，现在有了 templates 模板，我们只要事先定义好模板中的各个实体，比如：`items triggers、graphs` 等，然后在服务器中进行链接套用，这样就达到了简化 zabbix 的每台主机的相关配置，就像使用事先做好的模板去印刷，取代传统的手工抄写，节省时间和减少工作量，也就**简化了 Zabbix 配置**。如果后续有修改、新增等功能，只需要修改模板即可。

*下面就来实际操作一下具体如何去操作模板。*

## 3. 建立自定义模板

下面我们将会学习建立自定义模板。

### 3.1 创建模板

在配置一个模板时需要先定义模板的一般参数来创建，然后再添加相应的实体。

#### 3.1.1 创建模板

1.在 Zabbix 前端界面中顺序点击：`Configuration → Templates`，然后选择 `create template` 创建一个新的模板

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514976905865.png/wm)

接着进入 Templates 模板配置界面，填写相应的参数项，如下图所示：（系统中自带有许多写好的模板，有时间可以下来去查看下每个模板的具体信息）

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514976910478.png/wm)

**参数说明**

|参数|描述
|-|-|
|`Template name`|唯一的模板名称
|`Visible name`|如果填写可以显式的在列表，地图等中可见
|`Groups`|模板所属的主机/模板组。
|`New group`|创建一个新组来保存模板。可以忽略不填
|`Hosts/templates`|应用模板的主机/模板列表
|`Description`|模板描述

2.Linked templates

可以选择其他模板进行链接。这里我们先不做连接操作，后续再进行说明。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514976916053.png/wm)

3.Macros 宏

定义模板级的用户宏，如果选择继承模板的宏选项，则还可以从链接的模板和全局宏中查看宏。在这模板的所有定义的用户宏都可以显示它们所决定的值以及它们的起源。

+ Macro ：变量名称
+ value ：为变量值

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514976996091.png/wm)

配置完成后点击 ：`add` 按钮创建 Templates 的一般配置。

然后需要添加一些实体来具体实现模板的功能。下面介绍几个常用的实体的配置内容，**其中模板实体配置和主机单独配置相似。**

### 3.2 添加监控项（Item）

**注：items 监控项必须首先添加到模板中，否则没有相应的项目，triggers 和 graphs 无法进行添加**

#### 3.2.1 概述

监控项（item）是 Zabbix 中获得数据的基础。它主要由一个 `key` 值和相应的参数组成。例如：需要在监控项中获取 CPU 的信息，可以选择 `key ：system.cpu.load[参数]`。需要监控项获取网卡的信息，可以选择 `key：net.if.in[参数] 或 net.if.out[参数]`。需要监控项收集磁盘空闲空间的数据，可以选择 `key：vfs.fs.size[参数]` 等等。

#### 3.2.2 创建监控项

下面我们从一个实例来学习如何创建监控项实体。

1.**添加 items**

点击 `Configuration->Hosts` ，进入如下界面：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515401936095-wm)

然后点击：`Items`，就进入到 items 的界面，此时还没有任何 item 监控项。然后点击右上角 `create items` 新建一个 items。配置相关参数。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4131timestamp1515635229903.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977039707.png/wm)

**参数说明**

- `Name`：监控项的名称，可以使用宏定义（$1,$2,$3...）
- `Type`：监控项类型常见的有这样两种：
	+ Zabbix agent：被动监控方式
	+ Zabbix agent(active)：主动监控方式
- `Key`：监控项的 key 值，可以在支持的监控项类型中选择，key 在单个主机中必须是唯一的。（选择 key 之后需要指定里面相应的参数，不同的监控项使用的参数值可参考[该列表](http://www.ttlsa.com/zabbix/zabbix-agent-types-and-all-keys/)）
- `Host interface`：选择主机的 agent 接口。
- `Type of information`：执行转换后存储在数据库中的数据类型。（例如：numeric（unsigned）——64 位无符号整数、numeric（float）——浮点数）
- `Units`：若进行单位符号设置，则 zabbix 将对接收到的数据进行处理，并设置单位后缀来显示。如：B（字节）、Bps（每秒字节数）等。
- `Update interval（in sec）`：每 N 秒检索一次该项目的新值。
- `Custom interval`：创建用于检查监控项的自定义规则。（Flexible：创建更新间隔的异常；Scheduling：创建自定义轮询的时间表）
- `History storage period（in sec）`：数据库保留历史记录的天数。
- `Trend storage period(in days)`：数据库保留 N 天的详细历史记录（小时最小，最大，平均值，计数）。
- `show value` ：将值映射到监控项
- `New application`：输入监控项的新应用程序的名称。
- `Applications`：将监控项链接到一个或多个现有应用程序。用于对逻辑组中的项目进行分组。
- `Enabled`：表示启用该项目。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977101202.png/wm)

> 注：由于一些原因，某些 item 的数据无法获取到，但是 zabbix 依旧会在固定的时间间隔内重新去获取数据。

2.**监控项的 Key**

**参数的灵活性（Flexible）**

若参数的位置可以接受任何的参数，那么就是灵活的。例如：`vfs.fs.size[*]`，星号可以使用任何的参数，`vfs.fs.size[/etc/host]` 或 `vfs.fs.size[/]` 等。

**Key 格式**

key 的格式是由名称和它的参数构成，其中参数和名称都要符合规则。格式定义的规范主要是从左到右依次验证，如下图，若 key 名称和参数都依次符合就合法，否则不合法，若没有参数就跳过。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977128372.png/wm)

（图片来自 [zabbix 官方文档](https://www.zabbix.com/documentation/3.4/manual/config/items/item/key)）

**Key 名称**

名称的字符可以是：`0-9a-zA-Z_-.`

也就是数字、大小写字母、下划线、减号和点。（在字符中若有一个不符合，key 都不会合法）

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977133825.png/wm)

（图片来自 [zabbix 官方文档](https://www.zabbix.com/documentation/3.4/manual/config/items/item/key)）

**Key 参数**

key 的参数可以有一个或多个或没有，参数可以**带引号**或**不带引号的字符串**以及**数组**。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977140854.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977147202.png/wm)

（图片来自 [zabbix 官方文档](https://www.zabbix.com/documentation/3.4/manual/config/items/item/key)）

3.将监控项添加到模板中

接下来，标记需要添加到模板的 items 。选中需要复制的项目模板，然后点击：`copy` 按钮进行复制。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977437470.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977444020.png/wm)

到此，所有选定的 items 都被复制到了模板中了。

#### 3.2.3 zabbix 监控项类别

zabbix 提供多种类别的监控项，涵盖从系统获取数据的多种方法，常用的监控项类别有：zabbix agent 、SNMP、IPMI、JMX 等等。在主机定义中可以设置多个类别的接口，为了便于和第一个恰当的主机接口链接，通常会采用如下的搜索可用的主机接口的顺序：`zabbix agent → SNMP → JMX → IPMI`。

**zabbix agent 监控**

zabbix agent 进行通信实现数据采集中，主要有两种代理模式：

+ zabbix agent ：被动模式，zabbix server 向 agent 索要数据。
+ zabbix agent（active）：主动模式，agent 主动给 server 传递数据。

*详细参数说明以及其他监控类别大家感兴趣可以去 [zabbix 官方文档](https://www.zabbix.com/documentation/3.4/manual/config/items/itemtypes/zabbix_agent)查看。*

> 说明：triggers 的配置在后面实验中会详细讲解这里不做说明，不过它的配置和 items 相类似。

### 3.3 添加 screens

构建一个 screen 可以非常简单直观的将不同来源的信息分组并可以快速在屏幕上进行显示。

要配置一个 screen 首先需要定义它的一般属性来创建屏幕，然后就可以在单元格中添加元素。

点击模板栏上的 `Screens`，选择 `create screen`，出现如下界面，填写相关的参数项。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977547368.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977553132.png/wm)

点击 `add` 添加 `screens`，创建完 screen 的一般配置后，点击名称旁边的 ：`Constructor` 添加元素进行展示。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977557695.png/wm)

打开链接后可以看到屏幕上 `Change` 的按钮，点击即可打开一个窗口进行元素的填写，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977611150.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977617128.png/wm)

其中的参数 `Resource` 可以包含如下信息：

+ Action log  : 最近操作的历史
+ Clock : 显示当前服务器或本地时间的数字或模拟时钟
+ Data overview : 一组主机的最新数据
+ Graph : 单个自定义图形
+ Graph prototype : 来自低级别发现的自定义图形规则
+ History of events : 最新的事件
+ Host group issues : 由主机组过滤的触发器的状态
+ Host info : 高级主机相关信息
+ Host issues : 由主机过滤触发器的状态
+ Map : 单个地图纯
+ Plain text  : 纯文本数据
+ Screen : 屏幕（其中一个屏幕可能包含其他屏幕）
+ Simple graph : 单个简单图形
+ Simple graph prototype : 基于低级发现生成的项目的简单图形
+ Status of Zabbix : 高级信息关于Zabbix服务器
+ System status  : 显示系统状态（类似于仪表板）
+ Trigger info : 高级触发相关信息
+ Trigger overview  : 主机组触发器的状态
+ URL : 包括来自外部资源的内容

> 注：图形高度为小于 `120` 像素时，图例中不会显示任何触发器。

添加后就可以显示相应的图像了：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515404617249-wm)

### 3.4 添加 web scenarios

`Web scenarios` 是由一个或多个 HTTP 请求组成。`Web scenarios` 主要是收集场景或场景步骤中的数据信息，然后将其保存在数据库中，而收集到的数据主要用于 `Graph`、`triggers` 和 `notifications` 中。

配置步骤如下：

首先，点击 templates 栏中：`web scenarios`，接着点击右侧的 `create scenarios` 进行常规参数配置。（详细参数说明可以参看 zabbix 官方手册）

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977650379.png/wm)
![实验楼](https://dn-simplecloud.shiyanlou.com/87971515405128294-wm)

此时不要点击 Add 按钮，因为我们的 steps 还没有配置。选择 `Steps` 标签项，配置 web scenario 步骤信息，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977687272.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977692818.png/wm)


|参数|描述        | 
| ------------- |-------------|
|`Name`|步骤名称。用户宏和 {HOST。*}
|`URL`|连接到并能检索数据的 URL。
|`Query fields`|URL 的 HTTP GET 变量。指定为属性和值对。值自动被URL编码。
|`Post` |HTTP POST 变量。在表单数据模式下，指定为属性和值对。值自动被URL编码。
|`Variables`|用于 GET 和 POST 函数的步骤级变量。指定为属性和值对。格式：{macro} = value； {macro} = regex：<正则表达式>
|`Headers`|在执行请求时将自定义 HTTP 标头。
|`Follow redirects`|选中该复选框以遵循 HTTP 重定向。
|`Retrieve only headers`|将该复选框标记为仅从 HTTP 响应中检索标头。
|`Timeout`|定义了连接 URL 的最大时间和执行 HTTP 请求的最大时间。
|`Required string`|必需的正则表达式模式 除非检索的内容（HTML）匹配所需模式，否则步骤将失败。如果为空，则不执行检查。
|`Required status codes`|预期的 HTTP 状态码列表。如果 Zabbix 获取不在列表中的代码，则该步骤将失败。如果为空，则不执行检查。

点击 Add 按钮进入如下界面

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977697521.png/wm)

接着，选择 Authentication 选项配置验证方案。如下图：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977722095.png/wm)

点击 Add 即可添加成功：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515405573321-wm)


> 补充：其他实体的添加方法类似，根据具体需求进行添加，这里不做过多演示。


## 4. 编辑模板

在创建了一个模板后，我们可以根据具体的要求对它进行编辑操作，包括：**save、clone、delete 和 clear** 等。

我们点击刚才创建好的模板 `template` ，换到 `Macros` 界面可以看到下方有 `6` 个按钮，选择相应按钮就可以对模板进行编辑操作了。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977767163.png/wm)

|按钮 |说明|
|-|-|
|`Add`|添加模板
|`Update`|更新现有模板的属性
|`Clone`|根据当前模板的属性创建另一个模板，包括从链接模板继承的实体
|`Full clone`|根据当前模板的属性创建另一个模板，包括从链接模板继承并直接附加到当前模板的实体
|`Delete`|删除模板， 模板的实体保留在链接的主机中
|`Delete and clear`|从链接主机中删除模板及其所有实体
|`Cancel`|取消编辑模板属性

## 5. zabbix 主机链接模板

下面我们将会学习zabbix 主机链接模板。

### 5.1 主机链接模板

Host 链接模板之后，会继承模板里定义的 item，trigger 等实体，使用链接模板配置 Zabbix 监控会减少很多重复的工序，更加灵活。

在创建模板的时候就可以进行模板的链接了，选择主机，将我们之前创建的主机链接到模板中，点击下方的更新按钮即可。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977795115.png/wm)

**注：模板只能被链接到 host，不能链接到组里面。**

如果在创建模板的时候没有进行模板的链接，可以使用下面的操作方式，让指定主机链接模板。

点击 `Configuration->Hosts` 进入主机列表界面：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515407036375-wm)

点击 Host_shiyanlou 然后选择 Templates 标签。点击 `Link new templates` 旁的 `add` 按钮，选择至少一个模板进行添加更新。

![实验楼](https://dn-simplecloud.shiyanlou.com/87971515406906929-wm)

更新完成后，当前 host 就拥有了该模板所有的 item，trigger 等实体信息。

**唯一性**

+ 链接多个模板时，模板间不能含有相同的 items key。
+ trigger 和 graphs 中的 items 不能是来自多个模板。

### 5.2 多主机批量链接模板

将模板链接到多个主机，顺序点击：`configuration → Templates` 中选择 `Template shiyanlou` 模板，然后从 `other|group` 选项框中的相应组中选择主机，单击 `<` 并更新模板。
反之，如果在 `In` 中选择链接的主机，点击 `>` 并更新，从主机中取消链接模板。如图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977834939.png/wm)

**批量更新模板**

顺序点击：`Configuration — Hosts` 选择需要批量更新的主机，下方选择框选择 `Mass update`，切换到 `template` 界面，选择需要的模板，最后点击 `update` 即可。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977858886.png/wm)

### 5.3 解除模板链接

从主机取消链接模板，操作如下：顺序点击：`Configuration → Hosts`，选择编辑的主机切换到 template 界面，点击：`Unlink` 或者 `Unlink and clear` 按钮。最后点击更新即可。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977867176.png/wm)

+ `Unlink`：简单地移除与模板的关联，同时将所有实体留给主机。

+ `Unlink and clear`：删除与模板及其所有实体的关联。

## 6. 模板间的嵌套

下面我们将会学习模板间的嵌套。

### 6.1 概述

嵌套是指包含一个或多个其他模板的模板。

因为各种服务、应用等会分离出单个模板实体，所以最终可能会有很多的模板，这些模板都可能需要链接到更多的主机。为了简化，可以将这些模板链接到一个“嵌套”模板中。

### 6.2 配置嵌套模板

顺序点击：`Configuration → Templates`，选择目标模板，切换到 `linked templates` 选项中，`select` 需要的模板，然后点击 `Add`，最后更新即可。如下图：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1514977875358.png/wm)

配置完成后的模板会包含链接模板的所有实体。

要取消链接任何链接的模板，用相同的形式使用 `Unlink` 或者 `Unlink and clear` 按钮，然后单击更新即可。

## 7. 总结

通过本节实验，进一步对 Zabbix 的配置项中的 template 模板有了一定的认识，从中可以学习到如何去定义一个模板，以及模板的链接和嵌套等操作。
