---
show: step
version: 1.0
enable_checker: true
---
# Logstash 配置

## 1. 实验介绍

#### 1.1 实验内容

在上一节实验中，我们安装了 Elasticsearch、Logstash、Kibana 三个软件，搭建了 ELK Stack 的基本架构。本节实验，将继续完善 ELK Stack。主要包括对 Logstash 的了解与配置、Logstash 的作用是仅仅是整理日志的格式，最后还需要将整理好的日志发送到 Elasticsearch 做具体的分析和处理。

#### 1.2 实验知识点

* Logstash 工作流程
* Logstash 基本配置
* Logstash 启动
* Logstash 连接 Elasticsearch 配置

#### 1.3 推荐阅读

* [How Logstash Works](https://www.elastic.co/guide/en/logstash/current/pipeline.html#pipeline)
* [Multiline codec plugin](https://www.elastic.co/guide/en/logstash/current/plugins-codecs-multiline.html)

## 2. Logstash 工作流程

Logstash 是一个具有实时 pipeline 特性的开源数据收集引擎。在 ELK Stack 中，他的主要作用是对日志内文件数据的收集，在内部将日志数据进行格式化和规范化，然后发送到指定的接收者，通常是 Elasticsearch。Logstash 对搜集的数据处理工作，主要依靠丰富的内部插件来实现。

Logstash 工作流程一般分为三个部分：INPUTS（输入），FILTERS（过滤），OUTPUTS（输出）。各个部分也称为插件，其中 inputs 和 outputs 是必需的，filter 是可选的。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516849562125.png-wm)

**Inputs**：

输入插件，从外部将数据输入到 Logstash。有多种可选的数据输入方式：

* file：比如日志文件。Logstash 直接从文件系统读取日志文件内容
* syslog：linux 的系统日志管理服务。配置 Logstash 监听在 514 端口，即可搜集 syslog 日志数据
* redis：从 Redis 服务器读取数据，利用了 Redis 消息队列的特性。适合大规模日志集群场景。
* beats：比如 Filebeat，自动收集日志数据并发送到 Logstash

**Filters：**

过滤插件，是 Logstash 数据处理流程的中间步骤。Logstash 最常用的过滤插件是 grok，它能够解析任意的文本内容并将他们结构化输出。grok 也是目前 Logstash 将非结构化数据解析为结构化和可查询的最佳实现方式。

更多关于过滤插件的内容，可以点击[此处]([Filter Plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html))查看。

**Outputs:**

输出是 Logstash 处理数据的最后一步，输出的方式有很多种，比较常用的是 Elasticsearch 和 file。将输出方式设定为 Elasticsearch 后，就可以通过 Elasticsearch 对数据进行高效，便捷地查询。当输出的方式为 file 时，Logstash 会将数据写入到磁盘文件中。

## 3. Logstash 基本配置

首先确保 ELK 所需服务都已正常开启：

```sh
sudo service logstash status
sudo service kibana status
sudo service elasticsearch status
sudo service redis-server status
```

```checker
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
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep elasticsearch
  error: 没有启动 elasticsearch
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep redis
  error: 没有启动 redis-server
```

Logstash 默认会安装到 `/usr/share/logstash` 或 `/opt/logstash` 目录下，取决于安装方法。我们后续的步骤默认安装目录为 `/opt/logstash`，如果你是按照先前的 dpkg 的方式安装的，则安装目录为 `/usr/share/logstash`。

主要执行命令都安装在 `/opt/logstash/bin` 目录下。进入到此目录，执行下面一条命令：

```sh
cd /opt/logstash/bin
sudo ./logstash -e 'input{stdin{}} output{stdout{}}' 
```

> 如果是使用 APT 命令安装（Ubuntu 或 Debian），需要通过 `--path.settings` 选项来显示指定配置文件路径，完整命令为 `sudo /usr/share/logstash/bin/logstash -e 'input{stdin{}} output{stdout{}}' --path.settings /etc/logstash`。

执行过程可能比较慢，需要耐心等待一会儿：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516859075134.png-wm)

可以看到，在终端输入内容后，会输出一条规范的内容，这就是经过 Logstash 处理后的输出格式。使用快捷键 `ctrl+d` 结束进程。再次执行下面这条命令：

```sh
./logstash -e 'input{stdin{}} output{stdout{codec=>rubydebug}}'
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516859530675.png-wm)

现在可以理解输出内容每个字段的含义了。虽然我们只是简单输入了 `hello shiyanlou` ，但是经过 Logstash 的处理，默认添加几个字段，比如版本信息，时间信息，主机名等。

将刚才执行的命令格式化后，可以得到以下内容：

````sh
input {
  stdin {}
}
output {
  stdout {
    codec => rubydebug
  }
}
````

参数讲解：

* input：定义 Logstash 输入行为
* output：定义 Logstash 输出行为


* stdin {}：标准输入
* stdout {}：标准输出
* codec：指定输出的数据格式，这里指定为 rubydebug，常用的还有 json，multiline

执行的命令中，**-e** 参数表示 Logstash 直接从命令行读取配置信息。通过这种方式可以简单快速测试配置内容是否正确而不用编辑配置文件。

在 input 中，我们定义了一个标准输入，由于什么都没有，所以 Logstash 会从终端的标准输入中读取字符串，这也是为什么刚才在输入完命令后会出现等待输入的情况。

在 output 中，我们定义了一个标准输出，也就是说 Logstash 在处理完后会将结果从标准输出（终端）中输出，而 codec 字段，则说明了输出会遵循什么样的格式，这里定义的 codec 为 rubydebug，所以我们刚才看到了一套标准的输出格式。

Logstash 基本配置介绍视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/4-1.mp4
@`


