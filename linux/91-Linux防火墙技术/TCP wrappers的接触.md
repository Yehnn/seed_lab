---
show: step
version: 0.1
enable_checker: true
---

# TCP wrappers

## 一、实验介绍

TCP wrappers 是 Linux 中两大防护措施的其中之一，这层保护通过定义哪些主机允许或不允许连接到网络服务上来实现。TCP Wrappers为多种不同的服务提供访问把关，通过本实验我们将初试TCP wrappers的使用。

#### 实验涉及的知识点

- Tcp wrappers的认识
- Tcp wrappers的使用

## 二、Tcp wrappers的认识

Tcp Wrapper 是由 Wietse Venema 编写的，其创作出这个程序的主要原因是监视 Eindhoven 大学数学和计算机科学系的Unix工作站上的黑客活动，Wietse Venema一直维护这个程序到1995年；2001年6月1日，在其自己的BSD风格的许可证下发布。

Tcp Wrapper 是一个基于主机的网络访问控制的程序，用于过滤对类 Unix 系统（如 Linux 或 BSD ）的网络访问。

当一个系统或者主机在公开网络中充当服务器时,这个系统就可能成为被攻击的目标.对网络服务进行访问控制 是一件很重要的事情。像 Telnet、SSH、FTP、POP 和 SMTP 等很多服务都会用到 TCP Wrapper,它被设计为一个介于外来服务请求和服务回应的中间处理

它的基本过程是这样的：当服务器或主机接收到一个外来服务请求的时候，先由TCP Wrapper 来对请求进行分析处理，TCP Wrapper 会根据请求所需要的服务和对这个服务所设定的规则来判断请求方是否有权限，如果有，TCP Wrapper 会把这个请求按照配置文件所设定的规则转交给相应的守护进程去处理，同时记录这个请求动作。

TCP Wrappers虽然能对 TCP 协议的报文做过滤动作，但是并不是所有的 TCP 协议报文 TCP Wrappers 都可以过滤掉。只有该服务链接到TCP Wrappers的函式库才可以使用TCP Wrappers进行报文过滤。

因为 TCP Wrapper 的功能主要来自于 libwrap.a这个静态库，它是一个服务库。像 xinetd、sshd 和 portmap 等许多服务编译时都依赖于 libwrap.so 这个动态链接库，其他的网路服务甚至你自己编写的服务都可以加上这个编译选项来提供 TCP Wrapper 的功能。

我们可以使用这样一个命令来确定我们要过滤的服务是否链接到了libwrap 这个函数库中

```
ldd /path/service_name |grep libwrap

#例如
ldd /usr/sbin/sshd | grep libwrap

#若是不知道程序的所在位置可以这样
ldd $(which sshd) | grep libwrap

```

