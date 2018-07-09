---
show: step
version: 1.0
enable_checker: true
---
# Alertmanager 告警

## 1. 实验介绍

Prometheus 的告警功能分为两部分。一部分是产生警告，在 Prometheus Server 里配置告警规则来产生和发送警告到告警管理器（Alertmanager）。另一部分是管理警告，包括静默、抑制、聚合警告以及发送通知，这部分由告警管理器负责。

## 2. 实验知识点

- Prometheus Server 告警配置
- Alertmanager 安装和配置

### 2.1. Prometheus Server 告警配置

前面学习 Prometheus Server 配置的时候我们跳过了告警相关配置，包括告警规则和告警管理器配置，现在来继续学习。

#### 2.1.1. 告警规则配置

告警规则类似于记录规则，也是存放在单独的文件中，可以有多个。下面是一个示例：

```yaml
# 一个配置文件里包含多个组
groups:
- name: example # 组名
  # 触发规则列表
  rules:
  - alert: HighErrorRate # 警告名
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5 # 触发规则
    for: 10m # 规则触发持续多长时间发送告警
    # 告警附加标签
    labels:
      severity: page
    # 告警附加注释
    annotations:
      summary: High request latency
```

每次规则触发会产生警告，在未发送之前，警告处于待定（pending）状态。当某种警告的触发持续指定时长后，这些警告才会被发送出去。警告继承了时序的标签，附加标签可以覆盖这些标签，当然也可以新增。标签在告警管理器里用来分组，注释不会，它仅仅用来传递一些附加信息。

#### 2.1.2. 告警管理器配置

我们还需要配置告警管理器的地址，这样 Prometheus 才知道发送警告到哪。下面是一个示例：

```yaml
# 告警管理器配置
alerting:
  alertmanagers:
    - timeout: 10s # 发送超时时间
      path_prefix: / # 路径
      scheme: http # 协议
      # 告警管理器实例，一个或多个
      static_configs:
      - targets: ['localhost:9093']
```

## 3. 告警管理器

下面我们将会学习告警管理器。

### 3.1. 相关概念

告警管理器处理来自于 Prometheus Server 或其它客户端应用的警告。它会对警告进行去重（Deduplicating）、分组（Grouping）并路由（Routing）到正确的接收者，另外还能够静默（Silencing）和抑制（Inhibition）警告。

#### 3.1.1. 分组

分组将相似的警告归类到一起来通知。这在出现大规模服务中断的时候非常有用，这种时候大量出现故障的系统会同时发出成千上万的警告。经过分组后接收者只会收到少量的通知，但在通知详情里仍然可以看到具体是哪些服务和节点出现了问题。

分组规则、通知时效和通知接收者是通过配置文件里的路由树（Routing tree）来配置的。

#### 3.1.2. 抑制

抑制是当出现其它警告的时候压制当前警告的通知。比如当机房出现网络故障时，所有服务都将不可用而产生大量服务不可用警告，但这些警告并不能反映真实问题在哪，真正需要发出的应该是网络故障警告。当出现网络故障警告的时候，应当抑制服务不可用警告的通知。

抑制规则也是在配置文件里来配置。

#### 3.1.3. 静默

静默简单直接的在指定时段关闭告警。静默通过匹配器（Matcher）来配置，类似于路由树。警告进入系统的时候会检查它是否匹配某条静默规则，如果是则该警告的通知将忽略。

静默规则在告警管理器的 Web 界面里配置。

### 3.2. 安装配置

#### 3.2.1. 安装

告警管理器的安装跟 Prometheus Server 一样很简单，由于都是 Go 语言编写，只要下载编译好的对应平台的二进制包，解压即可。可从 [官网](https://prometheus.io/download/) 下载，实验楼也提供了 Linux 64 位平台的包 [alertmanager-0.15.0-rc.1.linux-amd64.tar.gz](http://labfile.oss.aliyuncs.com/courses/980/05/assets/alertmanager-0.15.0-rc.1.linux-amd64.tar.gz)。

可通过以下命令来下载和启动告警管理器。

```bash
wget http://labfile.oss.aliyuncs.com/courses/980/05/assets/alertmanager-0.15.0-rc.1.linux-amd64.tar.gz
tar -zxvf alertmanager-0.15.0-rc.1.linux-amd64.tar.gz
cd alertmanager-0.15.0-rc.1.linux-amd64
cp simple.yml alertmanager.yml
./alertmanager
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep alertmanager
  error: 没有启动 alertmanager
- name: check file
  script: |
    #!/bin/bash
	ls /home/shiyanlou/alertmanager-0.15.0-rc.1.linux-amd64/alertmanager.yml
  error: /home/shiyanlou/alertmanager-0.15.0-rc.1.linux-amd64 目录下没有 alertmanager.yml 文件
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5956timestamp1528873154019.png/wm)

`alertmanager` 命令默认会加载同目录下的配置文件 `alertmanager.yml`，如果找不到配置文件会报错。也可以通过 `--config.file` 参数来显示指定要使用的配置路径，支持相对路径。

启动之后，可通过地址 `http://localhost:9093/` 来访问告警管理器的 Web 界面。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5956timestamp1528873174587.png/wm)

`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/newweek5/05-22%20Alertmanager.mp4
@`

#### 3.2.2. 配置

告警管理器可通过命令行参数和配置文件来配置。命令行参数用来配置运行期间不可变的系统参数，配置文件用来配置抑制规则、通知路由和通知接收者等。

**命令行选项**

可通过 `./alertmanager -h` 来查看支持的命令行选项。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5956timestamp1528873193318.png/wm)

