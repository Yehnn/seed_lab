---
show: step
version: 0.1
enable_checker: true
---

# LVS 实战

## 一、实验介绍

**注意：本节实验环境特殊，请不要沿用上节实验的环境，需使用本节实验默认环境。** 

通过以上两节实验的学习，我们了解了 LVS 是什么，可以做什么，还有种种好处，以及 IP 负载均衡的技术，本节实验将结合 Docker 真正实现 LVS 集群负载均衡。

注意：注意：本节与上一节的实验环境略有差别，若是点下一个实验，继续使用上一个实验环境的同学，请退出，然后创建新环境。

#### 实验涉及的知识点

- Docker 的简单使用
- LVS/DR 的实现

## 二、 LVS/NAT 搭建

下面我们来实际搭建 LVS/NAT。

### 2.1 集群环境搭建

在本实验环境中我们没有办法为大家提供多台服务器来模拟集群环境，由此我们 docker 工具来创建多个 container 来模拟集群所需要的多台服务器。

> docker 可以简单的理解为非常轻量级的虚拟机工具，而 container 则理解为创建的虚拟机。

集群系统中，服务器资源可以简单分为两种角色：

- 一种是 Load Balancer，即负载均衡调度器，位于集群系统的前端，对后端服务器实现负载均衡作用，对外 IP 地址也称为 VIP（虚拟 IP 地址）。
- 另一种就是后端服务器群，处理由 Load Balancer 发来的请求。

整个集群系统结构：

- 宿主机环境（默认桌面环境）：装有 `ipvsadm`（LVS 的 IP 负载由 IPVS 内核模块完成，ipvsadm 是为 IPVS 编制规则的工具），充当负载均衡调度器
- 宿主机浏览器：通过宿主机中的浏览器来充当客户端
- RealServer1 的 container：部署 Nginx web 服务器，提供 Web 访问服务，充当服务器池中的一员
- RealServer2 的 container：部署 Nginx web 服务器，提供 Web 访问服务，充当服务器池中的一员

### 2.2 实验步骤

我们将通过这样的一些步骤来完成此次的实验：

- 本地安装 ipvsadm 工具，加载 IPVS 模块
- 通过 docker 创建两个 container 来模拟服务器池中的成员
- 配置两台 RealServer 的环境：
    + 安装 vim 与 nginx 工具
    + 修改默认的 nginx 展示页面
- 配置负载均衡调度机器：
  + 修改内核转发参数
    + 配置 ipvsadm 规则
- 测试实验效果

### 2.3 测试步骤

LVS 成功测试：我们能够通过 VIP 访问我们的 Nginx 站点，经过多次的刷新我们能够访问另一个站点的内容（以显示的内容以作区分，因为负载并不高，所以需要很多次刷新，点击地址栏，按住 F5 不放）

### 2.4 LVS/NAT 实现

#### 2.4.1 安装 ipvsadm 工具 

首先为了能够使用 IPVS 内核模块，我们将在宿主机中安装 ipvsadm，并尝试能否使用：

```bash
# 更新源 
sudo apt-get update

# 安装 ipvsadm 工具
sudo apt-get install ipvsadm

# 尝试使用 ipvsadm
sudo ipvsadm -L
```

