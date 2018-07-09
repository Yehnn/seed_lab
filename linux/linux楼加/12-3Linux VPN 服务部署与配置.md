---
show: step
version: 1.0
enable_checker: true
---
# Linux VPN 服务部署与配置

## 1 实验介绍

#### 1.1 实验内容

本实验将介绍一些关于 VPN 服务器的软件，以及如何安装配置 pptp 服务器和 openVPN 服务器，着重于动手部分，软件的安装与配置，理论部分会涉及较为深入的网络知识，为大家提供了相关链接，欢迎大家阅读。

#### 1.2 实验知识点

+ VPN 简介
+ 常见 VPN 软件
+ 搭建 pptp 服务
+ 搭建 openVPN 服务

#### 1.3 推荐阅读

- [PPTP 通俗讲解](https://www.zhihu.com/question/20174552)
- [PPTP 运行流程抓包解析](http://blog.csdn.net/hdxlzh/article/details/46711901)
- [VPN 运行过程](https://technet.microsoft.com/en-us/library/cc779919(v=ws.10).aspx)
- [CA 证书浅析](http://www.cnblogs.com/hyddd/archive/2009/01/07/1371292.html)
- [数字证书是什么](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)
- [Ubuntu 安装 OpenVPN](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-14-04)

## 2 VPN 简介

#### 2.1 概述

VPN（Virtual Private Network）中文翻译为虚拟专有网络。现在 VPN 被普遍定义为在公用互联网络上建立一个临时的、安全的网络，使用它可以对数据进行深度加密达到安全使用互联网的目的，因此被广泛使用企业办公当中远程访问内网。

#### 2.2 VPN 实现方式

虚拟专用网络能够利用 `Internet` 或其它公共互联网络的基础设施，将 IP 报文加密封装在另一个 IP 报文中的方式为用户创建一个虚拟通信的专用通道，通常称为隧道，让两个没有实际物理连接的节点却能在逻辑上直接连接通信，以达到安全的通信方式。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1513912490296.png/wm)
(此图来自于[Microsoft vpn 讲解](https://technet.microsoft.com/en-us/library/cc779919(v=ws.10).aspx))

VPN 的实现原理大家可以阅读推荐阅读中的 `VPN 运行过程`，其中会涉及到很多的网络知识，有兴趣的同学可以仔细阅读。

## 3 常见 VPN 软件

+ **OpenVPN**

OpenVPN 是一款基于 `SSL` 的开源 VPN 软件，它利用 `OpenSSL` 加密库中的 `SSLv3/TLSv1` 协议函数库来保证网络通讯的安全性，并且允许参与建立 VPN 的单点使用共享密钥，电子证书，或者用户名/密码来进行身份验证。同时避免了传统 SSL VPN 仅提供简单的 Web 应用的不足，它具有支持各种应用协议，以及支持 Windows，Linux，BSD，MAC OS 等多平台的特点。

+ **PPTP**

点对点隧道协议（英语：Point to Point Tunneling Protocol），缩写为PPTP是由包括微软和 3Com 等多家公司组成的 PPTP 论坛开发的一种点对点隧道协议，基于拨号使用的 `PPP` 协议使用 `PAP` 或 `CHAP` 等加密算法，或者使用 Microsoft 的点对点加密算法 `MPPE` 。使其能通过传输控制协议（TCP）创建控制通道来发送控制命令，以及利用通用路由封装（GRE）通道来封装点对点协议（PPP）数据包以发送数据，实现了按需的、多协议的虚拟专用网络，让远程客户端到专用企业服务器之间数据的安全传输。

+ **L2TP**

第二层隧道协议 (`L2TP`) 是 IETF 基于 `L2F` （ Cisco 的第二层转发协议）开发的 PPTP 的后续版本。是一种工业标准的 Internet 隧道协议，可以为跨越面向数据包的媒体发送点到点协议 (`PPP`) 框架提供封装。PPTP 和 L2TP 都使用 PPP 协议对数据进行封装，然后添加附加包头用于数据在互联网络上的传输。

## 4 搭建 PPTP 服务

我们将通过 PPTP 的搭建对 VPN 做进一步的了解。

### 4.1 检查系统是否支持

在搭建 pptp 的服务之前我们可以先检查该系统是否支持 pptp 的服务：

1.检查系统中是否有 MPPE 的驱动模块(在内核版本 2.6.14 时就已经将该驱动加入了内核驱动中)：

```bash
sudo modprobe ppp-compress-18 && echo "is OK"
```

`modprobe` 是一个用来添加、删除系统模块的工具，通过这个命令我们可以知道系统中是否有加载该模块，若是存在便会继续执行后续的 `echo` 命令，若是失败的话则会输出找不到该模块，例如：

```bash
modprobe: FATAL: Module ppp-compress-18 not found.
```

若是不存在需要安装该软件包：

```bash
sudo yum install kernel-devel
```

2. 检查系统中是否有 TUN/TAP 驱动（在内核版本 2.4 时便加入内核驱动中）

```bash
cat /dev/net/tun

cat: /dev/net/tun: 文件描述符处于错误状态
```

有这样的输出表示当前的驱动是支持的。

### 4.2 搭建 PPTP

通过以上的步骤让系统拥有了搭建 PPTP 的必备条件，MPPE 模块主要用于数据的加密，而 TUN/TAP 主要用于虚拟网卡，紧接着我们将安装相关的服务：

1.安装 `ppp` 与 `pptpd`：

```bash
# ppp
sudo yum install ppp pptpd
```

通过这样的方式我们可以确认我们成功的安装了 pptpd：

```bash
yum list installed |grep -E "ppp|pptpd"
```

成功安装 pptpd 之后我们先来简单说一下 ppp  ，以便理解我们的配置项。

早期我们上网是通过电脑外接一个 modem（调制解调器，也就是猫），然后通过电话线拨号来接入 ISP（Internet Service Provider，网络服务提供商，也就是移动、电信、联通等服务商），而这里的拨号接入 ISP 便是通过 PPP 协议来让数据包发送、校验与认证，接入 ISP 后会为本机分配 IP 地址，之后我们便可以通过这个 IP 地址与网络上的其他机器通信。

pptp 的原理与此类似，只是做了一些不同的处理。而我们接下来的配置项便是服务端在客户端通过验证之后将会为机器分配的 IP 地址范围。

2.编辑 `/etc/pptpd.conf` 配置文件

```
sudo vim /etc/pptpd.conf

localip 192.168.10.1
remoteip 192.168.10.100-200
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512357760767.png/wm)

> 注：
> - `localip` 是服务器的 IP 地址
> - `remoteip` 是将分配给连接到它的客户端的 IP。
>
> `localip` 与 `remoteip` 配置为使用 pptp 网络中所用的 IP 地址，并不会影响其自身的其他 IP 地址，所以这两个配置项的 IP 地址可以随意配置，`localip` 主要是作为 `remoteip` 的网关。
>
> 若是 `remoteip` 需要配置多个地址段只需要使用逗号 `,` 隔开即可，例如 `192.168.10.1-10,192.168.1.1-20`

我们在上文中说到客户端需要通过服务端的验证才可以获得 IP 地址，在 PPP 中使用的验证协议是 PAP 与 CHAP，接下来我们将要配置的便是 CHAP 的用户配置文件，只有用户名、密码与配置文件中的内容匹配才可以通过验证。

3.添加用户和密码，设置 PPTP 身份验证。 将其添加到`/etc/ppp/chap-secrets`

```bash
vim /etc/ppp/chap-secrets
```

打开配置文件，我们可以从注释中了解到，该配置项主要为四列，分别是：

- client：用户登录时所需要提供的用户名
- server：此处指的是本机的名字
- secret：用户登录时所需要提供的密码
- IP addresses：用户通过该用户登录是分配的 IP 地址（需要在刚刚配置的 remoteip 的范围内），若是值为 `*` 则为随机分配 `remoteip` 范围内的地址即可。

这里我们将创建三个用户，内容如下：

```bash
user1 pptpd 123 *
user2 pptpd qwe *
user3 pptpd asd *
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512357794168.png/wm)

### 4.3 配置 DNS 服务器

我们不仅可以配置登录用户的用户地址，还可以配置用户登录之后使用的 DNS，当然该配置项为可选配置项。

DNS 的地址可以是本机使用的 DNS、内网中的 DNS、或者常用流行的 DNS（电信，阿里，114等等）

若是配置本机使用的 DNS：

1.首先，通过 `cat /etc/resolv.conf` 查看本机的 DNS 服务器 IP 地址。

```bash
sudo cat /etc/resolv.conf
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4232timestamp1514073370609.png/wm)

2.然后添加相关的内容到 `/etc/ppp/options.pptpd`。

```bash
sudo vim /etc/ppp/options.pptpd

ms-dns 10.202.72.116
ms-dns 10.202.72.118
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512357828290.png/wm)

启动 PPTP 的守护进程：

```bash
sudo systemctl restart pptpd
```

验证 pptp 已经运行并已连接（`pptp` 默认端口号 `1723`）：

```bash
sudo netstat -alpn | grep :1723
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512357853183.png/wm)

### 4.4 配置转发

在 PPTP 服务器上启用 `IP 转发`是非常重要的。 这会允许使用 PPTP 设置的公共 IP 和私有 IP 之间进行数据包的转发。

出于安全考虑， Linux 系统默认是禁止数据包转发的。现在打开系统的转发功能（本环境里默认是打开转发的）

```bash
cat /proc/sys/net/ipv4/ip_forward # 该文件内容为 0，表示禁止数据包转发，1 表示允许，将其修改为 1。

# 切换到 root
sudo su

echo "1" > /proc/sys/net/ipv4/ip_forward # 修改文件内容及时生效，重启网络服务或主机后效果不在。
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512357882075.png/wm)

从输出信息可以看出系统默认已经是 `1`了。

### 4.5 为 iptables 创建 NAT 规则

通过以上的配置就可以让客户端连接上 pptp 的服务端，但是这样我们只能与同样连接了 pptp 服务端的机器相互通信，若是我们需要使用服务器中的内网资源，我们需要结合 iptables 做 pptp 的流量转发，例如：

```bash
# 通过 nat 将源地址为 192.168.10.0 网段的数据包转发至 eth0 网卡，并替换源地址
iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o eth0 -j MASQUERADE && iptables-save
```

>注意：这里选用 eth0 网卡是因为其为本机的内网网卡（可以通过 `ifconfig` 查看验证），通过将数据包转发至内网网卡，我们便可访问内网资源。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4232timestamp1514078381119.png/wm)

由此我们便完成了 pptp 服务端的所有配置工作。

### 4.6 安装客户端

接下来我们将通过安装客户端，本地连接来验证我们配置的成功与否（若有条件的同学可以用一台服务器与自己本地做测试，Linux 的客户端需要安装，Windows 之间创建新连接配置 vpn 即可，无需额外安装客户端）：

首先我们将安装 pptp 客户端：

```
sudo yum install pptp
```

添加 kernel 模块：
`ptpsetup` 脚本在运行时会检查核心是否支持 `MPPE` 模块，以及 PPP 是否支持 `MPPE` 加密。通过下面的命令检查内核是否支持 MPPE 补丁。

```
sudo modprobe ppp_mppe && echo "is OK"
```

创建一个 `/etc/ppp/peers/pptpserver` 文件，用于配置连接参数，添加以下几行（其中用户名与密码便是我们在上文中所配置的用户名、密码，而 pptp 的服务器地址因为此处是本地连接所以填写的是 `127.0.0.1`，若是连接远程服务器请填写远程服务器的 IP）：

```bash
sudo vim /etc/ppp/peers/pptpserver

pty "pptp 127.0.0.1 --nolaunchpppd"
name user1
password 123
remotename PPTP
require-mppe-128
```

通过 `call` 命令，尝试连接服务器：

```bash
# 切换到 root
sudo su

pppd call pptpserver
```

客户端连接之后并未有任何信息的输出，我们可以查看系统日志来证实。

通过命令可以看到 PPTP 服务日志中已经成功连接(可以通过 `ctrl+c` 退出这种持续查看的状态):

```
# 切换到 shiyanlou 用户
su shiyanlou  或者使用  exit

# 查看日志
sudo tail -f /var/log/messages
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4232timestamp1514079524439.png/wm)

还可以通过 `ifconfig` 进行查看，验证我们成功连接：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4232timestamp1514079692925.png/wm)

PPTP 服务搭建及配置操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week4/6-1.mp4
@`

## 5 搭建 openVPN 服务

虽然 PPTP 的解决方式不错，但是 PPTP 相对于 openvpn 来说，openvpn 较安全也更稳定。

### 5.1 openVPN 工作原理

openVPN 通过 `TLS` 加密的方式，这种方式使用公开密钥（非对称密钥，加密解密使用不同的密钥，一个是 Public key，另外一个是 Private key）对数据进行加密。

openVPN 在加密过程中，需要 `server 端`和 `client 端`具有相同的 `CA 证书`，双方通过交换证书并验证合法性，再决定是否建立 VPN 连接。然后通过加密方式发送对方的 CA 证书，因此，只有对方 CA 证书对应的 `Private key` 才能解密该数据，这样就保证了此密钥的安全性，并且此密钥会定期改变，防止攻击破解。

CA 证书是一个验证安全性的一个工具，其中会记录证书的签发日期、所属地区、什么机构等一些信息，对于证书的验证过程及为什么这样的方式会安全推荐 [hyddd 博客](http://www.cnblogs.com/hyddd/archive/2009/01/07/1371292.html) 供大家进一步了解。

### 5.2 安装 openVPN

1.安装 `openVPN` 软件：

```bash
sudo yum install openvpn
```

2.安装成功之后，可以查看下 `openVPN` 的版本信息：

```bash
openvpn --version
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4232timestamp1514080481530.png/wm)

3.下载制作证书的工具 `easy-rsa`：

```
wget http://labfile.oss.aliyuncs.com/courses/980/software/easy-rsa-old-2.3.3.tar.gz
tar zxf easy-rsa-old-2.3.3.tar.gz
```

### 5.3 制作证书

证书的种类有：

- 自签名证书
- 签名证书
- 认证中心（CA）证书
- 未签名证书

openvpn 主要是为了让服务端与客户端之间相互验证，所以可以直接使用自签名证书即可。其执行过程可以参考 [阮一峰博客中的解释](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)

根据 openvpn 的工作原理，证书分为三部分：`CA 证书`、`Server 端证书`、`Client 端证书`。

#### 5.3.1 制作 CA 证书

1.在 `/etc/openvpn/` 目录下创建 `easy-rsa` 文件夹。

```bash
cd /etc/openvpn
sudo mkdir easy-rsa
ll /etc/openvpn
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4232timestamp1514081199711.png/wm)

`easy-rsa` 为我们提供了制作证书的脚本与模板文件，我们从之前解压出来的文件夹里将其全部复制至 `/etc/openvpn/easy-rsa/` 下：

```bash
sudo cp -r /home/shiyanlou/easy-rsa-old-2.3.3/easy-rsa/2.0/* /etc/openvpn/easy-rsa
ll /etc/openvpn/easy-rsa/
```

2.编辑 vars 文件

`vars` 文件中主要申明了机构的信息：

```bash
sudo vim /etc/openvpn/easy-rsa/vars
```

我们将相关的城市信息与机构信息改成我们的信息：

```bash
export KEY_COUNTRY="CN"
export KEY_PROVINCE="SC"
export KEY_CITY="ChengDu"
export KEY_ORG="shiyanlou"
export KEY_EMAIL="shiyanlou@shiyanlou.com"
export KEY_CN=vpn.shiyanlou.com
export KEY_NAME="vpnshiyanlou"
export KEY_OU="shiyanlou"
export PKCS11_MODULE_PATH=changeme
export PKCS11_PIN=1234
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid4985timestamp1523344844614.png/wm)

执行脚本，使得声明的变量生效：

```bash
#　切换路径
cd /etc/openvpn/easy-rsa/

# 切换到 root 用户
sudo su

# 执行脚本
source vars
./clean-all
```

3.制作 CA 证书

通过官方为我们提供的简单脚本，会读取我们之前声明的变量生成 CA 证书：

```bash
./build-ca
```

因为我们已经修改了相关变量的值为我们所需，**所以此时创建时的默认值便是我们想要的，不断回车确认即可**：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512358137434.png/wm)

制作完成后，查看 keys 目录

```bash
ll keys/
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512358155322.png/wm)


