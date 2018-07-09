---
show: step
version: 1.0
enable_checker: true
---
# Zabbix 监控 Nginx

## 1. 实验介绍

#### 1.1 实验内容

通过前面的学习，我们基本掌握了 Zabbix 监控的搭建和基本配置，那么这节实验就来进行一下实战。让我们一起来学习如何用 Zabbix 监控 nginx 服务。

#### 1.2 实验知识点

+ Nginx
+ Zabbix 配置
+ 监控 Nginx

#### 1.3 推荐阅读

+ [Nginx document](http://nginx.org/en/docs/)
+ [Nginx 官方资源](https://www.nginx.com/resources/wiki/)
+ [Nginx Wiki](https://en.wikipedia.org/wiki/Nginx)

## 2. Nginx

下面我们将会学习Nginx。

### 2.1 Nginx

Nginx 是一个高性能的网页服务器，能够反向代理 `HTTP`、 `HTTPS`、`SMTP`、 `POP3`、 `IMAP`，也可以作为一个负载均衡器和 HTTP 缓存。Nginx 由 `Igor Sysoev` 开发设计来供俄罗斯的大型门户网站及搜索引擎 Rambler 使用。此软件 BSD-like 协议下发行，可以在 UNIX、GNU/Linux、BSD、Mac OS X、Solaris，以及 Microsoft Windows 等操作系统中运行。其特点是占用内存少，并发性强（在同类型的网页服务器中表现良好）。在中国大陆使用 Nginx 的网站用户有：新浪、网易、腾讯、百度、淘宝等。

> 反向代理（Reverse Proxy）方式是指以代理服务器来接受 internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器 。

### 2.2 Nginx 的优势

Nginx 是一个高性能的 Web 和反向代理服务器, 它具有有很多非常优越的特性:****

**作为 Web 服务器**：相比 Apache，Nginx 使用更少的资源，支持更多的并发连接，体现更高的效率，这点使 Nginx 尤其受到虚拟主机提供商的欢迎。能够支持高达 50,000 个并发连接数的响应。

**作为负载均衡服务器**：Nginx 既可以在内部直接支持 Rails 和 PHP，也可以支持作为 HTTP代理服务器 对外进行服务。Nginx 用 C 编写, 不论是系统资源开销还是 CPU 使用效率都比 Perlbal 要好的多。

**作为邮件代理服务器**: Nginx 同时也是一个非常优秀的邮件代理服务器（最早开发这个产品的目的之一也是作为邮件代理服务器）。

**Nginx 安装非常的简单，配置文件非常简洁（还能够支持 perl 语法），Bugs非常少的服务器**: Nginx 启动特别容易，并且几乎可以做到 7*24 不间断运行，即使运行数个月也不需要重新启动。你还能够在 不间断服务的情况下进行软件版本的升级。

注：（此段来自 Nginx 官方文档）

### 2.3 Nginx 安装

根据操作系统的不同，Nginx 也会以不同的方式进行安装。

+ 在 Linux 上安装

    使用 `nginx.org` 的 nginx 软件包进行安装。

    [nginx: Linux packages](http://nginx.org/en/linux_packages.html)

+ 在 FreeBSD 上安装

    在 FreeBSD 上，Nginx 可以从 `packages` 或通过 `ports` 系统安装。 `ports` 系统提供了更大的灵活性，允许选择范围较广的选项。 `ports` 将用指定的选项编译 `nginx` 并进行安装。

    [nginx：packages](https://www.freebsd.org/doc/handbook/pkgng-intro.html)

    [nginx：ports](https://www.freebsd.org/doc/handbook/ports-using.html)

+ 从源码进行安装

    虽然可以从各个系统自带的软件包仓库中进行安装管理，但是这些预先编译好的软件包都比较旧，所以直接从源进行编译安装就会比较灵活，但是对于新手来说也会比较复杂。

    [Building nginx from Sources](http://nginx.org/en/docs/configure.html)

**注：因为实验楼环境下已经安装了 Nginx ，大家可以在自己的本地环境上尝试安装搭建 Nginx 。**

可以查看下本实验环境下的 ngibx 版本和配置参数信息等。

```
$ nginx -V
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515147620738.png/wm)

从打印出来的信息可以知道，环境中 nginx 的版本信息是：`nginx/1.4.6 (Ubuntu)`，配置参数中指出了 Nginx 的安装路径（`--conf-path`）、启用的模块（如：`--with-http_stub_status_module` 启用 "server status" 页）等信息。在不同版本间，选项可能会有些许变化。

### 2.4 配置 Nginx

1.查看端口情况

```
$ netstat -lunapt | grep 80
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515147660442.png/wm)

因为在环境中启动了 Apache ，所以可以看到 80 端口目前是被占用的（此环境是在容器里，所以看不到服务的名称），而 Nginx 的默认端口也是 80，所以我们需要对其进行修改。

```
$ sudo vim /etc/nginx/sites-available/default
```

将里面的 80 全部替换掉，主要就是 listen 的那个配置项改成 8080 这个端口或者其他没有用的端口即可。

2.然后在 server 中的 location/ 配置块下面添加一个配置模块。

用于监控运行状态

```
location /nginx_status {
    stub_status on;
}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515147703352.png/wm)

3.启动 nginx

```
$ sudo service nginx start
$ netstat -lunapt | grep 8080
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515147742470.png/wm)

这时可以在 web 前端输入 `127.0.0.1:8080` 就可以进入到 nginx 的主页面了。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148780008.png/wm)

输入：`127.0.0.1:8080/nginx_status` 就可以查看 `Nginx` 的运行状态。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148832448.png/wm)

## 3. Zabbix agent 配置

在前面的学习中我们知道对服务的监控需要通过 agent 去采集数据，所以下面就该对 agent 进行配置。

在 `/etc/zabbix/zabbix_agentd.conf` 配置文件中的最底端添加下面的这个语句，让 agent 使用脚本来采集 Nginx 的数据，然后发送给 server。

```
$ sudo vim /etc/zabbix/zabbix_agentd.conf

UserParameter=nginx.status[*],/bin/bash /etc/zabbix/zabbix_agentd.d/nginx_status.sh $1
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515147771444.png/wm)

配置完后重启一下 agent 服务

```
$ sudo service zabbix-agent restart
```

接着我们来编写下用于采集数据的信息的脚本

```
$ sudo vim /etc/zabbix/zabbix_agentd.d/nginx_status.sh
```

```bash
#!/bin/bash

# 设置变量
BKUP_DATE=`/bin/date +%Y%m%d`
LOG="/data/log/zabbix/webstatus.log"
HOST=127.0.0.1  # 确保 CURL 能访问这个主机的 IP 地址
PORT="8080"    # 端口号

# 编写函数用于获取 nginx 的统计信息
function active {
  /usr/bin/curl "http://$HOST:$PORT/nginx_status" 2>/dev/null| grep 'Active' | awk '{print $NF}'
  }
function reading {
  /usr/bin/curl "http://$HOST:$PORT/nginx_status" 2>/dev/null| grep 'Reading' | awk '{print $2}'
  }
function writing {
  /usr/bin/curl "http://$HOST:$PORT/nginx_status" 2>/dev/null| grep 'Writing' | awk '{print $4}'
  }
function waiting {
  /usr/bin/curl "http://$HOST:$PORT/nginx_status" 2>/dev/null| grep 'Waiting' | awk '{print $6}'
  }
function accepts {
  /usr/bin/curl "http://$HOST:$PORT/nginx_status" 2>/dev/null| awk NR==3 | awk '{print $1}'
  }
function handled {
  /usr/bin/curl "http://$HOST:$PORT/nginx_status" 2>/dev/null| awk NR==3 | awk '{print $2}'
  }
function requests {
  /usr/bin/curl "http://$HOST:$PORT/nginx_status" 2>/dev/null| awk NR==3 | awk '{print $3}'
  }

$1

```

在 Zabbix 逻辑结构中知道，`Zabbix agent` 监控代理获取（采集）数据是通过 `zabbix_get` 进程来取得数据的。这里我们需要安装一下 `zabbix-get` 这个软件包。

```
$ sudo apt-get install zabbix-get
```

Zabbix agent 配置操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week6/11-1.mp4
@`

## 4. Zabbix 配置

下面就是 Zabbix 的配置，我们需要先创建一个模板来添加 items 监控项。

### 4.1 创建 Templates Nginx 模版

打开 zabbix 前端界面，点击：`Configuration → Templates`，然后选择 `create template` 创建一个新的模板，接着进入 `Templates` 模板配置界面，填写相应的参数项。如下图操作：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515147928800.png/wm)

### 4.2 创建 items 监控项

1.在模板中先添加如下 3 个连接操作的 item 监控项

+ nginx.accepts
+ nginx.handled
+ nginx.requests

```
Name                nginx.accepts
Type                Zabbix agent
Key                 nginx.status[accepts]
Type of information Numeric (unsigned)
Store value         1s # 每秒变化
Show  value         As is
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515147956422.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515147962404.png/wm)

（注：3 个监控项操作相似）

2.再创建 4 个交互操作的 items 监控项

+ nginx.active
+ nginx.reading
+ nginx.writing
+ nginx.waiting

```
Name                nginx.active
Type                Zabbix agent
Key                 nginx.status[active]
Type of information Numeric (unsigned)
Show  value         As is
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515147993828.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148022736.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148032015.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148044106.png/wm)

配置完成后，状态栏显示是 `enabled`

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148141361.png/wm)

