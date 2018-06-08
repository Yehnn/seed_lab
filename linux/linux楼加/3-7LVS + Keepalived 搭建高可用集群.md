<<<<<<< HEAD
---
show: step
version: 1.0
enable_checker: true
---
=======
>>>>>>> 3fa8bdf764cf14bf952ed26152bb4addb11c4e0c
# LVS + Keepalived 搭建高可用集群

## 1. 实验介绍

<<<<<<< HEAD
#### 1.1 实验内容

本节实验中，我们将学习如何使用 LVS(Linux Virtual Server) 搭配高可用集群软件 Keepalived 共同搭建高可用性集群系统。LVS 可以实现后端服务器的负载均衡，Keepalived 则可以实现调度器的高可用，防止单点故障。

#### 1.2 实验知识点
=======
### 1.1 实验内容

本节实验中，我们将学习如何使用 LVS(Linux Virtual Server) 搭配高可用集群软件 Keepalived 共同搭建高可用性集群系统。LVS 可以实现后端服务器的负载均衡，Keepalived 则可以实现调度器的高可用，防止单点故障。

### 1.2 实验知识点
>>>>>>> 3fa8bdf764cf14bf952ed26152bb4addb11c4e0c

- LVS/DR + Keepalived 搭建与配置

## 2. LVS/DR + Keepalived 实验说明

<<<<<<< HEAD
#### 2.1 环境介绍
=======
### 2.1 环境介绍
>>>>>>> 3fa8bdf764cf14bf952ed26152bb4addb11c4e0c

LVS/DR + Keepalived 是再原来的 LVS/DR 之上做的扩展，所以是在原来的基础之上增加了一个 LoadBalancer，环境结构如下：

- 宿主机（本实验桌面环境）模拟客户端；
- 启动一台 docker container 作为我们的 Load Balancer 1 (VIP：192.168.0.2)；
- 启动一台 docker container 作为我们的 Load Balancer 2 (VIP：192.168.0.3)；
- 启动一台 docker container 作为我们的 Real Server 1（VIP：192.168.0.4）；
- 启动一台 docker container 作为我们的 Real Server 2（VIP：192.168.0.5）；

宿主机模拟我们的客户端，用浏览器来访问。两个 Load Balancer 作为我们的 VRRP 组。

<<<<<<< HEAD
#### 2.2 实验步骤
=======
### 2.2 实验步骤
>>>>>>> 3fa8bdf764cf14bf952ed26152bb4addb11c4e0c

我们将通过这样的一些步骤来完成此次的实验：

- 本地安装 ipvsadm 工具，加载 IPVS 模块
- 通过 docker 创建四个 container 来模拟集群环境
- 配置两台 RealServer 的环境：
    + 安装 vim 与 nginx 工具
    + 修改默认的 nginx 展示页面
    + 修改内核参数，抑制 arp
    + 创建网卡别名与添加路由
- 配置两台 LoadBalancer 环境：
    + 安装 ipvsadm 与 Keepalived
    + 修改 Keepalived 的配置文件
    + 启动 Keepalived 使配置生效
- 测试实验效果

<<<<<<< HEAD
#### 2.3 测试步骤
=======
### 2.3 测试步骤
>>>>>>> 3fa8bdf764cf14bf952ed26152bb4addb11c4e0c

- LVS 成功测试一：我们能够通过 VIP 访问我们的 Nginx 站点，经过多次的刷新我们能够访问另一个站点的内容（以显示的内容以作区分，因为负载并不高，所以需要很多次刷新，点击地址栏，按住 F5 不放）

- LVS 成功测试二：当我们停止当前访问节点的 nginx 服务时，我们还能通过虚拟 IP 访问我们的站点，说明 LVS 在工作，能够将请求分发给另外一台 Real Server

- Keepalived 成功：我们使用 `arp -a` 查看当前的虚拟 IP 指向的 MAC 地址是 Master 节点的，然后停止 Master 节点的 keepalived 服务，然后还能通过我们的虚拟 IP 访问我们的节点，并且此时通过 `arp -a` 可以看到 虚拟 IP 指向的 Backup 节点的 Mac 地址

