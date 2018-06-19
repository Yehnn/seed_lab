---
show: step
version: 1.0
enable_checker: true
---
# Zabbix 报警配置

## 1. 实验介绍

#### 1.1 实验内容

本节实验将继续讲解 Zabbix 中一个重要的实体部分—— triggers（触发器）报警装置。其中包括了它的应用场景以及如何创建和实现等等知识点。

#### 1.2 实验知识点

+ triggers 触发器

+ triggers 表达式详解

+ triggers 严重性级别

+ triggers 的创建

+ triggers 依赖关系

+ triggers 处理

#### 1.3 推荐阅读

+ [zabbix 官方文档](https://www.zabbix.com/documentation/3.4/zh/manual/config/triggers)

## 2. triggers 触发器

下面我们将会学习triggers 触发器。

#### 2.1 triggers 介绍

在 `Zabbix` 逻辑结构中，`triggers` 触发器主要是在 `items` 监控项收集完数据之后，对数据进行自动评估，并作出判断的一个机制。

`triggers` 触发器中定义了一个表达式，来表示数据的阈值级别。一旦接收到的数据超过了定义的阈值，触发器就会被触发，或进入一个 `PROBLEM` 状态。后续就会采取 `action` 动作等。而当数据信息恢复到了严重级可接受的水平时，触发器就回到 `OK` 状态。

#### 2.2 triggers 状态

|值 | 描述
|-|-|
|`OK`|触发器的正常状态
|`PROBLEM`|非正常状态。比如：数据库不能提供服务，系统负载过高等

## 3. triggers 表达式详解

下面我们将会学习triggers 表达式。

### 3.1 概述

triggers 触发器最核心的机制就是在于阈值表达式的定义。

表达式格式：

```
 {<server>:<key>.<function>(<parameter>)}<operator><constant>

 #{主机：key.函数(参数)}<表达式>常数
```

### 3.2 function 函数

**说明：在 function 函数中有一个特殊的符号：**`#`**，后面一般接一个 `num`（数字），表示最新的 `num` 个数。**

1.avg()

指定判断周期中项目的平均值。

```
格式：avg(sec|#num ,<time_shift>)
```

+ sec or #num ：表示秒或最新值个数
+ time_shift ：时间偏移

eg 1：

```
avg(#10)  # 最新 10 个数值的平均值
avg(1h,1d) # 一天前的一个小时的平均值
```

2.count()

判断周期中值的个数。

```
格式：count (sec|#num,<pattern>,<operator>,<time_shift>)

```

+ pattern（可选）：模式（整型：精确匹配；浮点型：误差在 0.000001）
+ operator（可选）：大于（gt）、小于（lt）、等于（eq）、大于等于（ge）、小于等于（le）、不等于（ne）、like（只要包含 pattern 就匹配）、band（按位与）

> `band` 做第三个参数时，第二个 pattern 参数可以用两个数字表示， 以 `/` 分隔: number_to_compare_with/mask。

eg 2:

```
count(10m,12,"gt",1d) #一天前的前十分钟值大于 12 的个数
count(10m,6/7,"band") #过去 10 分钟值最低三个有效位是 '110' (十进制)的个数。
```

3.diff()

比较新获取的数据和历史数据是否相同。

返回值：

+ 1 两值不等
+ 0 两值相等

4.forecast()

估计未来值、最大值、最小值、差值、平均值。

```
格式：forecast (sec|#num,<time_shift>,time,<fit>,<mode>)
```

+ time：估计所需的时间
+ fit（可选）：匹配历史数据的函数
+ mode（可选）：要求输出（支持：value（默认）、max、min、delta（最大最小值）、avg（平均值））

eg 4：

```
forecast(1h,1d,12h) # 根据一天前的一个小时值估计十二个小时后的值
forecast(1h,,2h,polynomial3,max) # 根据过去一小时并按照三次多项式方式估计两小时的最大值
```

5.last()

判断周期中最近的值。

```
格式：last (sec|#num,<time_shift>)
```

**注：此处的 `#num` 的含义是指第 num 个值。**

eg 5：

```
last() # 等同于 last(#1)
last(#3) # 第三个最新值
```

### 3.3 操作符

| 优先级  | 操作    | 定义   |
| ---- | ----- | ---- |
| 1    | `-`   | 一元减号 |
| 2    | `not` | 逻辑非  |
| 3    | `*`   | 算数乘  |
| 3    | `/`   | 算数除  |
| 4    | `+`   | 算数加  |
| 4    | `-`   | 算数减  |
| 5    | `<`   | 小于   |
| 5    | `>`   | 大于   |
| 5    | `<=`  | 小于等于 |
| 5    | `>=`  | 大于等于 |
| 6    | `=`   | 等于   |
| 6    | `<>`  | 不等于  |
| 7    | `and` | 逻辑与  |
| 8    | `or`  | 逻辑或  |

> `not`, `and` 和 `or` 运算符区分大小写，而必须为小写。

### 3.4 举例

eg 1：服务器的处理器负载过高

```
{www.shiyanlou.com:system.cpu.load[all,avg1].last()}>5
```

服务器：`www.shiyanlou.com`，指定监控项的 key ，调用函数 last 引用最近的值，当服务器上的处理器负载的最近测量值大于 5 就会触发，进入 `PROBLEM` 状态。

eg 2：服务器无法访问

```
{www.shiyanlou.com:icmpping.count(30m,0)}>5
```

服务器在最近 30 min 内无法访问的次数达到 5 次以上，就会触发。

## 4. triggers 严重性级别

下面我们将会学习 triggers 严重性级别。

### 4.1 严重性

triggers 的触发功能也会根据 items 监控项的数据信息的严重程度进行等级划分。Zabbix 默认定义了 6 个触发器严重性，分别为：`Not classified`、`Information`、`Warning`、`Average`、`High`、`Disaster`。

|严重性| 定义|  颜色
|-|-|-|
|`Not classified`|未知|  灰色
|`Information`|一般信息|  浅绿
|`Warning`    |警告    |  黄色
|`Average`    |一般问题|  橙色
|`High`       |严重问题|  红色
|`Disaster`   |灾难,会带来损失|  深红

### 4.2 设置严重性

系统默认的严重性的名称和颜色是可以在 Zabbix web 后台进行自定义修改的。（系统默认的已经比较清晰，建议不用自定义）

**自定义触发器的严重性**

1.顺序点击：`Administration → General → Trigger severities`，然后就可以自定义修改 `custom severity`和`color`。修改完后进行保存即可。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515049874667.png/wm)

