# HAProxy + Keepalived 搭建高可用负载系统 

## 1. 实验介绍

### 1.1 实验内容

在本实验中我们将为大家介绍另一款与 LVS 功能类似的软件--HAProxy，通过 HAproxy 与 Keepalived 的联合，搭建高可用服务。

### 1.2 实验知识点

+ HAProxy 工具简介
+ HAProxy + Keepalived 搭建高可用服务

### 1.3 推荐阅读

- [HAProxy 官方文档](http://www.haproxy.org/#desc)

## 2. HAproxy 简介

HAProxy 是一个免费，非常快速和可靠的解决方案，为基于 TCP 和 HTTP 的应用程序提供高可用、负载均衡和代理。 它特别适合高流量的网站和拥有世界上访问量最大的网站。多年来，它已经成为标准开源负载平衡器，现在已经与大多数主流 Linux 发行版一起发布，并且通常默认部署在云平台上。

HAProxy 是一款专做负载均衡的软件，其主要作用与 LVS 非常的类似，但还是有很大的差别：

- LVS 运行于内核空间，所以其效率高 HAProxy 不少
- LVS 的额外功能点很少，配置项也非常的少

而在 HAProxy 中：

- 虽然 HAProxy 的负载均衡能力比不过 LVS 但是却高于 Nginx 的负载均衡能力，性能非常优秀，HAProxy 的单机宽带速度可以达到 10Gbit/s，适用于大流量的负载系统
- 能够解析非常多的应用层协议，很多都是 LVS 所不能支持的
- 能够支持 Session 的保持，Cookie 的引导；同时支持通过获取指定的 url 来检测后端服务器的状态。
- 拥有强大的 ACL 特性，可以更加更加灵活的实现智能负载均衡
- 自带强大的监控服务器状态页面，可以实时查看系统运行状态
- HAProxy 支持连接拒绝和全透明代理等功能等等

接下来我们将通过这样的一个实验带领大家认识 HAProxy。

## 3. HAproxy + Keepalived 实验说明

### 3.1 环境介绍

整个集群系统的结构拓扑图如下所示：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514102392052.png-wm)

|   服务器名称   |      IP      |               用途               |
| :------------: | :----------: | :------------------------------: |
| HAProxy-master | 192.168.0.2  |       提供负载均衡服务(主)       |
| HAProxy-backup | 192.168.0.3  |       提供负载均衡服务(备)       |
| nginx-server-1 | 192.168.0.4  |  对应 `www.shiyanlou.com` 站点   |
| nginx-server-2 | 192.168.0.5  | 对应 `static.shiyanlou.com` 站点 |
| nginx-server-3 | 192.168.0.6  |  对应 `api.shiyanlou.com` 站点   |
|      VIP       | 192.168.0.10 |          集群 VIP 地址           |

结构说明：

通过负载层两台 HAProxy 服务器实现后端三台服务器的负载均衡。

后端有三台真实服务器，也分别对应着一个域名，但是不是通过域名就能访问到此服务器。

这里有三个域名：

- `www.shiyanlou.com` 代表实验楼主站
- `static.shiyanlou.com` 代表实验楼的静态资源存放站点，比如图片视频等，
- `api.shiyanlou.com` 代表实验的 api 调用接口站点。

这三个域名均对应着 Keepalived 的虚拟 IP 地址(VIP)，即通过访问这三个域名，请求都会导向 HAProxy 服务器，HAProxy 服务器会根据域名的不同，智能调度请求到后端的真实服务器集群中的一个。后端服务器处理的结果将会返给 HAProxy 服务器，再由 HAProxy 服务器将结果返回给客户端。

例如：

- 当客户端通过域名 `www.shiyanlou.com` 访问网站时，HAProxy 服务器会将请求发送到 nginx-server1；
- 当客户端通过域名 `static.shiyanlou.com` 访问网站时，HAProxy 将请求发送给 nginx-server2；
- 当客户端通过域名 `api.shiyanlou.com` 访问网站时，HAProxy 将请求发送给 nginx-server3；

