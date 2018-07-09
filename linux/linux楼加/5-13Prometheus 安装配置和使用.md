---
show: step
version: 1.0
enable_checker: true
---
# Prometheus 安装配置和使用

## 1. 实验介绍

本次实验我们将先安装 Prometheus，然后学习 Prometheus 的常用配置，最后学习 Prometheus 的基本用法。

## 2. 实验知识点

- 安装
- 配置
- 使用

## 3. Prometheus 安装

下面我们将会学习Prometheus 安装。

### 3.1. 二进制安装包方式

二进制安装包方式非常简单，实验环境里推荐使用这种方式。首先下载安装包 [prometheus-2.2.1.linux-amd64.tar.gz](http://labfile.oss.aliyuncs.com/courses/980/05/assets/prometheus-2.2.1.linux-amd64.tar.gz)（实验楼提供的是 64 位 Linux 平台的安装包，其它平台可从 [官网](https://prometheus.io/download/) 下载），然后解压即可。

```bash
wget http://labfile.oss.aliyuncs.com/courses/980/05/assets/prometheus-2.2.1.linux-amd64.tar.gz
tar xvfz prometheus-2.2.1.linux-amd64.tar.gz
cd prometheus-2.2.1.linux-amd64
./prometheus
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep prometheus
	netstat -lunat |grep 9090 
  error: 没有启动 prometheus
```

上面的命令会在默认的 9090 端口启动 Prometheus 服务，打开地址 http://localhost:9090/ 即可看到其 Web 界面。

### 3.2. 源码方式

Prometheus 是开源的，因此可以下载源码到本地来编译安装。这种方式比较麻烦，适合想学习源码或做二次开发的人。感兴趣的同学可以自行研究，这里就不做讲解了。

### 3.3. Docker 方式

如果当前部署环境支持 Docker，那么可以采取 Docker 方式来运行 Prometheus 服务。使用下面的命令来启动一个 Prometheus 容器：

```bash
docker run -p 9090:9090 prom/prometheus
```

上面把容器内的 9090 端口映射到了宿主机的 9090 端口，因此可以在宿主机上通过 http://localhost:9090/ 来访问容器内的 Prometheus 服务。

> Docker 的使用后面会有一系列专门的实验来讲解，不熟悉的同学可以先跳过。

## 4. Prometheus 配置

下面我们将会学习Prometheus 配置。

### 4.1. 概述

执行 `prometheus` 命令的时候可以通过参数 `--config.file` 来指定配置文件路径，默认会使用同目录下的 `prometheus.yml` 文件。Prometheus 服务运行过程中如果配置文件有改动，可以给服务进程发送 `SIGHUP` 信号来通知服务进程重新从磁盘加载配置。这样无需重启，可以避免中断服务。

下面会逐一讲解 Prometheus 的核心配置，示例中用到的各种占位符包括：

- `<boolean>`: 布尔值，true 或 false
- `<duration>`: 持续时间，格式符合正则表达式 [0-9]+(ms|[smhdwy])
- `<labelname>`: 标签名，格式符合正则表达式 [a-zA-Z_][a-zA-Z0-9_]*
- `<labelvalue>`: 标签值，可以包含任意 unicode 字符
- `<filename>`: 文件名，任意有效的文件路径
- `<host>`: 主机，可以是主机名或 IP，后面可跟端口号
- `<path>`: URL 路径
- `<scheme>`: 协议，http 或 https
- `<string>`: 字符串
- `<secret>`: 密钥，比如密码
- `<tmpl_string>`: 模板字符串，里面包含需要展开的变量

配置文件格式为 YAML，下面是一个配置文件示例：

```yaml
# 全局配置对所有其它地方有效，作为其它地方配置的默认值。
global:
  # 抓取间隔
  [ scrape_interval: <duration> | default = 1m ]

  # 抓取超时时间
  [ scrape_timeout: <duration> | default = 10s ]

  # 规则评估间隔
  [ evaluation_interval: <duration> | default = 1m ]

# 抓取配置
scrape_configs:
  [ - <scrape_config> ... ]

# 规则配置文件列表，可使用通配符 *
rule_files:
  [ - <filepath_glob> ... ]

# 告警管理器配置
alerting:
  alert_relabel_configs:
    [ - <relabel_config> ... ]
  alertmanagers:
    [ - <alertmanager_config> ... ]
```

规则包括记录规则和报警规则，两者处理比较类似，所以放在一起。Prometheus 会定期评估规则，对于记录规则会在对应的时序里插入采样值，对于报警规则会判断条件是否满足，如果满足则发送警告到警告管理器（Alertmanager）。报警规则和告警管理器配置相关的内容我们会留到后续“Prometheus 告警”实验里再学习。

### 4.2. 抓取配置

抓取配置可以有多个，一般来说每个任务（Job）对应一个配置。单个抓取配置的格式如下：

```yaml
# 任务名
job_name: <job_name>

# 抓取间隔，默认为对应全局配置
[ scrape_interval: <duration> | default = <global_config.scrape_interval> ]

# 抓取超时时间，默认为对应全局配置
[ scrape_timeout: <duration> | default = <global_config.scrape_timeout> ]

# 协议，默认为 http，可选 https
[ scheme: <scheme> | default = http ]

# 抓取地址的路径，默认为 /metrics
[ metrics_path: <path> | default = /metrics ]

# 抓取地址的参数
params:
  [ <string>: [<string>, ...] ]

# 是否尊重抓取回来的标签，默认为 false
[ honor_labels: <boolean> | default = false ]

# 静态目标配置
static_configs:
  [ - <static_config> ... ]

# 单次抓取的采样值个数限制，默认为 0，表示没有限制
[ sample_limit: <int> | default = 0 ]
```

`honor_labels` 表示是否尊重抓取回来的标签。当抓取回来的采样值的标签值跟服务端配置的不一致时，如果该配置为 true，则以抓取回来的为准。否则以服务端的为准，抓取回来的值会保存到一个新标签下，该新标签名在原来的前面加上了“exported_”，比如 exported_job。

`static_configs` 下配置了该任务要抓取的所有实例，按组配置，包含相同标签的实例可以分为一组，以简化配置。单个组的配置格式如下：

```yaml
# 目标地址列表，地址由主机+端口组成
targets:
  [ - '<host>' ]

# 标签列表
labels:
  [ <labelname>: <labelvalue> ... ]
```

抓取目标除了采用静态配置方式，还可以动态发现。动态发现依赖于一个服务发现服务（比如 Consul，可以从这个服务里查询到目前系统里的服务列表），适合监控目标非常多并且经常变化的场景。因为使用场景比较少，在以后需要的时候大家可以去进一步研究。

### 4.3. 记录规则配置

记录规则允许我们把一些经常需要使用并且查询时计算量很大的查询表达式，预先计算并保存到一个新的时序。查询这个新的时序比从原始一个或多个时序实时计算快得多，并且还能够避免不必要的计算。在一些特殊场景下这甚至是必须的，比如仪表盘里展示的各类定时刷新的数据，数据种类多且需要计算非常快。

记录规则配置文件的格式如下：

```yaml
groups:
  [ - <rule_group> ]
```

记录规则配置按组来组织，一个组下的所有规则按顺序定时执行。单个组的格式如下：

```yaml
# 组名，在文件内唯一
name: <string>

# 规则评估间隔，默认为对应的全局配置
[ interval: <duration> | default = global.evaluation_interval ]

rules:
  [ - <rule> ... ]
```

每个组下包含多条规则，格式如下：

```yaml
# 规则名称，也就是该规则产生的时序数据的度量指标名
record: <string>

# PromQL 查询表达式，表示如何得到采样值
expr: <string>

# 关联标签
labels:
  [ <labelname>: <labelvalue> ]
```

### 4.4. 配置验证

Prometheus 的配置项比较多，容易出错，所以它提供了 `promtool` 工具来验证配置的正确性。该工具可使用下面的命令来安装：

```bash
go get github.com/prometheus/prometheus/cmd/promtool
```

> 请安装最新版的 Go 语言，低版本可能无法编译成功。由于需要从 GitHub 下载源码，时间会比较长，尽量使用代理。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5951timestamp1528872546300.png/wm)

