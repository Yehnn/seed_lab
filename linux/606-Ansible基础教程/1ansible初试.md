---
show: step
version: 1.0
enable_checker: true
---
# Ansible 初试

## 1. 实验介绍

#### 1.1 实验内容

随着云技术的日渐成熟以及服务器数量的增多，对于运维的日常管理也就逐渐繁杂，因此越来越多的运维管理就趋向于自动化的方式。所以从本周开始将带着大家认识和学习几个常用的自动化运维的工具。

本节主要讲解的是 Ansible 工具，虽然 Chef、Puppet、SaltStack and Fabric（后面章节会讲） 等等这些都是比较流行的自动化运维管理工具，但是相较于 Ansible 来说要复杂得多，不过每个工具也是各有各的好处，这里我们就先来学习这个比较简单的一款自动化运维工具—— Ansible。

#### 1.2 实验知识点

+ Ansible 的简介

+ Ansible 的安装

+ Ansible 的配置

+ AD-HOC 临时命令

#### 1.3 推荐阅读

+ [Ansible 官方文档](http://docs.ansible.com/ansible/latest/intro.html)
+ [Ansible wiki](https://en.wikipedia.org/wiki/Ansible_(software))

## 2. Ansible 简介

#### 2.1 概述

Ansible 是一款基于 python 开发，能够实现了批量系统配置、程序部署、运行命令等功能的自动化运维工具。Ansible 主要是基于模块进行工作的，本身没有批量部署的能力，真正实现部署功能的是运行的模块。

#### 2.2 结构框架

和 Chef、Puppet 刚好相反，Ansible 使用的是无代理体系结构，这种体系结构可以通过防止节点轮询控制机器来减少网络开销。Ansible 提供的结果框架如下所示：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081470635637349-wm)
（此图来源<http://tekslate.com/tutorials/ansible/>）

+ Ansible ：运行在中央计算机上；

+ Connection Plugins ：连接插件，主要用于本地与操作端之间的连接与通信；

+ Host Inventory：指定操作的主机，是一个配置文件里面定义监控的主机；

+ Modules：核心模块、自定义模块等等；

+ Plugins ：使用插件来完成记录日志、邮件等功能；

+ Playbooks：执行多任务，通过 SSH 部署模块到节点上，可多个节点也可以单个节点。

Ansible 主要有两种类型的服务器：控制机器和节点。控制机器用于控制协调，而节点由控制机器通过 SSH 进行管理，并且控制机通过 `inventory` 来描述节点的位置。在节点的编排上，Ansible 通过 SSH 部署模块到节点上，模块临时存储在节点上，并以标准输出的 JSON 协议进行通信，从而在远程机上检索信息，发送命令等。

#### 2.3 特点

+ Ansible 是基于 python 开发而来，维护相对简单，同时开发库也比基于 Ruby （一种面向对象的程序设计的脚本语言）的运维工具要多。

+ Ansible 默认通过 SSH 协议进行管理。同时 Ansible 是基于 python 的一个模块（paramiko）开发的，遵循 SSH 协议，支持加密和认证的方式来进行远程服务器连接，因此 Ansible 不需要客户端和服务端。

+ Ansible 可以通过命令来简单执行一些任务，也可以通过 palybook （后面会讲）的配置脚本来执行复杂任务，同时 playbook 不用分发到远程，在本地就可以执行。

+ Ansible 中的 playbook 使用的是 Jinja2 （基于 python 的模板引擎），简单易学。

+ Ansible 基于模块工作，易于扩展，而模块可以用任何语言编写，并以标准输出的 JSON协议进行通信。

+ Ansible 是开源的软件，在 [GitHub](https://github.com/ansible/ansible) 上有公开的代码。

## 3. 安装

这里我们介绍在 Ubuntu 14.04 上安装 Ansible 的方法。其他环境的安装方法大家可以参考 [Ansible 官方安装手册](http://docs.ansible.com/ansible/latest/intro_installation.html)。

官方手册提供了多种的安装方法（如：通过 git 源码、使用 pip 安装等），这里我们使用源的方法来安装。

1.首先，需要更新软件包的信息以及安装通用的管理软件库的工具（software-properties-common）。

```bash
$ sudo apt-get update
$ sudo apt-get install software-properties-common
```

```checker
- name: check pkg
  script: |
    #!/bin/bash
      dpkg -l software-properties-common
  error: 没有安装 software-properties-common
```

2.安装了软件库管理工具后，就可以通过 `add-apt-repository` 命令来添加 `ansible` 的源，将 PPA 添加到系统中去。

> `PPA（Personal Package Archives）`，个人软件包档案，Ubuntu Launchpad 网站提供的一项源服务，允许个人用户上传软件源代码，通过 Launchpad 进行编译并发布的二进制 `deb` 软件包，这样使用者就可以便捷的安装最新版的软件。

```bash
$ sudo python3.4 /usr/bin/add-apt-repository ppa:ansible/ansible
```

注：由于 python 升级之后使用 `add-apt-repository` 会报错（ImportError：No module named 'apt_pkg'），这是因为在 `/usr/lib/python3/dist-packages` 中 `apt_pkg` 的连接只有 `python3.4`，所以为了避免因为版本的问题这里我们就特别的指定 python 的版本信息。

![实验楼](https://dn-simplecloud.shiyanlou.com/2767331516098353036-wm)

3.最后需要更新一下软件包的信息，以便了解 PPA 中可用的包，然后安装 `ansible` 软件即可。

```
$ sudo apt-get update
$ sudo apt-get install ansible
```

```checker
- name: check pkg
  script: |
    #!/bin/bash
	ansible --version
  error: 没有安装 ansible
```

4.验证一下 `Ansible` 是否安装成功以及版本信息

```
$ ansible --version
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1997timestamp1516168235701.png/wm)

Ansible 安装完成后，不会添加数据库，也不会有守护进程启动或继续运行。你只需要把它安装在至少一台机器上，它可以从该中心点来管理远程机器了。

Ansible 安装操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week10/1-1.mp4
@`

## 4. SSH

在前面我们已经知道了 Ansible 是通过 SSH 来进行通信的。在和远程主机通信时，Ansible 默认假设使用 SSH 密钥，在需要时可以使用 `ansible` 命令的参数 `--ask-pass` 来进行密码认证。

这里实验楼提供了当前环境下 SSH 的密码（点击右边的工具栏 "SSH 直连"），因此无需设置 SSH 密钥。

## 5. Inventory

Ansible 能够同时对单台或多台机器亦或部分机器操作是通过 Inventory 来实现的， Inventory 默认保存在 `/etc/ansible/hosts` 配置文件中，而 Ansible 通过这个文件就可以知道要追踪的服务器了。

编辑打开配置文件

```
$ sudo vim /etc/ansible/hosts
```

可以看到配置文件中有很多的默认的示例配置，用 `'#'` 注释掉了。

![实验楼](https://dn-simplecloud.shiyanlou.com/2767331516098624338-wm)

在 Inventory 中列出我们需要操作的机器，可以单纯的列出这些主机，但是推荐有条理地为他们分组，这样在使用时就可以只对其中的某组操作。

Inventory 文件可以有多种不同的格式（如：INI、YAML 等），具体要取决于相应的插件，这里我们举几个 Ansible 的默认格式（INI）的示例，如下所示：

```bash
# 1.常用主机（IP 地址）分组，标题是组名，用于分类系统和决定系统的控制等，可以有一台或多台。
[test]
127.0.0.1
foo.example.com

# 2.分组后添加对该组机器的登录用户和验证方式。添加主机和用户名以及私钥文件。
[dev_test]
192.168.42.3 ansible_ssh_user=ubuntu ansible_ssh_private_key_file=/path/of/keyfile

# 3.不使用分组，采用文件别名的方式。通过端口及主机来描述。
Alias ansible_host=192.168.1.50 ansible_port=6666
```

下面我们来实际操作一下，在配置文件中添加如下语句：

```bash
[test]
127.0.0.1 ansible_ssh_user=shiyanlou ansible_ssh_pass=666666
```

当前实验环境的密码点击右侧工具栏中的 “SSH 直连”可获得，每个环境中密码都不同。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1997timestamp1516168322623.png/wm)

注：在 Asible 中并不推荐将 SSH 登录密码以文本形式存储在 Inventory 中的方式。但是此处使用只是因为本实验环境中的密码容易输入错误，避免后续可能出现的一些常见错误。

保存退出即可。

### Inventory 参数

如上的示例中，我们通过设置了一些变量来配置和远程主机的交互。下面我们列出几个常见的 inventory 的配置参数。

主机连接：

+ `ansible_connection` 连接到主机的类型，任何可能的连接插件名称，例如，SSH 协议类型中有：`ssh`、`smart` 或 `paramiko` 。

一般连接：

+ `ansible_host` 要连接的主机名称。

+ `ansible_port` ssh 端口号。

+ `ansible_user` 默认 ssh 用户名。

具体的 SSH 连接：

+ `ansible_ssh_pass` ssh密码

+ `ansible_ssh_private_key_file` 由 ssh 使用的私钥文件。

## 6. AD-HOC

**ad-hoc** ：临时命令，是在输入内容后，快速执行某些操作，但不希望保存下来的命令。

一般来说，Ansible 主要在于我们后面会学到的 playbook 的脚本编写，但是，ad-hoc 相较来说，它的优势在于当你收到一个临时任务时，你只用快速简单的执行一个 ad-hoc 临时命令，而不用去编写一个完整的 playbook 脚本就可以了。

我们知道 Ansible 主要是通过模块来实现各种功能的，下面我们就通过 `ping` 这个简单的模块来操作一下 `ad-hoc` 命令。

eg：

```bash
# 对 test 分组执行命令
$ ansible test -m ping

# 对所有机器执行命令
$ ansible all -m ping
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1997timestamp1516168374181.png/wm)

操作执行后可能会出现图上的报错，其主要原因是 ssh 连接时需要检查验证 `HOST KEY` ，可在 ssh 连接命令中使用 `-o` 参数将 `StrictHostKeyChecking` 设置为 no 来临时禁用检查。如果要保存设置，可修改 Ansible 配置文件，将 `/etc/ansible/ansible.cfg` 中的 `host_key_checking` 的注释符删除即可。如下操作：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1997timestamp1516168422630.png/wm)

修改完成后，再次执行 `ad-hoc` 操作的正确结果如下：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1997timestamp1516168450086.png/wm)

简单尝试后，我们大概了解了 Ansible 的 `ad-hoc` 的一般用法，如下是它的大致命令格式：

```bash
ansible 主机名或组名 -m 模块名 -a "模块参数" 其他参数
```

我们可以再举几个示例来感受下 `ad-hoc` 命令的操作。

eg1：执行命令查看 `setup` 模块中所有我们需要操作的机器的信息。

```bash
$ ansible all -m setup
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081470643195158-wm)

eg2：执行命令让操作的机器输出 `Hello shiyanlou`。

```bash
$ ansible test -a "/bin/echo Hello shiyanlou"
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1997timestamp1516169220218.png/wm)

eg3：执行命令让 test 组中的主机在指定目录下创建文件，并设置权限。

```bash
$ ansible test -m file -a "dest=/home/shiyanlou/file state=touch mode=777"
```

```checker
- name: check file
  script: |
    #!/bin/bash
	ls /home/shiyanlou/file
  error: /home/shiyanlou 目录下没有 file 文件
- name: check priv
  script: |
    #!/bin/bash
	stat -c %a /home/shiyanlou/file|grep 777
  error: /home/shiyanlou/file 权限不是 777
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1997timestamp1516169161655.png/wm)

从上面可以看出 Ansible 提供了很多的模块，还可以创建文件和文件夹、修改文件内容、创建用户、从源代码管理部署、管理软件包等操作。更多的模块及操作大家可以参考 Ansible 官方文档中的[模块章节](http://docs.ansible.com/ansible/latest/list_of_all_modules.html)。

对于 Ansible 来说基本支持我们平时常用的操作，并且它也在不断的完善来支持我们更多的操作。不过对于使用 shell 操作在 Ansible 中没有相应的模块支持的操作时，我们可以尝试的解决办法是直接使用 shell 模块来执行命令即可，如下这个例子：

```bash
$ ansible test -m shell -a 'free -m'
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid1997timestamp1516170159413.png/wm)

**AD-HOC 返回类型**

在之前的操作中我们可能看到返回的类型有如下几种：

+ `success`：这个结果表示操作成功，其中有两种情况，第一种情况是当执行一些查询的简单操作并且不需要修改内容时，表示该操作没问题；第二种情况就是当这个操作曾经执行过再执行时就会直接表示成功。

+ `changed`：true 这样的结果表示执行的一些修改操作执行成功，如上文的创建了一个文件，或者修改了配置文件，复制了一个文件等等这类的操作就会有这样的结果。

+ `failed`：这样的结果表示这个操作执行失败，可能是密码错误，参数错误等等，具体看提示中的 msg 的值。并且在 playbook 中会有多个任务，中间的某个任务出现这样的情况都不会继续往下执行。（playbook 会在后续的试验中详细讲解）

Ansible AD_HOC 命令操作视频：


`@
http://labfile.oss.aliyuncs.com/courses/980/week10/1-2.mp4
@`

## 7. 总结

本节实验主要学习了如何配置 Ansible 服务，以及尝试简单的 AD-HOC 命令的操作，来实现 Ansible 服务和控制的服务器间的通信，在后面实验中我们将继续学习 Ansible 另一个强大的功能——playbook。