从而实现集群的负载均衡。

一旦 HAProxy-master 发生故障，Keepalived 会将负载请求任务切换到 HAProxy-backup 服务器，实现负载均衡的高可用。

### 3.2 实验步骤

主要有以下的步骤：

- 使用 docker 创建所需要的服务器集群
- 在服务器中安装 HAProxy + Keepalived 服务
- 配置 HAProxy 实现高性能负载均衡系统
- 配置 Keepalived 实现高可用的负载调度器
- 分别测试负载负载均衡的可用性以及负载调度器的可靠性

### 3.3 测试步骤

通过这样的方式来测试我们内容的分发与 keepalived 的高可用：

- 在浏览器中分别访问三个域名，查看页面的展示
- 关闭与重新启动查看 keepalived 的状态变化
- 通过 HAProxy 的监控平台查看相关的信息

## 4. HAproxy + Keepalived 实验

### 4.1 创建服务器集群

通过如下的命令创建的我们的 container，按照顺序启动才能与我们所罗列的自列表中的 IP 地址相同：

```bash
docker run --privileged --name=HAProxy-master -tid ubuntu
docker run --privileged --name=HAProxy-backup -tid ubuntu
docker run --privileged --name=nginx-server-1 -tid ubuntu
docker run --privileged --name=nginx-server-2 -tid ubuntu
docker run --privileged --name=nginx-server-3 -tid ubuntu
```

使用 `docker ps` 查看创建且当前正在运行的容器:

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514105373814.png-wm)

### 4.2 安装与配置 nginx 服务器

以下操作在 nginx-server 服务器集群中操作。这里以 `ngxin-server-1` 为例，另外两台操作相同。

通过 `docker attach ngxin-server-1` 登录 server-1，依次执行以下命令：

```bash
apt-get update
apt-get install vim nginx -y
server nginx start	# 启动 nginx 服务
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514106306847.png-wm)

在其他的 nginx server 中作相同的安装步骤。

接下来分别修改三台 nginx 服务器中的 nginx 默认展示页面：

1.在 nginx-server-1 服务器中，修改响应 html 页面，命令如下：

```
vim /usr/share/nginx/html/index.html
```

修改内容如下图：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514106755820.png-wm)

因为此节点服务器对应 `www.shiyanlou.com` 的域名访问响应服务器。

2.nginx-server-2 修改如下：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514106969627.png-wm)

3.nginx-server-3 修改如下：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514107047067.png-wm)

三台 nginx 服务器都修改完成之后，打开宿主机的 firefox 浏览器，分别输入 nginx 服务器的 IP 地址，查看页面效果：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514107219109.png-wm)

上图输入了 nginx-server-1 的 IP 地址，页面显示对应域名 `www.shiyanlou.com` 的响应。

### 4.3 HAProxy 节点的安装

以下操作在 HAProxy-master 和 HAProxy-backup 两台服务器进行。

此处以 HAProxy-master 为示例，HAProxy-backup 操作相同。以下简称 `master` 和 `backup`。

在 master 和 backup 安装 HAProxy 和 Keepalived 软件，依次执行以下命令：

```bash
apt-get update 
apt-get install vim keepalived haproxy -y
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514105954428.png-wm)

实验环境安装操作视频:

`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/week8/1-install.mp4
@`

### 4.4 HAProxy 的 Keepalived 配置

HAProxy-master 作为主负载均衡器，HAProxy-backup 作为备用负载均衡器。VIP 地址为 `192.168.0.10`。 

进入 HAProxy-master 服务器，编辑配置文件：

```bash
# 进入 HAProxy-master
docker attach HAProxy-master 

# 编辑配置文件
vim /etc/keepalived/keepalived.conf	
```

配置文件内容如下所示(复制请去掉注释)：

