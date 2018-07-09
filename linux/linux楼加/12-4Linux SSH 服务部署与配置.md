---
show: step
version: 1.0
enable_checker: true
---
# Linux SSH 服务部署与配置

## 1. 实验介绍

#### 1.1 实验内容

在本节内容中，我们将会给大家简单描述 `SSH` 的原理，并且尝试 `SSH` 的几种验证方式。

#### 1.2 实验知识点

+ 基于主机的身份验证

+ 密码验证

+ 公钥验证

#### 1.3 推荐阅读

由于在 `SSH` 的连接过程中会涉及到有关密钥，签名等一些安全方面的知识，但是在实验内容中并不会涉及太多这方面的有关内容，而要对 `SSH` 的原理有一定的了解，必然绕不开这些知识，所以推荐大家阅读下列文章：

+ [数字签名是什么](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)

+ [非对称加密](https://www.zhihu.com/question/33645891/answer/57721969)

## 2. SSH 概述

#### 2.1 SSH 协议

`SSH` 的全称是 `Secure Shell`。可以通过其对远程主机进行安全的访问。在最开始设计的时候，`SSH` 被设计为替代 `telnet` 以及一些其它不安全远程 `shell` 协议。

`SSH` 基于 `C/S` 架构，即 **客户端/服务器**。通过客户端与服务器相连，进行安全的远程访问。

#### 2.2 OpenSSH

对于 `SSH` 协议有很多的实现。而最常用的开源实现为 `OpenSSH`。在很多操作系统中默认使用，例如在 `Ubuntu` 和 `CentOS` 中默认使用 `OpenSSH`。在下面的安装和配置的操作过程也将基于 `OpenSSH` 进行操作。

#### 2.3 SSH 的体系结构

根据 [RFC4251](https://tools.ietf.org/html/rfc4251) 关于 `SSH` 体系结构的定义。`SSH` 协议主要由三个组件组成：

+ 传输层协议（`SSH-TRANS`）：该协议提供服务器身份验证，隐私和具有完美转发隐私的完整性。传输层通常是通过 `TCP/IP` 连接运行。

+ 用户认证协议（`SSH-USERAUTH`）：验证连接到服务器的客户端用户。运行在传输层的协议之上。

+ 连接协议（`SSH-CONNECT`）：将加密隧道复用到多个逻辑通道。运行在用户认证协议上。

在 `SSH` 的三个组件中的 `SSH-USERAUTH`，用户认证协议。对于用户的身份认证，在 [RFC4252](https://tools.ietf.org/html/rfc4252) 中，有相关的身份认证方式的描述。在本节内容中，我们将带领大家配置以下几种身份认证方式：

+ 基于主机的身份验证（`hostbased`）

+ 使用密码 （`password`）

+ 公钥的认证方式（`public key`）

## 3. 服务的搭建与配置

下面我们将会学习服务的搭建与配置。

### 3.1 安装与启动

在实验环境中已经有安装相应的组件，若是需要在本机安装相应的内容，可以参考我下面的命令。

在 `Centos` 上，使用 `yum` 进行安装：

```bash
# 服务端
$ sudo yum install openssh-server

# 客户端
$ sudo yum install openssh-clients
```

在 `Ubuntu` 上，使用 `apt` 进行安装：

```bash
# 服务端
$ sudo apt-get install openssh-server

# 客户端
$ sudo apt-get install openssh-client
```

为避免服务未启动，可以手动尝试开启服务，使用如下命令：

```bash
$ sudo service sshd start
```

最后，我们可以将要修改的配置文件进行一个备份，以便还原初始值，使用如下命令：

```bash
# 备份服务器端的配置文件
$ sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.back

# 备份客户端的配置文件
$ sudo cp /etc/ssh/ssh_config /etc/ssh/ssh_config.back
```

### 3.2 配置文件

在安装好相应的软件包之后，会在 `/etc/ssh` 文件夹下生成一些相应的配置文件，下面我们简单说明：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514118804910.png/wm)

- `ssh_config` 为客户端的配置文件，
- `sshd_config` 为服务端的配置文件,
- 其它的文件属于服务端安装时自动生成的密钥文件，以 `.pub` 结尾的为公钥，对应的不以 `.pub` 结尾的为私钥文件。

在启动服务的时候，会加载相应的相应的配置文件。对应的还可以在用户目录下的 `~/.ssh/` 目录中编辑配置文件，例如编辑 `~/.ssh/config` 文件，则用户在启动客户端时，就会读取 `~/.ssh/config` 文件的配置项而不是 `/ect/ssh/ssh_config` 针对全局的配置。

在这里我们并不会介绍有关于各种加密算法的内容，需要大家自行去了解相关的知识。

### 3.3 信任模式

如上所示，在安装完服务器端之后，会有相应的密钥对文件（一个公钥一个私钥）生成，由于 `SSH` 支持多种不同的加密算法，所以可能会根据实际情况会有多个文件被生成。

每个服务器都有一个密钥，由于使用不同的算法可以拥有多个密钥。在密钥交换过程中使用服务器主机密钥来验证客户端是否与正确的服务器交谈，为了做到这一点，客户必须先知道服务器的公共主机密钥（公钥），在 [RFC4251](https://tools.ietf.org/html/rfc4252) 中给出了两种解决的思路或者说信任模式：

1. 客户端在本地保存主机名与对应的服务器的公钥信息。

2. 主机关联的公共密钥由受信任的证书机构进行认证（如 CA）

但是对于上述的两种方式来说，第 1 种需要一定的人力维护成本，而第 2 种，对于现在互联网上无数的服务器来说，明显是不可行的。

因此对于 `SSH` 协议而言，在首次连接到服务器时，允许事先通信而不需要密钥或证书，然而这一过程容易受到**中间人攻击**。

### 3.4 hostbased

hostbased 方式允许受信任主机上的指定用户无需输入密码即可直接登录进服务器。服务器会读取配置的可信任主机的信息，一般为 `/etc/hosts.equiv`。

在默认设置中，`SSH` 并未启用该验证方式，我们需要修改 `/etc/ssh/sshd_config` 的配置文件：

```bash
HostbasedAuthentication yes
```

上述编辑的 `HostbasedAuthentication` 可以选择设置是否启用基于主机的身份验证。将会从全局的 `/etc/hosts.equiv` 文件中匹配指定的主机和用户，以便允许这些主机上的这些用户无密码登录。

除了编辑服务器端的配置文件之外，我们还需编辑客户端的配置文件，这里我们编辑 `~/.ssh/config`，则仅针对当前用户的客户端配置文件 ：

```bash
# localhost 属于别名，未指定时只能使用 IP 地址
Host localhost
    HostName 127.0.0.1
    EnableSSHKeysign yes
    HostbasedAuthentication yes
```

注意，如果 `~/.ssh/config` 文件之前不存在，编辑完成之后需要修改其读写权限为 600 ，命令为 `chmod 600 ~/.ssh/config`，否则会报权限错误。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514337806194.png/wm)

另外全局的客户端配置文件 `/etc/ssh/ssh_config` 也需要按如下修改：

```bash
Host *
    GSSAPIAuthentication yes
    EnableSSHKeysign yes
    HostbasedAuthentication yes
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514120334963.png/wm)

上述图片中的 `Host *` 代表的是所有的服务器，就是该配置项下的子项针对所有的服务器。

最后还需要编辑 `/etc/hosts.equiv` 文件（不存在需要新创建）。该文件的格式为：

```bash
<IP ADDRESS> <USERNAME>
```

我们在该文件中添加一行内容，允许本机的 shiyanlou 用户无密码登录本机：

```bash
127.0.0.1 shiyanlou
```

重新启动服务器：

```bash
$ sudo service sshd restart
```

最后，我们可以使用 `ssh` 连接到本地的 `SSH` 服务器上，使用如下命令：

```bash
$ ssh shiyanlou@localhost
```

这里需要说明的是，使用 `ssh` 命令连接远程主机的格式为 `ssh [user@]hostname`，而 `SSH` 服务默认运行在 `22` 端口上，若是配置了使用其它端口，可以通过 `-p` 参数指定端口号。

我们可以看到显示结果如下图，对于初次连接服务器时，会检测是否已经有相应的记录（检测 `known_hosts` 文件），若是并未有相应的记录，则会提示是否继续（即在上述信任模式种提到的初次连接时 `SSH` 允许不需要预先知晓服务器的公共主机密钥）。如果选择继续则代表信任该服务器提供的公钥，并会将其保存到 `~/.ssh/known_hosts` 文件中。可以通过在 `/etc/ssh/sshd_config` 文件中设置 `IgnoreUserKnownHosts yes` 不进行检测。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514124743755.png/wm)

这时我们可以输入 `yes`，则会提示保存该公钥并且登陆成功，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514164086827.png/wm)

此时，我们可以查看 `~/.ssh/known_hosts` 文件的内容，与刚刚的提示信息进行比对，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514127592008.png/wm)

这里可以看到主机名与对应的公钥的信息，由于提示信息显示的公钥是经过 `HASH` 算法计算过的值，所以会与文件中的内容不一致，在下面的内容中，我们将会验证两者为同一内容。

这里需要说明的是，**由于我们是通过客户端直接连接当前的服务器端，即 `localhost`，在进行登陆时，登陆成功后不会有太明显的区别。有条件的同学可以在自己电脑上尝试通过客户端连接到服务器端，或者在进行登陆之前切换目录，不要停留在 `shiyanlou` 用户的家目录，因为登陆成功之后，会默认切换到被登陆用户的家目录。**

如下所示，我们首先使用 `exit` 命令退出登陆，然后切换到 `Code` 目录，再次尝试登陆，可以看出明显的区别，即登陆成功之后会切换到到 `/home/shiyanlou` 目录，退出登陆会回到 `~/Code` 目录下，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514164809455.png/wm)

### 3.5 password

`SSH` 默认使用的即是密码验证的方式，即在 `/etc/sshd_config` 配置文件中的 `PasswordAuthentication` 配置项默认为 `yes` ，如下所示，我们查看相关的配置文件：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514165030617.png/wm)

由于刚刚我们配置了基于主机的身份验证，所以需要将之前的设置重新还原，可以手动修改配置，也可以使用我们刚刚备份的配置文件，或者直接删除掉配置的 `/etc/hosts.equiv` 文件即可。

这里我们删除刚刚配置的 `/etc/hosts.equiv` 文件，并且停止 `sshd` 服务，新打开一个终端，使用调试模式启动 `sshd`，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514165908100.png/wm)

此时，我们使用 `ssh shiyanlou@localhost` 去连接服务器端，由于已经删除掉 `/etc/hosts.equiv` 配置文件，会提示基于主机的身份验证失败，需要输入密码，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514166100436.png/wm)

此时，输入相应的密码即可成功连接，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514166299425.png/wm)

注意，实验环境暂不支持密码登录，这里的截图仅做参考。

### 3.6 public key

最后我们要介绍的验证方式为 `public key` 即公钥认证，该方式属于上述讨论的信任模式中的第一种实现方式，即通过手动在客户端和服务端配置相应的密钥对，客户端保存私钥，服务器端保存公钥信息。通过公私钥的认证，能够保证客户端是与真正的服务器通信，并且客户端和服务器端互相信任，不需要进行密码认证。

这里所使用的公钥认证的过程与传统意义上公私钥认证方式没有什么不同。具体的原理大家可以参考我在之前给出的链接内容。

对于 `SSH` 支持的加密算法，我们可以通过 `ssh -Q key` 来查询，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514169448711.png/wm)

下面我们创建一个密钥对，用于配置 `public key` 认证方式，使用如下命令：

```bash
# 直接使用 ssh-keygen 会默认使用 `rsa` 算法，并且将生成的密钥文件放到 ~/.ssh/ 目录下
$ ssh-keygen

# 还可以使用 -t 参数指定使用的算法类型， rsa | ecdsa| ed25519 | dsa 等，上述命令等同于下列命令
$ ssh-keygen -t rsa
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514171244903.png/wm)

所有提示要输入的地方直接回车即可。生成的公私钥文件，分别保存在 `~/.ssh/id_rsa`，`~/.ssh/id_rsa.pub` 中。

`SSH` 默认启用了 `public key` 的认证方式，不需要进行配置，我们直接通过下图查看需要设置的相关配置项，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514171579068.png/wm)

