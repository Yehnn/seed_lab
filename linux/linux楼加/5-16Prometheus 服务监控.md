---
show: step
version: 1.0
enable_checker: true
---
# Prometheus 服务监控

## 1. 实验介绍

前面我们只使用 Prometheus 来监控其自身状态，算是小试牛刀。本次实验我们来学习如何使用 Prometheus 监控各种应用或服务。Prometheus 支持多种方式，包括在应用内插入监测代码来提供度量指标（Metric）服务端点（Endpoint）或推送度量指标到 Prometheus 网关，对于不能修改代码的第三方系统可以使用导出器（Exporter）来把这些系统的度量指标导出到 Prometheus 里。

## 2. 实验知识点

- 使用客户端库来提供度量指标服务端点
- 推送度量指标到 Prometheus 网关
- 使用导出器来导入第三方系统的度量指标

## 3. 使用客户端库来提供度量指标服务端点

应用要被监控，需要在应用代码里插入监测代码来暴露应用的各项度量指标。不过不用从零开始去编写监测代码，Prometheus 提供了多种语言的客户端库来简化这项工作。这些客户端库支持 Prometheus 的各种度量指标类型，它们在应用内提供了一个 HTTP 服务端点（Endpoint）。

### 3.1. 支持语言的客户端库列表

官方提供了下面这些语言的客户端库：