```sh
# 全局配置，在发现某个节点出故障的时候以邮件的形式同时管理员
global_defs {
   notification_email {	#设置报警邮件地址，可以设置多个
        shiyanlouAdmin@localhost #每行一个，如果开启邮件报警，需要开启本机的 Sendmail 服务
        shiyanlou@admin.com
   }
   notification_email_from root #设置邮件的发送地址
   smtp_server 127.0.0.1 #设置 STMP 服务器地址
   smtp_connect_timeout 30 #设置连接 SMTP 服务器的超时时间
   router_id LVS_DEVEL	#标识，发邮件时显示在邮件主题中的信息
}

#配置 vrrp 实例，可配置多个，主备结构只需一个
vrrp_instance HAProxy_1 {
    state MASTER  #指定 Keepalived 角色， MASTER 表示此主机是主服务器，BACKUP 表示此主机是备用服务器
    interface eth0    #指定 HA 检测网络的接口
    virtual_router_id 51 #虚拟路由标识，这个标识是一个数字，同一个 vrrp_instance 下，MASTER 和BACKUP 必须是一致的
    priority 100  #定义优先级，数字越大，优先级越高。在同一个 vrrp_instance 下，MASTER 的优先级必须大于 BACKUP 的优先级
    advert_int 1  #设定 MASTER 与 BACKUP 负载均衡器之间同步检查的事件间隔，单位是秒
    authentication { #配置 vrrp 直接的认证 
        auth_type PASS	#设定验证类型和密码，验证类型分为 PASS 和 AH 两种
        auth_pass 1111	#设置验证密码，在一个 vrrp_instance 下，MASTER 与 BACKUP 必须使用相同的密码才能通信
    }   
    virtual_ipaddress { #配置虚拟 IP，可以设置多个，每行一个
        192.168.0.10
    }
}
```

上面的内容就是 Keepalived 的主节点配置信息。保存并退出 vim。

进入 HAProxy-backup，修改 Keepalived 配置文件：

```sh
docker attach HAProxy-backup
vim /etc/keepalived/keepalived.conf
```

配置文件内容基本同上，可直接复制过去(去掉注释)，然后做一点差异修改即可。将 HAProxy-backup 的节点角色设置为 BACKUP，再将他的优先级设置比 HAProxy-master 低。具体差异如下图：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514126625722.png-wm)

除了上面标出的部分，其他内容完全相同。修改完之后，保存退出 vim。Keepalived 的配置就完成了。

Keepavlied 配置视频:

`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/week8/2-keepalived-config.mp4
@`

### 4.4 HAProxy 的配置

配置 HAProxy 的内容相对较多，不过也不是很复杂，可以很容易理解。在 HAProxy-master 与 HAProxy-backup 服务器上，两者的 HAProxy 配置完全相同，只要在其中一台机器配置好，再将内容复制到另外一台机器上即可。我们这里以 HAProxy-master 为示例，HAProxy-backup 操作步骤与配置内容完全相同。

进入 HAProxy-master 服务器，编辑 HAProxy 配置文件，默认路径为 `/etc/haproxy/haproxy.cfg`：