可以用 `zabbix_get` 来测试一下是否可以进行数据的采集。

```
$ zabbix_get -s 127.0.0.1 -k 'nginx.status[accepts]'
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515147882201.png/wm)

> `-s`: 指定客户端主机名或者 IP 。
>
> `-k`: 你想获取的 key 。

返回一个数据则表示配置成功。配置出错可以通过查看日志信息来进行排错。

### 4.3 创建 Graphs

点击：`Tempates -> Graphs`，创建两个 Graphs。

+ nginx_connect
+ nginx_interact

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148161986.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148169182.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148174353.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148179243.png/wm)

### 4.4 创建监控的 Screen

在 `Monitoring->Screens` 中创建监控 Nginx 的 Screen（注意配置为1列2行），将前面创建的两个图像添加进去。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid1labid3535timestamp1515576576653.png/wm)

点击添加后，选择 `Constructor` 将之前配置的两个 graph 添加到 screen 中。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148455083.png/wm)

### 4.5 查看 Nginx

在 Zabbix Web 页面中的 `Monitoring->Screens`  查看 `nginx screen` 的信息图表了：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid600timestamp1515148606811.png/wm)

Zabbix 配置操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week6/11-2.mp4
@`


## 5. 总结

通过本节实验的学习大家可以学到如何用 zabbix 去监控一项服务。