我们需要将 `~/.ssh/id_rsa.pub` 的公钥信息追加到 `~/.ssh/authorized_keys` 文件中，可以在其中添加多个公钥。

使用如下命令：

```bash
$ cat ~/.ssh/id_rsa.pub | tee -a ~/.ssh/authorized_keys
```

如果 `~/.ssh/authorized_keys` 是首次创建，需要修改其文件权限为 600，命令为 `chmod 600 ~/.ssh/authorized_keys`。

此时，可以直接使用 `ssh shiyanlou@localhost` 进行验证，就能够直接进行登陆而不需要输入密码：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514182279798.png/wm)

在客户端，也可以使用 `-i` 参数，指定要使用的私钥文件，使用如下命令：

```bash
$ ssh -i ~/.ssh/id_rsa shiyanlou@localhost
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514190406671.png/wm)

SSH 使用公私钥进行登录操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week4/7-1.mp4
@`


## 4. 其它配置项

对于 `SSH` 而言，还有很多其它的可用配置项，针对于 `sshd_config` 的配置项如下：

```bash

# 可以自定义端口，默认为 22，在客户端使用 `ssh` 连接服务端时，可以通过 -p 参数指定端口
Port 22

# 允许 root 用户使用 ssh 登陆，可以设置为 yes ，no ，prohibit-password，without-password  后面两个都代表允许登陆而不使用密码
PermitRootLogin yes

# 指定允许组内的用户登陆，可以指定多个组，以空格分隔。
AllowGroups shiyanlou mysql

类似还有 DenyUsers   AllowUsers   DenyGroups  等，它们的优先级顺序从高到低为 DenyUsers,AllowUsers,DenyGroups,AllowGroups。

# 指定应该监听的本地地址，可以指定多个，默认为监听正在使用的路由域上的所有本地地址
ListenAddress hostname:port
```