```bash
docker attach HAProxy-master
vim /etc/haproxy/haproxy.cfg
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514127315606.png-wm)

打开配置文件，可以看到里面包含了两部分的配置内容，一个是全局的配置信息(global)，和默认的配置信息(defaults)。这是默认的配置文件内容，我们需要在这个基础上做自己的配置。

### 4.4.1 HAProxy 配置文件讲解

HAProxy 的配置非常灵活丰富。根据实际的功能和用途，HAProxy 的配置文件主要分为 5 个部分，但不是所有的心都是必须的，可以根据实际需求做灵活配置，默认的配置文件，包含了 global 部分和 defaults 部分：

- global 配置部分：这部分一般用来设定全局的配置参数，是属于进程级别的配置，一般和操作系统的配置相关，所以很少去修改它。

- defaults 配置部分：配置默认参数，在这部分配置的内容，默认会自动被引用到下面的 frontend 和 backend、listen 配置部分，可以配置某些公共的参数，如果下面部分配置的参数与 defaults 部分的配置相同，则会覆盖 defaults 的配置参数。

- frontend 配置部分：这部分用于设置接收用户请求的前端虚拟节点。frontend 部分可以根据 ACL 规则动态指定要使用的后端(backend)。与此对应的部分是 backend。

- backend 配置部分：这部分配置内容主要用于设置后端集群服务器，即可以提供 web 服务真实的服务器。一个 backend 部分通常对应一个后端服务器或一组后端服务器集群。其类似于 LVS 的 Real-Server 。

- listen 配置部分：此部分是 frontend 与backend 部分的结合体。

### 4.4.1 HAProxy 配置文件修改后

以下内容为修改好的 HAProxy 配置文件，大家可以结合注释理解，只需要将 frontend 、backend 和 listen 部分添加到你自己的服务器上即可，因为默认已经有 global 和 defaults。

- frontend 配置了前端负载均衡器的参数，即用户直接访问的端口
- backend 部分配置了三个实例，这里分成 3 个是因为我们只有三台后端服务器，且要根据域名访问到不同的服务器。
  + 定义名为 www 的backend 实例，对应以 `www.shiyanlou.com` 域名访问的后端响应服务器集群
  + 定义名为 static 的 backend 实例，对应以 `static.shiyanlou.com` 域名访问的后端响应服务器集群
  + 定义名为 api 的 backend 实例，对应以 `api.shiyanlou.com` 域名访问的后端响应服务器集群。由于后端服务器数量有限，所以每一个 backend 实例中，都只有配置了一个 server。你也可以根据实际情况，在一个 backend 实例中，配置多个 server 实例。

```bash
global
        log /dev/log    local0 #日志输出配置，所有日志都记录在本机, 由 local0 设备输出，日志文件为 /var/log/syslog
        log /dev/log    local1 notice #全局日志配置，指定使用 syslog 服务中的 local1 日志设备，记录日志的等级为 info 级别, 日志文件为 /var/log/haproxy.log
        chroot /var/lib/haproxy	#HAProxy 的工作目录
        user haproxy	#设置运行 HAProxy 进程的用户和组
        group haproxy
        daemon #设置 HAProxy 已守护进程的方式运行

defaults
        log     global	#使用 global 的日志配置
        mode    http	#设置 HAProxy 实例默认运行模式，有 tcp、http、health 三个值可选。tcp 是第四层，http 是第七层，health 是健康监测(基本不再使用)
        option  httplog #启用日志记录 HTTP 请求，默认 HAProxy 日志不会记录 HTTP 请求
        option  dontlognull #启动这个配置参数，日志中将不会记录空连接。如果该服务上游没有其他的负载均衡器，建议不适用此参数
        contimeout 5000 #设置成功连接到一台服务器的最长等待时间，默认单位是毫秒，新版本的haproxy使用timeout connect替代，该参数向后兼容
        clitimeout 50000 #设置连接客户端发送数据时的成功连接最长等待时间，默认单位是毫秒，新版本haproxy使用timeout client替代。该参数向后兼容
        srvtimeout 50000 #设置服务器端回应客户度数据发送的最长等待时间，默认单位是毫秒，新版本haproxy使用timeout server替代。该参数向后兼容
        errorfile 400 /etc/haproxy/errors/400.http #指定部分错误状态码的html文件
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http
        
#下面部分为添加的部分
frontend shiyanlou #定义一个名为 shiyanlou 的前端部分
	bind 0.0.0.0:80 #定义前端部分监听地址端口
	mode http #同上
	option httplog #同上
	option forwardfor #使用后端可以获取到客户端的真实 IP
	option httpclose #客户端与服务器完成一次请求后，HAProxy 将主动关闭此 TCP 链接，对性能有一定的帮助。
	log global #使用全局的日志配置
	default_backend www #指定默认的后端服务器池

