---
show: step
version: 1.0
enable_checker: true
---
# ELK(R) 环境搭建

## 1.实验介绍

#### 1.1 内容简介

本节内容将会搭建  ELK(R) 的基本环境，包括安装 Elasticsearch，Logstash，以及 Kibana，R 表示 Redis。解决安装过程中遇到的环境问题，以及配置 Nginx 日志文件路径，用于后续 ELK 用于检索和分析。

#### 1.2 知识点

* ELK Stack 简单了解
* ELK 软件安装流程
* 配置 Nginx 日志

#### 1.3 实验流程

* 安装 Elasticsearch
* 安装 Logstash
* 安装 Kibana
* 解决软件启动问题
* 配置 Nginx 日志文件

#### 1.4 推荐阅读

[Elasticsearch Installation Steps](https://www.elastic.co/downloads/elasticsearch)
[Logstash Installation Setup](https://www.elastic.co/cn/downloads/logstash)
[Kibana Installation Setup](https://www.elastic.co/cn/downloads/kibana)

## 2.实验内容

> 目前 Elasticsearch、Logstash、Kibana 三个软件的最新版本都为 6.1.2。
>
> 由于在线实验的环境问题，暂时不能成功安装 Logstash 的最新版本，为了顺利完成实验，我们将全部安装 5.x 版本的 Logstash，Elasticsearch 和 Kibana。
>
> 如果在非在线实验环境中操作，建议都安装最新版本，以免出现不兼容的问题。

### 2.1 ELK 简介

ELK（Elasticsearch + Logstash + Kibana）是一套完整的日志分析解决方案，是三个开源软件产品的简称。Elasticsearch 是一个分布式的 RESTful 风格的搜索和数据分析引擎，它可以用于全文搜索，结构化搜索以及分析数据，采用 Java 语言编写，是整个日志分析系统的核心。

Logstash 开源的服务器端数据处理管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到指定的存储库，比如 Elasticsearch，采用 JRuby 语言编写，主要有三个组成部分：Shipper，发送日志数据；Broker，收集日志数据；Indexer，写入日志数据。

Kibana 是一款基于 Apache 开源协议，并且使用 JavaScript 语言编写，主要为 Elasticsearch 提供数据分析和可视化的 web 软件平台。你可以通过 Kibana 的交互页面，在 Elasticsearch 中查找数据，并生成直观的数据分析图表。

Elasticsearch + Logstash + Kibana 也称为 ELK Stack。三个软件协同工作，可以满足大部分场景需求，且性能方面也非常优秀。

Redis 本身并不属于 ELK 技术栈里的一份子，而是作为一个可选的应用存在于 ELK 技术栈中。Redis 提供了一个很好的缓冲区，它能够很好地帮助我们在主节点上屏蔽掉多个从节点之间不同日志文件的差异，负责管理日志端（从节点）的人可以专注于向 Redis 里生产数据，而负责数据分析聚合端的人则可以专注于从 Redis 内消费数据，达到分层目的。

关于 ELK 的相关实验，主要以实际操作为主。实验偏向环境的搭建和软件的运行测试过程。

ELK(R) Stack 工作原理：

![](https://doc.shiyanlou.com/document-uid29879labid1887timestamp1466386683374.png/wm)

### 2.2 系统环境检查

由于 Elasticsearch 是由 Java 编写的，所以运行它需要依赖 Java 环境，检查系统中是否已安装 JDK 以及 JDK 版本：

```
java -version
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516781882835.png-wm)

只要 JDK 的版本不低于 1.8 就满足要求。

Redis 作为 Logstash 的一个插件，Logstash 的运行依赖于 Redis 服务。检查环境中是否已安装 Redis：

```
redis-server -v
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516782092069.png-wm)

实验环境中已经安装了 Redis 服务。

启动 Redis 服务并查看启动情况：

```sh
sudo service redis-server start
sudo netstat -antulp | grep 6379
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep redis
	netstat -antulp | grep 6379
  error: 没有启动 redis
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516782248174.png-wm)

### 2.3 dpkg 方式安装 ELK（备选）

由于国内网络的原因。通过 apt-get 的方式安装三个软件，速度非常慢，为了顺利在实验环境中进行，我们已经提前准备好了相应的安装包(.deb)，使用 wget 命令即可下载到环境中，你也可以直接到官网去下载，不过比较费时。

```sh
#下载
wget http://labfile.oss.aliyuncs.com/courses/980/software/elasticsearch-5.6.6.deb
wget http://labfile.oss.aliyuncs.com/courses/980/software/kibana-5.6.6-amd64.deb
wget http://labfile.oss.aliyuncs.com/courses/980/software/logstash-5.6.6.deb

#安装
sudo dpkg -i elasticsearch-5.6.6.deb
sudo dpkg -i kibana-5.6.6-amd64.deb
sudo dpkg -i logstash-5.6.6.deb
```

```checker
- name: check pkg
  script: |
    #!/bin/bash
      dpkg -l elasticsearch
  error: 没有安装 elasticsearch
- name: check pkg
  script: |
    #!/bin/bash
      dpkg -l kibana
  error: 没有安装 kibana
- name: check pkg
  script: |
    #!/bin/bash
      dpkg -l logstash
  error: 没有安装 logstash
```

**注意：**在安装 Logstash 时，可能会遇到下面的警告信息：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516952194949.png-wm)

无法创建系统启动脚本，这是因为我们的实验环境是以 docker 创建 container，对于系统的启动工具有所限制，我们可以通过手动创建启动脚本来解决此问题：

```sh
sudo /usr/share/logstash/bin/system-install /etc/logstash/startup.options sysv
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516952418788.png-wm)

dpkg 方式安装 ELK 操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/3-1.mp4
@`

### 2.4 APT 方式安装 ELK（推荐）

安装软件的方式，推荐选择 apt-get 方式，通过这种方式安装的软件，可以很方便地管理软件运行状态。下面通过 apt-get 的方式来安装 ELK 环境：

1. 导入 Elastic PGP Key：

   ```
   wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
   ```

2. 添加官方安装源：

   ```sh
   sudo apt-get install apt-transport-https

   # 添加官方 apt 源信息
   echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
   ```

3. 安装 ELK 环境：

   ```sh
   sudo apt-get update
   sudo apt-get install elasticsearch	# 安装 elasticsearch
   sudo apt-get install kibana	# 安装 kibana
   sudo apt-get install logstash # 安装 logstash
   ```

通过以上步骤，即可完成 ELK Stack 环境的部署。

**启动 ELK 服务：**

**启动 Elasticsearch**

```sh
sudo service elasticsearch start
```


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516785326730.png-wm)

启动 Elasticsearch 时，由于在线环境的原因，会遇到如上图的警告信息，但是 Elasticsearch 也成功启动了。（这同样也是因为使用环境所导致）

使用以下方式解决警告信息：

```
sudo service elasticsearch stop
sudo vim /etc/init.d/elasticsearch
```

将 142 行至 152 行的内容注释（可能因版本不同，行数有所不同，注意注释内容即可）：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516873427826.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516873434167.png-wm)

再次启动 Elasticsearch，没有任何警告信息：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516785964710.png-wm)

若是解决了以上问题还是启动失败，我们可以修改 /etc/logstash/jvm.options 中的 Xms 与 Xmx 中的内存配置参数，默认情况下使用 2G 的内存，我们可以修改至 1G 或者是 512m 即可。

Elasticsearch 监听在本地的 9200 端口，浏览器打开地址：**localhost:9200**

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516787425891.png-wm)

**启动 Logstash**

```sh
sudo service logstash start
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516786313500.png-wm)

启动时，出现了错误信息。修复方式：

```sh
sudo vim /etc/init.d/logstash
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516952668914.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516952663551.png-wm)

注释掉第 67 行的内容(因版本不同，可能行数不同，注意内容即可)，并再次启动 Logstash：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516786611397.png-wm)

若是解决了以上问题还是启动失败，我们可以修改 /etc/elasticsearch/jvm.options 中的 Xms 与 Xmx 中的内存配置参数，默认情况下使用 2G 的内存，我们可以修改至 1G 或者是 512m 即可。

**启动 Kibana**

```Sh
sudo service kibana start
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516786239200.png-wm)