- [Go](https://github.com/prometheus/client_golang)
- [Java or Scala](https://github.com/prometheus/client_java)
- [Python](https://github.com/prometheus/client_python)
- [Ruby](https://github.com/prometheus/client_ruby)

社区维护的有以下这些：

- [Bash](https://github.com/aecolley/client_bash)
- [C++](https://github.com/jupp0r/prometheus-cpp)
- [Common Lisp](https://github.com/deadtrickster/prometheus.cl)
- [Elixir](https://github.com/deadtrickster/prometheus.ex)
- [Erlang](https://github.com/deadtrickster/prometheus.erl)
- [Haskell](https://github.com/fimad/prometheus-haskell)
- [Lua for Nginx](https://github.com/knyar/nginx-lua-prometheus)
- [Lua for Tarantool](https://github.com/tarantool/prometheus)
- [.NET / C#](https://github.com/andrasm/prometheus-net)
- [Node.js](https://github.com/siimon/prom-client)
- [PHP](https://github.com/Jimdo/prometheus_client_php)
- [Rust](https://github.com/pingcap/rust-prometheus)

当 Prometheus 从度量指标服务端点抓取数据的时候，客户端库会发送跟踪的所有度量指标的当前状态。由于涉及语言非常多，下面我们以 Python 客户端库为例来讲解用法，其它语言的客户端库用法请参考相关文档。不熟悉 Python 语言的可以先跳过，后面有专门的实验来介绍 Python 语言。

### 3.2. Python 客户端库用法

[Python 客户端库](https://github.com/prometheus/client_python) 由 Prometheus 官方提供，支持 Python 2 和 3。

首先，使用 `pip` 命令安装客户端库。

```bash
sudo pip3 install --upgrade pip
sudo pip3 install prometheus_client
```

```checker
- name: check pip
  script: |
    #!/bin/bash
	pip3 list|grep prometheus
  error: 没有安装 prometheus-client
```

然后，编写下面的 Python 脚本 `prometheus.py`。

```python
from prometheus_client import start_http_server, Summary
import random
import time

# 创建一个汇总类型的度量指标
REQUEST_TIME = Summary('request_processing_seconds', 'Time spent processing request')

# 使用度量指标的计时装饰器来采集函数的执行时长
@REQUEST_TIME.time()
def process_request(t):
    """模拟真实业务处理的时间消耗"""
    time.sleep(t)

if __name__ == '__main__':
    # 启动度量指标端点服务
    start_http_server(8000)
    # 模拟真实服务不断处理请求
    while True:
        process_request(random.random())
```

最后，运行脚本。注意：如果 8000 端口被占用的话，要先杀掉占用端口的程序。

```checker
- name: check file
  script: |
    #!/bin/bash
	ls /home/shiyanlou/prometheus.py
  error: /home/shiyanlou 目录下没有 prometheus.py 文件
```

```bash
python3 prometheus.py
```

这里使用 `python3` 来运行，该脚本也支持 Python 2。访问度量指标服务端点 http://localhost:8000/ ，结果如下：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5954timestamp1528872926369.png/wm)

可以看到里面除了我们显示定义的度量指标 `request_processing_seconds`，还多出了一些诸如内存占用、CPU 消耗、打开文件数等进程相关的度量指标。这些是 Python 客户端库自动采集的，不过只有在 Linux 平台运行的时候才会有。

结果里“# HELP”和“# TYPE”打头的行是接下来的度量指标的描述和类型。我们显示定义的 request_processing_seconds 度量指标的类型为 summary，因此出现了两个时间序列的采样值，分别是 request_processing_seconds_count（总数）和 request_processing_seconds_sum（总和）。像内存占用、打开文件数这些都是监测当前状态，所以类型为 gauge，而 CPU 消耗需要监测自程序启动以后消耗的 CPU 时间，类型自然为 counter。

除了上面使用的 Summary 类型的度量指标，Python 客户端库还支持 Counter、Gauge 和 Histogram 其它几种。更多用法请参考 Python 客户端库 GitHub 仓库的文档。

## 4. 推送度量指标到 Prometheus 网关

对于一些短生命周期的应用程序，比如批处理脚本、定时任务等，无法通过抓取的方式来获取它们的度量指标。这时可以采用推送方式，应用程序把采集到的度量指标主动推送到 Prometheus 网关里暂存，Prometheus Server 定时从网关里去抓取这些指标。

### 4.1. 安装和启动 Prometheus 网关

Prometheus 网关只暂存度量指标，不做任何聚合或分布式计数这样的计算。它只是原样保存推送过来的度量指标并等待下次被 Prometheus Server 抓走。

从 Prometheus 网关 [GitHub 仓库](https://github.com/prometheus/pushgateway/releases) 下载对应平台的安装包，实验楼提供了 Linux 平台的安装包 [pushgateway-0.4.0.linux-amd64.tar.gz](http://labfile.oss.aliyuncs.com/courses/980/05/assets/pushgateway-0.4.0.linux-amd64.tar.gz)。解压之后运行 `pushgateway` 这个可执行程序就可以启动网关，默认在 9091 端口监听。

```bash
wget http://labfile.oss.aliyuncs.com/courses/980/05/assets/pushgateway-0.4.0.linux-amd64.tar.gz
tar -zxvf pushgateway-0.4.0.linux-amd64.tar.gz
cd pushgateway-0.4.0.linux-amd64/
./pushgateway
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5954timestamp1528872948360.png/wm)

### 4.2. 推送度量指标到 Prometheus 网关

#### 4.2.1. 配置 Prometheus Server 从网关抓取数据

我们需要在 Prometheus Server 的配置文件里添加一个 job 来从网关抓取数据，参考配置如下：

```yaml
...

scrape_configs:
  ...

  - job_name: 'pushgateway'
    scrape_interval: 5s
    honor_labels: true
    static_configs:
      - targets: ['localhost:9091']
```

```checker
- name: check content
  script: |
    #!/bin/bash
	grep pushgateway /home/shiyanlou/prometheus-2.2.1.linux-amd64/prometheus.yml
  error: /home/shiyanlou/prometheus-2.2.1.linux-amd64/prometheus.yml 内容不对
```

#### 4.2.2. 使用客户端库来推送

大部分客户端库都支持向 Prometheus 网关推送数据。下面我们仍然以 Python 客户端为例。

首先，编写一个 Python 脚本 `pushgateway.py`，内容如下：

```python
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway

registry = CollectorRegistry()
g = Gauge('job_last_success_unixtime', 'Last time a batch job successfully finished', registry=registry)
g.set_to_current_time()
push_to_gateway('localhost:9091', job='batchA', registry=registry)
```

该脚本运行的时候会推送一个度量指标 `job_last_success_unixtime` 到网关，该指标表示任务上次成功执行的时间，属于 job batchA。因为默认的 registry 包含了一些其它指标，比如进程相关的，所以这里新建了一个。

可以从网关日志里看到推送数据后触发了数据的持久化。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5954timestamp1528872977990.png/wm)

大约几秒后，在 Prometheus 的 Web 控制台里能够查询到该度量指标，证明推送和拉取都成功了。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5954timestamp1528872994770.png/wm)

#### 4.2.3. 从命令行推送

除了使用客户端库在代码里推送，我们还可以从命令行通过 HTTP API 来推送。

推送一个度量指标 some_metric，采样值为 3.14。

```bash
echo "some_metric 3.14" | curl --data-binary @- http://localhost:9091/metrics/job/some_job
```

推送多个度量指标到 job=some_job 且 instance=some_instance 的时间序列里。

```bash
cat <<EOF | curl --data-binary @- http://localhost:9091/metrics/job/some_job/instance/some_instance
# HELP some_metric Just an example.
# TYPE some_metric counter
some_metric 42
# HELP another_metric Just another example.
# TYPE another_metric gauge
another_metric 2398.283
EOF
```

删除网关里暂存的 job=some_job 且 instance=some_instance 的所有时间序列的度量指标。

```bash
curl -X DELETE http://localhost:9091/metrics/job/some_job/instance/some_instance
```

删除网关里暂存的 job=some_job 的所有时间序列的采样值。

```bash
curl -X DELETE http://localhost:9091/metrics/job/some_job
```

### 4.3. Prometheus 网关注意事项

#### 4.3.1. honor_labels 抓取配置项

Prometheus Server 抓取网关的数据的时候，honor_labels 配置项需要设置为 true，否则从网关抓取的所有度量指标的 job 和 instance 标签的值都是网关的值，而不是实际 push 度量指标到网关的实例的值。

#### 4.3.2. 关于度量指标的时间戳

Prometheus Server 从网关（或者实例）抓取到度量指标后，关联到之上的时间是抓取时间而不是推送时间。抓取时间相比推送时间有延迟，为什么要这样？

Prometheus 是具备一定的容错性，但如果某个度量指标 5 分钟内都没有数据，则认为该度量指标不存在了。阻止这样的事情发生是使用网关的原因之一，网关使得可以在稍后抓取那些短暂任务的指标，而不是非得在它们运行时。如果使用推送时间作为度量指标的时间戳，那么就会使得 Prometheus 有可能抓取到 5 分钟前的陈旧数据，与 Prometheus 的设计原则相违背，所以干脆直接使用抓取时的时间作为度量指标的时间戳。

#### 4.3.3. 推送 API

Prometheus 网关提供了 REST 风格的 API，其 URL 形如 `/metrics/job/<JOBNAME>{/<LABEL_NAME>/<LABEL_VALUE>}`。job 名称必须提供，标签可以有一个或多个，或者没有。如果请求 body 里提供了跟 URL 同样的标签，那么以 URL 里的为准。

使用 PUT 方式推送时，会替换既有的跟 URL 里的键值对组合完全一样的所有度量指标。注意这里是整体替换，而不是部分更新，也就是哪怕本次推送的只有部分度量指标，那么之前存在的所有度量指标也都会被丢掉。而 POST 方式正好相反，它执行部分更新，只有本次推送的度量指标会被得到更新，其它的保持原样。DELETE 方式用来删除指定 URL 下的所有度量指标。

## 5. 使用导出器来导入第三方系统的度量指标

有的时候根本没办法通过插入监测代码的方式来采集度量指标，比如 HAProxy、Linux 系统指标等，这个时候只能通过导出器（Exporter）方式来将第三方系统里的度量指标导入到 Prometheus 里。

已经存在大量的第三方系统导出器，下面列出一些常见的，完整的请参考 [官方文档](https://prometheus.io/docs/instrumenting/exporters/)（带有 official 字样的由官方提供）。

### 5.1. 第三方系统导出器

**Databases**

- [Consul exporter](https://github.com/prometheus/consul_exporter) (official)
- [CouchDB exporter](https://github.com/gesellix/couchdb-exporter)
- [ElasticSearch exporter](https://github.com/justwatchcom/elasticsearch_exporter)
- [Memcached exporter](https://github.com/prometheus/memcached_exporter) (official)
- [MongoDB exporter](https://github.com/dcu/mongodb_exporter)
- [MSSQL server exporter](https://github.com/awaragi/prometheus-mssql-exporter)
- [MySQL server exporter](https://github.com/prometheus/mysqld_exporter) (official)
- [OpenTSDB Exporter](https://github.com/cloudflare/opentsdb_exporter)
- [Oracle DB Exporter](https://github.com/iamseth/oracledb_exporter)
- [PostgreSQL exporter](https://github.com/wrouesnel/postgres_exporter)
- [Redis exporter](https://github.com/oliver006/redis_exporter)

**Hardware related**

- [apcupsd exporter](https://github.com/mdlayher/apcupsd_exporter)
- [Node/system metrics exporter](https://github.com/prometheus/node_exporter) (official)

**Messaging systems**

- [Gearman exporter](https://github.com/bakins/gearman-exporter)
- [Kafka exporter](https://github.com/danielqsj/kafka_exporter)
- [MQTT blackbox exporter](https://github.com/inovex/mqtt_blackbox_exporter)
- [RabbitMQ exporter](https://github.com/kbudde/rabbitmq_exporter)
- [RabbitMQ Management Plugin exporter](https://github.com/deadtrickster/prometheus_rabbitmq_exporter)

**Storage**

- [Ceph exporter](https://github.com/digitalocean/ceph_exporter)
- [Gluster exporter](https://github.com/ofesseler/gluster_exporter)
- [Hadoop HDFS FSImage exporter](https://github.com/marcelmay/hadoop-hdfs-fsimage-exporter)

**HTTP**

- [Apache exporter](https://github.com/Lusitaniae/apache_exporter)
- [HAProxy exporter](https://github.com/prometheus/haproxy_exporter) (official)
- [Nginx metric](https://github.com/knyar/nginx-lua-prometheus) library
- [Nginx VTS](https://github.com/hnlq715/nginx-vts-exporter) exporter
- [Varnish exporter](https://github.com/jonnenauha/prometheus_varnish_exporter)

**APIs**

- [AWS ECS exporter](https://github.com/slok/ecs-exporter)
- [AWS Health exporter](https://github.com/Jimdo/aws-health-exporter)
- [AWS SQS exporter](https://github.com/jmal98/sqs_exporter)
- [Cloudflare exporter](https://github.com/wehkamp/docker-prometheus-cloudflare-exporter)
- [DigitalOcean exporter](https://github.com/metalmatze/digitalocean_exporter)
- [Docker Cloud exporter](https://github.com/infinityworksltd/docker-cloud-exporter)
- [Docker Hub exporter](https://github.com/infinityworksltd/docker-hub-exporter)
- [GitHub exporter](https://github.com/infinityworksltd/github-exporter)

**Logging**

- [Fluentd exporter](https://github.com/V3ckt0r/fluentd_exporter)
- [Google's mtail log data extractor](https://github.com/google/mtail)
- [Grok exporter](https://github.com/fstab/grok_exporter)

**Other monitoring systems**

- [AWS CloudWatch exporter](https://github.com/prometheus/cloudwatch_exporter) (official)
- [Cloud Foundry Firehose exporter](https://github.com/cloudfoundry-community/firehose_exporter)
- [Google Stackdriver exporter](https://github.com/frodenas/stackdriver_exporter)
- [Graphite exporter](https://github.com/prometheus/graphite_exporter) (official)
- [InfluxDB exporter](https://github.com/prometheus/influxdb_exporter) (official)
- [JMX exporter](https://github.com/prometheus/jmx_exporter) (official)
- [Munin exporter](https://github.com/pvdh/munin_exporter)
- [Nagios / Naemon exporter](https://github.com/Griesbacher/Iapetos)
- [New Relic exporter](https://github.com/jfindley/newrelic_exporter)
- [NRPE exporter](https://github.com/robustperception/nrpe_exporter)
- [Sensu exporter](https://github.com/reachlin/sensu_exporter)
- [SNMP exporter](https://github.com/prometheus/snmp_exporter) (official)
- [StatsD exporter](https://github.com/prometheus/statsd_exporter) (official)

**Miscellaneous**

- [Bitbucket exporter](https://github.com/AndreyVMarkelov/prom-bitbucket-exporter)
- [Blackbox exporter](https://github.com/prometheus/blackbox_exporter) (official)
- [BOSH exporter](https://github.com/cloudfoundry-community/bosh_exporter)
- [Confluence exporter](https://github.com/AndreyVMarkelov/prom-confluence-exporter)
- [Jenkins exporter](https://github.com/lovoo/jenkins_exporter)
- [JIRA exporter](https://github.com/AndreyVMarkelov/jira-prometheus-exporter)
- [PHP-FPM exporter](https://github.com/bakins/php-fpm-exporter)
- [SMTP/Maildir MDA blackbox prober](https://github.com/cherti/mailexporter)

还有一些系统直接暴露了 Prometheus 格式的度量指标，因此不再需要导出器，Prometheus Server 可以直接从这些系统抓取。带有 direct 字样的表示其内部实现直接使用了 Prometheus 的客户端库。

- [Ceph](http://docs.ceph.com/docs/master/mgr/prometheus/)
- [Collectd](https://collectd.org/wiki/index.php/Plugin:Write_Prometheus)
- [Concourse](https://concourse.ci/)
- [CRG Roller Derby Scoreboard](https://github.com/rollerderby/scoreboard) (direct)
- [Docker Daemon](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-metrics)
- [Doorman](https://github.com/youtube/doorman) (direct)
- [Etcd](https://github.com/coreos/etcd) (direct)
- [FreeBSD Kernel](https://www.freebsd.org/cgi/man.cgi?query=prometheus_sysctl_exporter&apropos=0&sektion=8&manpath=FreeBSD+12-current&arch=default&format=html)
- [Grafana](http://docs.grafana.org/administration/metrics/)
- [Kubernetes](https://github.com/kubernetes/kubernetes) (direct)
- [Linkerd](https://github.com/BuoyantIO/linkerd)
- [mgmt](https://github.com/purpleidea/mgmt/blob/master/docs/prometheus.md)
- [Netdata](https://github.com/firehol/netdata)
- [Pretix](https://pretix.eu/)
- [Quobyte](https://www.quobyte.com/) (direct)
- [RobustIRC](http://robustirc.net/)
- [Skipper](https://github.com/zalando/skipper)
- [SkyDNS](https://github.com/skynetservices/skydns) (direct)
- [Telegraf](https://github.com/influxdata/telegraf/tree/master/plugins/outputs/prometheus_client)
- [Weave Flux](https://github.com/weaveworks/flux)

### 5.2. MySQL 导出器用法

由于导出器太多，没法一个个讲解用法，这里我们以官方提供的 MySQL 导出器为例来讲解。MySQL 导出器支持 MySQL 5.1 及以上版本。

#### 5.2.1. 给导出器创建 MySQL 访问账号

```sql
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'exporterpass';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef |grep -v grep|grep mysql
  error: 没有启动 mysql
- name: check sql
  script: |
    #!/bin/bash
	mysql -u root shiyanlou001 -e "select host,user,password,select_priv from mysql.user"|grep "exporter"
  error: 没有创建 exporter 用户
```

#### 5.2.2. 安装并运行 MySQL 导出器

MySQL 导出器是用 Go 语言编写的，可从 [GitHub](https://github.com/prometheus/mysqld_exporter/releases) 下载已编译好的对应平台的二进制包，不嫌麻烦的话也可以下载源码到本地后自己编译。实验楼提供了 Linux 平台的下载地址 [mysqld_exporter-0.10.0.linux-amd64.tar.gz](http://labfile.oss.aliyuncs.com/courses/980/05/assets/mysqld_exporter-0.10.0.linux-amd64.tar.gz)。按照下面的操作步骤即可启动 MySQL 导出器，DATA_SOURCE_NAME 里的帐号密码为之前所创建。

```bash
wget http://labfile.oss.aliyuncs.com/courses/980/05/assets/mysqld_exporter-0.10.0.linux-amd64.tar.gz
tar -zxvf mysqld_exporter-0.10.0.linux-amd64.tar.gz
cd mysqld_exporter-0.10.0.linux-amd64/
DATA_SOURCE_NAME='exporter:exporterpass@(localhost:3306)/' ./mysqld_exporter
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5954timestamp1528873018009.png/wm)

#### 5.2.3. 配置 Prometheus Server 从 MySQL 导出器抓取度量指标

```yaml
...

scrape_configs:
  ...

  - job_name: 'mysql global status'
    scrape_interval: 5s
    static_configs:
      - targets:
        - 'localhost:9104'
    params:
      collect[]:
        - global_status

  - job_name: 'mysql performance'
    scrape_interval: 5s
    static_configs:
      - targets:
        - 'localhost:9104'
    params:
      collect[]:
        - perf_schema.tableiowaits
        - perf_schema.indexiowaits
        - perf_schema.tablelocks
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep prometheus
  error: 没有启动 prometheus
```

上面的配置里添加了两个 job，一个用来采集 MySQL 全局状态，另一个用来采集 MySQL 性能相关指标。

大约几秒钟后，我们就可以在 Prometheus 的 Web 控制台里查询到 MySQL 相关的度量指标。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5954timestamp1528873036997.png/wm)

## 6. 实验总结

本次实验我们学习如何使用 Prometheus 来监测我们的应用和服务，可以看到 Prometheus 功能非常强大，相关生态也已经十分完善。