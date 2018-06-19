---
show: step
version: 1.0
enable_checker: true
---
# Kibana 数据可视化

## 1. 实验介绍

#### 1.1 实验内容

前面的两节实验中，我们已经可以从 Nginx 日志文件中获取数据到 Logstash，经过 Logstash 的处理后，以规范的形式输出到 Elasticsearch。本节将会使用 Kibana 的数据可视化服务，用来搜索，查看和存储 Elasticsearch 索引中存储的数据。

#### 1.2 实验知识点

* Kibana 简单使用
* Lucene 基本查询

#### 1.3 推荐阅读

* [Kibana Docs](https://www.elastic.co/guide/en/kibana/current/getting-started.html)

## 2. Kibana 简单使用

下面我们将会学习 Kibana 简单使用。

### 2.1 创建 Kibana 索引模式

在前面的实验中，我们将 Logstash 运行在后台，可以不断地搜集 Nginx 访问日志信息，并发送到 Elasticsearch。我们可以在浏览器中，通过 Kibana 查看这些日志数据。

输入地址：`localhost:5601`

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516878787892.png-wm)

Kibana 提示需要配置一个索引模式，这里可以直接使用默认索引模式就行，点击创建：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516878991588.png-wm)

可以看到，Kibana 默认为我们创建了 16 个字段（fields）。

点击左侧 Discover ，进入到 Kibana 的全局搜索界面，可以看到更多统计信息：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516891204779.png-wm)

右上角可以选择数据记录的时间段：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516891294824.png-wm)

### 2.2 Visualize 使用

Kibana 具有丰富的图表工具，能够以多种方式来分析数据，我们将尝试使用图表来分析每分钟 nginx 日志记录。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516891809578.png-wm)

尝试创建一个面积图（Area）：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516893206595.png-wm)

选择索引模式，创建新的搜索数据：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516893213425.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516893199725.png-wm)

右侧即是面积图，只是此时有一个维度的数据。Y 轴代表 nginx 日志的请求总数 Count，可以看到有 16 条记录。

> 如果数据较少，可以认为向 access.log 中添加伪造数据，或者多次刷新 nginx 页面

我们还需要添加 X 轴的统计指标。点击 X-Axis 进而创建 X 轴上的 Aggregation。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516893966951.png-wm)

配置指标：

Aggressive：Date Histogram

Field：@timestamp

Interval：Minute

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516893974043.png-wm)

右侧统计表变化：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516893980084.png-wm)

此时：X 代表分钟间隔。Y 轴代表日志数量。从这个表中，可以很直观的看出 Nginx 每分钟日志记录数量，以及变化趋势。从而可以判断当前 Nginx 的负载情况。

Kibana 也支持将统计结果保存，以便于下次快速查看最新统计结果。右上角的 Save 按钮，输入保存的名字，点击保存即可。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516894665907.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516894671182.png-wm)

下次再打开 Visualize 时，就可以看到保存的记录，就可以快速查看统计情况：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516894659061.png-wm)

### 2.3 Dashboard

如果你保存了多个可视化数据时，你可以创建一个 Dashboard，以便于可以快速查看全部统计数据：

创建新的 Dashboard：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516895786766.png-wm)

添加已保存的统计图表数据：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516895775932.png-wm)

我们刚才保存的 `Nginx logs1`

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516895781381.png-wm)

Nginx logs1 的统计图表添加到了下方面板：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516895767858.png-wm)

## 3. Lucene 基本查询

在 Kibana 中如果要进行更加精确的查询，则需要使用 Lucene。

例如，我们要在某个字段中搜索某个字符串是否存在。

首先我们在终端中构造数据，新打开一个终端，开启 Logstash：

```
sudo /opt/logstash/bin/logstash -f /etc/logstash/conf.d/logstash-shipper.conf --path.settings /etc/logstash
```

等待命令执行完毕，在终端中一次输入： hello world、hello shiyanlou、learn by doing，how are you。Logstash 会输出 4 条记录：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516930462308.png-wm)

在 Kibana 中设置页面自动刷新时间为 5s 一次。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516930615333.png-wm)

可以看到，刚才输入的数据，页面中已经展示了统计结果

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516930609527.png-wm)

开始搜索，我们查找带有 shiyanlou 这个单词的记录，搜索框输入：message：\*hello* 。`message` 指定要搜索的字段，`hello` 表示要搜索的字符，`*` 表示通配符。

输入后，点击搜索。下方已经查找出了满足条件的两条记录：`hello world` 和 `hello shiyanlou`，且匹配字符都已高亮显示。且统计图表也反映了消息的时间和数量。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516930689851.png-wm)

> 注意：Lucene 语法的搜索并不是Kibana持有的，Kibana 只是一个用于数据可视化的 Web 前端，真正的搜索引擎其实是构建在了 Elasticsearch 之中。
>
> 更详细的 Lucene 语法可以参考：[Lucene 查询分析器语法](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)

## 4. 实验总结

本节实验我们搞定了 Kibana 的可视化部分，并初步了解了一下在 Elasticsearch 中需要使用的 Lucene 查询语法。

严格意义上来说，ELK Stack 的技术栈基本搭建到此就结束了，我们的实验中在一台机器上跑起了基本的 ELK 环境，并在 Kibana 中实验了我们的数据。

作为进阶，我们将在下一个实验中启用 Redis 作为缓存，构建以本地环境模拟而搭建的分布式 ELK+Redis 日志环境。