生成了 `ca.crt` 和 `ca.key` 两个文件，其中 `ca.crt` 就是 CA 证书。

把 CA 证书的 `ca.crt` 文件复制到 openvpn 的启动目录 `/etc/openvpn` 下。

```bash
cp keys/ca.crt /etc/openvpn/

ll /etc/openvpn/
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4272timestamp1514131123241.png/wm)

#### 5.3.2 制作 server 端证书

1.制作证书

```bash
./build-key-server vpnshiyanlou
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512358240305.png/wm)

查看生成的 Server 端证书

```bash
ll /etc/openvpn/easy-rsa/keys/
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512358255844.png/wm)

可以看到已经生成了`vpnshiyanlou.crt`、`vpnshiyanlou.key`和`vpnshiyanlou.csr`三个文件。其中`vpnshiyanlou.crt`和`vpnshiyanlou.key`两个文件是后面需要的。

2.为服务器生成加密交换时的 `Diffie-Hellman` 文件

```
./build-dh
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512369026716.png/wm)


查看生成文件

```bash
ll /etc/openvpn/easy-rsa/keys/
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512369082087.png/wm)

看到已经生成了`dh2048.pem`这个文件

3.把`vpnshiyanlou.crt`、`vpnshiyanlou.key`、`dh2048.pem`复制到`/etc/openvpn/`目录下

```
cd /etc/openvpn/easy-rsa/
cp keys/vpnshiyanlou.crt keys/vpnshiyanlou.key keys/dh2048.pem /etc/openvpn/
```

#### 5.3.3 制作 Client 端证书

```bash
./build-key shiyanlou
```

查看生成的证书

```bash
ll keys/
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512369129605.png/wm)