![3-2.4.1](https://dn-simplecloud.shiyanlou.com/uid/108299/1516011723528.png-wm)

#### 2.4.2 创建服务器池成员

使用 docker 创建所需 container，使用以下命令创建：

```bash
docker run --privileged --name=RealServer1 -tdi ubuntu
docker run --privileged --name=RealServer2 -tdi ubuntu
```

命令讲解：

>  docker run：创建 docker 容器
>
>  name 参数：给容器命名，方便区分
>
>  tid 参数：分配 tty，能够与之交互
>
>  ubuntu：指定容器镜像，这里使用 ubuntu 镜像

![3-2.4.2](https://doc.shiyanlou.com/document-uid113508labid2124timestamp1516732770433.png/wm)

创建命令执行完成后，目前有两个 container：

* RealServer1：IP 地址为 `192.168.0.2`。下文简称 `RIP1`
* RealServer2：IP 地址为 `192.168.0.3`。下文简称 `RIP2`

> 可以通过 `ifconfig` 命令查看各自的 IP 地址，此处的地址是因为按顺序，且按默认配置创建所导致

#### 2.4.3 配置服务器池成员

RealServer 部署 Nginx 来提供 Web 服务，RealServer1 和 RealServer2 操作步骤相同，此处以 RealServer1 为示例：

首先登录 RealServer1:

```sh
# 通过 attach 命令登录 RealServer1
docker attach RealServer1
```

安装相关工具

```bash
apt-get update
apt-get install vim -y 
apt-get install nginx -y
service nginx start
```

为了区分 RealServer1 和 RealServer2 的 Nginx 响应页面，需要修改默认 nginx 的展示 html 页面。

```sh
#进入 RealServer1 container 环境
vi /usr/share/nginx/html/index.html
```

![3-2.4.3](https://dn-simplecloud.shiyanlou.com/uid/108299/1516013480902.png-wm)

> 按 `i` 键插入，按 `esc` 再输入 `:wq` 保存退出。
>
> 注意若完成了 RealServer1 的配置之后，如果我们不想打开新的终端，可以通过 `ctrl+p+q` 的组合快捷键脱离当前机器的登录，切勿使用 `exit` 的方式退出 container，这样的方式关闭服务器的。
> 脱离之后便会返回到 shiyanlou 的 zsh 交互，可以通过 `docker attach RealServer2` 的命令来登录另一台机器，然后做类似的操作（同上的软件安装操作以及 nginx 启动操作）

接下来就修改 nginx 页面，如下所示：

```sh
#在 RealServer2 container 环境
vi /usr/share/nginx/html/index.html
```

![3-2.4.3-2](https://dn-simplecloud.shiyanlou.com/uid/108299/1516013537528.png-wm)

完成两台服务器的配置之后，我们通过 `service nginx start` 启动服务器中 nginx 服务。

至此我们完成两台 Web 服务器的配置，我们可以打开宿主机 firefox 浏览器，地址栏分别输入两个 IP 地址，来检验我们的配置成功：

![3-2.4.3-3](https://doc.shiyanlou.com/document-uid8797labid2124timestamp1527582859419.png/wm)

![3-2.4.3-4](https://doc.shiyanlou.com/document-uid8797labid2124timestamp1527582859854.png/wm)

#### 2.4.4 配置调度器规则

1.为避免影响实验结果，关闭宿主机环境的 nginx 服务：

```bash
sudo service nginx stop
```

LoadBalancer 的对外 IP 地址为 VIP，即 VIP 地址为 `120.26.15.9` （注意，你的 VIP 地址可能和我的不一样，根据自己实际情况来）。对内 IP 称为 RIP，此时 RIP 为 `192.168.0.1`。

2.开启 LoadBalancer 的内核路由转发：

```sh
echo '1' | sudo tee /proc/sys/net/ipv4/ip_forward
```

查看当前机器内核路由转发开启情况：

```bash
cat /proc/sys/net/ipv4/ip_forward
```

得到的值为 1，说明此机器已开启内核路由转发。进行下一步。

4.使用 ipvsadm 添加 ipvs 规则。定义集群服务：

```
sudo ipvsadm -A -t 120.26.15.9:80 -s rr         #定义集群服务
sudo ipvsadm -a -t 120.26.15.9:80 -r 192.168.0.2 -m #添加 RealServer1
sudo ipvsadm -a -t 120.26.15.9:80 -r 192.168.0.3 -m #添加 RealServer2
sudo ipvsadm -l                 #查看 ipvs 定义的规则
```

上面命中 ipvsadm 参数讲解：

```
# 添加集群服务
-A：添加一个新的集群服务
-t: 使用 TCP 协议
-s: 指定负载均衡调度算法
rr：轮询算法(LVS 实现了 8 种调度算法)
120.26.15.9:80 定义集群服务的 IP 地址（VIP） 和端口

# 添加 Real Server 规则
-a：添加一个新的 RealServer 规则
-t：tcp 协议
-r：指定 RealServer IP 地址
-m：定义为 NAT 
上面命令添加了两个服务器 RealServer1 和 RealServer2
```

![3-2.4.4](https://dn-simplecloud.shiyanlou.com/uid/108299/1516026881486.png-wm)

#### 2.4.5 测试配置

打开浏览器，输入 VIP 地址：`120.26.15.9`：

![3-2.4.5](https://dn-simplecloud.shiyanlou.com/uid/108299/1516026980496.png-wm)

可以发现，我们访问 VIP 地址，得到的是 RealServer1 的 Nginx 响应页面。多次刷新页面，页面也会发生变化：

![3-2.4.5-2](https://dn-simplecloud.shiyanlou.com/uid/108299/1516027089606.png-wm)

> 因为访问压力比较小，调度算法不会请求切换服务器，可以按住 F5 快速多次刷新查看页面变化效果

以上便实现了 LVS 的 NAT 负载均衡系统。

## 三、LVS/DR 搭建

下面我们来实际搭建 LVS/DR 。

### 3.1 集群环境搭建

与 NAT 方式相同，我们将通过 docker 来模拟我们的集群环境。

集群系统中，服务器资源可以简单分为两种角色：

- 一种是 Load Balancer，即负载均衡调度器，位于集群系统的前端，对后端服务器实现负载均衡作用，对外 IP 地址也称为 VIP（虚拟 IP 地址）。
- 另一种就是后端服务器群，处理由 Load Balancer 发来的请求。

整个集群系统结构：

- 宿主机环境（默认桌面环境）：充当客户端访问 web 服务
- LoadBalancer1 的 container：装有 ipvsadm，充当负载均衡调度器
- RealServer1 的 container：部署 Nginx web 服务器，提供 Web 访问服务，充当服务器池中的一员
- RealServer2 的 container：部署 Nginx web 服务器，提供 Web 访问服务，充当服务器池中的一员

### 3.2 实验步骤

我们将通过这样的一些步骤来完成此次的实验：

- 本地安装 ipvsadm 工具，加载 IPVS 模块
- 通过 docker 创建三个 container 来模拟服务器池中的成员
- 配置两台 RealServer 的环境：
    + 安装 vim 与 nginx 工具
    + 修改默认的 nginx 展示页面
    + 修改内核参数，抑制 arp
    + 创建网卡别名与添加路由
- 配置一台 LoadBalancer 环境：
  + 安装 ipvsadm
  + 配置网卡别名
    + 配置 ipvsadm 规则
- 测试实验效果

### 3.3 测试步骤

- LVS 成功测试：我们能够通过 VIP 访问我们的 Nginx 站点，经过多次的刷新我们能够访问另一个站点的内容（以显示的内容以作区分，因为负载并不高，所以需要很多次刷新，点击地址栏，按住 F5 不放）

- 查看 ipvsadm 中的统计数据。

### 3.4 LVS/DR 实现

若是我们沿用 NAT 的实验环境，我们需要做环境的清理：

1.首先清除 ipvsadm 的规则：

```bash
sudo ipvsadm -C
```

2.删除之前所创建的 container，虽然都是提供 Web 服务，但是在 DR 模式中需要修改内核参数与创建网卡别名，需要超级权限，所以不能沿用之前的 container：

```bash
# 关闭所有的 container
docker stop `docker ps -aq`
# 删除所有的 container
docker rm `docker ps -aq`
```

#### 3.4.1 安装 ipvsadm 工具 

因为在 NAT 实验中我们已安装所以可跳过该步骤，若是新启动的环境请参考 NAT 中的步骤，此处提示务必在宿主机环境中执行 `ipvsadm -L` 的验证步骤，若是不执行该步骤，在 LoadBalancer 的 container 中我们将无法加载 IPVS 的内核模块。

#### 3.4.2 创建与配置服务器池成员

同样我们使用 docker 来模拟我们的集群环境，创建三台 container：

```bash
docker run --privileged --name=LoadBalancer -tid ubuntu
docker run --privileged --name=RealServer1 -tid ubuntu
docker run --privileged --name=RealServer2 -tid ubuntu
```

其中 `--privileged` 参数用于给予容器超级权限。

完成我们服务器池成员的创建之后，我们参照 NAT 中配置步骤完成 RealServer 中的：

- **nginx 与 vim 的安装**
- **默认展示页面的修改**
- **nginx 服务的启动**

在完成这样的配置之后我们需要一些额外的操作：

1.修改内核参数

以 `RealServer1` 为例，登录 container：

执行下列命令：

```sh
# 设置只回答目标IP地址是来访网络接口本地地址的ARP查询请求 
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore

# 为了保险自己可以查看一下是否成功修改
cat /proc/sys/net/ipv4/conf/lo/arp_ignore

# 设置对查询目标使用最适当的本地地址.在此模式下将忽略这个IP数据包的源地址并尝试选择与能与该地址通信的本地地址.首要是选择所有的网络接口的子网中外出访问子网中包含该目标IP地址的本地地址. 如果没有合适的地址被发现,将选择当前的发送网络接口或其他的有可能接受到该ARP回应的网络接口来进行发送.
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce

# 使得上面的配置立即生效
sysctl -p
```

> ARP 的内核参数：
> **arp_ignore 部分参数：**定义了本机响应 ARP 请求的级别
>
> `0 `表示目标 IP 是本机的，则响应 ARP 请求。默认为 0
>
> `1 `如果接收 ARP 请求的网卡 IP 和目标 IP 相同，则响应 ARP 请求
>
> **arp_announce 参数：**定义了发送 ARP 请求时，源 IP 应该填什么。
>
> `0` 表示使用任一网络接口上配置的本地 IP 地址，通常就是待发送的 IP 数据包的源 IP 地址 。默认为 0
>
> `1` 尽量避免使用不属于该网络接口(即发送数据包的网络接口)子网的本地地址作为 ARP 请求的源 IP 地址。大致的意思是如果主机包含多个子网，而 IP 数据包的源 IP 地址属于其中一个子网，虽然该 IP 地址不属于本网口的子网，但是也可以作为ARP 请求数据包的发送方 IP。
>
> `2` 表示忽略 IP 数据包的源 IP 地址，总是选择网络接口所配置的最合适的 IP 地址作为 ARP 请求数据包的源 IP 地址(一般适用于一个网口配置了多个 IP 地址)

2.配置网卡别名

只有目的 IP 是本机器中的一员时才会做相应的处理，所以需要添加网卡别名：

```bash
# 配置虚拟IP
ifconfig lo:0 192.168.0.10 broadcast 192.168.0.10 netmask 255.255.255.255 up

# 添加路由，因为本就是相同的网段所以可以不添加该路由
route add -host 192.168.0.10 dev lo:0

service networking restart
```

![3-3.4.2](https://dn-simplecloud.shiyanlou.com/uid/108299/1516029742820.png-wm)

同两台 Web 服务器中都做该配置，即完成了所有 Web 服务器所需的配置工作。

#### 3.4.3 配置调度器规则

紧接着我们登录 LoadBalancer 机器：

```bash
docker attach LoadBalancer
```

安装 ipvsadm 软件：

```sh
apt-get update
apt-get install -y ipvsadm
ipvsadm -L #查看 ipvsadm 能否正常使用
```

![3-3.4.3](https://dn-simplecloud.shiyanlou.com/uid/108299/1516028854500.png-wm)

![3-3.4.3-2](https://dn-simplecloud.shiyanlou.com/uid/108299/1516028894171.png-wm)

![3-3.4.3-3](https://dn-simplecloud.shiyanlou.com/uid/108299/1516029004118.png-wm)

创建 eth0 的别名并绑定 VIP 地址，作为集群同时使用：

```sh
ifconfig eth0:0 192.168.0.10 netmask 255.255.255.0 up
```

查看网卡信息：`ifconfig`

![3-3.4.3-4](https://dn-simplecloud.shiyanlou.com/uid/108299/1516029261794.png-wm)

在 LoadBalancer 中添加 IPVS 规则：

```sh
ipvsadm -A -t 192.168.0.10:80 -s rr         # 定义集群服务
ipvsadm -a -t 192.168.0.10:80 -r 192.168.0.3 -g # 添加 RealServer1
ipvsadm -a -t 192.168.0.10:80 -r 192.168.0.4 -g # 添加 RealServer2
ipvsadm -l                  # 查看 ipvs 定义的规则
```

ipvsadm 命令参数讲解：

```bash
# 添加集群服务
-A：添加一个新的集群服务
-t: 使用 TCP 协议
-s: 指定负载均衡调度算法
rr：轮询算法(LVS 实现了 8 种调度算法)
192.168.0.10:80 定义集群服务的 IP 地址（VIP） 和端口

# 添加 Real Server 规则
-a：添加一个新的 RealServer 规则
-t：tcp 协议
-r：指定 RealServer IP 地址
-g：定义为 DR 模式
上面命令添加了两个集群服务器 RealServer1 和 RealServer2
```

![3-3.4.3-5](https://dn-simplecloud.shiyanlou.com/uid/108299/1516031321919.png-wm)

使用两组组合快捷键 （ctrl+p）+（ctrl+q），脱离当前容器环境。

由此我们便完成了所有的配置工作。

#### 3.4.4 测试配置

打开宿主环境的 firefox 浏览器，地址栏输入 VIP 地址：`192.168.0.10`：

![3-3.4.4](https://dn-simplecloud.shiyanlou.com/uid/108299/1516031462752.png-wm)

此时由 **RealServer1** 的 Nginx 响应页面，多次刷新页面，页面发生变化：

![3-3.4.4-2](https://dn-simplecloud.shiyanlou.com/uid/108299/1516031530603.png-wm)

页面结果由 **RealServer2** 的 Nginx web 服务器响应。

> 因为访问压力比较小，调度算法不会请求切换服务器，可以按住 F5 快速多次刷新查看页面变化效果

以上操作就实现了 LVS/DR 模式，实现集群系统的负载均衡。


## 四、实验总结

通过本实验的学习，我们使用 docker+nginx+LVS/DR 简单地实现了 IP 负载均衡的模拟。在实际生活中可能有更复杂的环境，但是万变不离其宗。
