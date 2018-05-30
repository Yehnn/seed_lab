---
show: step
version: 0.1
enable_checker: true

---

# LVS + keepalived 实战

## 一、实验介绍

通过上节实验，我们了解了 keepalived 的作用，其可以解决我们 Load Balancer 的单点故障问题。而仅仅了解其原理，明白理论知识是没有用的，我们将通过本次实验带领大家一起完成 LVS + keepalived 的高可用集群负载的实验。

注意：本节与上一节的实验环境略有差别，若是点下一个实验，继续使用上一个实验环境的同学，请退出，然后创建新环境。

#### 实验涉及的知识点

- LVS + keepalived 实战
- docker 基础操作

## 二、LVS + Keepalived 实战

通过上文的学习我们了解到 keepalived 是如何帮我们解决单点故障的问题，但是该如何与 LVS 结合使用呢？我们通过这样的例子来学习。

我们的模拟环境是这样的一个结构：

- 宿主机（本实验桌面环境）模拟客户端；
- 启动一台 docker container 作为我们的 Load Balancer 1 (VIP：192.168.0.10，作为主路由)；
- 启动一台 docker container 作为我们的 Load Balancer 2 (VIP：192.168.0.10，作为备份路由)；
- 启动一台 docker container 作为我们的 Real Server 1（VIP：192.168.0.10）；
- 启动一台 docker container 作为我们的 Real Server 2（VIP：192.168.0.10）；

通过这样的步骤来验证我们的实验是否成功：

- LVS 成功测试一：我们能够通过 VIP 访问我们的 Nginx 站点，经过多次的刷新我们能够访问另一个站点的内容（以显示的内容以作区分，因为负载并不高，所以需要很多次刷新，点击地址栏，按住 F5 不放）

- LVS 成功测试二：当我们停止当前访问节点的 nginx 服务时，我们还能通过虚拟 IP 访问我们的站点，说明 LVS 在工作，能够将请求分发给另外一台 Real Server

- keepalived 成功：我们使用 `arp -a` 查看当前的虚拟 IP 指向的 MAC 地址是 Master 节点的，然后停止 Master 节点的 keepalived 服务，然后还能通过我们的虚拟 IP 访问我们的节点，并且此时通过 `arp -a` 可以看到 虚拟 IP 指向的 Backup 节点的 Mac 地址

我们整体的网络结构如图所示：