## 3. LVS/DR + Keepalived 实验

<<<<<<< HEAD
下面我们将会开始 LVS/DR + Keepalived 实验。

=======
>>>>>>> 3fa8bdf764cf14bf952ed26152bb4addb11c4e0c
### 3.1 配置宿主机

与之前 LVS 实战步骤相同，首先更新源并安装 ipvsadm 工具：

```bash
# 更新源
sudo apt-get update

# 安装 ipvsadm 工具
sudo apt-get install ipvsadm

# 验证安装成功
sudo ipvsadm -l
```

> 与之前相同 ipvsadm 验证步骤不可忽略，否则 LoadBalancer 机器无法使用 ipvs 模块

另外要停止宿主机上的 nginx 服务：

```bash
service nginx stop
```

### 3.2 使用 docker 创建集群环境

同样我们使用 docker 来模拟我们的集群环境，创建四台 container：

- 启动一台 docker container 作为我们的 Load Balancer 1 (VIP：192.168.0.10，作为主路由)；
- 启动一台 docker container 作为我们的 Load Balancer 2 (VIP：192.168.0.10，作为备份路由)；
- 启动一台 docker container 作为我们的 Real Server 1（VIP：192.168.0.10）；
- 启动一台 docker container 作为我们的 Real Server 2（VIP：192.168.0.10）；

实际的 IP 地址请查看，以实际的为主，当前通过该顺序创建并以默认的配置参数，IP 地址会这样分配，通过如下的命令来创建：

```bash
docker run --privileged --name=LoadBalancer1 -tid ubuntu
docker run --privileged --name=LoadBalancer2 -tid ubuntu
docker run --privileged --name=RealServer1 -tid ubuntu
docker run --privileged --name=RealServer2 -tid ubuntu
```

相关参数与命令的作用在 LVS 实战中做了详细的解释，在第 10 周我们会深入的学习 docker，此处不必过于深究，当作轻量级虚拟机即可。

通过 `docker ps` 我们可以验证我们成功的创建：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081516776769477-wm)

### 3.3 配置两台 RealServer 的环境

两台配置的步骤类似，我们提供 RealServer1 的配置步骤

#### 3.3.1 安装 vim 与 nginx 工具

首先我们通过 `docker attach` 命令登录 RealServer1

```bash
# 通过 container 的 name 或者 ID 即可登录
# 登录上之后换行没有反应不是卡住，回车即可看到命令提示符
docker attach RealServer1
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/113508/1516777326330.png-wm)

然后安装 nginx 来提供 web 服务，vim 来提供编辑器：

```bash
apt-get update
apt-get install nginx vim
```

#### 3.3.2 修改默认的 nginx 展示页面

修改默认的 nginx 展示页面，将其中的 `Welcome to Nginx` 修改成 `Welcome to RealServer1`，在 RealServer2 中的操作则修改成 `Welcome to RealServer2`:

```bash
vim /usr/share/nginx/html/index.html
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081516778777234-wm)

完成之后不要忘记启动 nginx：

```bash
service nginx start 
```

最后使用 `ctrl+p` + `ctrl+q` 快捷键来退出容器。

#### 3.3.3 修改内核参数，抑制 arp

修改 arp 的内核参数配置，来防止 LVS 的集群的 arp 表，从而影响负载均衡机器数据包的接受：

```bash
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce
```

回顾一下两个内核参数配置的作用：

- `arp_ignore`：定义了本机响应 ARP 请求的级别。
- `arp_announce`：定义了发送 ARP 请求时，源 IP 应该填什么。

