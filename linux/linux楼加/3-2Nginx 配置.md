---
show: step
version: 1.0
enable_checker: true
---
# Nginx 配置

## 1. 实验介绍

#### 1.1 实验内容

在本实验中将带领大家学习继 Apache 之后迅猛崛起的 Web 服务工具：Nginx。

Nginx 以其高效处理并发连接而迅速崛起，加上其高扩展性、高可靠性、低内存消耗等等优势不断吞噬市场的占有率，同样因其强大的功能与繁多的配置项我们没有办法做到面面俱到，我们仅从简单的配置与常用的使用来入门 Nginx 的使用。 

#### 1.2 实验知识点

+ Nginx 简介
  + Nginx 是什么
  + Nginx 的优势
+ Nginx 安装与配置
  + Nginx 的安装
  + Nginx 的模块
  + Nginx 的配置讲解
+ Nginx 配置实例
  + Nginx 实现负载均衡
  + Nginx 与 PHP 搭建服务

#### 1.3 推荐阅读

+ [Nginx 官方文档](https://nginx.org/en/docs/)

+ [Nginx 源码剖析](http://tengine.taobao.org/book/chapter_02.html)

## 2. Nginx 简介

#### 2.1 Nginx 是什么

Nginx 是一个高性能的网页服务器，能够反向代理 `HTTP`、 `HTTPS`、`SMTP`、 `POP3`、 `IMAP` ，也可以作为一个负载均衡器和 HTTP 缓存。是一个免费的、开源的、高性能的 HTTP 服务器。

Nginx 以其高性能、稳定性、丰富的特性、以及简单配置和低资源消耗而著称。Nginx 是由 `Igor Sysoev` 开发设计来供俄罗斯的大型门户网站和搜索引擎 Rambler 的使用。此软件在 BSD-like 协议下发行，可以在 UNIX、GNU/Linux、BSD、Mac OS X、Solaris，和 Microsoft Windows 等操作系统中运行。

#### 2.2 Nginx 的优势

与传统的服务器不同，`Nginx` 不依赖线程来处理请求。相反，它使用了一个更具可扩展性的事件驱动（异步）体系结构。这种体系结构使用较小的内存量，但更重要的是，在负载下可预测。即使你不希望同时处理数千个请求，但仍然可以从 `Nginx` 的高性能和小内存占用中受益。`Nginx` 在所有方向都可以扩展：从最小的 `VPS（Virtual Private Servers）`到大型的服务器集群。

## 3. Nginx 安装与配置

下面我们将会学习Nginx 安装与配置。


### 3.1 Nginx 的安装

Nginx 的安装方式同样分为两种：

+ 包管理工具安装；
+ 从源码进行编译安装；

两种安装方式的选择上，包管理工具方式安装简单、便捷，但是一般系统源为了稳定性所以版本会有所滞后，而手动编译安装方式稍显麻烦，但却可以安装最新版本，及时修补问题，并且在编译时添加自定义参数如修改安装位置、加载模块、修改环境变量、开关部分功能等等。

在 Nginx 中添加新的模块并不如 Apache 那样的方便，安装相关的工具，通过命令加载即可，在 Nginx 中添加新的模块需要在编译时添加相关的参数，然后安装才行。

所当我们需要添加一些默认不提供的模块我们必须使用编译安装。

1.包管理工具安装

在我们的实验环境中使用的是 ubuntu 14.04 的系统，使用的是 apt 的管理工具，所以我们只需要通过这样一个简单的命令即可安装：

```bash
sudo apt-get install nginx
```

当然即想简单的安装，有想安装最新版的 Nginx，可以通过添加源的方式来完成：

将下面内容添加到 `/etc/apt/sources.list.d` 文件下，这里我们重新创建一个文件 `nginx.list` 来存放下面的源信息：

```bash
$ sudo vim /etc/apt/sources.list.d/nginx.list

# 添加内容
deb http://nginx.org/packages/ubuntu/ trusty nginx
deb-src http://nginx.org/packages/ubuntu/ trusty nginx
```

需要执行下面的命令来添加验证密钥：

```bash
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys ABF5BD827BD9BF62
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517278288955.png-wm)

```bash
$ sudo apt-get update
$ sudo apt-get install nginx
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2767331517278548989-wm)

这样即可安装 nginx1.12.2 目前最新的稳定版本了。

### 3.2 Nginx 的配置讲解

在 Nginx 的配置文件中包含了各种指令，其中指令又分为简单指令和块指令：

- 简单指令是由名称和参数组成，之间以空格分隔，并以分号（;）结束；
- 块指令和简单指令具有相似的结构，不过不是由分号结尾，而是由一组大括号（{}）包含一些附加指令。这样的块指令又称为上下文（如：event、http、server 和 location）。

Nginx 及其模块的工作方式是在配置文件中确定的，默认的配置文件（nginx.conf）存放在目录 `/etc/nginx` 下。

如下为默认的配置文件内容：

![实验楼](https://dn-simplecloud.shiyanlou.com/2767331517286196039-wm)

可以将 Nginx 的配置文件的结构抽象成如下示意图：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517307303225.png-wm)

#### Nginx 的模块

Nginx 的架构是以高度模块化为设计的基础，除了非常少量的核心代码，其他的一切皆是模块，高度抽象的模块接口，结构的设计简单，使得 Nginx 十分的灵活与高效，默认情况下只会加载默认、必须的模块，其他的一些功能实现需要加载一些第三方的模块。

而配置文件中的各个指令配置项其实便是对模块的一个功能配置。

Nginx 在运行时至少需要加载几个核心的模块和一个事件类模块，这些模块运行时所支持的配置项称为基础配置，这样的配置项很多，一般将其分为四大类：

- 调试配置项
- 必备配置项
- 优化配置项
- 事件类配置项

有一些基础配置项没有在 nginx.conf 配置文件中写出，但是会有默认的值，这里只是简单的罗列配置文件中有的配置项：

```bash
user nginx;  # 运行用户，为必备配置项
worker_processes 1; # 启动工作进程数量，通常设置为可用的 CPU 内核数，为优化配置项

# 错误日志文件的路径与级别，为调试配置项
error_log /var/log/nginx-error.log warn; 

# pid  logs/nginx.pid; # 定义 PID 文件位置，为必备配置项

# 事件配置块上下文，其中指定影响连接处理的指令。
events {
    worker_connections 2048; # 设置一个工作进程可以打开的同时连接的最大数量。默认为 1024
}
```

其他更多核心配置项可以参考[官方说明](http://nginx.org/en/docs/ngx_core_module.html#example)

在 Nginx 中有一个非常重要的核心模块就是 `ngx_http_core_module` 模块，它实现了静态 Web 服务器的主要功能，而其相关的配置项都放在 `http{}` 配置块中：

```bash
http {
	 # 设置 mime 的类型，类型由 mime.type 文件定义
     include       /etc/nginx/mime.types;
     default_type  application/octet-stream;

	 # 设定日志格式
     log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                       '$status $body_bytes_sent "$http_referer" '
                       '"$http_user_agent" "$http_x_forwarded_for"';

     access_log  /var/log/nginx/access.log  main;

    # sendfile 指令指定 nginx 是否调用 sendfile 函数来输出文件，
    # 对于普通应用，必须设为 on,
    # 如果用来进行下载等应用磁盘 IO 重负载应用，可设置为 off，
    # 以平衡磁盘与网络 I/O 处理速度，降低系统的 uptime
     sendfile        on;
     #tcp_nopush     on;

    # 连接超时时间
     keepalive_timeout  65;

     # 开启 gzip 压缩
     # gzip  on;

     # 包含 /etc/nginx/conf.d 下的所有以 conf 结尾的配置文件
     include /etc/nginx/conf.d/*.conf;
 }
```

一个典型、完整的静态 Web 服务器还会包含多个 server 配置块和 location 配置块，例如（/etc/nginx/site-avaliabel/default.conf）：

```bash
# 设置虚拟主机配置
 server {
        # 侦听 80 端口
        listen    80;
        # 设置虚拟服务器的名称
        server_name  www.nginx.cn;

        # 定义服务器的默认网站根目录位置
        root /data/www/html;

        # 设定本虚拟主机的访问日志路径与级别
        access_log  logs/nginx.access.log  main;

        # location 块命令，根据请求目录或文件配置处理的行为

        # location / 表示当访问根目录时所做的行为
	    location / {
            # 按顺序查找访问的 uri 文件是否存在，或者相关的目录，若是不存在则返回 404
		    try_files $uri $uri/ =404;
	    }
        ……
}
```

server 配置块类似与 Apache 中的 `VirtualHost` 配置块，用于配置虚拟主机。

其中 location 用于匹配请求的 URI（URI 表示的是访问路径，除域名以外的内容），匹配的方式有多种：

- 精准匹配
- 忽略大小写的正则匹配
- 大小写敏感的正则匹配
- 前半部分匹配

其语法如下：

```bash
location [ = | ~ | ~* | ^~ ] pattern {
    ......
    ......
}

```

其中各个符号的含义：

- `=`：用于精准匹配，想请求的 uri 与 pattern 表达式完全匹配的时候才会执行 location 中的操作
- `~`：用于区分大小写的正则匹配；
- `~*`：用于不区分大小写的正则匹配；
- `^~`：用于匹配 URI 的前半部分；

我们以这样的实例来进一步理解：

```bash
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```

- 当访问 `www.shiyanlou.com` 时，请求访问的是 `/`，所以将与配置 A 匹配；
- 当访问 `www.shiyanlou.com/test.html` 时，请求将与配置 B 匹配；
- 当访问 `www.shiyanlou.com/documents/document.html` 时，请求将匹配配置 C;
- 当访问 `www.shiyanlou.com/images/1.gif` 请求将匹配配置D；
- 当访问 `www.shiyanlou.com/docs/1.jpg` 请求将匹配配置 E。

当一个 URI 能够同时配被多 location 匹配的时候，则按顺序被第一个 location 所匹配。

在 location 中处理请求的方式有很多，如上文中的 `try_files`，还有常用的 `pass_proxy` 等等，下面将通过几个配置实例带领大家进一步的学习。

## 4. Nginx 配置实例

前面我们主要讲了默认配置文件的一些命令，下面我们就来实际操作，进一步深入了解 Nginx 的各项配置。

较好的习惯是将所有的虚拟主机配置文件（也就是 server 配置块的内容）存放在 `/etc/nginx/conf.d/` 文件目录中，在主配置文件中已经默认声明了会读取 `/etc/nginx/conf.d/` 文件下所有 `*.conf` 文件。

这样做的目的是为了方便维护我们 server 相关配置，不会让某一个配置文件过于庞大。

补充：更多的执行指令说明可以参考[官方说明](http://nginx.org/en/docs/dirindex.html)

### 4.1 Nginx 实现负载均衡

负载均衡是优化资源的利用率、最大化吞吐量、减少延迟以及确保容错配置的常用技术。使用 Nginx 作为高效的 HTTP 负载平衡器来将流量分配给多个应用程序服务器，并提高 Web 应用程序的可伸缩性和可靠性性能。而负载均衡的实现主要依赖于反向代理与 upstream 相关模块的结合使用。

注意：反向代理方式是指用当前的服务器（一般称为代理服务器）来接受用户的连接请求，然后将请求转发给内部网络中的其他服务器（一般称为上游服务器），并将从上游服务器上获得结果返回给用户。由于 Nginx 有着很强大的高并发、高负载的能力，所以一般会作为前端服务器直接用户提供静态文件服务，而不适合 Nginx 处理的多变业务将使用反向代理的功能交给适合处理的 Web 服务上。

1.我们可以先配置一个最简单的负载均衡，如下：

```bash
$ sudo vim /etc/nginx/conf.d/LoadBalance.conf
```

```bash
upstream shiyanlounode {
    # 2 个不同端口的服务地址来模拟多节点运行相同的服务，此时没有特别的配置负载均衡的方法，所以默认为循环。
     server 127.0.0.1:8080;
     server 127.0.0.1:8081;
}
# 设置虚拟主机
server {
    # 监听 800 端口
     listen 81;

    # location 块
     location / {
       # 反向代理指令，将所有的请求都发送给 shiyanlounode 机器组中的机器
       proxy_pass http://shiyanlounode;
     }
}

# 创建多个节点来模拟上游服务器
server {
    listen 8080;
    root /usr/share/nginx/html;
	location / {
		try_files $uri $uri/ =404;
	}
}

server {
    listen 8081;
    root /usr/share/nginx/html;
	location / {
		try_files $uri $uri/ =404;
	}
}
```

`proxy_pass` 便是 HTTP proxy module 中重要配置项，通过该配置项我们可以配置将我们的请求转发给哪一台服务节点，或者哪一个服务群。

当设置转发给某一个节点时，其格式为 `proxy_pass http://ip或者域名:端口`，例如：

```bash
proxy_pass http://192.168.1.2:8080
```

当需要转发给某一个服务器群时，需要结合 upstream 使用，如上文所示，其格式如下：

```bash
upstream 群名字 {
    server ip或者域名:端口；
    server ip或者域名:端口；
    ......
    ......
}

server {
    ...
    ...
    location / {
        proxy_pass 群名字
    }
}
```

2.为了让展示效果更明显，我们可以修改调度节点的 root，值改为 `/home/shiyanlou`，并在 `/home/shiyanlou` 目录下创建一个 index.html：

```bash
echo "Shiyanlou Louplus" > /home/shiyanlou/index.html
```
```checker
- name: check file
  script: |
    #!/bin/bash
	ls /home/shiyanlou/index.html
  error: /home/shiyanlou 目录下没有 index.html 文件
```

3.在让配置生效之前我们可以通过这样的命令来检查我们的配置文件是否有误：

```bash
# 该命令可以简单检查配置文件中的语法问题
sudo nginx -t
```
```checker
- name: check nginx conf
  script: |
    #!/bin/bash
	nginx -t
  error: nginx 配置有误
```
若是无误会输出 `successful` 的提示，若是有误则会提示语法错误的位置，以便于我们根据需求修改

4.在确认配置文件无误之后我们便让配置文件生效：

```bash
sudo nginx -s reload
```

reload 是一个优雅的查新加载配置文件的做法，在不用中断 nginx 服务的情况下使得我们的配置文件生效。

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517349296358-wm)

```checker
- name: check url
  script: |
    #!/bin/bash
	wget --spider -q -o /dev/null  --tries=1 -T 5 http://localhost:81
  error: http://localhost:81 访问失败
```

在默认的负载均衡上，nginx 还支持一些调度算法来配置：

+ 最少连接负载均衡：下一个请求被分配给连接数最少的服务器

配置如下所示：

```bash
upstream shiyanlounode {
     least_conn;
     server 127.0.0.1:8080;
     server 127.0.0.1:8081;
    }
```

+ 加权负载均衡：通过使用权重，来实现负载均衡

```bash
upstream shiyanlounode {
     server 127.0.0.1:8080 weight=3;
     server 127.0.0.1:8081;
    }
```

+ ip-hash 负载均衡：将客户端的IP地址被用作哈希键，以决定服务器组中应该为客户端请求选择什么服务器。此方法可确保来自同一客户端的请求将始终定向到同一服务器，除非此服务器不可用:

```bash
upstream shiyanlounode {
    ip_hash;
     server 127.0.0.1:8000;
     server 127.0.0.1:8001;
     server 127.0.0.1:8002;
     server 127.0.0.1:8003;
}
```

Nginx 实现负载均衡配置实例操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/2-1.mp4
@`
### 4.2 Nginx 与 PHP 搭建服务

Nginx 一般为用户提供静态文件服务，动态的页面服务本生并无法提供，Nginx 不像 Apache 默认加载了 php/python 等解释器模块，所以 Nginx 该如何结合 PHP 搭建服务呢？

首先 Nginx 无法解析 PHP，所以必然需要第三方模块，通常 Nginx 通过 fastcgi 将相关的请求发送给 php-fpm，通过 php-fpm 解析后将数据返回，Nginx 再将获取的数据返回给客户端。

注：CGI 是为了保证 web server 传递过来的数据是标准格式，而 fastcgi 是一个提高 cgi 性能的协议，而 php-fpm 是一个实现了该协议，并能够调度 php-cgi 解释器的程序。

1.启动环境中的 php-fpm

```bash
sudo service php5-fpm start
```
```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep php-fpm
  error: 没有启动 php5-fpm
```

2.创建 php 访问页面

```bash
sudo vim /usr/share/nginx/html/index.php

# 添加如下内容，展示 php 的参数信息
<?php
phpinfo();
?>
```

```checker
- name: check file
  script: |
    #!/bin/bash
	ls /usr/share/nginx/html/index.php
  error: /usr/share/nginx/html 目录下没有 index.php 文件
```

3.配置 nginx

```bash
sudo vim /etc/nginx/conf.d/php.conf


# 添加如下内容

server {
    listen 801;

    root /usr/share/nginx/html;
    index index.php;
    server_name localhost;

    location / { 
        try_files $uri $uri/ =404;
    }   

    # 用于处理访问的所有 php 页面
    location ~ \.php?.*$ {   
        fastcgi_index index.php;

        # 将相关的请求发送给 php5-fpm 处理
        fastcgi_pass   unix:/var/run/php5-fpm.sock;  
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;  
        include        fastcgi_params;
    }
}
```

```checker
- name: check content
  script: |
    #!/bin/bash
	grep "index.php" /etc/nginx/conf.d/php.conf
  error: /etc/nginx/conf.d/php.conf 内容不对
```

4.重新加载 nginx 配置文件

```bash
sudo service nginx reload
```

5.验证

在浏览器中访问 `localhost:801/index.php`，我们便成功的看到了 php 参数的展示页面：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517348967217-wm)

```checker
- name: check url
  script: |
    #!/bin/bash
	wget --spider -q -o /dev/null  --tries=1 -T 5 http://localhost:801/index.php
  error: http://localhost:801/index.php 访问失败
```

Nginx 与 PHP 搭建服务操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/2-2.mp4
@`

## 5. 总结

本节实验中我们更多的是对 nginx 的一个认识与虚拟主机的添加和 location 的简单应用，任何不清楚的地方欢迎与我们交流：

- Nginx 简介
  + Nginx 是什么
  + Nginx 的优势
- Nginx 安装与配置
  + Nginx 的安装
  + Nginx 的模块
  + Nginx 的配置讲解
- Nginx 配置实例
  + Nginx 与 PHP 搭建服务
  + Nginx 访问重定向
  + Nginx 实现负载均衡
  + Nginx 实现访问控制

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。