## 4. Logstash 配置文件

正式项目中，肯定不能直接在命令行中配置 Logstash，特别是配置内容较多的时候。所以需要创建配置文件。

进入 `/etc/logstash/conf.d/`，创建配置文件`logstash-shipper.conf`。

```sh
cd /etc/logstash/conf.d
sudo vim logstash-shipper.conf
```

```checker
- name: check file
  script: |
    #!/bin/bash
	ls /etc/logstash/conf.d/logstash-shipper.conf
  error: /etc/logstash/conf.d 目录下没有 logstash-shipper.conf 文件
```

添加以下内容：

```Sh
input {
  stdin {}
  file {
    path => "/home/shiyanlou/Code/elk/access.log"
    start_position => beginning
    codec =>  multiline {
      'negate' => true
      'pattern' => '^\d'
      'what' => 'previous'
    }
  }
}
output {
    stdout {
        codec => rubydebug
    }
    elasticsearch {
        hosts=>["localhost:9200"]
        index=>"logstash-%{+YYYY.MM.dd}"
    }
}
```

配置文件中，有两种数据输入方式：标准输入，文件读取（Nginx 日志文件）。对文件读取也有一定的要求：指定文件路径，读取开始位置，读取的格式。读取的内容用到了一个 mutiline 的插件。这个插件的作用是将多行日志记录处理为一个日志事件。

mutiline 插件的一般配置格式：

```sh
multiline {
      pattern => "pattern, a regexp"
      negate => "true" or "false"
      what => "previous" or "next"
    }
```

* pattern：指定正则表达式，输入的内容匹配这个表达式，用于确定输入的内容是属于前一个事件的内容还是新的事件的内容。
* negate：正则表达式的否定形式。若为 true，表示不匹配指定的 pattern。
* what：如果符合匹配的正则表达式，则应该将一行内容合并到前一行还是后一行。

在我们的项目中。查看 nginx 日志信息：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516863637683.png-wm)

每一条日志信息都以 ip 地址开头，即以数字开头。我们配置的多行处理方式为：

```Sh
multiline {
      'negate' => true
      'pattern' => '^\d'
      'what' => 'previous'
    }
```

表示，如果不以数字开始的一行内容，就属于上一条日志的内容，否则就是一条新的日志记录。 Logstash 采用 `tail -F filename` 的方式动态读取日志记录。

配置文件中，也定义了两种输出方式：标准输出和输出到 Elasticsearch。如果要将数据输出到 Elasticsearch，需要配置 Elasticsearch 的主机名和端口号。同时，为每条日志都增加了一条自定义的索引。

执行命令检测配置文件是否正确：

```sh
/opt/logstash/bin/logstash -t -f /etc/logstash/conf.d/logstash-shipper.conf --path.settings /etc/logstash
```

> -t：test，测试，不会真正执行
>
> -f：file，指定配置文件路径

需要等待一会儿执行完成：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516865852043.png-wm)

配置没问题后，还是上面那条命令，去掉 `-t` 参数，再次执行：

```
/opt/logstash/bin/logstash -f /etc/logstash/conf.d/logstash-shipper.conf --path.settings /etc/logstash
```

等待一会儿执行完成：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1516876625141.png-wm)

执行完成后，终端中应该能够看到日志文件的输出，如果终端没有输入日志内容，原因可能是因为日志文件太小了，Logstash 的监控做了一些限制，不过并不影响实际的使用。解决办法可以参考这一篇博文：[logstash file输入，无输出原因与解决办法](http://www.cnblogs.com/yangwenbo214/p/6195135.html)

此时 Logstash 运行在前台，如果使用 `ctrl+c` 就会结束当前进程，Logstash 就不会向 Elasticsearch 发送数据。可以使用以下方式将 Logstash 运行在后台：

```sh
nohup /opt/logstash/bin/logstash -t -f /etc/logstash/conf.d/logstash-shipper.conf --path.settings /etc/logstash &
```

配置 Logstash 操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/4-2.mp4
@`


## 5. 总结

本节实验的主要内容就是对 Logstash 的了解，理解它的工作流程和组件关系。Logstash 处理数据的三个流程：输入，过滤，输出。将无结构的数据解析并结构化为规范的数据，并发送到 Elasticsearch，用作后续分析。