> **arp_ignore 部分参数：**
>
> `0 `表示目标 IP 是本机的，则响应 ARP 请求。默认为 0
>
> `1 `如果接收 ARP 请求的网卡 IP 和目标 IP 相同，则响应 ARP 请求
>
> **arp_announce 参数：**
>
> `0` 表示使用任一网络接口上配置的本地 IP 地址，通常就是待发送的 IP 数据包的源 IP 地址 。默认为 0
>
> `1` 尽量避免使用不属于该网络接口(即发送数据包的网络接口)子网的本地地址作为 ARP 请求的源 IP 地址。大致的意思是如果主机包含多个子网，而 IP 数据包的源 IP 地址属于其中一个子网，虽然该 IP 地址不属于本网口的子网，但是也可以作为ARP 请求数据包的发送方 IP。
>
> `2` 表示忽略 IP 数据包的源 IP 地址，总是选择网络接口所配置的最合适的 IP 地址作为 ARP 请求数据包的源 IP 地址(一般适用于一个网口配置了多个 IP 地址)

#### 3.3.4 创建网卡别名与添加路由

只有在相应 RealServer 中配置了虚拟 IP 地址，该机器才会接受并处理负载均衡机器上发来的数据包（该操作与上一步的修改内核参数都是需要超级权限的，这也就是为什么我们在创建 RealServer 的时候会添加 privileged 参数）：

```bash
# 添加网卡别名
ifconfig lo:0 192.168.0.10 broadcast 192.168.0.10 netmask 255.255.255.255 up

# 添加路由
route add -host 192.168.0.10 dev lo:0
```

由此我们便完成了其中一台 RealServer 的环境配置，我们只需要在另外一台做相同的操作即可，完成两台 RealServer 的配置之后我们通过 Firefox 浏览器来验证我们的 Web 服务是否正常工作。

完成 RealServer 的配置之后，紧接着便是 LoadBalancer 机器的配置

### 3.4 配置两台 LoadBalancer 环境

#### 3.4.1 安装 ipvsadm 与 Keepalived

同样我们首先登录 LoadBalancer1 中更新源与安装相关的工具：

```bash
# 因为镜像中默认是 ubuntu 原生源，所以有时候比较慢
apt-get update

# 安装 ipvsadm 与 keepalived
apt-get install ipvsadm keepalived vim

# 验证 ipvsadm
ipvsadm -l
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081516781346373-wm)

紧接着在 LoadBalancer2 中做相同的操作

#### 3.4.2 修改 Keepalived 的配置文件

因为 Keepalived 就是为 LVS 而诞生的，他会调用 IPVS 模块，所以此时我们并不需要再去通过 ipvsadm 工具来编写规则，我们直接将我们要做的配置写在配置文件中，Keepalived 会根据配置文件自动的为我们配置。

首先我们修改 LoadBalancer1 中的 Keepalived 配置文件：

```bash
vim /etc/keepalived/keepalived.conf
```

因为我们将 LoadBalancer1 作为我们的主路由器，所以其配置文件为：

```bash
#全局配置，在发现某个节点出故障的时候以邮件的形式同时管理员
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

#配置 vrrp 实例
vrrp_instance VI_1 {
    state MASTER  #指定 Keepalived 角色， MASTER 表示此主机是主服务器，BACKUP 表示此主机是备用服务器
    interface eth0    #指定 HA 检测网络的接口
    virtual_router_id 51 #虚拟路由标识，这个标识是一个数字，同一个 vrrp_instance 下，MASTER 和BACKUP 必须是一致的
    priority 101  #定义优先级，数字越大，优先级越高。在同一个 vrrp_instance 下，MASTER 的优先级必须大于 BACKUP 的优先级
    advert_int 1  #设定 MASTER 与 BACKUP 负载均衡器之间同步检查的事件间隔，单位是秒
    authentication { #配置 vrrp 直接的认证 
        auth_type PASS	#设定验证类型和密码，验证类型分为 PASS 和 AH 两种
        auth_pass 1111	#设置验证密码，在一个 vrrp_instance 下，MASTER 与 BACKUP 必须使用相同的密码才能通信
    }   
    virtual_ipaddress { #配置虚拟 IP，可以设置多个，每行一个
        192.168.0.10
    }
}

