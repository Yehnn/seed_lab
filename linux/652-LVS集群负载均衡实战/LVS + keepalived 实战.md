# LVS + keepalived 实战

## 一、实验介绍

通过上节实验，我们了解了 keepalived 的作用，其可以解决我们 Load Balancer 的单点故障问题。而仅仅了解其原理，明白理论知识是没有用的，我们将通过本次实验带领大家一起完成 LVS + keepalived 的高可用集群负载的实验。

注意：本节与上一节的实验环境略有差别，若是点下一个实验，继续使用上一个实验环境的同学，请退出，然后创建新环境。

### 1.1 实验涉及的知识点

- LVS + keepalived 实战
- docker 基础操作

## 二、LVS + Keepalived 实战

通过上文的学习我们了解到 keepalived 是如何帮我们解决单点故障的问题，但是该如何与 LVS 结合使用呢？我们通过这样的例子来学习。

我们的模拟环境是这样的一个结构：

- 宿主机（本实验桌面环境）模拟客户端；
- 启动一台 docker container 作为我们的 Load Balancer 1 (实际 IP 地址：172.17.0.2，VIP：172.17.0.10)；
- 启动一台 docker container 作为我们的 Load Balancer 2 (实际 IP 地址：172.17.0.3，VIP：172.17.0.10)；
- 启动一台 docker container 作为我们的 Real Server 1（RIP：172.17.0.4，VIP：172.17.0.10）；
- 启动一台 docker container 作为我们的 Real Server 2（RIP 地址：172.17.0.2，VIP：172.17.0.10）；

通过这样的步骤来验证我们的实验是否成功：

- LVS 成功测试一：我们能够通过 VIP 访问我们的 Nginx 站点，经过多次的刷新我们能够访问另一个站点的内容（以显示的内容以作区分，因为负载并不高，所以需要很多次刷新，点击地址栏，按住 F5 不放）

- LVS 成功测试二：当我们停止当前访问节点的 nginx 服务时，我们还能通过虚拟 IP 访问我们的站点，说明 LVS 在工作，能够将请求分发给另外一台 Real Server

- keepalived 成功：我们使用 `arp -a` 查看当前的虚拟 IP 指向的 MAC 地址是 Master 节点的，然后停止 Master 节点的 keepalived 服务，然后还能通过我们的虚拟 IP 访问我们的节点，并且此时通过 `arp -a` 可以看到 虚拟 IP 指向的 Backup 节点的 Mac 地址

我们整体的网络结构如图所示：

