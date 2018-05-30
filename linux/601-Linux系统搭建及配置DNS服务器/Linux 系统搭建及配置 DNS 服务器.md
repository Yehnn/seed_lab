---
show: step
version: 0.1
enable_checker: true
---

# Linux DNS 服务部署与配置

## 1. 实验介绍

### 1.1 实验内容

IP 地址是与服务器内容没有意义关联的一串数字，不便于记忆，通过 DNS 可以帮助我们将这些无意义、难以记忆的数字与有意义的描述做关联，帮助我们记忆，本实验便将带领大家搭建自己的 DNS 服务器。

### 1.2 实验知识点

+ DNS 简介
+ DNS 解析
+ DNS 搭建
+ DNS 验证

### 1.3 推荐阅读

- [BIND 官方配置项文档](https://ftp.isc.org/isc/bind/9.9.4/doc/arm/Bv9ARM.ch06.html)
- [How To Configure BIND as a Private Network DNS Server on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-14-04)
- [BIND9 管理员参考手册](https://www.centos.bz/manual/BIND9-CHS.pdf)
- [BIND相关资料](https://www.isc.org/downloads/bind/)
- [段海新的论文](http://netsec.ccert.edu.cn/duanhx/?p=1479)

## 2. DNS 简介

DNS 是 Domain Name System（域名系统）的简称，DNS 是一个分层的分布式命名系统，用于连接到互联网或专用网络的计算机，服务或其他资源。它将各种信息与分配给每个参与实体的域名关联起来。

简单来说其最大的功能就是将域名翻译成 IP 地址，例如实验楼的 IP 地址就是一串毫无规则的数据，非常不好记忆，想的起实验楼但是想不起它的 IP 地址。但是域名与站点的意义是相关联的，便于记忆，想起实验楼，就能想起它的域名是 `www.shiyanlou.com`。举一个不恰当的例子，人与人之间通电话拨电话的时候，都是通过名字查找电话本中的电话号码，然后直接拨通，电话号码是无逻辑的一串数字，名字是有意义的。

人的姓名是由姓氏与名字构成的，同样域名也是由多个部分构成的。构成域名的每个部分称为一个区域（zone），主要是这样分级的：`主机名.二级域名.顶级域名.`

不同域直接通过 `.` 来分割，域名与国外人命名一般，名在前，姓在后，越是大的域越在后面。也就是说靠前的是后面的子集，最右边的是根域可以忽略不写（因为其无名，nameless），在顶级域名的点后面，根域紧接着的就是顶级域名（top-level domains），紧跟着的就是二级域名(second-level)，就这样依此类推 +1，在 DNS 的配置时我们需要留着顶级域名后面的 `.`，在平时使用的时候因为浏览器会帮助我们自动填充，所以我们可以忽略不写，例如 `www.shiyanlou.com` 不用写成 `www.shiyanlou.com.`。

![2](https://doc.shiyanlou.com/document-uid113508labid1986timestamp1510191991239.png)

- 主机名：一般代表公司或者组织的主机名或者某个服务名，一般在最左边；例如 `www.shiyanlou.com` 中 `www` 是三级域名也是主机名了，一般这样的比较常见
- 二级域名（second-level domains）：一般代表公司或者组织的名字，例如 `www.shiyanlou.com` 中的 `shiyanlou`，例如 `www.baidu.com` 中的 `baidu`
- 顶级域名（top-level domains，TLDS）：一般代表的是该公司或者组织在社会中的某个领域，或者所说的国家。在顶级域名中包含两种：
    + 通用顶级域名（generic top-level domains，gTLDs）：通用的顶级域名代表的便是在社会中的某个领域，例如 `com` 表示的是商业，`edu` 表示的教育机构，`gov` 表示的政府机构，`org` 表示的组织机构（例如开源组织什么的）
    + 国家代码顶级域名（country code top-level domains，ccTLDs）：国家代码顶级域名显然就是代表所属国家了，例如 `cn` 代表的是中国，`jp` 代表的是日本，`ca` 代表的是加拿大等等

当然 gTDLS 与 ccTLDs 不是说只能用一个，例如南开大学的域名 `www.nankai.edu.cn`.

## 3. DNS 解析

上一节中我们了解到了域名是什么，域名的命名规则，接下来我们将简单地了解一下 DNS 是如何将一个域名解析成 IP 地址的，使得我们的浏览器或者其他工具能够访问到我们需要的服务。

我们通过下图来了解整个过程：

![3](https://doc.shiyanlou.com/document-uid113508labid1986timestamp1510197302739.png)
(此图来自于[ictedu](http://www.ictedu.info/e-learning/redhat-linux-course/04-dns-server/me-shume-detaje-mbi-dns/how-dns-query-works))

通过浏览器访问 `www.shiyanlou.com`：

- 浏览器首先查看其自己的解析记录缓存：
    + 若是缓存记录中存在域名所对应的 IP 地址，则直接使用该 IP 地址
    + 若是缓存记录中不存在该域名的相关记录则进入下一步
- 查找系统的 DNS 解析缓存，因为 DNS 每次查询的记录都会记录在缓存在中，以免再次查询。
    + 在 Windows 中系统启动时便会加载位于 `C:\windows\system32\drivers\etc\` 目录中的 `hosts` 文件于 DNS 缓存中，所以系统在查看 DNS 解析缓存时也等同于在查看 hosts 文件中的记录(在 windows 中并不会直接查询 hosts 文件，而是通过缓存的方式间接查看)
    + Linux 中的 hosts 文件位于 `/etc/hosts`，其与 Windows 中使用 hosts 类似，但具体实现方式不同

> 注意：hosts 文件是 DNS 的前身，早期电脑的数量还很小，一个文件列表中的域名与 IP 的对应足以应付，只是到了后期不便更新维护以及数量大大增加，所以诞生了 DNS 分布式的数据库。

- 若是在系统的 DNS 解析缓存中也未能找到相关的记录，此时便会去请求系统中设置的 DNS 服务器（系统中设置的 DNS 服务器可能是路由器分配的默认，也可能是自己设置的权威机构的，也可以能运营商自己的）
- 部分路由器会有 DNS 解析的缓存，此时会在路由器中查询
- 若还是没有便会将查询的请求交给 ISP 的 DNS 缓存服务器
- 依旧没有查询到相关的记录便会开始递归搜索
    + 由本地 DNS 服务器或者你配置的服务器将相关的请求转发给根域服务器
    + 根域服务器查询相关的顶级域服务器，然后将请求转发给相关的顶级域服务器
    + 顶级域服务器查询相关的二级域服务器，然后将请求转发相关的二级域服务器
    + 最后将查询结果返回给主机
- 若还是没有相关记录则访问失败

>注意：大部分的请求基本在运营商处可以得到满足，在中国没有 IPv4 的根域，只有镜像。

### 3.1 DNS 解析优先级

在上文中我们提到早期没有 DNS 服务器存在的时候使用的是 hosts 文件，到后期才出现了 DNS 服务器，新的服务方式提出并不代表老得服务方式被彻底替换，hosts 文件的方式因为快速、小巧而被保留了下来。

在 Linux 中我们可以通过 `/etc/nsswitch.conf` 配置文件修改 DNS 查询的顺序：

通过 `sudo vim /etc/nsswitch.conf` 打开该配置文件，往下查看就可以看到这样的配置：

```bash
#hosts:     db files nisplus nis dns
hosts:      files dns myhostname
```

这里的 `files` 代表的就是 `/etc/hosts` 文件，`dns` 代表的是系统配置的 DNS 服务器地址。所以在 Linux 中默认是先查询 hosts 文件中的记录，然后再请求 DNS 服务器。

1.hosts 文件

因为 hosts 文件在本地相比于请求 DNS 服务器的方式快速，所以得到保留。

我们通常会在 hosts 文件中存放常用的 DNS 记录与开发中测试使用的服务器记录.

hosts 文件的记录格式非常的简单：

```bash
IP地址  域名或者hostname
```

例如我们当前机器中的 hosts 文件 `cat /etc/hosts`:

![3.1](https://doc.shiyanlou.com/document-uid8797labid1986timestamp1527486954991.png)

第二行中 `::1` 是 IPv6 的本地地址表示方式，类似于 IPv4 中 `127.0.0.1` 后面紧跟着的就是主机名。

后续的记录中记录着内网中的地址与主机名的映射关系，我们可以将测试服务器的 IP 地址与测试域名映射关系存放于此，这种记录是不会放在公共 DNS 服务器中，一般存放于本地 DNS 或者 hosts 中。

2.DNS 服务器配置文件

系统设定的 DNS 服务器配置文件位于 `/etc/resolv.conf`:

![3.1-2](https://doc.shiyanlou.com/document-uid8797labid1986timestamp1527487094511.png)

`nameserver` 后面对应的便是内网中的 DNS 服务器地址，一般会配置两个，防止一个不可用而导致无法域名解析。

## 4 DNS 搭建

一切的理论都是为了帮助我们更好的理解软件中的各项配置，以及在出问题时的进行调试，但是仅仅只有理论是不够的，我们需要通过 DNS 的搭建来进一步的了解。

### 4.1 DNS 的选用

DNS 的使用方案有许多，常见的开源软件有：

- `BIND`：全名为 Berkeley Internet Name Domain，是早在 1980 年左右有 Berkeley 大学公开出来的 DNS 服务实现，也是使用最为广泛的方案。后由 ISC 基于 BIND 重写发布 BIND9
- `PowerDNS`：PowerDNS 由 C++ 实现于 1990 年末，起源一个商业软件于 2002 年开源，相对于 BIND 在数据库选用上与集群上功能更多更灵活。
- `CoreDNS`：由 SkyDNS 进化而来，主要作为一种可插拔的中间件。
- `DNSpod-sr`：一款由国内服务商开源的一套 DNS 的实现

此次我们将选用最为成熟、拥有良好 License 的 `BIND9` 来搭建属于我们自己的 DNS 服务器。

### 4.2 DNS 的安装

#### 4.2.1 安装和配置BIND

```
sudo apt-get update  # update 可能会报两个警告，可以忽略

sudo apt-get install -y bind9 bind9utils bind9-doc
```

#### 4.2.2 激活IPv4 Mode

在继续配置 DNS 服务器之前，先修改一下 bind9 service 参数文件，设置 BIND 为 IPv4 Mode。

```
sudo vi /etc/default/bind9
```

把 "-4" 添加到 OPTIONS 变量，如下所示 */etc/default/bind9*

```
OPTIONS="-4 -u bind"
```

#### 4.2.3 确定主DNS服务器

一般都会设置两个 DNS 服务器，一个主要的，一个备用的，这次的实验受实验环境的限制，我们只做主服务器。 首先查看本机的ip地址

```
ifconfig -a
```

我的 ip 地址是`192.168.42.5`，每个人的是不一样的。 然后获取几个有效的内网 ip ，可以试着去 ping 一下，我找的内网 ip 是 `192.168.42.1`。

```bash
ping -c 3 192.168.42.1
```

```checker
- name: check option
  script: |
    #!/bin/bash
    grep 4 /etc/default/bind9
  error: 没有配置 /etc/default/bind9 文件
```

### 4.3 DNS 服务器的配置

BIND 的配置由多个文件组成，这些文件包含在主配置文件中`named.conf`。这些文件名以“named”开头，因为这是BIND 运行的进程的名称。我们将开始配置选项文件。 

```bash
cat /etc/bind/named.conf
```

![4.3](https://doc.shiyanlou.com/document-uid8797labid1986timestamp1527488516741.png/wm)

有关各配置详细解释可参考 [BIND 官方配置项文档](https://ftp.isc.org/isc/bind/9.9.4/doc/arm/Bv9ARM.ch06.html) 

#### 4.3.1 配置选项文件

打开`named.conf.options`文件进行编辑： 

```
sudo vi /etc/bind/named.conf.options
```

在现有的文档之上，添加一个新的 ACL 块并命名 "trusted"，我们把之前找到的内网 ip 添加到可信客户端列表里，并称为 ns1 ,只允许它们查询 DNS 服务器 ， 提高安全性。

```
acl "trusted" {
  	        192.168.42.5; #ns1	信任的名单
};
options {
	directory "/var/cache/bind";

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk.  See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.

		recursion  yes;                 # enables resursive queries
        allow-recursion { trusted; };  # allows recursive queries from "trusted" clients
        listen-on { 192.168.42.5;  };   # ns1 private IP address - listen on private network only
        allow-transfer { none; };      # disable zone transfers by default

        forwarders {
               	114.114.114.114;        # 当本地的dns服务器中找不到记录时向上查询
        };
	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys.  See https://www.isc.org/bind-keys
	//========================================================================
	dnssec-validation auto;

	auth-nxdomain no;    # conform to RFC1035
	listen-on-v6 { any; };
};
```

> 注意 ip 地址根据你自己的 ip 地址填写。
>
> 按 `esc` ，然后输入`:wq`  保存并退出 `named.conf.options`，通过上面的设置只有信任的主机才能查询 DNS 服务器，其他主机不能。

#### 4.3.2 配置本地文件

打开 `named.conf.local `并编辑：

```
sudo vi /etc/bind/named.conf.local
```

我们修改配置文件，完成后如下，在配置文件中添加了[正向解析和反向解析](http://blog.csdn.net/jackxinxu2100/article/details/8145318)的文件在系统中的位置，简单的说，正向解析就是根据域名查 ip，反向就是根据 ip 查域名

```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "shiyanlou.example.com" {
    type master;
    file "/etc/bind/zones/db.shiyanlou.example.com"; # zone file path
};

zone "168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168";  
};
```

> 注意，反向区域名称以 `168.192` 开头，即 `192.168` 反转。

添加完成后，保存退出。这样我们在 BIND 我们就有了指定的文件，接下来我们就编写正向解析和反向解析的域文件。

#### 4.3.4 创建正向解析域文件

> 正向解析：通过主机名获取其对应的广域网 IP 地址，我们以 "host1.shiyanlou.example.com" 为例来编写来这个文件。

```
sudo mkdir /etc/bind/zones
```

我们根据现有的域文件`db.local`，来复制一份作为`db.shiyanlou.example.com` 编辑这个正向解析文件

```
sudo cp /etc/bind/db.local /etc/bind/zones/db.shiyanlou.example.com
sudo vi /etc/bind/zones/db.shiyanlou.example.com
```

我们在原来文件的基础上，修改了很多，具体如下

```
;
; BIND data file for local loopback interface
$TTL    604800
@       IN      SOA     ns1.shiyanlou.example.com. admin.shiyanlou.example.com. (
                  3       ; Serial
             604800     ; Refresh
              86400     ; Retry
            2419200     ; Expire
             604800 )   ; Negative Cache TTL
;
;
; name servers - NS records
     IN      NS      ns1.shiyanlou.example.com.
; name servers - A records
ns1.shiyanlou.example.com.          IN      A       192.168.42.5

host1.shiyanlou.example.com.        IN      A      192.168.42.1
```

> **注意上面加点的地方不要漏写**
>
> ip 地址根据自己的实际地址填写。

#### 4.3.5 创建反向解析域文件

> 反向域名解析与通常的正向域名解析相反，提供IP地址到域名的对应。 IP 反向解析主要应用到邮件服务器中来阻拦垃圾邮件，特别是在国外。多数垃圾邮件发送者使用动态分配或者没有注册域名的IP地址来发送垃圾邮件，以逃避追踪，使用了域名反向解析后，就可以大大降低垃圾邮件的数量。

在之前的 `named.conf.local` 的文件里我们写了一个反向域名解析的文件名`db.192.168`，现在来编写它：

```
sudo cp /etc/bind/db.127 /etc/bind.zones/db.192.168
sudo vi /etc/bind/zones/db.192.168
```

文件的具体内容如下:

```
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@       IN      SOA     shiyanlou.example.com. admin.shiyanlou.example.com. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
; name servers
      IN      NS      ns1.shiyanlou.example.com.

; PTR Records
5.42   IN      PTR     ns1.shiyanlou.example.com.    ; 192.168.42.5
1.42   IN      PTR     host1.shiyanlou.example.com.  ; 192.168.42.1
```

> ip 地址根据自己的实际地址填写。
>
> PTR 首位根据后面的 ip 反转。

#### 4.3.6 检验 BIND 配置文件语法错误

运行下面的命令检验 `named.conf*` 有没有语法错误：

```
sudo named-checkconf
```

如果没有报错则编写是正确的，如果报错了，那就根据报的错误去修改文件。 运行下面的命令查看正向解析和反向解析文件是否正确。

```
sudo named-checkzone shiyanlou.example.com /etc/bind/zones/db.shiyanlou.example.com
sudo named-checkzone 168.192.in-addr.arpa /etc/bind/zones/db.192.168
```

![4.3.6-1](https://doc.shiyanlou.com/document-uid8797labid1986timestamp1527496483672.png)

做到这里，离成功已经不远了:-)，我们重启 BIND service

```
sudo service bind9 restart
```

![4.3.6-2](https://doc.shiyanlou.com/document-uid8797labid1986timestamp1527496521741.png)

```checker
- name: check options
  script: |
    #!/bin/bash
    grep recursion /etc/bind/named.conf.options
  error: 没有配置 /etc/bind/named.conf.options 文件
- name: check local
  script: |
    #!/bin/bash
    grep type /etc/bind/named.conf.local
  error: 没有配置 /etc/bind/named.conf.local
- name: check db1
  script: |
    #!/bin/bash
    ls /etc/bind/zones/db.shiyanlou.example.com
  error: 指定目录下没有正向解析域文件
- name: check db2
  script: |
    #!/bin/bash
    ls /etc/bind/zones/db.*|wc -l
  error: 指定目录下没有反向解析域文件
- name: check service
  script: |
    #!/bin/bash
    ps -ef |grep -v grep | grep named
  error: 没有运行 BIND
```

### 4.4 DNS 客户端配置

配置 DNS 客户端，在我们的实验环境里，客户端和服务器是一体的，用自己的电脑的话，可以试试局域网的其他电脑做客户端。将客户端的 DNS 修改为服务器的 ip 地址。 因为我们用的是 Ubuntu 的系统，所以运行命令

```
sudo vi /etc/resolvconf/resolv.conf.d/head
```

把文档修改为如下所示：

```
nameserver 192.168.42.5  # ns1 private IP address
```

然后打开文档`resolv.conf`

```
sudo vi /etc/resolv.conf
```

编辑为：

```
options timeout:1 attempts:1 rotate
nameserver  192.168.42.5
```

接下来运行 `resolvconf` 生成一个新的 `resolv.conf`文件

```
sudo resolvconf -u
```

好了，你的客户端可以检验你的服务器是否可以正常使用了。

```checker
- name: check head
  script: |
    #!/bin/bash
    grep nameserver /etc/resolvconf/resolv.conf.d/head
  error: 没有配置 head 文件
- name: check resolv
    #!/bin/bash
    grep nameserver /etc/resolv.conf
  error: 没有配置 resolv.conf
```

## 5. 排错以及验证

#### 排错

```
使用named -g可以让dns服务器在前台运行，并输出所有信息到屏幕，在里面可以看到错误信息 
```

#### 验证

测试我们的 DNS 服务器是否可以正常运行，使用 [nslookup](https://www.ezloo.com/2011/04/nslookup.html),来查询你的服务器，（若使用其他的客户端， ip 地址 需要加入到 "trusted" ACL 里面）。先测试正向解析，通过输入网址找到ip地址 运行下面的代码

```
nslookup host1.shiyanlou.example.com
```

![5-1](https://doc.shiyanlou.com/document-uid8797labid1986timestamp1527497092104.png/wm)

再测试反向解析，通过 ip 地址找到它的网址。**如果你的反向域名文件写错了，或者放置的位置不对，就会报错，可以在上文重新看看操作**

```
nslookup 192.168.42.1
```

![5-2](https://doc.shiyanlou.com/document-uid8797labid1986timestamp1527497092608.png/wm)

好了，到此为止，一个简单的 DNS 服务器就搭建好了，你可以试试添加更多的 ip 地址，快来试试自己的 DNS 服务器吧！

## 6. 总结

DNS 配置上还有很多可配置的东西，例如日志的种类筛选、还有黑/白名单的配置、DNSSEC、TXT 记录等等，有兴趣继续深入研究的同学可查看推荐阅读帮助自己理解与实践。

## 7.参考资料

- [How To Configure BIND as a Private Network DNS Server on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-14-04)
- [BIND9 管理员参考手册](https://www.centos.bz/manual/BIND9-CHS.pdf)
- [鳥哥的 Linux 私房菜](http://linux.vbird.org/linux_server/0350dns.php)
- [BIND相关资料](https://www.isc.org/downloads/bind/)