![5-2](https://dn-simplecloud.shiyanlou.com/1135081473327585218-wm)

宿主机模拟我们的客户端，用浏览器来访问。两个 Load Balancer 作为我们的 VRRP 组。

### 2.1 安装 ipvsadm 工具

开始我们的实战：

首先我们先得在宿主机上安装 ipvsadm 工具，以及使用 ipvsadm 看能否使用（若是没有这一步，在 docker 中将无法使用）：

```
#使用 apt-get 安装 ipvsadm 工具
sudo apt-get install ipvsadm

#使用 ipvsadm 看能否正常工作，并且使用一次，docker 中才能使用，否则会出现 docker 显示读取不到 ipvs 模块的情况
sudo ipvsadm -l
```

![5-2.1](https://dn-simplecloud.shiyanlou.com/2294111473264342377-wm)

### 2.2 使用 docker 创建集群环境

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

相关参数与命令的作用在 LVS 实战中做了详细的解释。

通过 `docker ps` 我们可以验证我们成功的创建：

![5-2.2](https://dn-simplecloud.shiyanlou.com/1135081516776769477-wm)

### 2.3 配置两台 RealServer 的环境

两台配置的步骤类似，我们提供 RealServer1 的配置步骤

#### 2.3.1 安装 vim 与 nginx 工具

首先我们通过 `docker attach` 命令登录 RealServer1

```bash
# 通过 container 的 name 或者 ID 即可登录
# 登录上之后换行没有反应不是卡住，回车即可看到命令提示符
docker attach RealServer1
```

![5-2.3.1](https://dn-simplecloud.shiyanlou.com/uid/113508/1516777326330.png-wm)

然后安装 nginx 来提供 web 服务，vim 来提供编辑器：

```bash
apt-get update
apt-get install nginx vim
```

#### 2.3.2 修改默认的 nginx 展示页面

修改默认的 nginx 展示页面，将其中的 `Welcome to Nginx` 修改成 `Welcome to RealServer1`，在 RealServer2 中的操作则修改成 `Welcome to RealServer2`:

```bash
vim /usr/share/nginx/html/index.html
```

![5-2.3.2](https://dn-simplecloud.shiyanlou.com/1135081516778777234-wm)

完成之后不要忘记启动 nginx：

```bash
service nginx start 
```

#### 2.3.3 修改内核参数，抑制 arp

修改 arp 的内核参数配置，来防止 LVS 的集群的 arp 表，从而影响负载均衡机器数据包的接收：

```bash
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce
```

回顾以下两个内核参数配置的作用：

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

#### 2.3.4 创建网卡别名与添加路由

只有在相应 RealServer 中配置了虚拟 IP 地址，该机器才会接收并处理负载均衡机器上发来的数据包（该操作与上一步的修改内核参数都是需要超级权限的，这也就是为什么我们在创建 RealServer 的时候会添加 privileged 参数）：

```bash
# 添加网卡别名
ifconfig lo:0 192.168.0.10 broadcast 192.168.0.10 netmask 255.255.255.255 up

# 添加路由
route add -host 192.168.0.10 dev lo:0
```

由此我们便完成了其中一台 RealServer 的环境配置，我们只需要在另外一台做相同的操作即可，完成两台 RealServer 的配置之后我们通过 Firefox 浏览器来验证我们的 Web 服务是否正常工作。

完成 RealServer 的配置之后，紧接着便是 LoadBalancer 机器的配置。

### 2.4 配置两台 LoadBalancer 环境

#### 2.4.1 安装 ipvsadm 与 Keepalived

同样我们首先登录 LoadBalancer1 中更新源与安装相关的工具：

```bash
# 因为镜像中默认是 ubuntu 原生源，所以有时候比较慢
apt-get update

# 安装 ipvsadm 与 keepalived
apt-get install ipvsadm keepalived vim

# 验证 ipvsadm
ipvsadm -l
```

![5-2.4.1](https://dn-simplecloud.shiyanlou.com/1135081516781346373-wm)

紧接着在 LoadBalancer2 中做相同的操作 .

#### 2.4.2 修改 Keepalived 的配置文件

因为 Keepalived 就是为 LVS 而诞生的，它会调用 IPVS 模块，所以此时我们并不需要再去通过 ipvsadm 工具来编写规则，我们直接将我们要做的配置写在配置文件中，Keepalived 会根据配置文件自动的为我们配置。

首先我们修改 LoadBalancer1 中的 Keepalived 配置文件：

```bash
vim /etc/keepalived/keepalived.conf
```

因为我们将 LoadBalancer1 作为我们的主路由器，所以其配置文件为：

```bash
#全局配置，在发现某个节点出故障的时候以邮件的形式通知管理员
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
    advert_int 1  #设定 MASTER 与 BACKUP 负载均衡器之间同步检查的时间间隔，单位是秒
    authentication { #配置 vrrp 直接的认证 
        auth_type PASS	#设定验证类型和密码，验证类型分为 PASS 和 AH 两种
        auth_pass 1111	#设置验证密码，在一个 vrrp_instance 下，MASTER 与 BACKUP 必须使用相同的密码才能通信
    }   
    virtual_ipaddress { #配置虚拟 IP，可以设置多个，每行一个
        192.168.0.10
    }
}

#配置虚拟服务器
virtual_server 192.168.0.10 80 { #配置虚拟服务器，需要指定虚拟 IP 地址和端口，IP 与端口用空格隔开
    delay_loop 6 #设置运行情况检查时间，单位是秒
    lb_algo rr   #设置负载调度算法，这里设置为 rr，即论叫算法
    lb_kind DR   #设置 LVS 实现负载均衡的机制，有 NAT，TUN，DR 三个模式可选，这里选择 DR
    #persistence_timeout 50 会话保持时间，单位是秒，一般针对动态网页很有用，这里需要这个配置
    protocol TCP #指定协议转发类型，有 TCP 和 UDP 两种。
    
    real_server 192.168.0.4 80 { #配置 real server 的信息，服务节点1
        weight 1 #配置该节点的权重，权值大小用数字表示，设置权值的大小可以分不同性能的服务器分配不同的负载，性能较低的方服务器，设置权值较低，这样能合理地利用和分配系统资源
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

#### 2.4.3 启动 Keepalived 使配置生效

完成两台机器的 Keepalived 的配置，便启动该服务：

```bash
service rsyslog start
service keepalived start 
```

由此我们便完成了所有的配置，启动成功之后我们可以通过 `ipvsadm -l` 查看到 keepalived 自动的为我们配置好了规则：

![5-2.4.3](https://dn-simplecloud.shiyanlou.com/1135081516795303754-wm)

### 2.5 测试实验效果

#### 2.5.1 LVS 成功测试一

我们能够通过 VIP 访问我们的 Nginx 站点，经过多次的刷新我们能够访问另一个站点的内容（以显示的内容以作区分，因为负载并不高，所以需要很多次刷新，点击地址栏，按住 F5 不放）

![5-2.5.1](https://dn-simplecloud.shiyanlou.com/1135081516798350280-wm)

#### 2.5.2 LVS 成功测试二

当我们停止当前访问节点的 nginx 服务时，我们还能通过虚拟 IP 访问我们的站点，说明 LVS 在工作，能够将请求分发给另外一台 Real Server

![5-2.5.2](https://dn-simplecloud.shiyanlou.com/1135081516798454831-wm)

#### 2.5.3 Keepalived 成功测试

我们使用 `arp -a` 查看当前的虚拟 IP 指向的 MAC 地址是 Master 节点的:

![5-2.5.3](https://dn-simplecloud.shiyanlou.com/1135081516798806475-wm)

也就是我们 LoadBalancer1 节点，同时我们也可以通过 LoadBalancer1 的 `/var/log/syslog` 日志中看到当前的节点为 Master，而在 LoadBalancer2 节点中的日志我们可以看到其为 BACKUP

然后停止 Master 节点的 keepalived 服务，然后还能通过我们的虚拟 IP 访问我们的节点，并且此时通过 `arp -a` 可以看到 虚拟 IP 指向的 Backup 节点的 Mac 地址：

![5-2.5.3-2](https://dn-simplecloud.shiyanlou.com/uid/113508/1516799030104.png-wm)

同是我们也可以通过 LoadBalancer2 的日志中看到其 vrrp 的角色变化：

![5-2.5.3-3](https://dn-simplecloud.shiyanlou.com/uid/113508/1516799108129.png-wm)

由此我们完成了 LVS + Keepalived 的配置与验证 。

## 三、实验总结

通过本实验的学习，我们将以前学习到的看似不相关的知识点串联前来，完成我们的高可用集群负载均衡，让我们的服务更加的可靠，即使是某个站点出现故障也不会被用户察觉到。