看到已经生成`shiyanlou.csr`、`shiyanlou.crt`和`shiyanlou.key`这个三个文件。其中`shiyanlou.crt`和`shiyanlou.key`两个文件后面会用到。

### 5.4 配置 Server 端

Server 端的配置文件，可以从 openvpn 自带的模板中进行复制。

```bash
cp /usr/share/doc/openvpn-2.4.4/sample/sample-config-files/server.conf /etc/openvpn/

cd /etc/openvpn/
```

修改 `server.conf` 文件

先用下面命令查看下要修改的文件内容：

```bash
grep -vE "^#|^;|^$" server.conf
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512369190271.png/wm)

再用 `vim` 编辑器对文件进行修改，主要修改下面几项内容。

```bash
vim server.conf

proto tcp # 环境建议使用 TCP 协议

# 修改相关证书
ca /etc/openvpn/ca.crt
cert /etc/openvpn/vpnshiyanlou.crt 
key /etc/openvpn/vpnshiyanlou.key
# 修改 Diffie-Hellman 文件
dh dh2048.pem
```

>注意：
> 若是改成了 `tcp` 注意注释以下行(此处为两行注释)：    
> \#explicit-exit-notify 1
> \#tls-auth ta.key 0

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512369295960.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512369301813.png/wm)

> 注：`server.conf` 文件中 `vpnshiyanlou.crt`、`vpnshiyanlou.key`、`dh2048.pem` 要与 `/etc/openvpn/` 目录下的相关文件一一对应。

> 如果上述文件如果没有存放在 `/etc/openvpn/` 目录下，在`server.conf` 文件中，要填写该文件的绝对路径。

可以再次查看下文件内容：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512369331097.png/wm)

配置文件修改完毕后，可以对其做与 pptp 同样的 iptables 处理，将进入的数据包都通过 nat 转换。因为之前搭建 
`PPTP` 服务的时候已经执行过，所以这里无需再执行。

然后我们需要创建一个启动脚本:

```bash
vim /usr/lib/systemd/system/openvpn.service

