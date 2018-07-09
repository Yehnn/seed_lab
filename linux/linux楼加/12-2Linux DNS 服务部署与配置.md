---
show: step
version: 1.0
enable_checker: true
---
# Linux DNS 服务部署与配置

## 1. 实验介绍

#### 1.1 实验内容

IP 地址是与服务器内容没有意义关联的一串数字，不便于记忆，通过 DNS 可以帮助我们将这些无意义、难以记忆的数字与有意义的描述做关联，帮助我们记忆，本实验便将带领大家搭建自己的 DNS 服务器。

#### 1.2 实验知识点

+ DNS 简介
+ DNS 解析
+ DNS 搭建
    - DNS 安装
    - DNS 配置
        - DNS 服务配置
        - DNS 域配置
+ DNS 验证
+ DNS 安全（chroot）

#### 1.3 推荐阅读

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

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1986timestamp1510191991239.png/wm)

- 主机名：一般代表公司或者组织的主机名或者某个服务名，一般在最左边；例如 `www.shiyanlou.com` 中 `www` 是三级域名也是主机名了，一般这样的比较常见
- 二级域名（second-level domains）：一般代表公司或者组织的名字，例如 `www.shiyanlou.com` 中的 shiyanlou，例如 `www.baidu.com` 中的 baidu
- 顶级域名（top-level domains，TLDS）：一般代表的是该公司或者组织在社会中的某个领域，或者所说的国家。在顶级域名中包含两种：
    + 通用顶级域名（generic top-level domains，gTLDs）：通用的顶级域名代表的便是在社会中的某个领域，例如 `com` 表示的是商业，`edu` 表示的教育机构，`gov` 表示的政府机构，`org` 表示的组织机构（例如开源组织什么的）
    + 国家代码顶级域名（country code top-level domains，ccTLDs）：国家代码顶级域名显然就是代表所属国家了，例如 `cn` 代表的是中国，`jp` 代表的是日本，`ca` 代表的是加拿大等等

当然 gTDLS 与 ccTLDs 不是说只能用一个，例如南开大学的域名 `www.nankai.edu.cn`.

## 3. DNS 解析

上一节中我们了解到了域名是什么，域名的命名规则，接下来我们将简单地了解一下 DNS 是如何将一个域名解析成 IP 地址的，使得我们的浏览器或者其他工具能够访问到我们需要的服务。