2.如果更改了严重性名称，则在所有语言环境中都使用自定义名称，并且需要额外的手动翻译。

> 如果使用 Zabbix 前端翻译，默认情况下，自定义严重性名称将覆盖已翻译的名称。

3.添加内容到 `frontend.po`。

```
<frontend_dir> / locale / <required_locale> /LC_MESSAGES/frontend.po

# 添加 2 行：

msgid "<Custom name>"
msgstr "<translation string>"
```

+ msgid：匹配新的自定义严重性名称
+ msgstr：特定语言的翻译

4.按照 `<frontend_dir> / locale / README` 中的描述创建 `.mo` 文件

**注：因为每个严重性名称更改后都要执行此过程，所以这里不做名称修改操作，感兴趣的可以下来研究。**

## 5. triggers 创建

**创建触发器步骤:**

1.顺序点击： `Configuration → Hosts`

2.选择需要的 hosts 相关行的 trigger

3.点击右上角：`create trigger`

4.在 triggers 表中填写相关参数

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515049958222.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515049989587.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515049978536.png/wm)

**相关参数说明：**

+ Name ：在列表或其他地方显示的触发器名称。

  + 可以包含`宏变量`：
    {HOST.HOST}，{HOST.NAME}，{HOST.CONN}，{HOST.DNS}，{HOST.IP}，{ITEM.VALUE},{ITEM.LASTVALUE}，{$MACRO}
  + `$1, $2…$9`：可以被用来关联表达式的常量
  + 示例：
     + `name`：Cpu load too high $1 on {HOST.NAME}
     + 表达式 `$1`：host:system.cpu.load[percpu,avg1].avg(180)}>2
     + 显示为：CPU load too high on host for 3 minutes

+ Severity ：设置触发器的严重级。

+ Expression ：描述触发器的逻辑表达式。eg：{host:system.cpu.load.avg(180)}>2

+ OK event generation：ok 事件生成选项。（Expression，ok 事件基于和问题事件相同的表达式）

+ PROBLEM event generation mode：

+ OK event closes：

+ Tags：设置自定义标签来标记触发事件。

+ Allow manual close：允许手动关闭由触发器生成的问题事件。