#配置虚拟服务器
virtual_server 192.168.0.10 80 { #配置虚拟服务器，需要制定虚拟 IP 地址和端口，IP 与端口用空格隔开
    delay_loop 6 #设置运行情况检查时间，单位是秒
    lb_algo rr   #设置负载调度算法，这里设置为 rr，即论叫算法
    lb_kind DR   #设置 LVS 实现负载均衡的机制，有 NAT，TUN，DR 三个模式可选，这里选择 DR
    #persistence_timeout 50 会话保持时间，单位是秒，一般针对动态网页很有用，这里需要这个配置
    protocol TCP #制定协议转发类型，有 TCP 和 UDP 两种。
    
    real_server 192.168.0.4 80 { #配置 real server 的信息，服务节点1
        weight 1 #配置该节点的权重，权值大小用数字表示，设置权值的大小可以分不同性能的服务器分配不同的负载，性能较低的方服务器，设置权值较低，这样能合理的利用和分配系统资源
        HTTP_GET {	#设置健康检查
            url {	#访问这个地址，判断状态码是否 200
              path /
          status_code 200
            }
            connect_timeout 3	#表示3秒无响应超时
            nb_get_retry 3	#表示重试次数
            delay_before_retry 3	#表示重试间隔
        }
    }
    real_server 192.168.0.5 80 { #配置服务节点2
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
```

以上配置内容为 LoadBalancer1 的 Keepalived.conf 配置内容。在编写配置内容时。一定要注意文字和语法格式，因为 Keepalived 在启动时并不会检测配置文件的正确性，及时没有配置文件，Keepalived 也能正常启动，所以一定要保证配置文件内容的正确性。

紧接着我们在 LoadBalancer2 中做相同的配置，主需要将 "Master" 修改成 "BACKUP"，以及优先级的调整即可。

#### 3.4.3 启动 Keepalived 使配置生效

完成两台机器的 Keepalived 的配置，便启动该服务：

```bash
service rsyslog start
service keepalived start 
```

由此我们便完成了所有的配置，启动成功之后我们可以通过 `ipvsadm -l` 查看到 keepalived 自动的为我们配置好了规则：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081516795303754-wm)

### 3.5 测试实验效果

#### 3.5.1 LVS 成功测试一

我们能够通过 VIP 访问我们的 Nginx 站点，经过多次的刷新我们能够访问另一个站点的内容（以显示的内容以作区分，因为负载并不高，所以需要很多次刷新，点击地址栏，按住 F5 不放）

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081516798350280-wm)

#### 3.5.2 LVS 成功测试二

我们来停止 RealServer2 上的服务，测试 LVS 的健康检查是否能正常工作。

```bash
docker attach RealServer2
service nginx stop
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081516798454831-wm)

多刷新几次，可以看到我们始终访问到的是 RealServer1 上的服务，LVS 不会再把请求发给已经停止服务的 RealServer2。健康检查工作正常。

#### 3.5.3 Keepalived 成功测试

我们使用 `arp -a` 查看当前的虚拟 IP（192.168.0.10） 指向的 MAC 地址是 Master 节点（LoadBalancer1）的:

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081516798806475-wm)

同时我们也可以通过 LoadBalancer1 的 `/var/log/syslog` 日志中看到当前的节点为 Master，而在 LoadBalancer2 节点中的日志我们可以看到其为 BACKUP

接下来我们停止 Master 节点的 keepalived 服务。

```bash
docker attach LoadBalancer1
service keepalived stop
```

此时还是可以通过虚拟 IP 访问网站。通过 `arp -a` 可以看到 虚拟 IP 已经指向到 LoadBalancer2 的 Mac 地址：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/113508/1516799030104.png-wm)

同是我们也可以通过 LoadBalancer2 的日志中看到其 vrrp 的角色变化：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/113508/1516799108129.png-wm)

由此我们完成了 LVS + Keepalived 的配置与验证。

`@
http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/week8/4-1.mp4
@`

## 4. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

- LVS/DR + Keepalived 搭建与配置