我们通过下图来了解整个过程：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1986timestamp1510197302739.png/wm)
(此图来自于[ictedu](http://www.ictedu.info/e-learning/redhat-linux-course/04-dns-server/me-shume-detaje-mbi-dns/how-dns-query-works))

通过浏览器访问 `www.shiyanlou.com`：

- 浏览器首先查看其自己的解析记录缓存：
    + 若是缓存记录中存在域名所对应的 IP 地址，则直接使用该 IP 地址
    + 若是缓存记录中不存在该域名的相关记录则进入下一步
- 查找系统的 DNS 解析缓存，因为 DNS 每次查询的记录都会记录在缓存在中，以免再次查询。
    + 在 Windows 中系统启动时便会加载位于 `C:\windows\system32\drivers\etc\` 目录中的 hosts 文件于 DNS 缓存中，所以系统在查看 DNS 解析缓存时也等同于在查看 hosts 文件中的记录(在 windows 中并不会直接查询 hosts 文件，而是通过缓存的方式间接查看)
    + Linux 中的 hosts 文件位于 `/etc/hosts`，其与 Windows 中使用 hosts 类似，但具体实现方式不同
    + > 注意：hosts 文件是 DNS 的前身，早期电脑的数量还很小，一个文件列表中的域名与 IP 的对应足以应付，只是到了后期不便更新维护以及数量大大增加，所以诞生了 DNS 分布式的数据库。
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

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514238781733.png/wm)

第一行中 `::1` 是 IPv6 的本地地址表示方式，类似于 IPv4 中 `127.0.0.1` 后面紧跟着的就是主机名。

后续的记录中记录着内网中的地址与主机名的映射关系，我们可以将测试服务器的 IP 地址与测试域名映射关系存放于此，这种记录是不会放在公共 DNS 服务器中，一般存放于本地 DNS 或者 hosts 中。

2.DNS 服务器配置文件

系统设定的 DNS 服务器配置文件位于 `/etc/resolv.conf`:

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514239192729.png/wm)

`nameserver` 后面对应的便是内网中的 DNS 服务器地址，一般会配置两个，防止一个不可用而导致无法域名解析。

在上一个实验中我们了解到在分发地址的同时可以配置分发机器的网关、DNS 服务器等，若是 IP 地址是通过 DHCP 的方式获取，那么获取 IP 地址的同时会根据数据包中的内容修改系统的 `resolv.conf`（当然可以配置不让其修改，在配置网卡是添加 `PEERDNS=no` 参数）

DNS 解析介绍视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week4/2-1.mp4
@`

## 4 DNS 搭建

一切的理论都是为了帮助我们更好的理解软件中的各项配置，和在出问题时的进行调试，但是仅仅只有理论是不够的，我们通过 DNS 的搭建来进一步的了解。

### 4.1 DNS 的选用

DNS 的使用方案有许多，常见的开源软件有：

- BIND：全名为 Berkeley Internet Name Domain，是早在 1980 年左右有 Berkeley 大学公开出来的 DNS 服务实现，也是使用最为广泛的方案。后由 ISC 基于 BIND 重写发布 BIND9
- PowerDNS：PowerDNS 由 C++ 实现于 1990 年末，起源一个商业软件于 2002 年开源，相对于 BIND 在数据库选用上与集群上功能更多更灵活。
- CoreDNS：由 SkyDNS 进化而来，主要作为一种可插拔的中间件。
- DNSpod-sr：一款由国内服务商开源的一套 DNS 的实现

此次我们将选用最为成熟、拥有良好 License 的 BIND9 来搭建属于我们自己的 DNS 服务器。

### 4.2 DNS 的安装

这里我们将安装的是 bind 与 bind-chroot:

>注意：我们知道默认情况下 `/` 为我们的根目录，而使用 chroot 可能修改我们根目录的位置，通过 bind-chroot 我们可以限制 bind 读取文件，执行文件的范围，这样相对更加安全一些。

```bash
sudo yum install bind-chroot
```

因为 bind-chroot 依赖于 bind，所以直接安装 bind-chroot 的同时也会自动安装 bind：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514248920481.png/wm)

>注意：
>1.在 centos 中与高版本的 ubuntu 中都使用 systemd 来管理程序的状态与程序的状态。
>
>2.在 centos 中 bind 的后台为 named

### 4.3 DNS 的配置

#### 4.3.1 服务配置

安装成功之后默认配置文件位于 `/etc/named.conf`:

```bash
sudo vim /etc/named.conf
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514252235175.png/wm)

从配置文件中我们可以看出配置文件的主要格式是：

```bash
声明 {
    配置参数；
    配置参数 {
        子配置项；
        子配置项；
    }
    .......;
    .......;
}
```

通过系统提供的配置文件模版来了解常用的配置参数：

声明的参数主要有这样的一些配置：

| 声明    | 作用                                       |
| ------- | ------------------------------------------ |
| options | 配置服务的全局参数和一些其他配置项的默认值 |
| logging | 服务日志相关信息配置                       |
| zone    | 服务域的定义                               |

（1）options 的配置

在 options 中主要配置了一些全局的参数，例如监听的端口、服务的目录、保存文件的名字与路径等等，在默认的配置文件中主要分三个部分：

1.named 服务运行上的配置

| 配置项            | 作用                                 |
| ----------------- | ------------------------------------ |
| listen-on port    | 指定服务在 IPv4 上使用的端口         |
| listen-on-v6 port | 指定服务在 IPv6 使用的端口           |
| directory         | 服务的工作目录                       |
| pid-file          | 运行服务产生的 pid 文件路径          |
| recursion         | 在要求递归下，做递归查询回答查询请求 |

2.控制命令 rndc 运行后产生文件的目录配置：

| 配置项             | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| dump-file          | 运行 `rndc dumpdb` 备份缓存资料保存的文件名与路径(rndc 是 named 提供的控制命令) |
| statistics-file    | 运行 `rndc stats` 统计信息保存的文件名                       |
| memstatistics-file | 服务器输出的内存使用统计文件的路径名                         |

3.named 服务运行的安全问题

| 配置项                 | 作用                               |
| ---------------------- | ---------------------------------- |
| allow-query            | 指定可以查询主解析记录的服务器地址 |
| dnssec-enable          | 是否开启 DNS 安全模块              |
| dnssec-validation      | 是否进行 DNS 安全确认              |
| bindkeys-file          | 配置 DNS 安全模块所使用的密钥      |
| managed-keys-directory | 配置 DNS 安全模块管理密钥的目录    |
| session-keyfile        | 更新策略的 key 文件名              |

>注意：DNSSEC 即 DNS Security Extension，是 DNS 在安全上做的扩展，以应对 DNS 欺骗攻击、缓存污染等一些现有的问题，其原理与产生原因推荐[段海新的论文](http://netsec.ccert.edu.cn/duanhx/?p=1479)，讲解的非常详细

（2）logging 的配置

主要配置日志的日志类别与输出通道，日志类别能够帮我们过滤出想要的部分日志，通道指定我们日志输出的位置，通常分别使用 `category`、`channel`

在 channel 中：

- file：定义错误日志输出的文件名与目录
- serverity：定义日志的级别

（3）zone 的配置

域的定义，配置对于请求不同域的处理类型

除此之外还有一些常用的声明配置：

| 声明     | 作用                           |
| -------- | ------------------------------ |
| acl      | 定义访问控制列表，添加黑白名单 |
| inclued  | 引入其他的配置文件             |
| controls | 配置一些常用的操作             |
| view     | 定义视图                       |

从上述的解释中我们大概了解了默认配置模板中各配置项的意思，每种声明中都有非常多的子配置项，可以非常灵活的配置，我们不会注意讲解其意思与部分高级用法，有兴趣的同学可以查看推荐阅读中的[BIND 官方配置项文档](https://ftp.isc.org/isc/bind/9.9.4/doc/arm/Bv9ARM.ch06.html)

#### 4.3.2 域文件配置

接下来我们主要讲解 `zone` 声明的配置，因为 `zone` 涉及到 DNS 的主要功能。

我们通过这样的一个例子来理解：

我们将创建两个自定义的域，一个用于正向解析一个用于反向解析：

- 正向解析：用于将域名翻译成 IP 地址
- 反向解析：用于将 IP 地址翻译成域名，主要用于邮件系统，邮件系统开启反向解析功能若是无法通过 IP 获取域名的邮件拒绝接收，以此来降低收到垃圾邮件的数量

我们创建一个自定义域 `test-shiyannlou.com` 与自定义域 `113.29.10.in-addr.arpa` 分别用于正向解析与反向解析。

>注意：
>- 域名中可以有短横线，只是在国内用的较少，主要是考虑到不好表达、不好输入等一些习惯，在国外接受度高一些
>- 在域名表示习惯从左到右是小范围到大范围所以在反向解析的时候需要将网段反过来写，例如当前我的环境中所处的是 `10.29.113.0` 这个网段，这里的域便需要写成 `113.29.10.in-addr.arpa`
>- `.in-addr.arpa` 是反向解析域中的一个命名规范与表示

```bash
$sudo su
$vim /etc/named.conf

# 添加内容

zone "test-shiyanlou.com" IN {

};

zone "113.29.10.in-addr.arpa" IN {

};
```

在 zone 中主要的配置项有：

| 配置项         | 作用                                                |
| -------------- | --------------------------------------------------- |
| type           | 配置域的类型可用的值为 hint、master、salve、forward |
| file           | 配置定义域数据文件名                                |
| notify         | 更是数据之后时候通知其他服务器                      |
| allow-update   | 允许哪些机器更新域数据信息                          |
| allow-transfer | 允许可以下载数据文件                                |

其中 type 的四个值的意义是：

- hint：当本地找不到解析时可查询根域名服务器
- master：设置为主域名服务器
- slave：设置为次域名服务器
- forward：定义转发的域名服务器

此处将我们的两个自定义创建为主域名服务器，并配置域名信息文件：

```bash
zone "test-shiyanlou.com" IN {
    type master;
    file "test-shiyanlou.zone";
};

zone "113.29.10.in-addr.arpa" IN {
    type master;
    file "113.29.10.zone";
};
```

保存并退出配置文件的编辑，紧接着创建上面配置的信息记录文件 `test-shiyanlou.zone` 与 `113.29.10.zone`。这两个配置文件默认会放置在 `/var/named/` 目录下。

```
touch /var/named/test-shiyanlou.zone
touch /var/named/113.29.10.zone
```


在 zone file 中，每一条映射值为一条记录，每条记录都会有他的记录类型，针对不同的记录会有不同的格式，首先我们来简单了解一下记录类型的常用种类：

| 记录类型   | 作用                                   |
| ---------- | -------------------------------------- |
| A  记录    | 正向解析记录，域名到 IP 地址的映射     |
| NS 记录    | 域名服务器记录（NS 为 NameServer）     |
| MX 记录    | 邮件记录                               |
| PTR 记录   | 反向解析记录，IP 地址到域名的映射      |
| CNAME 记录 | 别名记录，为主机添加别名               |
| SOA 记录   | 域权威记录，说明本服务器为域管理服务器 |
| AAAA 记录  | 正向解析记录，域名到 IPv6 地址的映射   |

1.A 记录主要便是域名与 IP 地址这两个信息，所以格式为：

```bash
hostname IN A IP-address

# 该记录表示访问 www.test-shiyanlou.com 解析为 10.29.113.73
www IN A 10.29.113.73

# 该记录表示访问 test.test-shiyanlou.com 解析为 10.29.113.73
test IN A 10.29.113.73
# 等同于上一条记录
test.test-shiyanlou.com. IN A 10.29.113.73
```

从上述的例子中我们便可看出在写 hostname 是可以省略区域名 `test-shiyanlou.com.` 的部分，这是因为配置文件中通过 zone 去声明这个域，所以在其他记录中也是同样的效果。

2.NS 记录指出该域名的 NS 服务器，其格式为：

```bash
IN NS nameserver-name
```

例如：

```bash
IN NS dns1

# 等同于
test-shiyanlou.com. IN NS dns1.test-shiyanlou.com.

# 但若是这样指定必须指出 dns1 的 A 记录
dns1 IN A 10.29.113.73
```

3.MX 记录为 Mail Exchange 的缩写，也就是邮件记录：

>注意：
>- 邮件记录的域名必须是使用 fully qualified domain name (FQDN)，也就是完整的域名，不能像之前那般省略
>- 邮件增加了一个 `preference-value` 值来指定该记录的优先级

```bash
IN MX preference-value email-server-name

# 例如优先级为 5 的 mail.test-shiyanlou.com. 
test-shiyanlou.com. IN MX 5 mail.test-shiyanlou.com.

# 当然该记录的相关 A 记录必须存在
mail IN A 10.29.113.73
```

4.PTR 记录为反向解析记录，将 IP 翻译成域名：

```bash
# 格式为
last-IP-digit IN PTR FQDN-of-system

# 例如在 113.29.10.zone 配置文件中，named 中我们声明了，所以可省略 113.29.10-in-addr.arpa
73  IN PTR localhost.

# 完整表示为
73.113.29.10-in-addr.arpa. IN PTR localhost.
```

因为我们此处是配置的小网段，只用配置一位，若是面对大网段则写两位、三位即可。

5.CNAME 为机器的别名

```bash
# 其格式为
alias-name IN CNAME real-name

# 例如
server1  IN  A      10.0.1.5
www      IN  CNAME  server1
```

在上述例子中表示 server1.test-shiyanlou.com 将会被解析到 `10.0.1.5`，而 `www.test-shiyanlou.com` 会被解析到 `server1.test-shiyanlou.com` 也就是 `10.0.1.5`

6.SOA 记录表示的是域管理服务器

除了一些命令放在 zone file 的开头，所有记录的第一条记录应该为 SOA 记录，它的格式如下：

```bash
@  IN  SOA  primary-name-server hostmaster-email (
       serial-number
       time-to-refresh
       time-to-retry
       time-to-expire
       minimum-TTL )
```

(1)其中 @ 表示的是本机意思，等同于 `$ORIGIN` 命令，而 `$ORIGIN` 代表的值为我们在声明中所自定义域的名字。

例如在 `test-shiyanlou.zone` 文件中其 `$ORIGIN` 为 `test-shiyanlou.com.`。
例如在 `113.29.10.zone` 文件中其 `$ORIGIN` 为 `113.29.10.in-addr.arpa.`。

当然我们可以在配置文件的开头指定该值：

```bash
$ORIGIN test-shiyanlou.com.
```

(2)其中 `primary-name-server` 表示的是主要的域名服务器的域名，例如 `dns.test-shiyanlou.com.`

(3)其中 `hostmaster-email` 表示的是建立服务器的负责人邮件地址，因为 @ 在此处有特殊意义所以不能直接使用，需要用 `.` 代替，例如 `shiyanlou@qq.com` 需要改成 `shiyanlou.qq.com`

(4)其中 `serial-number` 是指定的一个同步序列号，每当 zone file 变化的时候该值便会递增，slave 便知道该同步，该重新加载 zone file 了

(5)其中 `time-to-refresh` 是指 slave 多久会主动检查 serial 的值，以便更新 zone file

(6)其中 `time-to-retry` 是指 slave 若是连接 master 失败，多久之后会再次尝试

(7)其中 `time-to-expire` 是指 slave 若是一直连接 master 失败，多久之后会放弃

(8)其中 `minimum-TTL` 是指定最小清除 cache 的时间

>注意：时间的单位都是秒

例如在 `test-shiyanlou.com` 的 zone file 中就是这样定义的：

```bash
@  IN  SOA  dns1.test-shiyanlou.com. admin.test-shiyanlou.com. (
       2017122704
       21600
       3600
       604800
       86400 )
```

该记录表示的是:

- `dns1.test-shiyanlou.com.` 是本域的主要域名服务器
- 主要的管理人员的邮箱是 `admin@test-shiyanlou.com`
- 初始序列号为 `2017122704`
- 每 6 小时主动检查一次 zone file 的序列号
- 每 1 小时重连一次，若是连接失败的话
- 一周都连接失败则放弃连接
- 每天清除一次缓存

这便是我们常用的一些记录，当然还有 `AAAA` 可以解析 IPv6 的地址，还有 `TXT`、`HINFO` 等等记录，大家可以在后期掌握之后详细了解。

在掌握了如何写 zone file 之后我们便可来修改两个域信息文件：

首先修改 `test-shiyanlou.zone` 文件：

>注意：在 zone file 中 `;` 是注释

```bash
# 编辑文件
vim /var/named/test-shiyanlou.zone

# 添加如下内容

@  IN  SOA  dns1.test-shiyanlou.com. admin.test-shiyanlou.com. (
       2017122704
       21600
       3600
       604800
       86400 )

; 在上文中提到果 @ 代表本机，可忽略，可写出来
@    IN    NS    dns1
@    IN    NS    dns2

; MX 记录
@    IN    MX    10    mail.test-shiyanlou.com.

;A 记录
dns1    IN    A    10.29.113.73
dns2    IN    A    10.29.113.74
www     IN    A    10.29.113.73
ftp     IN    A    10.29.113.73
mail    IN    A    10.29.113.73
api     IN    A    10.29.113.75

; CNAME
staging    IN    CNAME    www
devlop     IN    CNAME    www

```

紧接着我们来创建反向解析记录的域文件：

```bash
vim /var/named/113.29.10.zone

@  IN  SOA  dns1.test-shiyanlou.com. admin.test-shiyanlou.com. (
       2017122704
       21600
       3600
       604800
       86400 )

    IN    NS    dns1.test-shiyanlou.com.
    IN    NS    dns2.test-shiyanlou.com.

;注意域名需要 FQDN
73    IN    PTR    www.test-shiyanlou.com.
73    IN    PTR    ftp.test-shiyanlou.com.
73    IN    PTR    ftp.test-shiyanlou.com.
75    IN    PTR    api.test-shiyanlou.com.
```

### 4.4 配置检查

经过上述过程我们完成了 bind9 的主要服务配置文件 `/etc/named.conf` 的修改，与域文件 `test-shiyanlou.zone`、`113.29.10.zone` 的编写。

#### 4.4.1 检查服务配置文件

我们可以通过 `named-checkconf` 检查我们的服务配置文件是否有误：

```bash
named-checkconf
```

若是没有错误就没有任何的信息返回，若是有误则输出错误的附近行数，例如我将上述配置文件中的引号删除一个：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514327234769.png/wm)

然后使用该命令：

```bash
named-checkconf
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514325119623.png/wm)

