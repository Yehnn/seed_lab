---
show: step
version: 1.0
enable_checker: true
---
# Redis 与 ELK

## 1. 实验介绍

#### 1.1 实验内容

本节实验将开启两个 Logstash 进程。一个向 redis 写入日志数据，一个从 redis 读取日志数据。利用 redis 模拟分布式 redis 存储日志的结构。

#### 1.2 知识点

* Logstash 配置 Redis

## 2. ELK 集群应用 

在真实的项目中，服务器都是以集群的形式对外提供服务。如果要对集群中的全部服务器做日志分析，可以采用下图所示的结构图：

![](https://dn-anything-about-doc.qbox.me/document-uid29879labid1887timestamp1466386683374.png/wm)

管理服务器集群时，选取一台服务器作为主节点（Master Node），其他服务器作为从节点（Slave Node）。主节点服务器上部署 ELK + Redis 环境，配置 Logstash 的 input 为 Redis，output 为 Elasticsearch，每个从节点服务器部署 Logstash，配置 input 为日志文件，配置 output 为主节点的 Redis。部署完成后，日志的处理流程：全部 Slave Node 的日志数据发送到 Master Node 的 Redis，Redis 以队列的形式存储这些日志数据，Logstash 从 Redis 中读取日志数据进行分析和查询。

从节点（Slave Node），可以作为事件的传递者（Shipper），将各种日志数据发送至主节点（Master Node），从节点只需运行 Logstash 代理（agent）程序；

主节点（Master Node），可运行包括中间转发器（Broker）、索引器（Indexer）、搜索和存储器（Search and Storage）、Web界面端（Web Interface）在内的各个组件，以实现对日志数据的接收、处理和存储。

Redis 本身并不属于 ELK 技术栈里的一份子，而是作为一个插件存在于 ELK 技术栈中。Redis 提供了一个很好的缓冲区，它能够很好的帮助我们在主节点上屏蔽掉多个从节点之间不同日志文件的差异，从而使用 Redis 对数据的产生和使用进行了分层。

由于环境不好实现集群效果，所以可以通过开启多个 Logstash 进程来模拟 Slave Node。

## 3. Logstash 配置 Redis

**确保 ELK+R 服务都已启动**。

**模拟 Master Node 的 Logstash 配置，进程角色为 Indexer：**

修改配置文件，将 Logstash 的 input 设置为 Redis，从 Redis 中读取日志数据，复制 `logstash-shipper.conf` 为 `logstash-indexer.conf`，并编辑 `logstash-indexer.conf`：

```sh
sudo cp /etc/logstash/conf.d/logstash-shipper.conf /etc/logstash/conf.d/logstash-indexer.conf
sudo vim /etc/logstash/conf.d/logstash-indexer.conf
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516943413784.png-wm)

将 input 的 file 部分修改为 Redis：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516943419362.png-wm)

**模拟 Slave Node 的 Logstash 配置，进程角色为 Shipper：**

配置 Logstash 的 output 为 Redis，将日志数据输出到 Redis，编辑 ``logstash-shipper.conf`:

```sh
sudo vim /etc/logstash/conf.d/logstash-shipper.conf
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516944022935.png-wm)

将 Elasticsearch 部分修改为 Redis：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516944017276.png-wm)

这两个配置文件就是我们需要理解的核心内容，大家需要在实验环境中使用 `logstash -t -f` 检查下配置文件是否有错误。没有错误之后再继续后续的操作。

Logstash 主节点和从节点配置文件介绍视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/6-1.mp4
@`

如果你已经将 Logstash 运行在后台，可以使用命令：`kill %1` 结束后台进程。

打开两个终端页面：一个模拟模拟 Slave Node  的 Logstash 进程，一个模拟 Master Node  的 Logstash 进程。

启动 Shipper 进程，指定配置文件为 `logstash-shipper.conf`：

```sh
/opt/logstash/bin/logstash -f /etc/logstash/conf.d/logstash-shipper.conf --path.settings /etc/logstash
```

启动 Indexer 进程，指定配置文件为 `logstash-indexer.conf`：

```Sh
mkdir -p /home/shiyanlou/data/logstash
/opt/logstash/bin/logstash -f /etc/logstash/conf.d/logstash-indexer.conf --path.settings /etc/logstash --path.data /home/shiyanlou/data/logstash
```

> 再同一台主机上启动多个 logstash 进程需要将它们的数据目录分开，这里通过 `--path.data` 选项给第二个进程分配了一个单独的目录。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516944924589.png-wm)

在 Shipper 中输入数据：`hello shiyanlou`

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516944955412.png-wm)

右侧的 Indexer 进程也会接收到新产生的数据。这个过程中，左侧的 Shipper 产生数据，输入到 Redis 队列中，右侧的 Indexer 从 Redis 中读取新的数据。在集群环境中，可以将 Shipper 进程部署到各个节点服务器，只需要在 master 节点服务器上部署 Indexer 即可管理全部日志信息。


## 4. 实验总结

本节实验我们为单机环境下的 ELK 部署了 redis，也模拟了一个分布式环境的效果。

当这两个配置文件运行在不同服务器上时，便能做到真正的分布式了，唯一需要修改的地方就是 redis 的 host 字段值，其中 Logstash Shipper 中的地址应该填写 Logstash Indexer 所在地址，而 Logstash Indexer 中的 redis host 字段不需要修改，因为 redis 和 Loststash Indexer 运行在同一台机器上。