安装完成后，可使用 `promtool check config <config-files>` 和 `promtool check rules <rule-files>` 来分别检查主配置文件和规则配置文件。

## 5. Prometheus 使用

学会安装和配置之后，接下来我们通过使用 Prometheus 监控其自身来学习它的基本用法。

### 5.1. 配置 Prometheus 监控其自身

Prometheus 服务本身也通过路径 `/metrics` 暴露了其内部的各项度量指标，我们只需要把它加入到监控目标里就可以。

```yaml
global:
  # 全局默认抓取间隔
  scrape_interval: 15s

scrape_configs:
  # 任务名
  - job_name: 'prometheus'

    # 本任务的抓取间隔，覆盖全局配置
    scrape_interval: 5s

    static_configs:
      # 抓取地址同 Prometheus 服务地址，路径为默认的 /metrics
      - targets: ['localhost:9090']
```

配置完成后启动服务：

```bash
./prometheus
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5951timestamp1528872568304.png/wm)

可打开地址 `http://localhost:9090/metrics` 来确认是否有抓取到数据。

### 5.2. 使用表达式浏览器来查询数据

在本地打开页面 `http://localhost:9090/`，会自动跳转到 Graph 页。在这个页面里可以执行表达式来查询数据，数据展示方式支持 Console 文本方式和 Graph 图形方式。可从 `Execute` 后面的下拉列表里选择某个度量指标，或者手动输入任意合法的 PromQL 表达式。

比如选择或输入 `prometheus_target_interval_length_seconds` 会查询到该度量指标相关的所有时序，这里共有四个。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5951timestamp1528872585204.png/wm)

Console 方式列出匹配的所有时序，以及每个时序的最新采样值。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5951timestamp1528872604842.png/wm)

Graph 方式以图形式展示匹配的所有时序最近一段时间内的采样值，默认为最近一小时。

`prometheus_target_interval_length_seconds` 这个度量指标的含义是实际抓取目标时的间隔秒数。可以使用表达式 `prometheus_target_interval_length_seconds{quantile="0.99"}` 来查询 0.99 分位线的采样值，也就是小于这个采样值的数量低于总数的 99%。使用表达式 `count(prometheus_target_interval_length_seconds)` 可以查询到该度量指标包含的采样值个数。关于查询表达式的更多语法后续实验会讲到。

`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/newweek5/05-17%20Prometheus.mp4
@`

## 6. 实验总结

本次实验我们学会了 Prometheus 的安装、配置和基本查询，接下来我们将学习 PromQL 查询语言。