我们将错误修正回来。

#### 4.4.2 检查 zone 配置文件

我们可以通过 `named-checkzone` 检查 zone file 是否有错误：

例如检查反向解析配置文件，第一个参数是 named.conf 中的 zone 名称，第二个参数是 zone 文件的路径：

```bash
named-checkzone 113.29.10.in-addr.arpa /var/named/113.29.10.zone
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514325392105.png/wm)

我们可以看到配置无误便会输出 `OK` 的字样。

同样我们可以制造一些错误来看看其输出，同上会提示你错误的行数。同学们可以试试将域名 `dns1.test-shiyanlou.com.` 改成 `dns1` 看看检查的结果。

DNS 服务器配置操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week4/2-2.mp4
@`

## 5 验证

通过上述过程我们完成了 DNS 的搭建与配置，通过 `systemctl restart named` 来重启 named 使得配置文件生效。

使用 `ps -ef |grep named` 与 `netstat -lunapt |grep 53` 确保 named 是否成功启动

然后修改 `/etc/resolv.conf`：

```bash
vim /etc/resolv.conf
```

`resolv.conf` 用于配置我们使用的 DNS 服务器，因为是按顺序读取的配置，所以我们将如下添加在第一行：

```bash
nameserver 127.0.0.1
```

我们将通过这样三个工具来验证我们的 DNS 配置是否成功：