![struct](https://dn-simplecloud.shiyanlou.com/1135081473327585218-wm)

宿主机模拟我们的客户端，用浏览器来访问。两个 Load Balancer 作为我们的 VRRP 组。

开始我们的实战：

首先我们先得在宿主机上安装 ipvsadm 工具，以及使用 ipvsadm 看能否使用（若是没有这一步，在 docker 中将无法使用）：

```
#使用 apt-get 安装 ipvsadm 工具
sudo apt-get install ipvsadm

#使用 ipvsadm 看能否正常工作，并且使用一次，docker 中才能使用，否则会出现 docker 显示读取不到 ipvs 模块的情况
sudo ipvsadm -l
```

![install-ipvsadm](https://dn-simplecloud.shiyanlou.com/2294111473264342377-wm)

做好环境的准备工作之后我们需要创建两个 Load Balancer 的 docker 容器，同时为他们安装上 ipvsadm 与 keepalived：

```
#创建 Load Balancer 1
docker run --privileged --name=load1 -ti ubuntu:latest

#此时我们连接上了 load 1，更新源的信息，然后为其安装两个工具
apt-get update
apt-get install -y ipvsadm keepalived

#当然为了严谨我们可以试验下 ipvsadm 能否工作
ipvsadm -L
```

![run docker load1 ](https://dn-simplecloud.shiyanlou.com/2294111473264885000-wm)

通过以上的操作我们成功的在 load1 中安装好了我们需要的工具，所以我们需要为 load2 安装上同样的工具：

```
#首先通过 （ctrl+p） + （ctrl+q） 的快捷键脱离 load 1，当我们重新看到 shiyanlou 的抬头说明我们退出来了

#创建 Load Balancer 2
docker run --privileged --name=load2 -ti ubuntu:latest

#此时我们连接上了 load 2，更新源的信息，然后为其安装两个工具
apt-get update
apt-get install -y ipvsadm keepalived

#当然为了严谨我们可以试验下 ipvsadm 能否工作
ipvsadm -L
```

![run docker load2](https://dn-simplecloud.shiyanlou.com/2294111473265181909-wm)

![ipvsadm -l](https://doc.shiyanlou.com/document-uid113508labid1987timestamp1473265442563.png/wm)

完成了两个 Load Balancer 的容器启动，我们便开始我们的 Real Server 节点的启动与部署

首先是我们的 Real Server 1 节点，因为需要 nginx 的页面的读取与内容区分鉴别，以及使用 LVS/DR 模式，使用虚拟 IP 作为源地址，但是又不对该 IP 地址请求做相应，所以需要安装 nginx 以及对 arp 的响应与宣告做配置，还有路由策略的配置

1. 安装 nginx 服务：提供 web 服务
2. 修改 nginx 首页内容： 做显示上的区分，才能了解到是那台 real server 在工作
3. 启动 nginx 服务：提供 web 服务
4. 修改 arp 策略配置： 使得在响应包时使用 VIP 作为源地址，但是又不对内网的 VIP 请求做出响应
5. 使得修改后的配置立即生效
6. 配置虚拟 IP：在 lo 网卡上配置别名
7. 添加虚拟 IP 的路由

```
#首先通过 （ctrl+p） + （ctrl+q） 的快捷键脱离 load 2，当我们重新看到 shiyanlou 的抬头说明我们退出来了

#创建 real server 1
docker run --privileged --name=realserver1 -ti ubuntu:latest

#更新源 
apt-get update

#安装 nginx 服务
apt-get install nginx
```

![run realserver1 install nginx](https://doc.shiyanlou.com/document-uid113508labid1987timestamp1473266384870.png/wm)

```
#修改首页内容，- 开头的为修改行，+ 开头为修改后的内容
vim /usr/share/nginx/html/index.html

-<h1>Welcome to Nginx！</h1>
+<h1>11Welcome to shiyanlou! This is Real Server1</h1>
```

![vim nginx](https://doc.shiyanlou.com/document-uid113508labid1987timestamp1473266728559.png/wm)

```
#启动 nginx 服务
service nginx start

#修改 arp 的配置
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce

#使其立即生效
sysctl -p

#配置 VIP 在 lo 网卡的别名上
ifconfig lo:0 172.17.0.10 broadcast 172.17.0.10 netmask 255.255.255.255 up

#添加路由，让响应报文知道该何去何从
route add -host 172.17.0.10 dev lo:0
```

![config arp and route](https://dn-simplecloud.shiyanlou.com/2294111473267565243-wm)

就这样我们就完成了 real server 1 的所有配置，接下来我们只需要将同样的操作在 real server 2 上执行一遍即可，注意 nginx 首页内容记得修改：

```
#首先通过 （ctrl+p） + （ctrl+q） 的快捷键脱离 realserver1，当我们重新看到 shiyanlou 的抬头说明我们退出来了

#创建 real server 2
docker run --privileged --name=realserver2 -ti ubuntu:latest

#更新源 
apt-get update

#安装 nginx 服务
apt-get install nginx

#修改首页内容，- 开头的为修改行，+ 开头为修改后的内容
vim /usr/share/nginx/html/index.html

- <h1>Welcome to Nginx！</h1>
+ <h1>22Welcome to shiyanlou! This is Real Server2</h1>

#启动 nginx 服务
service nginx start

#修改 arp 的配置
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce

#使其立即生效
sysctl -p

#配置 VIP 在 lo 网卡的别名上
ifconfig lo:0 172.17.0.10 broadcast 172.17.0.10 netmask 255.255.255.255 up

#添加路由，让响应报文知道该何去何从
route add -host 172.17.0.10 dev lo:0
```

由此我们便完成了 Real Server 的所有配置。细心的同学会发现我们似乎还没有配置 ipvsadm 的规则，因为 keepalived 与 LVS 深度结合，我们把这个规则都写入 keepalived 的配置文件中。

接下来我们便登入 load 1 中，写其 keepalived 的配置文件，将其配置为我们的 backup，备用 Load Balancer

```
#首先通过 （ctrl+p） + （ctrl+q） 的快捷键脱离 realserver2，当我们重新看到 shiyanlou 的抬头说明我们退出来了

#我们登入 load 1
docker attach load1

#看见开头变成 root@xxxxxx 说明我们成功登入

#编辑我们的配置文件
vim /etc/keepalived/keepalived.conf
```

以下便是我们配置文件的内容，请勿将注释内容写入
```
#全局配置，在发现某个节点出故障的时候以邮件的形式同时管理员
global_defs {
   notification_email {
        richardwei@localhost #通知管理员的邮箱
   }
   notification_email_from root
   smtp_server 127.0.0.1 
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

#配置 vrrp
vrrp_instance VI_1 {
    state BACKUP  #设置为备用节点
    interface eth0    #设置发送消息的网卡
    virtual_router_id 51
    priority 99  #设置备用节点的优先级
    advert_int 1   #master 与 backup 之间同步检查时间间隔，单位秒
    authentication { #配置 vrrp 直接的认证 
        auth_type PASS
        auth_pass 1111
    }   
    virtual_ipaddress { #配置虚拟 IP
        172.17.0.10
    }
}

#配置虚拟 IP
virtual_server 172.17.0.10 80 {
    delay_loop 6 #健康检查时间
    lb_algo rr   #ipvs 的调度算法选择 rr
    lb_kind DR   #使用 VS/DR 模式
    nat_mask 255.255.255.0 
    #persistence_timeout 50
    protocol TCP #转发协议类型
    real_server 172.17.0.4 80 { #配置 real server 的信息
        weight 1 #配置该节点的权重
        HTTP_GET { #健康检查方式
            url {  #检查的路劲
              path /
          status_code 200
            }
            connect_timeout 2 #连接超时时间
            nb_get_retry 3   #重新连接次数
            delay_before_retry 1 #重连时间间隔
        }
    }
    real_server 172.17.0.5 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            connect_timeout 2
            nb_get_retry 3
            delay_before_retry 1
        }
    }
}

```

保存，这便完成了备用节点的配置，接下来我们便登入 load 2 中，写其 keepalived 的配置文件，将其配置为我们的 master，主 Load Balancer

其中配置文件中只有 state 改为 MASTER 与 优先级修改比备用高即可，其他都一样

```
#首先通过 （ctrl+p） + （ctrl+q） 的快捷键脱离 load1，当我们重新看到 shiyanlou 的抬头说明我们退出来了

#我们登入 load 2
docker attach load2

#看见开头变成 root@xxxxxx 说明我们成功登入

#编辑我们的配置文件
vim /etc/keepalived/keepalived.conf
```

以下便是我们配置文件的内容，请勿将注释内容写入
```
#全局配置，在发现某个节点出故障的时候以邮件的形式同时管理员
global_defs {
   notification_email {
        richardwei@localhost #通知管理员的邮箱
   }
   notification_email_from root
   smtp_server 127.0.0.1 
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

#配置 vrrp
vrrp_instance VI_1 {
    state MASTER  #设置为主节点
    interface eth0    #设置发送消息的网卡
    virtual_router_id 51
    priority 101  #设置主节点的优先级
    advert_int 1   
    authentication { #配置 vrrp 直接的认证 
        auth_type PASS
        auth_pass 1111
    }   
    virtual_ipaddress { #配置虚拟 IP
        172.17.0.10
    }
}

#配置虚拟 IP
virtual_server 172.17.0.10 80 {
    delay_loop 6 
    lb_algo rr   #ipvs 的调度算法选择 rr
    lb_kind DR   #使用 VS/DR 模式
    nat_mask 255.255.255.0 
    #persistence_timeout 50
    protocol TCP #使用的协议
    real_server 172.17.0.4 80 { #配置 real server 的信息
        weight 1 #配置该节点的权重
        HTTP_GET {
            url {
              path /
          status_code 200
            }
            connect_timeout 2
            nb_get_retry 3
            delay_before_retry 1
        }
    }
    real_server 172.17.0.5 80 {
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            connect_timeout 2
            nb_get_retry 3
            delay_before_retry 1
        }
    }
}

```

保存之后这样我们便完成了 LVS + keepalived + docker + nginx 的高可用负载均衡集群配置了。

完成配置之后我们可以通过这样的命令来启动 keepalived：

```
service keepalived start
```

接下来我们便来验证，我们是否配置成功了：

LVS 是可以成功运行的：

![LVS is success](https://doc.shiyanlou.com/document-uid113508labid1987timestamp1473268911560.png/wm)

查看 arp 可以看到当前是 master Load Balancer 在工作：

![MAC arp info](https://doc.shiyanlou.com/document-uid113508labid1987timestamp1473269008944.png/wm)

当我们关掉 master 节点的 keepalived，也就是停止了其 LVS 的工作，模拟其故障情况，此时我们还可访问站点，并且通过 arp 我们可以了解到是我们的 keepalived 结构解决了单点故障的问题：

![keepalived is work](https://doc.shiyanlou.com/document-uid113508labid1987timestamp1473269193646.png/wm)

就此我们完成了 LVS + keepalived + docker + nginx 的高可用集群负载均衡实验，通过本实验将以前学习到的知识做一个综合的应用。

## 实验总结

通过本实验的学习，我们将以前学习到的看似不相关的知识点串联前来，完成我们的高可用集群负载均衡，让我们的服务更加的可靠，即使是某个站点出现故障也不会被用户察觉到。