Kibana 的监听端口为 5601，浏览器打开地址：**localhost:5601**：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516872988682.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516873040200.png-wm)


```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep elasticsearch
  error: 没有启动 elasticsearch
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep logstash
  error: 没有启动 logstash
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep kibana
  error: 没有启动 kibana
```

启动 ELK 操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/3-2.mp4
@`


### 2.5 配置 Nginx 日志

ELK 用来构建日志分析系统，所以还需要配置日志信息。常用的 WEB 服务器有 Apache 和 Nginx，这里已 Nginx 的配置为示例。

环境中已经默认安装了 Nginx 服务。修改 Nginx 的配置文件，指定日志存放路径。Nginx 默认的配置文件位于 `/etc/nginx/sites-available/default`。

```sh	
sudo vim /etc/nginx/site-available/default
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516789624880.png-wm)

这里指定的是 Nginx 的访问日志存放路径。这个路径此时并不存在，所以需要提前创建相应的目录和文件，否则 Nginx 不能正常启动。

```sh
cd /home/shiyanlou/Code && sudo mkdir elk && cd elk && sudo touch access.log
```

```checker
- name: check file
  script: |
    #!/bin/bash
	ls /home/shiyanlou/Code/elk/access.log
  error: /home/shiyanlou/Code/elk 目录下没有 access.log 文件
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516788631601.png-wm)

启动 Nginx web 服务器：

```sh
sudo service nginx stop
sudo service nginx start
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep nginx
  error: 没有启动 nginx
```

浏览器打开地址 **localhost**

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516788862956.png-wm)

查看日志记录：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516789983117.png-wm)

配置 Nginx 日志操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/3-3.mp4
@`

## 3. 总结

本节实验主要介绍 ELK 环境的安装部署。推荐使用 apt-get 的方式，从官方提供的安装源中安装 Elasticsearch，Logstash，以及 Kibana，使用这种方式安装。

因为网络原因，使用这种方式安装速度会比较慢，也可以使用我们提供的安装包安装，不过这是备用方案。