# 添加下面的内容

[Unit]
Description=OpenVPN
After=network.target
[Service]
PrivateTmp=true
Type=forking
PIDFile=/var/run/op.pid
ExecStart=/usr/sbin/openvpn --daemon --writepid /var/run/op.pid --cd /etc/openvpn/ --config server.conf
[Install]
WantedBy=multi-user.target

```

开始启动 `openvpn`:

```bash
sudo systemctl start openvpn

netstat -tunlp | grep 1194
```

可以看出 openvpn 已经启动，并且使用的 TCP 协议的 `1194` 端口。

### 5.5 配置 Client 端

1.下载证书及配置文件

```
sudo su
cd /etc/openvpn/easy-rsa/keys
cp shiyanlou.crt shiyanlou.key ca.crt /home/shiyanlou/

cp /usr/share/doc/openvpn-2.4.4/sample/sample-config-files/client.conf /home/shiyanlou/
```

2.修改文件的用户属性

```bash
cd /home/shiyanlou

chown shiyanlou:shiyanlou shiyanlou.*

chown shiyanlou:shiyanlou ca.crt

chown shiyanlou:shiyanlou client.conf
```

3.退出 root 用户，切换到 `shiyanlou` 用户的家目录下。

```bash
# 退出 root 用户
su shiyanlou  

