---
show: step
version: 1.0
enable_checker: true
---
# SELinux 简介

## 1 实验介绍

#### 1.1 实验内容

本节实验主要是对 SELinux 进行一个初步的认识，让大家明白 SELinux 的用途，以及学习它的一个规划，最后是对其的一些简单操作。

#### 1.2 实验知识点

+ SELinux 是什么 
+ SELinux 简单流程
+ SELinux 学习内容
+ SELinux 开启与关闭
+ SELinux 相关工具
+ SELinux 日志

#### 1.3 推荐阅读

+ [SELinux 的历史](https://www.ibm.com/developerworks/cn/linux/l-secure-linux-ru/index.html)
+ [SELinux Documentation](https://www.nsa.gov/what-we-do/research/selinux/documentation/index.shtml)
+ [SELinux wiki about Tools](https://github.com/SELinuxProject/selinux/wiki/Tools)
+ [DAC 权限](http://blog.51cto.com/zhaotianyu/1795178)
+ [Redhat Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-security-enhanced_linux-working_with_selinux-which_log_file_is_used)

## 2. SELinux 是什么 

#### 2.1 概述

SELinux(Security-Enhanced Linux)是 Linux 内核集成的一个安全模块，为了增加传统 Linux 操作系统的安全性而开发。

那么可能大家会有一点疑惑，为什么需要增加 Linux 操作系统的安全性呢？

在之前的课程我们有学习到 Linux 中一切皆文件，并且每个文件都有其所属者与相对应的权限，通过用户与关联的读、写、执行权限来做一个相对简单的限制，这便是传统 Linux 的主要安全规则之一（后续会对该方法做一个详细的介绍，名为 DAC），而这样的限制方式有两个很大的弊端，其中之一便是这一切的限制其实对 root 用户是无效的，因为 root 用户拥有着至高无上的权利，所以一旦 root 用户做了一些错误的操作也将造成很严重的问题，还有一个弊端便是若是一些敏感的文件被没有经验的管理员设置了 777 的权限这样也容易被人所利用，造成一些不必要的麻烦。

而美国国家安全局（NSA）联合 Secure Computing Corporation (SCC)与犹他大学设计出了增强的安全系统架构 Flask，因为当时的 Linux 开源，便在 Linux 中实现他们的研究成果，最后以 Security-Enhanced Linux 的名称向公众发布。

而 SELinux 非常的灵活，不是简单的用户/组权限的限制，而是定义了一种混合的安全性策略，由基于角色的访问控制（RBAC，Role-based access control）、类型实施（TE，type enforcement）以及多级别安全性（MLS，Multi-Level Security）/多类安全性（MCS，Multi-Category Security）组成，有了这样复杂的安全机制，root 用户便不能为所欲为，弥补了传统的 DAC 模式的缺点，给系统带来了进一步的安全保障，减少因管理员的失误或者配置上的失误而造成错误的几率。

#### 2.2 DAC

DAC（Discretionary Access Control）自主访问控制，是一种经典 Unix 安全机制，最早的操作系统并没有用户的概念，后来因为多人使用相同的环境，而存储空间的紧张出现了其他使用者的使用资源或者数据被随意修改或者删除的现象，从而增加了用户的概念，并且增加了不同用户所属文件权限，从而防止这样的恶性事件继续发生。

这也就是为什么我们之前通过 `ls -lah` 命令可以看到所有的文件都有相关的所属用户与组，并且针对 owner（所属者）、group（组）、other（其他）都有 read（读）、write（写）、execute（执行）三个权限的设置

由此我们明白 DAC 主要针对的对象或者说主体是用户，以这样的因素来限制权限。其实这种方式是较为自由较为宽松的安全机制，说他宽松、自由是因为它尊重用户个人的主观意识，并没有其他更多的限制。

#### 2.3 MAC

MAC（Mandatory Access Control）强制访问控制，我们所说的 SELinux 正是使用这样的控制方式，它的提出正是因为 root 的权限太大、DAC 的方式太过于个人主义化，所以 MAC 的控制方式不再是只针对用户，而是增加了更多的其他因素与策略从而限制过于松散的安全制度。

这些因素包括角色（类似 DAC 中的用户）、类型、域、级别，这些因素管理员都可以灵活的配置，这样使得安全制度不会过于的个人主义化，同时 MAC 的方式不仅针对用户，文件同时也针对服务，对服务权限的限制使得 root 进程也不再是万能的，从而限制住了 root 的部分权限，这样就算是被被入侵了，也不会使得损失最大化。这也是在 flask 中提出的一个主要的特性最小特权。

这就是 SELinux，简单理解就是增加更多的标签来限制用户、限制进程。

## 3. SELinux 简单流程

很多时候我们都是 DAC + SELinux 同时启动来保护我们机器的安全，那么当这两个安全机制同时开启的时候，其检查的顺序是这样的：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1871timestamp1512095625106.png/wm)

当我在执行操作的时候，例如我们查看某个文件，或者是某个进程要读取某个文件的时候会执行这样的一些安全操作：

- 我们将我们的这些操作称为事件流，事件发生之前，首先会到 DAC 检测机制中，检查当前用户是否有对当前文件的读取或者写或者执行操作的权限；
  + 若是当前用户并没有相关的权限则会拒绝访问；
  + 若是当前用户有相关的权限，DAC 允许其操作，则进入下一步；
+ DAC 通过之后会将事件发送给 LSM（Linux Security Modules，Linux 安全模块。当时 Linux 只支持了 DAC 与部分 POSIX.1e 标准草案的 capabilities 安全机制，后来为了灵活的加入更多的安全机制就设计出了 LSM，各安全模块在系统启动初始化的时候会在 LSM 中注册并初始化）；
+ LSM 收到之后会将其发送给 SELinux Abstraction & Hook Logic 子系统；
+ SELinux Abstraction & Hook Logic 子系统接收到之后会发送给 Policy Enforcement Server 模块；
+ Policy Enforcement Server 模块与 Access Vector Cache 子系统通信，查询相关的策略；
- Access Vector Cache 子系统会到 System security policy database 中去查找相关的策略；
  + 若是查找到了 Access Vector Cache 子系统将会记录并缓存，下次查询同样的记录便可直接返回结果；
  + 若是没有找到也会将结果返回给 Policy Enforcement Server 模块；
- Policy Enforcement Server 模块获取策略之后判断其是否有权限执行操作；
  + 若是没有权限执行操作会记录到日志文件中，通常会在 message 或者 audit 中，然后将结果返回给 SELinux Abstraction & Hook Logic 子系统；
  + 若是有权限执行操作则直接将结果返回给 SELinux Abstraction & Hook Logic 子系统；
+ SELinux Abstraction & Hook Logic 子系统在接收到结果之后便返回给 LSM；
- LSM 再获取相关结果之后便执行相关的结果；
  + 若是没有权限则操作驳回并提示用户没有相关的权限；
  + 若是有权限则执行下一步；

这便是我们在执行一个操作的时候大概会走的一个流程。

## 4. SELinux 的配置文件

通过上述一个描述相信大家对 SELinux 会有一定的感官了，至少知道它是干嘛的，与 DAC 的判断顺序。接下来我们便开始 SELinux 的使用学习，我们会按照这样的顺序来逐步学习 SELinux：

- SELinux 的状态
  + SELinux 的开启与关闭
  + SELinux 的状态查询
  + SELinux 的状态切换
- SELinux 的日志
- SELinux 的相关工具
- SELinux 的上下文
  - SELinux 上下文的概念
    + SELinux 的身份标示
    + SELinux 的角色
    + SELinux 的类型
    + SELinux 的级别
  - SELinux 上下文的操作
    + SELinux 上下文的查看
    + SELinux 上下文的临时修改
    + SELinux 上下文的永久修改
    + SELinux 上下文的恢复
    + SELinux 上下文的保持
- SELinux 的策略与管理
  + SELinux 策略查看
  - SELinux 策略管理
    + SELinux 策略配置
    + SELinux 策略布尔值
- SELinux 的应用实例

在本实验剩下的部分我们会学习到 SELinux 的日志部分，而 SELinux 的核心也就是上下文与策略部分我们将在下一个实验中讲解，最后通过一个应用的案例实验来检验我们是否对 SELinux 真正的理解。

## 5. SELinux 的状态

下面我们将会学习 SELinux 的状态。

### 5.1 SELinux 的开启与关闭

SELinux 的开启与关闭是通过一个配置文件，而不是通过简单命令进行开关。这是因为 Linux 系统的 LSM 在初始化的时候会调用相关安全策略的注册函数，也就是 Linux 内核加载的前期就会初始化 SELinux 模块相关的内容，确保 SELinux 能够尽早的启动，以便在创建进程和对象时对其进行标识。

SELinux 的配置文件位是 `/etc/selinux/config`。改配置文件主要配置两个内容：

 + SELinux 的开关
 + SELinux 加载的策略

`/etc/selinux/config `文件是 SELinux 主要的配置文件。它控制着 SELinux 是启用还是禁用，以及使用哪种 SELinux 模式和 SELinux 策略。

我们来查看一下该配置文件的内容 `cat /etc/selinux/config`：

![cat-selinux-config](https://doc.shiyanlou.com/document-uid113508labid1871timestamp1512115816650.png/wm)

以 `#` 开头的为注释，是对配置项的一个解说，我们可以看到配置中生效的只要这两个：

- SELINUX=permissive
- SELINUXTYPE=targeted

他们的作用是：

- SELINUX 配置项是用来控制 SELinux 的运行模式或者直接关闭它，他可以设置的值有三个：
  + disabled：关闭 SELinux，一般系统中该值为默认值，也就是系统默认情况下是关闭了 SELinux 的；
  + permissive：SELinux 的一种运行模式，在这种模式下 SELinux 开启，当操作越过了 SELinux 设置的权限、策略的时候，会将其记录在日志文件中，但是不会阻止它；
  + enforcing：SELinux 的一种运行模式下，在这种模式下 SELinux 开启，当操作越过了 SELinux 设置的权限、策略的时候，会将其记录在日志文件中，并且阻止该行为；

- SELINUXTYPE 配置项是用来设置 SELinux 加载的策略，他可以设置的值同样有三个：
  + targeted：针对服务进程进行访问控制
  + minimum：针对选择部分的进程保护，由 target 修订而来
  + mls：针对所有进程进行多级别访问控制

默认情况下系统的 SELINUX 配置的是 disabled，若是想开启 SELinux 我们需要将其改成 permissive 或者是 enforcing，然后保存配置文件，重启电脑使其生效，需要重启使其生效的原因已经在上文中解释。

在生产环境中我们会直接使用 enforcing 的模式，而 permissive 的模式更多是用来调试，因为 SELinux 的权限控制非常细，所以很容就会发生一些文件访问不了或者一些没有想到的异常情况，这个时候就会切换成 permissive 的运行模式。

因为我们是初次学习 SELinux，所以环境中我们将其切换成 permissive 的运行模式。

同样我们的加载的策略也是较为轻量的 targeted 方式，当然也是因为 centos7 中也只自带了这种策略方式，例如我们若是想使用 minimum 策略，我们需要做这样一些操作：

+ 安装这个策略模式：` yum -y install selinux-policy-minimum`
+ 修改配置文件中的 SELINUXTYPE
+ 重启设备使其生效：`reboot`

这就是 SELinux 的开关配置。

### 5.2 SELinux 的状态查询

除了查看配置文件我们同样有其他的方式可以帮助我们快速的查看 SELinux 运行的状态，例如 `getenforce` 命令：

![getenforce](https://doc.shiyanlou.com/document-uid113508labid1871timestamp1512119026525.png/wm)

`getenforce` 查看的当前的 SELinux 的状态，而配置文件查看的是初始配置 SELinux 的模式，学习下一小节我们就知道 SELinux 的运行模式是可以切换的.

这个命令获取的内容非常的简略，若是我想看看当前的 SELinux 处于什么样的状态，是否开启，或者说用的什么策略，稍微详尽一点的信息，我们可以通过 `sestatus` 命令：

![sestatus](https://doc.shiyanlou.com/document-uid113508labid1871timestamp1512119238014.png/wm)

当然该命令通过不同的参数还可以查看更加详尽的内容，例如上下文、布尔值等，这些我们将放在后续相关内容的时候进行讲解。

### 5.3 SELinux 的状态切换

我们在上文说到 SELinux 的开启与关闭需要通过配置文件来控制，但是 SELinux 运行的模式是可以通过命令控制，也就是说 SELinux 的运行模式是不用修改配置文件然后重启机器令其生效的，我们只需要通过 `setenforce` 命令即可轻松地切换 SELinux 运行的模式，当然这里的切换只是临时的切换，当机器重启之后，初始化的时候还是读取的配置文件中的参数，所以若是需要永久的修改运行模式我们需要修改配置文件，若只是想调试一下权限问题，临时修改的话便可以 `setenforce` 命令直接切换

`setenforce` 只需要一个参数，这个参数就是需要运行的模式，这个参数可以为这两个值：

+ 0 ：转成 permissive 宽容模式
+ 1 ：转成 Enforcing 强制模式

![change-selinux](https://doc.shiyanlou.com/document-uid113508labid1871timestamp1512283077355.png/wm)

从图中我们可以看到我们可以通过 setenforce 命令来回切换两种运行的状态。

SELinux 状态操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week3/9-1.mp4
@`

## 6. SELinux 的日志

我们在上文的流程中有提到 SELinux 在管理权限的时候会将日志权限不通过的请求都记录在日志中，以便后续管理员的审计与分析，而当启动的服务不同时 SELinux 日志信息记录的文件也是不同的，我们通过这样一张表来了解：

| 日志工具 | 日志文件
|--------|--------
| audit [on] | /var/log/audit/audit.log
| audit [off]/rsyslogd [on] | /var/log/messages
| setroubleshooted/rsyslogd/audit | /var/log/audit/audit.log 翻译后日志 /var/log/messages

我们在之前的 Linux 日志系统的章节中了解到 Linux 的日志系统主要是 syslog 系统的架构，而 Linux 提供了一套审计系统 audit，提供了系统安全信息记录的方法，也就是当用户违反了系统安全规则时及时地将警告信息记录下来以便管理员的查看。内核实现的 audit 只会将信息写入套接字缓冲队列中，若需要写在 log 文件中的话还需要启动用户级的设计后台 auditd，在 Centos 中默认安装与启动用户级的 audit，而在 Debian、Ubuntu 中并没有安装启动。

所以当 audit 开启的时候 SELinux 相关的日志都会写入 audit.log 中，而若是 audit 没有开启而 rsyslogd 日志系统开启了便会将相关的信息写入 messages 中，此时记录的信息可读性不是很好，例如：

```bash
type=AVC msg=audit(1512349649.744:48393): avc:  denied  { getattr } for  pid=18865 comm="httpd" path="/var/www/html/index.html" dev="vda1" ino=140588 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:admin_home_t:s0 tclass=file
```

- 发生的时间是以时间戳的方式告知，必须经过一次换算
- 告知我们发生错误文件的路径与上下文，但是却不知道哪里错了

我们可以通过这样的方式来制造上述错误：

首先我们安装 httpd（在 centos 中 apache 是 httpd，在 ubuntu 中为 apache2）：

```bash
sudo yum install -y httpd
```

紧接着我们通过 shiyanlou 账户创建一个 `index.html` 文件放置 httpd 默认访问目录：

```bash
echo "hello shiyanlou" > index.html
sudo mv index.html /var/www/html/
```

随后我们启动 httpd：

```bash
sudo service httpd start
```

启动之后我们打开浏览器访问 `localhost`，若是在 `Permissive` 状态下可以访问，若是在 `Enforcing` 的状态下便是直接 403 返回了：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid1labid4244timestamp1513674702845.png/wm)

同时我们查看 `/var/log/audit/audit.log` 就会查看到类似的报警记录了：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid1labid4244timestamp1513674876111.png/wm)

为了增加日志的可读性，便出现了 setroubleshoot 工具，它可以帮我们将 audit 的日志信息翻译成更加人性化的方式，当 audit 与 rsyslog 同时开启并安装了 setroubleshoot 软件包时，会先在 audit.log 中记录日志信息，然后 setroubleshoot 将其翻译后写入 message 中，例如：

```bash
Dec  4 09:25:19 iZbp1jer7bx9h4he8tbo0nZ setroubleshoot: SELinux is preventing httpd from getattr access on the file /var/www/html/index.html. For complete SELinux messages run: sealert -l 5437555c-0d6d-4e3f-95c8-b5de2b01387a
Dec  4 09:25:19 iZbp1jer7bx9h4he8tbo0nZ python: SELinux is preventing httpd from getattr access on the file /var/www/html/index.html.#012#012*****  Plugin restorecon (99.5 confidence) suggests   ************************#012#012If you want to fix the label. #012/var/www/html/index.html default label should be httpd_sys_content_t.#012Then you can run restorecon.#012Do#012# /sbin/restorecon -v /var/www/html/index.html#012#012*****  Plugin catchall (1.49 confidence) suggests   **************************#012#012If you believe that httpd should be allowed getattr access on the index.html file by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'httpd' --raw | audit2allow -M my-httpd#012# semodule -i my-httpd.pp#012
```

我们可以通过这样的方式来开启 setrouble：

1.工具的安装

```bash
sudo yum install -y setroubleshoot-server 
```

2.重启 auditd 使其生效

```bash
sudo service auditd restart 
```

此时我们再访问一遍 `localhost`，然后我们查看 `/var/log/message` 就可以查看到上述类似信息了：

```bash
sudo tail /var/log/messages
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid1labid4244timestamp1513675120165.png/wm)


信息中会告诉我们问题的大概信息，与如何查看问题的详情，同时相关的组件会告诉我们具体是哪里出问题了，并且给出了解决问题的方案（截图中的命令来自于上述日志， 在 `Enforcing` 模式下运行 截图中的命令 会报错，请看下文注意中的解决方式）：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid1labid4244timestamp1513675366665.png/wm)

>**注意**：
> - 1.在 centos7 中 setroubleshoot 工具不是长期运行一个后台进程，仅仅只是在日志信息到达 audit 的时候运行；
> - 2.在 setroubleshoot 3.3 之前，在桌面终端执行并且同时处于 enforcing 模式的时候 sealert 运行会报错（一个 python 运行错误），切换至 permissive 或者在字符界面即可；
> - 3.上述的例子来自于后面的应用实验，将于应用实验中为大家演示。

SELinux 日志操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week3/9-2.mp4
@`

## 7. SELinux 的相关工具

SELinux 中并不是所有的工具都默认安装的，这里为大家简单介绍几个较为常用的工具集：

+ setroubleshoot：提供了 SELinux 排查故障的工具
+ setroubleshoot-server：提供了上述所介绍的将 AVC 拒绝日志转化成更为人性化、通俗易懂的语言，并提供了 sealert 工具，查看日志详情。
+ libselinux-utils：提供了一些常用简单的工具集，例如 getenforce、setenforce、getsebool、selinuxenabled 等等。
+ policycoreutils：提供了 SELinux 的管理工具，例如 restorecon、secon、load_policy 等等
+ policycoreutils-python：提供了 SELinux 的一些策略管理工具，例如 semanage、audit2allow 等等
- 安全策略包
  + selinux-policy-targeted：提供 targeted 类型的策略
  + selinux-policy-minimum： 提供 minimum 类型的策略
  + selinux-policy-mls： 提供 mls 类型的策略
- 开发 API 工具包：
  + libselinux：提供 SELinux 应用程序的 API
  + libselinux-python：提供 SELinux 应用程序的 API，适用于 python 开发。

## 8. 总结

通过本节实验，大家可初步对 SELinux 一定的认识，知道该如何去学习 SELinux，与 SELinux 的状态改变与切换还有日志、工具的相关内容，相对来说本章节的内容不是很难，也不多，SELinux 的核心在于下一章节，在下一章节将带领大家学习 SELinux 的策略与规则的管理。

这是本章节的内容总揽：

- SELinux 的状态
  + SELinux 的开启与关闭
  + SELinux 的状态查询
  + SELinux 的状态切换
- SELinux 的日志
- SELinux 的相关工具

大家能够通过章节的内容回忆出相关的知识点吗？

## 7.参考文件

+ [SELinux 的历史](https://www.ibm.com/developerworks/cn/linux/l-secure-linux-ru/index.html)
+ [SELinux wiki about Tools](https://github.com/SELinuxProject/selinux/wiki/Tools)