- host
- dig
- nslookup

首先需要安装这些工具：

```
yum install bind-utils
```

### 5.1 host 验证

host 命令常用的有两个参数

- `-a`:通过完整的域名列出该域名的所有信息

例如：

```bash
host -a api.test-shiyanlou.com
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514326110139.png/wm)

- `-l`:通过域名列出所有的子域名与对应关系

例如：

```bash
host -l test-shiyanlou.com
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514326275323.png/wm)

### 5.2 dig 验证

例如：

```bash
dig www.test-shiyanlou.com
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514326536727.png/wm)

在使用 `-t` 参数时主要是指定我们要查看的记录类型，例如查看 NS 记录：

```bash
dig -t ns test-shiyanlou.com
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514327528857.png/wm)

通过 `-x` 参数查看泛解析的记录：

```bash
dig -x 10.29.113.75
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514327630054.png/wm)

### 5.3 nslookup 验证

nslookup 也是一个非常常用的命令:

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514327772637.png/wm)

DNS 服务器验证操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week4/2-3.mp4
@`

## 6. chroot 使用

为了更加的安全，我们将使用 `bind-chroot` 来实现，首先关闭运行中的 named：

```bash
systemctl stop named
```

然后通过这个脚本开启 bind-chroot 的环境：

```bash
/usr/libexec/setup-named-chroot.sh /var/named/chroot on
```

`/var/named/chroot` 是我们要 chroot 的目录，执行该脚本时同时也会帮我们将配置文件移动到相应的目录中。

紧接着启动 chroot：

```bash
systemctl start named-chroot
```

此时使用的就是 chroot 的环境：

```bash
ps -ef |grep named
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4128timestamp1514328734072.png/wm)

named 运行的时候它的根目录 `/` 实际上是 `/var/named/chroot`

它读取的 `/etc/named.conf` 配置文件实际上是 `/var/named/chroot/etc/named.conf`。

### 7. 总结

DNS 配置上还有很多可配置的东西，例如日志的种类筛选、还有黑/白名单的配置、DNSSEC、TXT 记录等等，有兴趣继续深入研究的同学可查看推荐阅读帮助自己理解与实践。欢迎大家在 QQ 中与我们讨论、交流。