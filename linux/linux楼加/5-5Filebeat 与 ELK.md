---
show: step
version: 1.0
enable_checker: true
---
# Filebeat 与 ELK

## 1. 实验介绍

#### 1.1 内容简介

在前面的实验中，我们已经完整的搭建了一套 ELK(R) 的日志分析系统。本节实验将要介绍并使用另外一款软件 Filebeat，一个高效的日志文件采集器，他与 Logstash 的功能类似，既可以与 Logstash 协同工作，也可接替 Logstash 的工作。他的作用是对日志文件做数据采集，通过配置指定要采集的目录或文件，将采集到的数据发送给 Logstash 或者 Elasticsearch，相比 Logstash，Filebeat 更加轻量级，此外，与他类似的采集器还有 Heartbeat（运行时间监控），Packetbeat（搜集网络数据）等。

#### 1.2 知识点

* Filebeat 工作流程与原理
* 配置 Filebeat 使用 Logstash
* 配置 Filebeat 使用 Elasticsearch

#### 1.3 推荐阅读

* [How Filebeat works](https://www.elastic.co/guide/en/beats/filebeat/current/how-filebeat-works.html)
* [Configuring Filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-howto-filebeat.html)

## 2. 实验内容

接下来介绍实验内容。

### 2.1 Filebeat 工作流程

Filebeat 是一款轻量级的日志采集器。与 Logstash 的功能有些类似，他的工作流程如下图所示：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516958759797.png-wm)

Filebeat 可以对多个日志文件进行数据采集，将采集的数据汇总后，可以直接发送给 Logstash、Elasticsearch 或者 Redis。

当启动 Filebeat 服务后，它会启动一个或多个 Prospector，根据配置文件中配置的日志文件路径，监控所有的日志文件或目录，对于每一个日志文件，都会有一个对应 Harvester，它的作用是按行读取日志文件中的内容，将新读取到的内容，发送到内部的 Spooler（6.x 版本已移除），在 Spooler 中，聚合日志事件并将汇总的数据发送到指定的 output。同时 Filebeat 内部维护着一个记录文件读取信息的注册文件，文件中记录着每个 harvvester 最后读取位置的偏移量。

当将数据发送到 Logstash 或 Elasticsearch 时，Filebeat 使用背压敏感协议，以考虑更多的数据量。如果 Logstash 正在忙于处理数据，则可以让 Filebeat 减慢读取速度。一旦拥堵得到解决，Filebeat 就会恢复到原来的读取速度并继续运行，从而避免 Logstash 或者 Elasticsearch 出现过载的情况。

### 2.2 prospector 与 harvester

prospector 相当于是一个查找器或者监视器，根据配置文件中指定的数据读取源：单个日志文件，或日志文件所在目录，去对应的本地路径去查找是否存在相应的文件。只要存在可以读取的日志文件，才可以进行下一步读取操作。如果 Filebeat 的 input 数据类型为 log，那么 prospector 会根据配置的路径去匹配对应的日志文件。input 的类型也可以为 stdin。

示例：

```sh
filebeat.prospectors:
- type: log
  paths:
    - /var/log/*.log
    - /var/path2/*.log
```

如果 Filebeat 的配置信息如上，那么启动 Filebeat 后，会启动两个 prospector，每个 prospector 去各自的路径下检查匹配的日志文件，如果找到日志文件，就可以开始下一步读取操作。也可以通过配置对某些特定的日志文件忽略，不会读取它的内容。

harvester 负责对单个日志文件的内容读取与采集。每个 harvester 对每个日志文件按行读取内容，并把读取的内容发送到指定的 output 中。在一个 harvester 运行过程中，会始终保持此文件处于打开状态，即使重命名或者删除此文件，Filebeat 依然保持对此文件句柄的引用，占用此文件对应的磁盘空间。

简单的理解，Filebeat 有两个组件组成，prospector 负责监控日志文件或目录，harvester 负责读取每个日志的内容并发送到指定的 output。

### 2.3 配置 Filebeat 使用 Elasticsearch

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517048880315.png-wm)

首先需要安装 Filebeat，由于前面的步骤已经配置了安装源和Elastic PGP Key，现在可以直接使用 apt-get 安装：

```sh
sudo apt-get install filebeat
```

编辑 Filebeat 的配置文件，默认路径为：`/etc/filebeat/filebeat.yml`：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517033927138.png-wm)

在 12 行左右，是关于 prospector 的配置部分，默认日志输入类型为 log，然后配置日志文件的路径信息，默认为 `/var/log/*.log`，即 `/var/log/` 目录下，所有以 `.log` 结尾的文件，都属于目标文件。