或者使用 

exit

# 切换到用户的家目录 
cd ~
```

4.将`client.conf`文件重命名为`client.ovpn`，然后进行编辑

```bash
mv client.conf client.ovpn
vim client.ovpn
```

修改内容如下：

```bash
proto tcp # 修改使用协议，和 server 端保持一致

remote localhost 1194 # 修改为 server 端的地址 localhost 

# 修改证书名称
ca ca.crt
cert shiyanlou.crt
key shiyanlou.key

还需要注释 tls-auth ta.key 1
```

修改完毕后，把这几个文件放在同一个`/test`文件夹中，并且一定要保持 `client.ovpn` 这个文件名称是唯一的。

```bash
mkdir test
cp ca.crt client.ovpn shiyanlou.crt shiyanlou.key test
```

5.连接 server 端

```
cd  /home/shiyanlou/test
sudo openvpn --config client.ovpn
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512369627474.png/wm)

6.验证

开启一个新的 `terminal`

```bash
ifconfig
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4105timestamp1512369549746.png/wm)

可以看到在本机中虚拟出了 tun0 的虚拟网卡。

搭建及配置 openVPN 服务操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week4/6-2.mp4
@`

##  6 总结

通过本实验的学习，可以掌握 VPN 的相关软件特点以及如何自行搭建起一个 VPN 服务。在本实验中我们接触到了非常多的新网络概念，例如加密、非对称加密、证书等等，希望大家不能能够自行搭建 VPN 系统，还能知道其中原理。