+ URL：若不为空，则输入的网址可以在点击监控中的触发器名称的时候链接到这里的网址。

+ Description：描述

+ Enabled：没选中则表示禁止此触发器

相关参数配置完成后点击：`Add` 即可完成触发器的一般配置。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515050025487.png/wm)

## 6. triggers 依赖关系

下面我们将会学习triggers 依赖关系。

### 6.1 概述

有这样一个用插座给手机和 Mac 充电的应用场景，组件包括了：手机、Mac、插座，其中我们假定每一个组件都有一个类似 Zabbix 的 trigger 触发装置来进行报警行动。某天在充电的时候发现插座的开关没有开启，这时手机和 Mac 都没法充电，因此，这三个组件就会被触发从而发出警报信息，但是在检测中，只有插座的警报信息是有用的，所以在这种情况下，Zabbix 机制就出现了一种依赖关系来灵活处理这种类似的情况。

接着上面的场景，我们分别给手机和 Mac 配置依赖关系到插座的相应触发器，有了依赖关系后，当插座的触发器处于 `PROBLEM` 状态时，手机和 Mac 端不会改变状态，也就不会发送警告通知。

注意的几点：

+ 当插座和手机同时故障，并且配置了依赖关系，Zabbix 不会执行手机端的触发器的动作。
+ 触发器的依赖可以从任何主机的依赖添加到另一个主机的触发器，但是不能发生死循环依赖。例如：触发器 A 依赖触发器 B，B 依赖 C ，触发器 C 又依赖 A 。这种情况在 Zabbix 中不会出现，系统自动会报错，无法进行依赖。
+ 触发器依赖可以从模板的触发器添加到主机触发器。

### 6.2 配置

下面来动手定义依赖关系。

打开触发器 trigger 表单，选择 `Dependencies` 选项。点击：`add` 按钮选择一个或多个需要建立依赖关系的 triggers 。最后更新表单即可。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515050046688.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515050054204.png/wm)

这样就建立好了一个依赖关系。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515050060967.png/wm)

### 6.3 多依赖关系

还是刚才的场景，在手机充电过程中会需要到充电线这个组件，同样，充电线也具有触发器机制。这时，发生插线板故障，在没有依赖关系的情况下，插座、充电线和手机均会被触发从而发送警报通知。

因此，知道了依赖关系这一机制下，我们可以分别定义两个依赖关系。

+ 手机故障的触发器依赖于充电线故障的触发器
+ 充电线故障的触发器依赖于插座故障的触发器

这样，Zabbix 只用递归执行检查就可以快速发现问题根源。

## 7. triggers 处理

下面我们将会学习triggers 处理。

### 7.1 批量更新

批量更新可以省去一次次更改多个触发器。

操作步骤如下：

+ 选中要更新的触发器
+ 点击：批量更新 `Mass update` 按钮
+ 标记要更新的属性
+ 标记要更新属性的新值，点击：`Update` 更新按钮

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515050106710.png/wm)

> 注：参数 `Replace dependencies` 和 `Replace tags`：将使用批量更新中指定的触发器依赖关系和标签（如果有）来替换已存在的触发器依赖关系和标签。

### 7.2 事件 event

在前面我们学习了如何去配置监控项 items 和触发器 triggers 等实体项，这些都是为了报警做的准备，下面就一起来学习下报警是如何实现的。

在 Zabbix 逻辑结构中，当 trigger 被触发时会触发一个事件 event ，而事件是基于时间戳进行标记的，然后会采取一个动作 action 进行发送通知。

其中，事件主要基于以下几种：

+ 触发器(trigger)事件：`Trigger events`

 触发器状态每次发生改变，都会生成相应“事件”，且通常包含详细信息，如发生的时间及新的状态等。

 触发器会创建两种类型的事件：`问题（problem）事件`和`正常（OK）事件`。

+ 发现(discovery)事件：`Discovery events`

Zabbix 会周期性地扫描“网络发现规则”中指定的 IP 范围，一旦发现主机或服务，就会生成一个或几个发现事件。发现事件有 8 类：

  + `Service Up`：Zabbix 检测到活跃的服务
  + `Service Down`：Zabbix 无法检测到服务
  + `Host Up`：活跃的服务中至少有一个 IP
  + `Host Down`：所有服务都没有响应
  + `Service Discovered`：服务在维护时间后恢复或第一次被发现
  + `Service Lost`：服务在运行后丢失
  + `Host  Discovered`：主机在维护时间后恢复或第一次被发现
  + `Host Lost`：主机在运行后丢失