backend www #定义一个名为 www 的后端部分, 对应 www.shiyanlou.com 站点
	mode http #同上
	balance source #设置调度算法，可以是以下几种算法之一：roundrobin,static-rr,source,leastconn,uri,uri_param,hdr(<name>)
	cookie SERVERID #允许向cookie插入SERVERID，每台服务器的SERVERID可在下面使用cookie关键字定义，此处可选
	option httpchk GET /index.html #开启对后端服务器的健康检测，通过 GET /index.html 的方式来判断后端服务器的健康情况
	server nginx-server-1 192.168.0.4:80 cookie server-1 weight 1 check inter 2000 rise 3 fall 3
	#server 关键字定义一个或一组后端真实服务器使用格式如下：
	#server <name> <ip>[:port] [param*]
	#param 是为后端服务器设定的一系列参数，这里列举几个常用的
	# check：表示启用对后端服务器健康检查
	# inter 设置健康检查的时间间隔
	# rise 检测正常多少次后可被认为后端服务器是可用的
	# fall 检测失败多少次以后可被认为是不可用的
	# weight 分发请求的权重
	# backup 设置为后端真实服务器的备份服务器，只有其他服务器都不可用时，才会启动此服务器

backend static #定义一个名为 static 的后端部分, 对应 static.shiyanlou.com 站点
	mode http #同上
	balance source #设置调度算法，可以是以下几种算法之一：roundrobin,static-rr,source,leastconn,uri,uri_param,hdr(<name>)
	cookie SERVERID #允许向cookie插入SERVERID，每台服务器的SERVERID可在下面使用cookie关键字定义，此处可选
	option httpchk GET /index.html #开启对后端服务器的健康检测，通过 GET /index.html 的方式来判断后端服务器的健康情况
	server nginx-server-2 192.168.0.5:80 cookie server-2 weight 1 check inter 2000 rise 3 fall 3

backend api #定义一个名为 api 的后端部分, 对应 api.shiyanlou.com 站点
	mode http #同上
	balance source #设置调度算法，可以是以下几种算法之一：roundrobin,static-rr,source,leastconn,uri,uri_param,hdr(<name>)
	cookie SERVERID #允许向cookie插入SERVERID，每台服务器的SERVERID可在下面使用cookie关键字定义，此处可选
	option httpchk GET /index.html #开启对后端服务器的健康检测，通过 GET /index.html 的方式来判断后端服务器的健康情况
	server nginx-server-3 192.168.0.6:80 cookie server-1 weight 1 check inter 2000 rise 3 fall 3

listen HAProxy_status	#定义了一个名为 HAProxy_status 的监控实例，也相当于定义了 HAProxy 的监控统计页面
	bind 0.0.0.0:3000 #绑定监控页面的地址，端口
	stats uri /haproxy-status #设置 HAProxy 监控页面的 url 地址
	stats refresh 30s #设置监控页面自动刷新时间
	stats realm welcome \login HAProxy #设置监控页面登录时的文本提示信息
	stats auth admin:admin	#设置登录监控页面的用户名和密码
	stats hide-version	#隐藏监控统计页面的 HAProxy 版本信息
	stats admin if TRUE  #设置此选项，可以在监控页面上手动启动或禁用后端真实服务器，仅在 HAProxy 1.4.9 以后版本有效
```

HAProxy 配置视频:

`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/week8/3-haproxy-config.mp4
@`

### 4.5 修改 hosts 文件

前面介绍过，我们将会有三个域名：

- `www.shiyanlou.com` 
- `static.shiyanlou.com`
- `api.shiyanlou.com` 

三个域名都会指向负载均衡层，对应 Keepalived 配置的虚拟 IP(VIP)。

因为三个域名我们只是在本地作测试使用，所以需要配置 hosts 文件，使用本地的域名解析即可。

我们需要在宿主机中修改 hosts 文件，脱离 docker 容器，进入默认环境终端：

编辑 hosts 配置文件：

```sh
vim /etc/hosts
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514358104237.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514358283752.png-wm)

