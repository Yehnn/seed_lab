---
show: step
version: 0.1
enable_checker: true
---

#Linux系统监控工具——Nagios

## 一、实验简介

Nagios 是一个监视系统运行状态和网络信息的监视系统。Nagios能监视所指定的本地或远程主机以及服务，同时提供异常通知功能等。

Nagios 可运行在Linux/Unix平台之上，同时提供一个可选的基于浏览器的WEB界面以方便系统管理人员查看网络状态，各种系统问题，以及日志等等。

Nagios 可以监控的功能有：

1、监控网络服务（SMTP、POP3、HTTP、NNTP、PING等）；

2、监控主机资源（处理器负荷、磁盘利用率等）；

3、简单地插件设计使得用户可以方便地扩展自己服务的检测方法；

4、并行服务检查机制；

5、具备定义网络分层结构的能力，用"parent"主机定义来表达网络主机间的关系，这种关系可被用来发现和明晰主机宕机或不可达状态；

6、当服务或主机问题产生与解决时将告警发送给联系人（通过EMail、短信、用户定义方式）；

7、可以定义一些处理程序，使之能够在服务或者主机发生故障时起到预防作用；

8、自动的日志滚动功能；

9、可以支持并实现对主机的冗余监控；

10、可选的WEB界面用于查看当前的网络状态、通知和故障历史、日志文件等；

### 1.1 安装Nagios

**首先update一下，然后安装Nagios，同时安装Apache、PHP5、Postfix，所以下面会涉及简单的邮件服务器的配置。**

```
$ sudo apt-get update
$ sudo apt-get install nagios3 apache2 php5 libapache2-mod-php5 postfix
```

操作截图：

点击“TAB”键选择确定：

![3-1.1-1](https://doc.shiyanlou.com/userid42227labid998time1431307259433/wm)

回车确定：

![3-1.1-2](https://doc.shiyanlou.com/userid42227labid998time1431307295708/wm)

填写邮件服务器域名，此处选择默认主机名
，直接“TAB+回车”确定：

![3-1.1-3](https://doc.shiyanlou.com/userid42227labid998time1431307392587/wm)

填写Nagios管理员密码：

![3-1.1-4](https://doc.shiyanlou.com/userid42227labid998time1431307468079/wm)


### 1.2 安装完毕,查看配置文件分布
```
$ sudo apt-get install tree

$ cd /etc/nagios3

$ tree
```

操作截图：

![3-1.2](https://doc.shiyanlou.com/userid42227labid998time1431307626805/wm)

### 1.3 访问测试

启动apache2与nagios：
```
$ sudo service apache2 start

$ sudo service nagios3 start
```

```checker
- name: check service
  script: |
    #!/bin/bash
    ps -ef |grep -v grep|grep -E 'nagios3|apache2'
  error: 没有启动 apache2 或 nagios3
```

在火狐浏览器中输入下面的网址进入nagios，用户名为nagiosadmin密码为安装时设定的密码效果如下图所示:

```
http://127.0.0.1/nagios3/
```

操作截图：

![3-1.3-1](https://doc.shiyanlou.com/userid42227labid998time1431308045851/wm)

----------


![3-1.3-2](https://doc.shiyanlou.com/userid42227labid998time1431308107770/wm)


## 二、使用

在nagios的web页面中，可以看到一些目录，点击左边目录中的“service”，可查看所有用户的服务状态详细信息。

操作截图：

![3-2](https://doc.shiyanlou.com/userid42227labid998time1431310641299/wm)

当然根据左边的目录可查看相应的部分。

###参考文档：
(1)  http://baike.baidu.com/link?url=YzwvXDZDmnJ9vV5ra-XmVfLofFD35dfhjl_irfydsmsGJbCbdURTuk_S3D4m0XlnBeQqFxOP7HwCgfSbY3odX_