+ 主动 agent 自动发现事件：Auto registtation events

当一个此前状态未知的主动 agent 发起检测请求时会生成此类事件。

+ 内部事件：Internal events

Item 变成不再支持的状态或 trigger 变成未知状态。

### 7.3 事件通知

事件通知主要包括了：`Media types` 和 `Actions`。

#### 7.3.1 Media types

**通知媒介类型**

+ Email ：电子邮件，以邮件方式传送信息
+ SMS：手机短息，需要短息网关设备
+ Jabber：是一个开放的、基于 XML 的协议，能够实现基于 Internet 或 LAN 的即时通讯服务
+ 自定义通知脚本

**媒介操作（operation）方式**

+ send message

发送警告信息。

+ remote command

执行 server 端的命令，相对于 agent 端为远程命令。

**定义通知媒介**

顺序点击： `Aministration -> Media types`，可以看到系统默认的三个通知报警媒介，但现在我们选择自定义，点击：`create media type`。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515050176035.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515050183271.png/wm)

+ SMTP server：SMTP 邮件服务器，localhost 表示使用本服务器做邮件服务器就只有本系统内的用户才能收到邮件
+ SMTP helo：SMTP helo 值，通常情况下是顶级域名。SMTP 服务器间需要交换数据时，先会探测对发是否在线，一般和 SMTP 邮件服务器配置成一样
+ SMTP email：发件人

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515050209293.png/wm)

#### 7.3.2 Actions

当一个事件的发生而涉及到发送通知的时候，则需要配置一个动作（action）来执行这个事件的发送操作。

Action 主要是由指定条件（condition）和各种 operations （操作）组成，当满足条件之后就执行一个 action。

> 指定的 condition 的具体内容大家可以参照[官方文档](https://www.zabbix.com/documentation/3.4/manual/config/notifications/action/conditions)

**定义操作（operations）**

顺序点击：`configuration -> Actions -> 选择事件来源 -> create action`

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515050415087.png/wm)

1.Action

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515052985952.png/wm)

+ Name：唯一动作名称
+ conditions：动作条件
+ New condition：新的操作条件
+ Enable：选中则启用该操作

2.Operations

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515053021512.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515053027157.png/wm)

+ Default operation step duration：默认操作步骤时间
+ Default subject：默认的消息主题，可以使用宏
+ Default message：默认的消息，可以包含宏
+ Pause operations while in maintenance：选中此复选框以在维护期间延迟操作的开始。
+ Operations：添加操作
  + steps：分配操作的步骤
  + Step duration：步骤的自定义持续时间
  + Operation type：操作类型
    + send message：发送信息
    + remote command：远程命令

3.Recovery operations

恢复操作意为允许在问题得到解决的时候收到通知。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515053060180.png/wm)

+ Default subject：默认邮件主题用于恢复通知，可以使用宏
+ Default message：恢复通知的默认消息，可以使用宏
+ operations：显示恢复操作的详细信息，若需要配置新的回复操作，点击新建

4.Acknowledgement operations

允许在确认问题的时候收到通知。可以用于 triggers 事件源的操作，同时支持发送消息和远程命令。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515053076016.png/wm)

+ Default subject：用于确认通知的默认消息主题，可以使用宏
+ Default message：确认通知的默认消息，可以使用宏
+ operations：显示确认操作的详细信息，若需要配置新的回复操作，点击新建

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515053095201.png/wm)

至此 action 配置完成。

#### 7.3.3 绑定用户

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515050313095.png/wm)

点击：`Add`，填写参数

+ type：媒介类型，可以选择之前创建的
+ send to：收件人
+ when active：报警时间设定
+ Use if severity：严重性类型，只接收指定的类型
+ Status：媒介状态 Enabled（启用中）Disabled（已禁用）

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4128timestamp1515050318446.png/wm)

#### 7.3.4 查看触发器状态

在 `Monitoring —> Triggers` 查看触发器的状态。（因为我们自定义的 trigger 目前还没有任何警报信息，所以此时就没有报警的信息，不过可以看看其他主机的情况）

## 8. 总结

通过本节实验主要的可以了解到什么是触发器机制，它是如何工作，以及各个触发器之间的依赖关系等等。