上图中，我添加了三条本地 DNS 域名解析记录，三个域名都对应 `192.168.0.10` 即 VIP。通过任何一个域名访问，都会指向 VIP 地址，到达 HAProxy 负载均衡层。配置完成之后，保存退出。

### 4.6 服务运行

完成所有的配置之后，分别启动 HAProxy-master 和 HAProxy-backup 启动 Keepalived：

```bash
service keepalived start
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514359521787.png-wm)

上图中，Keepalived 已经成功启动，但是出现一些错误信息，这是因为 keepalived 在调用 IPVS 的内核模块失败所导致的，暂不处理，使用 `ctrl+c` 结束任务，使用命令：`ip a` 查看当前机器 IP 地址的情况：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514359668449.png-wm)

VIP 已经被绑定到当前服务器。说明 Keepalived 已经成功启动并且正在运行。

接下来启动 HAProxy 服务：

```sh
haproxy -c -f /etc/haproxy/haproxy.conf #检查配置文件是否存在语法错误
haproxy -d -f /etc/haproxy/haproxy.conf	#以调试模式启动 HAProxy
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514361147607.png-wm)

上图中，通过 `-c` 参数，检查了配置文件的正确性，没有问题。然后程序以 debug 模式运行，指定配置文件的路径。

HAProxy 服务就已经成功启动了，但是还是错误信息输出，这是因为 HAProxy 没有将日志信息发送给 rsyslog，我们需要提前 rsyslog(先使用 `ctrl+c` 停止 HAProxy)：

```bash
service rsyslog start
```

执行之后，重启 Keepalived 服务，然后再启动 HAProxy。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514362338036.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514362514702.png-wm)

通过手动指定参数的方式启动是因为我们希望让大家看到在启动出错时的一个调试信息的输出，若是想通过 service 命令来启动我们的 HAProxy 的话我们首先需要修改 `/etc/default/haproxy` 文件，将其中的 `ENABLED` 变量值修改为 1 即可，修改之后我们就可以通过：

```bash
service haproxy start
```

启动我们的 haproxy 服务了。

HAProxy 启动视频:

`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/week8/4-running-haproxy.mp4
@`

## 5. 配置测试

在地址栏输入域名：`www.shiyanlou.com`

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514362641090.png-wm)

页面正确返回。可以看到是 ngxin-server-1 响应了请求。

若是我们 `www.shiyanlou.com` 服务有多个节点，我们只需要在 www 的 backend 中配置多个 server 即可，这样便可达到 LVS 的效果。

此时我们访问 `api.shiyanlou.com` 与 `static.shiyanlou.com` 并不能达到我们预期的效果，返回的还是 `www.shiyanlou.com` 节点的内容：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514362747437.png-wm)

这是因为我们在 frontend 中配置了：

```bash
default_backend www
```

默认将所有的请求转发给 `www.shiyanlou.com` 节点，而多个 backend 的正确使用方式需要结合 HAProxy 的 ACL 与 `use_backend` 使用。

在没有配置 ACL 的情况，所有的请求都会发送给 `default_backend`，若是没有配置 ACL 也没有配置 `default_backend` 的话，HAProxy 将不知道转发给谁，此时会给我们返回 503 的错误页面，表示无法访问。

ACL 即访问控制技术。因为 HAProxy 可以工作在 7 层模型(即应用层)，因此，HAProxy 可以通过强大的 ACL 规则实现智能负载均衡系统。HAProxy 可以通过 ACL 规则实现以下两种主要的功能：