![2-2-1](https://dn-simplecloud.shiyanlou.com/1135081470122734066-wm)

也许我们会发现有些程序并没有链接到这个动态链接库但是 Tcp wrappers 也可以对他们执行过滤的审核，这是因为可能该程序将该函数库编译进程序中了

## 三、Tcp wrappers的使用

要决定一个客户是否被允许连接一项服务,TCP Wrappers会参考以下两个文件：

- /etc/hosts.allow
- /etc/hosts.deny

当一个TCP Wrappers接收到一个客户请求时,它会运行以下几个步骤:

![2-3-1](https://dn-simplecloud.shiyanlou.com/1135081470124912676-wm)
（此图的来源：http://images.slideplayer.com/12/3366547/slides/slide_75.jpg）

1. 检查 `/etc/hosts.allow`.TCP会绕服务会循序地解释/etc/hosts.allow文件并应用第一个为这个服务所指定的规则.如果找到了一个匹配的规则,则允许连接.如果找不到匹配的规则,就会进行下一个步骤。
2. 检查 `/etc/hosts.deny`.TCP会绕的服务会循序地来解释/etc/hosts.deny文件.如果找到一个匹配的规则,则拒绝这个连接.如果找不到匹配的规则,则允许连接到这个服务。
3. 若是没有匹配的规则便进入服务

 在使用 TCP Wrappers 保护网络服务时应该考虑以下几个要点:

- hosts.allow 或 hosts.deny 的配置改变会立即生效。
- 两个规则文件是有读取顺序的，hosts.allow中的规则会先被读取应用,hosts.deny中的规则后读取,如果访问一项服务在hosts.allow中设置为允许,那么在hosts.deny中同一项服务的拒绝访问设置则不会生效
- 规则文件内的读取也是有顺序的，每个文件中的各项规则是由上到下被读取的,第一个匹配的规则生效。因此,规则的排列顺序也很重要。

我们可以通过下图看到，hosts.allow 的配置文件，默认的文档给予了我们一些例子

![2-3-2](https://dn-simplecloud.shiyanlou.com/1135081470125218573-wm)

通过例子我们可以看出这个配置文件是有语法格式的。首先空行或以井号(#)开始的行会被忽略

每条规则都使用以下基本格式来对网络服务的访问进行控制:

```
<daemon list>: <client list> [: <option>: <option>: ...]

#动作可有可无
服务程序列表： 客户机地址立标 [:执行的动作]
```

列表中存在多个服务或者主机的时候用 `,`逗号分隔开。

例如我们要TCP Wrappers监测 10.3.1.1 这台主机向我们的 ssh 的守护进程发出链接，就拒绝掉，那么我们就把下面这条命令写在 hosts.deny 这个文件中

```
sshd : 10.3.1.1
```

我们来做这么一个实验来证明 Tcp wrappers 是起作用的

首先我们可以看到本地的 ssh 服务是开启的，并且实验能够正常地使用 ssh

```
#我们可以看到 ssh 的守护进程正在运行中
ps -ef | grep sshd
```

![2-3-3](https://dn-simplecloud.shiyanlou.com/1135081470127027311-wm)

```
#尝试连接我们本地的22号端口，也是成功的
telnet localhost 22
```

![2-3-4](https://dn-simplecloud.shiyanlou.com/1135081470127060208-wm)

```
#这是最直接的证明方式，直接ssh我们的本地，密码可以通过旁边工具栏的ssh直连得到
ssh shiyanlou@localhost

#然后我们可以通过top的第一栏看到，当前存在的用户有两个，说明我们成功登入了
top
```

![2-3-5](https://dn-simplecloud.shiyanlou.com/1135081470127324706-wm)

用了这么多的方式我们证明了，ssh 是没有问题的。然后我们通过这个命令将规则写入 hosts.deny中

```
sudo vim /etc/hosts.deny 

#这是需要写入的规则
sshd localhost
```

![2-3-6](https://dn-simplecloud.shiyanlou.com/1135081470127539192-wm)

这时候我们再来尝试连接 ssh 服务，就用以上的三种方式，可以看到 sshd 并没有停止服务，还在运行中，但是 telnet、ssh 都没有办法连接上去

![2-3-7](https://dn-simplecloud.shiyanlou.com/1135081470127679340-wm)

若我们将 hosts.deny中的规则删除掉亦或者是注释掉，再尝试一次肯定是没有问题的。

在 hosts.deny 与 hosts.allow 的语法是支持通配符的，通配符有以下这些

| 通配符   | 作用                                            |
| -------- | ----------------------------------------------- |
| ALL      | 完全匹配,可以用在守护进程列表和客户列表中       |
| LOCAL    | 与任何不包括点(.)的主机匹配,也就是localhost本机 |
| KNOWN    | 与任何带有已知主机名和主机地址匹配              |
| UNKNOWN  | 与任何带有未知主机名和主机地址匹配              |
| PARANOID | 与任何带有主机名和主机地址不相匹配的主机匹配    |

比如还是上面的例子

![2-3-8](https://dn-simplecloud.shiyanlou.com/1135081470128797160-wm)

上文提到过，我们的规则是有读取顺序的，如我们在 hosts.allow 中加入 `ssh: LOCAL` 我们会发现 ssh 会是成功的，hosts.deny 中的那条规则对本机并没有生效。

![2-3-9](https://dn-simplecloud.shiyanlou.com/1135081470128908606-wm)

模式可以用在访问规则的客户领域里,能够更有效率地匹配我们想要匹配的用户，这里只列举常用的方式，需要更深入学习的可以使用 man 命令查看

1. 主机名以点(.)开始,如果在一个主机名的开始放置一个圆点,那么就与所有共享这个主机名中列出的相同组成部分的主机匹配.如: `.simplecloud.com` 适用于匹配 `simplecloud.com` 域名内的所有主机.
2. IP地址以点(.)结束,如果在一个IP地址的末尾放置一个点,那么就与最后一个点前同一个网段的所有主机匹配.如: `192.168.` 适用于 `192.168.x.x` 网络内的任何主机.

![2-3-10](https://dn-simplecloud.shiyanlou.com/1135081470130237195-wm)

3. `IP地址/子网掩码`,这样的格式来控制某个网段的地址的访问.如: `192.168.1.0 /255.255.255.0` 适用于地址区间从`192.168.1.0` 到 `192.168.1.255` 的所有主机.

4. [IPv6地址] 的地址也是可以读取的

5. 星号(*)，星号与shell中的通配符作用相同

我们还可以使用这个动作来执行一些 shell 命令

| 动作  | 作用                             |
| ----- | -------------------------------- |
| spwan | 开启一个shell执行你指定的命令    |
| twist | 替换访问者的请求成我们指定的命令 |

如图中，我们若是在 hosts.deny 中这样写，会有这要的反馈效果，我们可以了解到是谁试图攻击我们，想登陆我们的主机了，然后可以封掉他的 IP。

![2-3-11](https://dn-simplecloud.shiyanlou.com/1135081470132610055-wm)

同样我们可以尝试一下 twist 的使用，在  /etc/hosts.deny 中写下这个命令：

```
sshd : localhost : twist /bin/echo "Hello，attacker，You are prohibited from accessing this service!!"
```

然后我们用 telnet 来尝试链接他，会得到这样的结果：

![2-3-12](https://dn-simplecloud.shiyanlou.com/1135081470147349286-wm)


在其中我们可以用到这样的一些参数，这里只列举了常用的一些参数：

| 参数 | 作用           |
| ---- | -------------- |
| %a   | 客户端的ip地址 |
| %A   | 服务端的ip地址 |
| %d   | 守护进程的名字 |
| %h   | 客户端的主机名 |
| %H   | 服务端的主机名 |
| %p   | 守护进程的pid  |
| %u   | 客户端的用户名 |

本实验只是粗略的了解与尝试 TCP wrappers，在实际的生产环境中 xinetd  与 TCP wrappers  是黄金搭档，也同样能够做到防止 DOS 攻击，与客户端交换，控制连接数等等的高级功能。若想更加地深入的学习，大家可以多看看 man，与网上查查资料。

```checker
- name: check twist
  script: |
    #!/bin/bash
    grep twist /etc/hosts.deny
  error: 没有配置 /etc/hosts.deny
```

## 四、实验总结

通过本实验我们初步的了解了Tcp wrappers 是一个什么样的服务，能帮助我们达到什么样的效果。


## 五、参考资料

【1】参考于：<http://www.aboutlinux.info/2005/10/using-tcp-wrappers-to-secure-linux.html>