**可视化路由树编辑器**

当路由树比较复杂时通过文本不容易看清数据流向，可通过在线的 [可视化路由树编辑器](https://prometheus.io/webtools/alerting/routing-tree-editor/) 来生成图形方式的路由树。输入默认的配置文件内容，将得到以下的图形显示。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5956timestamp1528873208492.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5956timestamp1528873208846.png/wm)

**配置文件**

大部分的配置需要在配置文件里完成，配置文件在告警管理器运行期间如有更改，可以发送 `SIGHUP` 信号给告警管理器进程来重新加载配置。具体命令为 `kill -SIGHUP <pid>`，其中 \<pid> 为告警管理器进程 ID。

配置文件描述中会使用下面这些占位符：

- `<duration>`: 时长，格式符合正则表达式 [0-9]+(ms|[smhdwy])
- `<labelname>`: 标签名，格式符合正则表达式 [a-zA-Z_][a-zA-Z0-9_]*
- `<labelvalue>`: 标签值，可包含任意 Unicode 字符
- `<filepath>`: 文件路径
- `<boolean>`: 布尔值，true 或 false
- `<string>`: 普通字符串
- `<secret>`: 保密字符串，比如密码
- `<tmpl_string>`: 模板字符串，使用前需要进行变量替换
- `<tmpl_secret>`: 模板密码，使用前需要进行变量替换

顶层配置文件格式如下：

```yaml
# 全局配置，其下选项值会作为其他地方的默认值
global:
  # 分组多久没有接收到新告警则可认为告警已解除
  [ resolve_timeout: <duration> | default = 5m ]

  # SMTP 邮件发送地址、账号和密码
  [ smtp_from: <tmpl_string> ]
  [ smtp_smarthost: <string> ]
  [ smtp_hello: <string> | default = "localhost" ]
  [ smtp_auth_username: <string> ]
  [ smtp_auth_password: <secret> ]
  [ smtp_auth_identity: <string> ]
  [ smtp_auth_secret: <secret> ]
  [ smtp_require_tls: <bool> | default = true ]

  # 各种接收通知的媒体地址，包括 Slack、微信等
  [ slack_api_url: <string> ]
  [ victorops_api_key: <string> ]
  [ victorops_api_url: <string> | default = "https://alert.victorops.com/integrations/generic/20131114/alert/" ]
  [ pagerduty_url: <string> | default = "https://events.pagerduty.com/v2/enqueue" ]
  [ opsgenie_api_key: <string> ]
  [ opsgenie_api_url: <string> | default = "https://api.opsgenie.com/" ]
  [ hipchat_api_url: <string> | default = "https://api.hipchat.com/" ]
  [ hipchat_auth_token: <secret> ]
  [ wechat_api_url: <string> | default = "https://qyapi.weixin.qq.com/cgi-bin/" ]
  [ wechat_api_secret: <secret> ]
  [ wechat_api_corp_id: <string> ]

  # 默认的 HTTP 客户端配置
  [ http_config: <http_config> ]

# 模板文件路径，可配置多个，可使用通配符 *
templates:
  [ - <filepath> ... ]

# 告警路由树
route: <route>

# 接收者列表
receivers:
  - <receiver> ...

# 抑制规则列表
inhibit_rules:
  [ - <inhibit_rule> ... ]
```

告警路由树节点格式如下：

```yaml
# 接收者
[ receiver: <string> ]

# 用来分组的标签，所有标签值相同的警告会分为一组
[ group_by: [ <labelname>, ... ] ]

# 如果匹配上，是否继续匹配兄弟节点，子节点不会再匹配
[ continue: <boolean> | default = false ]

# 普通匹配规则，精确匹配所有指定标签的值
match:
  [ <labelname>: <labelvalue>, ... ]

# 正则表达式匹配规则，正则匹配所有指定标签的值
match_re:
  [ <labelname>: <regex>, ... ]

# 新组创建后等待多长时间再发送通知
[ group_wait: <duration> | default = 30s ]

# 首次通知发出后，后续如果有新警告加入到组，等待多长时间再发送通知
[ group_interval: <duration> | default = 5m ]

# 当通知成功发出后，如果分组警告为消除，隔多长时间再重复发送
[ repeat_interval: <duration> | default = 4h ]

# 子路由，会继承父节点的配置，也可以覆盖父节点配置
routes:
  [ - <route> ... ]
```

> 注意，根节点需要匹配所有警告，所以不能配置 match 或 match_re。

抑制规则格式如下：

```yaml
# 要被抑制的警告普通匹配规则，精确匹配所有指定标签的值
target_match:
  [ <labelname>: <labelvalue>, ... ]
# 要被抑制的警告正则表达式匹配规则，正则匹配所有指定标签的值
target_match_re:
  [ <labelname>: <regex>, ... ]

# 要触发抑制的警告普通匹配规则，精确匹配所有指定标签的值
source_match:
  [ <labelname>: <labelvalue>, ... ]
# 要触发抑制的警告普通匹配规则，精确匹配所有指定标签的值
source_match_re:
  [ <labelname>: <regex>, ... ]

# 被抑制警告和触发抑制警告的对应关系，当抑制触发时，只会抑制跟触发警告具有同样标签值的警告
[ equal: '[' <labelname>, ... ']' ]
```

接收者配置格式如下：

```yaml
# 接收者名字，唯一
name: <string>

# 接收者接收通知的方式，一个或多个
email_configs:
  [ - <email_config>, ... ]
hipchat_configs:
  [ - <hipchat_config>, ... ]
pagerduty_configs:
  [ - <pagerduty_config>, ... ]
pushover_configs:
  [ - <pushover_config>, ... ]
slack_configs:
  [ - <slack_config>, ... ]
opsgenie_configs:
  [ - <opsgenie_config>, ... ]
webhook_configs:
  [ - <webhook_config>, ... ]
victorops_configs:
  [ - <victorops_config>, ... ]
wechat_configs:
  [ - <wechat_config>, ... ]
```

上面只是列出了最核心的配置，完整配置格式请参考 [官方](https://prometheus.io/docs/alerting/configuration/) 文档。下面是一个具体的配置文件示例：

```yaml
global:
  smtp_smarthost: 'localhost:25'
  smtp_from: 'alertmanager@example.org'
  smtp_auth_username: 'alertmanager'
  smtp_auth_password: 'password'

templates: 
- '/etc/alertmanager/template/*.tmpl'

route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 3h
  receiver: team-X-mails
  routes:
  - match_re:
      service: ^(foo1|foo2|baz)$
    receiver: team-X-mails
    routes:
    - match:
        severity: critical
      receiver: team-X-pager

  - match:
      service: files
    receiver: team-Y-mails
    routes:
    - match:
        severity: critical
      receiver: team-Y-pager

  - match:
      service: database
    receiver: team-DB-pager
    group_by: [alertname, cluster, database]
    routes:
    - match:
        owner: team-X
      receiver: team-X-pager

    - match:
        owner: team-Y
      receiver: team-Y-pager

inhibit_rules:
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  equal: ['alertname', 'cluster', 'service']

receivers:
- name: 'team-X-mails'
  email_configs:
  - to: 'team-X+alerts@example.org'

- name: 'team-X-pager'
  email_configs:
  - to: 'team-X+alerts-critical@example.org'
  pagerduty_configs:
  - service_key: <team-X-key>

- name: 'team-Y-mails'
  email_configs:
  - to: 'team-Y+alerts@example.org'

- name: 'team-Y-pager'
  pagerduty_configs:
  - service_key: <team-Y-key>

- name: 'team-DB-pager'
  pagerduty_configs:
  - service_key: <team-DB-key>
  
- name: 'team-X-hipchat'
  hipchat_configs:
  - auth_token: <auth_token>
    room_id: 85
    message_format: html
    notify: true
```

#### 3.2.3. 通知模板

如果觉得内置的模板满足不了需求，可以自定义模板。类似于 Prometheus Server，告警管理器的模板也是用的 Go 语言模板语法。自定义模板属于高级话题，这里就不展开了，感兴趣的同学可以阅读 [官方文档](https://prometheus.io/docs/alerting/notifications/)。

## 4. 实验总结

本次实验中我们学习了告警管理器的安装和配置，接下来的挑战我们会实际来配置一下，检验学习成果。