- 通过设置的 ACL 规则检查客户端请求是否合法，如果符合 ACL 规则，则可以通过，否则，则会直接中断请求（这是在没有指定 default_backend 的情况下）。
- 符合 ACL 规则的请求将被提交到相应的后端服务器集群，因此可以实现基于 ACL 规则的智能负载均衡，

HAProxy 中的 ACL规则一般放在 frontend 部分，语法规则如下：

```
acl alc名称 acl方法 -i [匹配的路径或文件]
```

> HAProxy 定义了很多 ACL 方法，常用的有 hdr_reg(host)、hdr_dom(host)、hdr_beg(host)、url_sub、url_dir、path_beg、path_end 等，更多关于 HAProxy 的 ACL 规则配置，请参考[官方文档](http://cbonte.github.io/haproxy-dconv/1.9/configuration.html#7.1)。

ACL 规则搭配 `use_backend` 使用时，`use_backend` 参数后面加上一个 backend 实例名，表示满足哪个 ACL 规则即可将请求发往哪个 backend 实例。

接下来就配置 ACL 规则，分别在 HAProxy-master 和 HAProxy-backup 负载调度器上进行，两者配置相同：

```bash
# 根据 PID，结束 HAProxy 进程
kill -9 $(ps -ef |grep haproxy|awk '{print $2}') 

# 编辑配置文件
vi /etc/haproxy/haproxy.cfg	
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514367255626.png-wm)

上图中，添加了 3 条 ACL 规则。

第一条规则表示：使用正则表达式匹配请求域名 ，如果以 www 开头，则此规则符合规则。

第二条规则表示：使用域名匹配，如果请求的域名为 `static.shiyanlou.com` ，则此规则符合规则。

第三条规则表示：使用子串匹配，如果请求的 uri 地址中(不包含域名)是否包含 `name=` 的字符串，若有则此规则符合规则。

分别在 HAProxy-master 和 HAProxy-backup 两台服务器上配置以上内容，然后分别重新启动 HAProxy 服务：

```bash
# 若是之前没有修改 /etc/default/haproxy 文件这样启动
haproxy -D -f /etc/haproxy/haproxy.cfg

# 反之使用 service 启动
service haproxy start
```

紧接着打开 firefox 的隐私窗口，地址栏分别输入 url：`www.shiyanlou.com` ，`static.shiyanlou.com` ，`api.shiyanlou.com?name=admin`。查看页面响应效果：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514369003018.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514369040755.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514369091360.png-wm)

上面三张图，分别展示不同域名访问，根据 ACL 规则，得到了相应得服务器的响应，由此便得到了我们期望的结果

### 6. 使用 Web 监控平台管理 HAProxy

在配置文件中，我们配置了一个 listen 的部分，即是 web 管理控制台。在新版的 HAProxy 中，自带了一个基于 Web 的监控平台，通过这个平台，可以实时查看此集群系统中所有后端服务器的运行状态，监控页面上会通过不同的颜色来展示服务器故障信息，极大地方便的运维人员的维护工作。

访问 `192.168.0.10:3000/haproxy-status` 地址，进入监控页面时需要验证，用户名和密码是你在配置文件写好的：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514369588753.png-wm)

验证之后即可进入监控页面：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514369708241.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514369807980.png-wm)

在这个页面上，可以查看到后端所有的服务器信息，Frontend 前端负载均衡器 `shiyanlou`，3 个 Backend 后端服务器：`www`、`static`、`api`。甚至可以直接管理每个后端服务的运行状态，可以关闭或开启，功能非常强大。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/108299/1514369984907.png-wm)

Squid 配置 ACL 及监控页面:

`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/week8/5-fix-and-monitor.mp4
@`

## 6. 总结

本实验中我们接触了一个新的负载均衡软件 HAProxy，并通过实践的方式学习了：

- HAProxy 的安装
- HAProxy 的配置
- HAProxy 的 ACL
- HAProxy 的监控平台

并将其与我们之前所学的 Keepalived 结合使用。