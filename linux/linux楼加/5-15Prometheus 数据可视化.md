---
show: step
version: 1.0
enable_checker: true
---
# Prometheus 数据可视化

## 1. 实验介绍

Prometheus 支持多种数据可视化方式，除了内置的表达式浏览器和控制台模板两种方式，还可使用开源的时序数据可视化工具 [Grafana](http://grafana.org/) 来展示监控数据。表达式浏览器功能最简单，控制台模板最灵活但使用门槛也最高，Grafana 兼顾了功能、灵活性和美观度，本次实验会重点讲解这种方式。

## 2. 实验知识点

- 表达式浏览器
- Grafana
- 控制台模板

## 3. 表达式浏览器

表达式浏览器在前面的实验中已经介绍过，打开 Prometheus 服务的路径 `/graph` 即可使用。其功能比较简单，也无需额外配置，这里就不再多做讲解。

## 4. Grafana

Grafana 是一个开源的、漂亮的数据分析和监控平台，也是时序数据分析工具的领头羊。它支持非常多的数据源，截止到目前包括 Prometheus 在内共有 40 多种。使用它可以将企业里存储在各个地方的数据汇总到一个地方来进行分析和监控。

### 4.1. 安装

Grafana 支持 Ubuntu/Debian、CentOS/Redhat、Windows、Mac、Docker 等多种系统或平台，并为每种平台都提供了安装包或镜像。甚至还可以通过编译源码的方式来安装，因为它是开源的。下面我们重点讲解我们的实验环境使用的 Ubuntu/Debian 系统下的安装，其它系统可参考 [官方文档](http://docs.grafana.org/installation/)。

#### 4.1.1. Deb 安装包方式

首先，下载 Deb 安装包。官方提供的下载地址在国外，可能速度会比较慢，可以从实验楼下载 [grafana_5.1.0_amd64.deb](http://labfile.oss.aliyuncs.com/courses/980/05/assets/grafana_5.1.0_amd64.deb)。

其次，使用 `dpkg` 命令来安装。安装之前还要先安装几个依赖软件。

```bash
wget http://labfile.oss.aliyuncs.com/courses/980/05/assets/grafana_5.1.0_amd64.deb
sudo apt-get install -y adduser libfontconfig
sudo dpkg -i grafana_5.1.0_amd64.deb
```

#### 4.1.2. APT 仓库方式

首先，添加 Grafana 的 APT 源。

```bash
deb https://packagecloud.io/grafana/stable/debian/ stretch main
```

虽然上面的版本写的是 `stretch`，也就是 Debian 9，但其它 Ubuntu/Debian 版本也都可以使用上面的源。

其次，添加源站 Package Cloud 的签名。

```bash
curl https://packagecloud.io/gpg.key | sudo apt-key add -
```

最后，使用 `apt-get` 命令来安装。

```bash
sudo apt-get update
sudo apt-get install grafana
```

在一些较老版本的系统上可能不支持 HTTPS 下载安装包，遇到这种情况需要先安装 `apt-transport-https` 这个工具。

```bash
sudo apt-get install -y apt-transport-https
```

APT 仓库方式和 Deb 包方式最终的安装效果都一样，区别只在于安装包的下载方式，一个是手动，一个是自动。它们安装到系统里的文件有：

- 二进制可执行文件 /usr/sbin/grafana-server
- Init.d 脚本 /etc/init.d/grafana-server
- 默认文件（环境变量） /etc/default/grafana-server
- 配置文件 /etc/grafana/grafana.ini
- Systemd 服务 grafana-server.service
- 日志文件 /var/log/grafana/grafana.log
- Sqlite3 数据库文件 /var/lib/grafana/grafana.db
- HTML/JS/CSS 等其它资源文件 /usr/share/grafana

如果系统使用 Systemd 来管理服务（Ubuntu 16 和 Debian 8 及以上），那么可以使用下面的命令来启动 Grafana 服务。

```bash
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5953timestamp1528872677713.png/wm)

否则使用 Init.d 的命令来启动。

```bash
sudo service grafana-server start
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep grafana
  error: 没有启动 grafana-server
```

#### 4.1.3. 二进制包方式

首先，下载二进制包。同样考虑到网速原因，实验楼也提供了下载 [grafana-5.1.0.linux-x64.tar.gz](http://labfile.oss.aliyuncs.com/courses/980/05/assets/grafana-5.1.0.linux-x64.tar.gz)。

然后，解压二进制包。

```bash
tar -zxvf grafana-5.1.0.linux-x64.tar.gz
```

二进制包无需安装，解压出来的目录里包含了所有相关文件，包括可执行程序和配置文件。

进入到解压目录，执行下面的命令即可启动 Grafana 服务。

```bash
cd grafana-5.1.0.linux-x64
./bin/grafana-server web
```

### 4.2. 使用

Grafana 服务启动之后，默认会在本地 3000 端口上监听，打开 http://localhost:3000/ 可访问其 Web 控制台。首次访问需要登录，默认的管理员账号/密码为 admin/admin。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5953timestamp1528872697602.png/wm)

登录完成后会进入到 Home 页，里面显示安装已完成，接下来需要 `Add data source`。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5953timestamp1528872724563.png/wm)
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5953timestamp1528872725113.png/wm)

在添加数据源页，填写或选择 Name、Type、URL 几项，其余保持默认即可。注意 Type 一定要选择 Prometheus，URL 填写 Prometheus 服务的地址。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5953timestamp1528872762781.png/wm)

填写完数据源表单后，点击 `Save & Test`，如果没有错误会显示“Data source is working”。然后通过左侧导航条里的链接回到 Home 页，可以看到“Add data source”任务已完成，接下来需要 `New dashboard`。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5953timestamp1528872780642.png/wm)

Dashboard 用来将相关 Panel 组织在一起，方便集中浏览。在创建 Dashboard 页，添加一个类型为 Graph 的 Panel。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5953timestamp1528872807493.png/wm)

刚添加的 Panel 为空，里面没有任何数据。点击“Panel Title”，展开下拉菜单，选择 `Edit` 操作来编辑 Panel。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5953timestamp1528872826058.png/wm)

在 Metrics Tab 下添加一个 Metric http_requests_total，就可以看到这个 Metric 相关的时间序列数据在上方图表里展示出来。一个 Panel 里可以添加多个 Metric，以方便对比数据。切换到 General Tab，修改 Panel 名称为“Prometheus Requests Total”。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5953timestamp1528872844412.png/wm)

第一个 Panel 添加完成后，点击顶部工具条里的 `Save dashboard` 按钮来保存 Dashboard。第一次保存 Dashboard，需要给 Dashboard 取一个名字。填写 Name 为 “Prometheus”，然后 `Save`。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5953timestamp1528872865120.png/wm)

第一个 Dashboard 创建好后，确保左上方的当前 Dashboard 显示的是刚才创建的 Prometheus，就会看到我们刚才创建的 Panel “Prometheus Requests Total”。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5953timestamp1528872881834.png/wm)

在 Prometheus Dashboard 里选择顶部工具条里的 `Add panel` 来再添加两个 Panel，分别用于展示 Metric http_request_duration_microseconds 和 http_request_size_bytes。这样 Prometheus Dashboard 里就包含了三个 Panel，可以通过拖动来调整各个 Panel 的位置和大小。

## 5. 控制台模板

控制台模板允许你使用 [Go 模板语言](http://golang.org/pkg/text/template/) 来创建任意想要的控制台界面，它在 Prometheus 服务端里渲染。控制台模板方式能力最强，但刚开始的学习曲线较陡，对于新手应尽量使用 Grafana 这样的方式。这里不再详细展开，需要的时候大家可以查阅 [官方资料](https://prometheus.io/docs/visualization/consoles/)。

## 6. 实验总结

本次实验我们着重学习了如何使用 Grafana 来展示 Prometheus 采集到的数据，可以看到两者搭配起来非常完美。