修改路径信息为 nginx 日志文件，注释默认路径：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517033921430.png-wm)

将 `access.log` 作为 Filebeat 检测的目标文件。

找到 output 配置部分，第 81 行左右，可以看到，Filebeat 默认的 output 配置为 Elasticsearch：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517041532570.png-wm)

保存退出。

**加载索引模板：**

> 在 Elasticsearch 中，索引模板用于定义设置和映射，以确定如何分析字段。 Filebeat 的建议索引模板文件由 Filebeat 软件包安装。如果在 filebeat.yml 配置文件中接受模板加载的默认配置，Filebeat 会在成功连接到 Elasticsearch 后自动加载该模板。如果模板已经存在，它不会被覆盖，除非单独配置 Filebeat 来执行此操作。 如果要禁用自动模板加载，或者要加载自己的模板，可以在 Filebeat 配置文件中更改模板加载的设置。如果选择禁用自动模板加载，则需要手动加载模板。

 默认情况下，如果启用了 Elasticsearch 输出，Filebeat 会自动加载推荐的模板文件 `filebeat.template.json`。这里不做过多讲解，使用默认配置即可。可查阅[文档](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-template.html#configuration-template)了解更多。

配置完成后，关闭 Logstash 进程，重新启动 Filebeat。

浏览器访问 `127.0.0.1`。

查看 Kibana 统计页面：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517048889140.png-wm)

可以看到，没人任何统计数据。这是因为通过 Filebeat 直接向 Elasticsearch 发送数据，默认的索引模式为 `filebeat-*`，而此时查看的索引模式是 `logstash-*` ，即 Logstash 发送的数据。如果要查看 Filebeat 发送的数据，只需要在 Kibana 中创建新的索引模式（Index Patterns）即可查询相关数据。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517049271552.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517049277078.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517049263749.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517049288926.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517049282563.png-wm)

查看统计结果：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517049424857.png-wm)

可以看到，此时已经可以正常获取 Filebeat 发送的日志数据。

### 2.4 配置 Filebeat 使用 Logstash

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517035523060.png-wm)

> 在 6.x 版本中，Filebeat 的 output 只能配置一个。

编辑 Filebeat 的配置文件：

```sh
sudo vim /etc/filebeat/filebeat.yml
```

修改 output 的配置为 Logstash。找到 81 行左右，注释 Elasticsearch 部分，应用 Logstash 的 output 配置：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517034317413.png-wm)

修改为：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517034461146.png-wm)

因为我们是在单机上部署环境，所以 host 为 localhost。如果是分布式部署，只需要将 localhost 修改为 Logstash 所在主机的 IP 地址即可。配置修改完成后，保存并退出 vim 。

启动 Filebeat 服务：

````sh
sudo service filebeat start
````

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517034720384.png-wm)

结束运行的 Logstash 进程：

```sh
kill %1			#kill 后台运行的 Logstash 进程
```

复制 `logstash-indexer.conf` 为 `logstash-filebeat.conf`，配置 Logstash 从 Filebeat 读取数据：

```sh
cp /etc/logstash/conf.d/logstash-indexer.conf /etc/logstash/conf.d/logstash-filebeat.conf
sudo vim /etc/logstash/conf.d/logstash-filebeat.conf
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517035773093.png-wm)

修改为：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517035779171.png-wm)

配置 beats 端口为本机的 5044。即可获取 Filebeat 发送的数据：

启动 Logstash ，应用配置文件 `logstash-filebeat.conf`，在前台启动方便观察效果：

```sh
opt/logstash/bin/logstash -f /etc/logstash/conf.d/logstash-filebeat.conf 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517037674595.png-wm)

启动成功后，这是还没有新的数据过来。可以尝试访问 `127.0.0.1`，access.log 会增加一条访问日志：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517037669390.png-wm)

查看 Logstash 的输出内容：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517037662946.png-wm)

刚才新增的访问日志信息已经发送到 Logstash 。打开 Kibana 控制台，可以查看刚才的日志统计：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1517038529586.png-wm)

Filebeat 与 ELK 操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/7-1.mp4
@`

## 3. 实验总结

本节实验主要介绍了数据采集器 Filebeat 的使用方法，相比 Logstash 而言，前者更加轻量级，性能也非常优异。Filebeat 使用了背压敏感写协议，更加只能的传输日志数据。实验中，了解了 Filebeat 的工作流程和实现原理，并配置 Filebeat 与 Logstash 和 Elasticsearch 协同工作，虽然实验中采用的是单机模式部署，但是也很容易应用到分布式环境中。