## 5. scp 和 sftp

这里我们将会简单介绍的 `scp` 和 `sftp` 两个传输文件的工具。 

### 5.1 scp

`scp` 可以在多个主机之间复制文件，使用 `SSH` 传输数据，并且使用它的认证方式进行身份认证。

`scp` 的使用方式为：

```bash
scp source target
```

可以将源(`source`) 指定为本地路径名，将目标（`target`）指定为远程主机，代表将本地的文件复制到远程主机。

也可以将源指定为远程主机，将目标指定为本地路径，代表的是将远程主机上的文件复制到本地。

对于远程主机的格式为 `[user@]host:[path]`，或者 `scp://[user@]host[:port][/path]`。

本地路径则使用绝对路径或相对路径即可。

如下所示，我们分别在 `/home/shiyanlou` 目录下创建一个 `test1` 文件，在 `/home/shiyanlou/Code` 目录下创建一个 `test2` 文件，并使用 `scp` 复制文件：

```bash
# 创建文件
$ touch /home/shiyanlou/test1  /home/shiyanlou/Code/test2

# 切换到 /home/shiyanlou 目录
$ cd ~

# 将当前目录下的 test1 上传到远程服务器上的 /home/shiyanlou/Code 目录下
$ sudo scp test1 shiyanlou@localhost:/home/shiyanlou/Code

# 将远程服务器上的 /home/shiyanlou/Code 目录下的 test2 复制到本地
$ sudo scp shiyanlou@localhost:/home/shiyanlou/Code/test2 /home/shiyanlou
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514192364886.png/wm)

如上所示，我们成功进行了文件的复制。`scp` 使用的是 `SSH` 进行数据的传输和身份的验证，在之前的内容中，我们配置了 `public key` 的认证方式，所以能够直接复制成功，而不需要输入密码等验证操作。

除此之外，`scp` 还有一些常用的参数，例如 `-r` 递归复制目录，`-p` 指定端口等。

### 5.2 sftp

跟 `scp` 类似的还有 `sftp`，它 **通过SSH** 来安全的上传和下载文件，是常用的文件传输工具。关于`sftp` 详细的使用方法与 `ftp` 类似，可以参考前面关于 `ftp` 的章节。

简单示例如下：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1514271324988.png/wm)

同样，上述操作也是由于在刚刚的 `SSH` 身份验证我们启用了公钥验证并设置相关的内容，所以可以直接连接而不需要输入密码。

scp 与 sftp 操作介绍视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week4/7-2.mp4
@`

## 6. 总结

在本节实验，我们学习了有关于 `SSH` 服务的知识。但是对于相关的工具和配置项来说，并不算太多，若是愿意继续深入了解 `SSH` 的内容，可以到 `openssh` 的官网 http://www.openssh.com/ 查找相关的资料，并结合实验楼的实验环境进行实践学习。