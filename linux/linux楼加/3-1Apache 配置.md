---
show: step
version: 1.0
enable_checker: true
---
# Apache 配置

## 1. 实验介绍

#### 1.1 实验内容

在本实验中将带领大家学习最流行的 Web 服务工具：Apache。

Apache 在实现 HTTP 服务器功能之上提供非常多的人性化功能，加上其模块结构使其非常的灵活与强大，所以在本实验中仅仅简单学习其配置文件与一些常用的功能，由此入门。

#### 1.2 实验知识点

- Apache 简介
  + Apache 是什么
  + Apache 的优势
- Apache 安装与配置
  + Apache 的安装
  - Apache 的配置
    + Apache 的配置文件概览
    + Apache 的主配置文件
    - Apache 的站点配置文件
      + 基于端口配置
      + 基于 IP 地址配置
      + 基于 ServerName 配置
      + 配置规范
    + Apache 的模块配置文件
- Apache 安全配置

#### 1.3 推荐阅读

- [Apache 官方文档](https://httpd.apache.org/docs/2.4/getting-started.html)
- [IBM Apache 模块编写](https://www.ibm.com/developerworks/cn/opensource/os-cn-apachehttpd/index.html)
- [IBM Apache 防护](https://www.ibm.com/developerworks/cn/linux/l-cn-apache-secure/index.html)

## 2. Apache 简介

下面我们将会学习Apache。

### 2.1 Apache 是什么

在网络协议栈的应用层中最为常用的是 HTTP 协议，基于 HTTP 协议实现 Web 服务，让我们能够通过访问网址就可以获得我们想要访问的页面。

整个 Web 分为两部分，一部分是客户端也就是我们的浏览器等工具，而另外一部分就是服务器端。Apache 便是世界使用排名第一的 Web 服务器软件，他是 Apache 软件基金会的一个开放源码的 Web 服务器软件，因为免费、可靠、快速、灵活被大家所认可，起源于 1995 的 NCSAhttpd，不断的修改、增强使用至今。

### 2.2 Apache 的优势

相对于其他的 Web 服务器软件来说 Apache 有这样的一些优势：

- 少 bug，经过 23 年的发展，并且同时被大多数的企业使用，而且开源，所以相对来说 bug 较少
- 稳定性高，运行非常的稳定
- 模块多，因为非常的普及，又是开源，所以有非常多的模块被开发出来，能够应对各种各样的需求，Apache 中很多强大的功能都是靠模块实现的。
- Apache 对 PHP 的支持非常的便捷，相对于 Nginx 来说
- 重定向、重写功能非常的强大，相对于 Nginx 来说

Nginx 同样作为一款免费、开源的 Web 服务器软件，备受市场认可，现在其市场占有率仅排在 Apache 之后，所以常常将 Apache 与 Nginx 作比较。

## 3. Apache 安装与配置

在简单的了解 Apache 是什么之后我们将通过对其安装与配置的查看做进一步的认识。

### 3.1 Apache 的安装

**在本实验环境中已经为大家安装好了 Apache，所以大家不需要再一次的安装了。**以下只是做个介绍。

Apache 的安装方式有两种：

- 安装包管理工具安装
- 手动编译安装

两种安装方式的选择上，安装包管理工具方式安装简单、便捷，但是一般系统源为了稳定性所以版本会有所滞后，而手动编译安装方式稍显麻烦，但却可以安装最新版本，及时修补问题，并且在编译时添加自定义参数如修改安装位置、优化参数、运行模式、加载模块、修改环境变量、开关部分功能等等。

1.安装包管理工具安装

在我们的实验环境中使用的是 ubuntu 14.04 的系统，使用的是 apt 的管理工具，所以我们只需要通过这样一个简单的命令即可安装：

```bash
sudo apt-get install apache2
```

当然既想简单的安装，又想安装最新版的 Apache，可以通过添加源的方式来完成：

```bash
# 若是没有添加源工具需要该步骤
sudo apt-get install software-properties-common

# 实验环境 python3 默认指向的是 python3.5，但后面要用到的 add-apt-repository 工具不支持 python3.5，需要执行下面的命令来切换 python3 版本为 python3.4
sudo update-alternatives --config python3

# 添加源与密钥
sudo add-apt-repository ppa:ondrej/apache2

# 更新源
sudo apt-get update

# 安装最新版
sudo apt-get install apache2
```

2.编译安装

其实在之前的课程中我们也有尝试过编译安装，也并不是太过困难，这里只为大家罗列步骤：

- 下载与解压 Apache 源码，[官方链接](http://httpd.apache.org/download.cgi)
- 安装编译所需的依赖包与依赖库，如 gcc 等等
- 通过 configure 可执行文件添加编译参数，创建编译文件，可以添加的[参数项](https://httpd.apache.org/docs/2.4/programs/configure.html)非常多
- 通过 make 编译源码
- 通过 make install 安装编译好的源码


### 3.2 Apache 的配置

#### 3.2.1 配置总览

在完成 Apache 的安装之后我们便来查看其配置文件，首先我们来认识配置文件夹的结构与配置文件的作用。

默认的配置文件位于 `/etc/apache2` 中（当然在 Red Hat 系列中，Apache 名为 httpd，所以位于 `/etc/httpd` 中），我们来查看配置文件夹的结构：

![3-3-1-3.2.1](https://dn-simplecloud.shiyanlou.com/1135081517272969024-wm)

- apache2.conf：是 Apache 的主要配置文件，全局的一些配置都会在这里面
- ports.conf：是 Apache 监听端口的配置文件，由主配置文件所包含读取
- envvars：是 Apache 环境变量的配置文件
- magic：是在 Apache 加载了 mod_mime_magic 模块之后，用户辅助判断文件的 MIME 类型的配置文件

剩余的文件夹分为三个大类：

- conf：单独的指令配置文件
  + conf-available：可用的指令配置文件
  + conf-enabled：生效的指令配置文件
- mods：模块的加载与相关参数的配置文件：
  + mods-available：可用的模块配置文件
  + mods-enabled：生效的模块配置文件
- sites：站点的配置文件：
  + sites-available：可用的站点配置文件
  + sites-enables：生效的站点配置文件

看着有非常多的配置文件，但其结构很清晰，主要起作用的是 `apache2.conf` 主配置，其他的配置文件都是在主配置文件中通过 `Include` 指令包含读取。

通过包含的方式来读取其他的配置是因为将这些配置放置在一个配置文件中，会非常的大、较为混乱，不好维护，所以将其拆分成多个子配置文件能够清楚的理清各个作用，方便维护。

在主配置文件中只会读取 enable 中的配置文件，而 enable 中的配置文件是我们确认并加载 available 中配置文件的一个软连接。

Apache 配置介绍视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/1-1.mp4
@`

#### 3.2.2 主配置文件

在认识了我们配置文件的结构之后我们便来详细查看主配置文件中的内容。

配置文件中主要分这样几个部分：

- 注释信息，以 `#` 开头的为注释部分
- 配置指令，例如 `Include`、`ServerRoot`、`LogFormat` 等等
- 指令配置块（在书籍中称为指令上下文），例如以 `<xxxx> </xxxx>` 这样的方式包括配置指令的部分。
  + 虚拟主机上下文，<VirtualHost></VirtualHost>
  + 局部上下文，针对目录或者 URI 的配置块，例如 `<Directory>` 等等
  + 条件上下文，针对只在特殊情况下起作用的指令，例如 `<ifModule>` 等等

我们通过这样的方式忽略配置文件中的空行和注释内容：

```bash
cat /etc/apache2/apache2.conf |grep -v -E "^#|^$" |less
```

![3-3-1-3.2.2-1](https://dn-simplecloud.shiyanlou.com/1135081517277963104-wm)

1.配置指令

我们可以看到配置文件的开头即为多个配置命令，如 Mutex、PidFile、Timeout 等等，而其中配置值中的变量配置（例如 `${APACHE_LOCK_DIR}`）在 `/etc/apache2/envvars` 中。

2.指令上下文

在配置命令之后就是一段局部上下文的配置内容，如 `<Directory>` 和 `<FilesMatch>`。在全局中的配置指令应用于全局，而在上下文会继承全局中的指令值，其中特定配置的指令只在上下文中生效，若是配置了于全局中相同的配置项则会覆盖。

其中 `<Directory>` 的作用是配置某个目录的或路径的一个访问权限和展示情况等等，例如：

```bash
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```

其配置的是 `/var/www/` 的目录，`Options` 指令的完整语法为：`Options [+|-]option [[+|-]option] ...`，可以附加指定多种服务器特性，特性选项之间以空格分隔。

而 `Indexes` 表示在访问目录时，若是没有 `index.html` 这样的文件就以列表的方式展示该目录中的文件与子目录。

`FollowSymLinks` 表示的是允许读取目录中的符号链接方式。

`AllowOverride` 表示在使用 Apache 的 rewrite 模块重写的时候能否读取 `.htaccess` 中的重写规则，`None` 表示的是不能读取。

`Require` 是一个利用 mod_authz_host 模块来实现访问控制的一个指令，`Require all granted` 表示允许所有来源的一个访问，而拒绝所有来源的方式便是 `Require all denied`，若是开放某个 IP 段的访问 `Require ip 172.10 192.168`，多个 IP 段之间使用空格分割。还有可以指定域名等等用法。

在配置目录或者文件的时候不仅可以明确的指定路径（如上文中的例子），可以使用正则表达式例如：

```bash
<Directory ~ "^/www/[0-9]{2}">
.......
</Directory>
```

配置 www 目录中包含两个数字的特殊权限，在使用正则表达式我们需要添加 `~` 符号，若是怕忘记添加该符号可以直接使用 `DirectoryMatch` 指令块，例如：

```bash
<DirectoryMatch "^/www/[0-9]{2}">
.......
</Directory>
```

与之类似的还有 File 指令块。


Apache 主配置文件介绍视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/1-2.mp4
@`


#### 3.2.3 端口配置文件

在主配置文件中我们有看到其包含了 `ports.conf` 配置文件，我们来查看该配置文件：

```bash
less /etc/apache2/ports.conf
```

![3-3-1-3.2.3](https://dn-simplecloud.shiyanlou.com/1135081517280940271-wm)

通过 `Listen` 指令指定了默认 Apache 启动时监听的端口是 80 端口。

通过 `<IfModule>` 指令块，判断当前是否加载了 `ssl_module` 模块，若是加载了便同时监听 443 端口。

#### 3.2.4 站点配置文件

接下来我们来查看我们最为常用的站点配置文件：

```bash
less /etc/apache2/sites-available/000-default.conf
```

![3-3-1-3.2.4](https://dn-simplecloud.shiyanlou.com/1135081517281262914-wm)

所有的站点信息都是通过 `<VirtualHost>` 来包含。配置多个 `<VirtualHost>` 即可创建多个虚拟主机，访问多个虚拟主机可以通过这样的方式来区分：

- IP 地址
- 监听的端口号
- 访问的 Servername

（1）其中默认配置文件中的就是通过基于端口的方式来配置，配置 `<VirtualHost *:80>` 表示无论访问的是哪个 IP 地址的 80 端口（这里所说的无论哪个 IP，是因为一台机器上一般有多个地址，有内网网卡地址，外网网卡地址，环回地址等等），都会给予配置，然后以该上下文中的配置为准，我们可以来验证：

1.修改 ports.conf 文件

```bash
sudo vim /etc/apache2/ports.conf
```
```checker
- name: check content
  script: |
    #!/bin/bash
	grep 8080 /etc/apache2/ports.conf
  error: /etc/apache2/ports.conf 内容不对
```

既然是基于端口配置，就需要先让我们的服务监听在新增的端口上：

![3-3-1-3.2.4-2](https://dn-simplecloud.shiyanlou.com/1135081517288920862-wm)

2.修改站点配置文件

```bash
sudo vim /etc/apache2/sites-available/000-default.conf
```
```checker
- name: check content
  script: |
    #!/bin/bash
	grep VirtualHost /etc/apache2/sites-available/000-default.conf
  error: /etc/apache2/sites-available/000-default.conf 内容不对
```

新增加一个 VirtualHost，匹配所有访问 8080 端口的请求，为了看出区别我们修改了其 DocumentRoot 目录，该配置项的作用就是指定访问该 VirtualHost 时对应文件系统的目录（为了看到完整的配置项，我们通过 grep 不显示注释内容）：

![3-3-1-3.2.4-3](https://dn-simplecloud.shiyanlou.com/1135081517289195006-wm)

3.在 /var/www 目录下增加新增站点默认显示内容

```bash
echo "Hello Louplus2" | sudo tee /var/www/index.html
```
```checker
- name: check file
  script: |
    #!/bin/bash
	ls /var/www/index.html
  error: /var/www 目录下没有 index.html 文件
```

4.重新加载配置文件使其生效

```bash
sudo service apache2 reload
```

5.验证

![3-3-1-3.2.4-4](https://dn-simplecloud.shiyanlou.com/1135081517289352930-wm)

我们可以看到在访问不同端口的时候，展现出了不同的内容。


Apache 站点配置实验操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/1-3.mp4
@`



（2）若是以 IP 地址的方式，例如使用环回地址则这样配置 `<VirtualHost 127.0.0.1:80>`，这样只有在使用 `127.0.0.1` 的方式才可以访问。我们可以来验证一下：

1.首先访问我们的站点：

![3-3-1-3.2.4-5](https://dn-simplecloud.shiyanlou.com/1135081517282402122-wm)

我们可以看到不管是以内网地址，还是环回地址访问都没有问题。

2.修改配置文件

```bash
sudo vim /etc/apache2/sites-available/000-default.conf

# 将 *:80 修改成如下所示
<VirtualHost 127.0.0.1:80>
```

然后通过 reload 的方式重新加载使其生效

```bash
sudo service apache2 reload
```

3.验证

![3-3-1-3.2.4-6](https://dn-simplecloud.shiyanlou.com/1135081517282641344-wm)

我们可以看到只有访问 `127.0.0.1` 的时候才能看到我们默认的展示页面，而访问其他两个地址我们只能看到目录的索引列表，这是因为我们在主配置文件中配置了 `/var/www` 目录的 Indexes，可以在没有 `index.html` 文件的时候展示目录的索引列表。若是我们删除该配置的 `Indexes` 指令项，我们就看到这样的结果了：

![3-3-1-3.2.4-7](https://dn-simplecloud.shiyanlou.com/1135081517283302603-wm)

注意：默认在没有找到相关的 `VirtualHost` 匹配的配置时访问的目录是 `/var/www`，这是默认的配置，我们可以通过这样的命令查看到：

```bash
apache2ctl -S
```

![3-3-1-3.2.4-8](https://dn-simplecloud.shiyanlou.com/1135081517290196511-wm)

其中的 `Main DocumenRoot` 指定的就是主访问的根目录。


Apache IP 地址站点配置实验操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/1-4.mp4
@`



（3）最后便是以 Servername 来区分，也就是多个虚拟主机可以使用相同的端口，之间通过访问的域名来区分，例如：

1.修改 Apache 配置文件

```bash
sudo vim /etc/apache2/sites-available/000-default.conf
```

我们增加了一个 `VirtualHost`，并配置各自的 `ServerName`，为了凸显第二个站点的区别，我们同时修改第二个站点的 `DocumentRoot`，该配置项的作用就是指定访问该 VirtualHost 时对应文件系统的目录（为了看到完整的配置项，我们通过 grep 不显示注释内容）：

```checker
- name: check content
  script: |
    #!/bin/bash
	grep louplus /etc/apache2/sites-available/000-default.conf
  error: /etc/apache2/sites-available/000-default.conf 内容不对
```

![3-3-1-3.2.4-9](https://dn-simplecloud.shiyanlou.com/1135081517287764499-wm)

2.在 /var/www 目录下增加新增站点默认显示内容

```bash
echo "Hello Louplus2" | sudo tee /var/www/index.html
```

3.增加 `/etc/hosts` 的地址解析

```bash
sudo vim /etc/hosts
```
```checker
- name: check content
  script: |
    #!/bin/bash
	grep -E 'louplus1|louplus2' /etc/hosts
  error: 没有修改 /etc/hosts 文件
```

![3-3-1-3.2.4-10](https://dn-simplecloud.shiyanlou.com/1135081517287978077-wm)

4.重新加载配置文件

```bash
sudo service apache2 reload
```

5.验证

![3-3-1-3.2.4-11](https://dn-simplecloud.shiyanlou.com/1135081517288115872-wm)

我们可以看到，展现出了不同的信息，当然若是我们访问了一个域名也指向了该机器，但是 VirtualHost 中找不到对应的 ServerName 就会使用相关端口的第一个 VirtualHost 配置。

在平时的使用习惯上，我们不会将不同的 VirtualHost 的配置放在相同的配置文件中，我们会在 `sites-available` 中创建新的配置文件单独存放。

在命名上一般以其实际的使用意义来命名，并以 `.conf` 来结尾，例如 louplus 的配置我们就会放在 `/etc/apache2/sites-available/louplus.conf` 配置文件中

在创建新的配置文件之后我们需要使用 `sudo a2ensite louplus.conf` 命令来让 Apache 创建一个软连接到 `sites-enable` 中，只有这样 Apache 才会读取相关的配置文件。

当然让 Apache 加载了相关的配置文件之后，我们需要 `sudo service apache2 reload` 让我们的配置文件生效。


Apache ServerName 站点配置实验操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/1-5.mp4
@`

#### 3.2.5 模块配置文件

同样的模块配置文件我们会在 mods-avaliable 中放加载与相关模块配置的文件，然后通过 `a2enmod 模块名` 来使我们的模块被加载。

然后通过 `service apache2 restart` 来使得我们的模块真正的被加载。

## 4. Apache 安全配置

在上文中为大家介绍了最常用的配置，如 VirtualHost、端口、目录访问控制，但其实 Apache 的细节配置项还有很多，我们没有办法一一列举。当前学习的配置内容能够帮助我们解决大部分基础的需求，当我们有更细节的需求时我们便知道从那个相关的配置块或者模块中获取我们的答案。

Apache 是一个 Web 服务，那么它所提供的服务是会面向大众的，是公开出来的，而公开出来的服务面临最大的问题就是：安全。

当我们以新手的身份去配置 Apache 的时候我们需要注意这样的一些配置，来防止攻击者有机可乘：

1.Apache 运行用户

Apache 以 root 用户来运行，攻击者可以通过其来获得服务器的 root 权限，由此可以在其中胡作非为了，所以我们需要配置 Apache 启动的运行用户，现在默认情况下 Apache 以 `www-data` 用户来运行：

```bash
cat /etc/apache2/envvars |grep APACHE_RUN
```

![3-3-1-3.2.5-1](https://dn-simplecloud.shiyanlou.com/uid/276733/1517308141160.png-wm)

我们可以看到运行的用户与用户组默认是以 `www-data` 用户与组来运行的。

2.Apache 的信息隐藏

当我们访问一个不存在的页面或者遇到后端服务崩溃的时候我们会看到一个错误页面，例如：

![3-3-1-3.2.5-2](https://dn-simplecloud.shiyanlou.com/1135081517296399899-wm)

我们访问了一个不存在的页面，给我提示了 `Not Found` 错误信息，同时在下方中显示了我 Apache 的版本信息，与我使用的系统版本。

这样的提示是非常不安全的，因为 Apache 是开源的，当不怀好意的用户得知使用版本之后，可以根据相关的版本存在的漏洞查找或者自己编写相关的攻击脚本，所以这样是非常不安全的，我们可以通过配置 `ServerSignature` 来关闭，我们通过这样的方式来查看当前该配置：

```bash
grep -R ServerSignature /etc/apache2/
```

![3-3-1-3.2.5-3](https://dn-simplecloud.shiyanlou.com/1135081517296794630-wm)

我们可以看到在 `conf-available/security.conf` 中该配置项是打开的，我们修改该配置项：

```bash
sudo vim /etc/apache2/conf-available/security.conf
```

将该配置的 on 改成 off：

```checker
- name: check content
  script: |
    #!/bin/bash
	cat /etc/apache2/conf-available/security.conf |grep ServerSignature|grep -i Off
  error: 没有修改 /etc/apache2/conf-available/security.conf
```

![3-3-1-3.2.5-4](https://dn-simplecloud.shiyanlou.com/1135081517297033204-wm)

然后通过 `sudo service apache2 reload` 使我们的配置生效，我们再来看错误的页面变化：

![3-3-1-3.2.5-5](https://dn-simplecloud.shiyanlou.com/1135081517297093942-wm)

3.Apache 的访问控制

在上文中我们学到每个 VirtualHost 都会有一个 DocumentRoot，也就是在访问该 VirtualHost 的根路径时，DocumentRoot 对应的目录即为根目录，而目录我们可以通过 `<Dirctory>` 来配置其相关的权限，相关的指令有：

- Require 命令，可以通过、拒绝所有，可以根据 IP 地址控制，可以根据主机名控制
- Order 命令，类似于 TCP Wrapper 的命令，通过 allow、deny 来控制具体地址的访问权限

4.通过 chroot 来改变运行目录

通过 chroot 工具可以改变运行目录，只是较为麻烦。这里不再详述，大家可以网上搜索相关资料来学习。

5.Apache 用户认证

我们想通过让用户提供用户名、密码的方式来认证，我们可以利用自带的认证模块，配合 htpasswd 来配置，例如：

（1）创建登录用户名与密码的数据文件：

```bash
# 安装相关工具
sudo apt-get install apache2-utils

# 创建密码文件，指定文件的位置与登录的用户名
sudo htpasswd -c /var/www/.htpasswd shiyanlou

# 然后在交互界面中输入密码
```

（2）配置目录访问的认证模式

```bash
sudo vim /etc/apache2/apache2.conf
```
```checker
- name: check content
  script: |
    #!/bin/bash
	grep '/var/www' /etc/apache2/apache2.conf
  error: 没有配置 /etc/apache2/apache2.conf
```

![3-3-1-3.2.5-6](https://dn-simplecloud.shiyanlou.com/1135081517301122088-wm)

增加了 Auth 模块相关的配置：

- AuthType 配置认证的方式
- AuthName 配置认证时的提示信息
- AuthUserFile 配置认证时读取的用户密码文件

并将 Require 修改成指定用户才可以登录，若是将 `user shiyanlou` 改成 `valid-user` 则是所有有效用户都可以登录。

完成修改之后我们通过 `service apache2 reload` 让配置生效。

3.验证

![3-3-1-3.2.5-7](https://dn-simplecloud.shiyanlou.com/1135081517301386635-wm)

这样我们在登录的时候就需要提供用户名密码了。


Apache 安全配置实验操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week9-mp4/1-6.mp4
@`


## 5. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

- Apache 简介
  + Apache 是什么
  + Apache 的优势
- Apache 安装与配置
  + Apache 的安装
  - Apache 的配置
    + Apache 的配置文件概览
    + Apache 的主配置文件
    - Apache 的站点配置文件
      + 基于端口配置
      + 基于 IP 地址配置
      + 基于 ServerName 配置
      + 配置规范
    + Apache 的模块配置文件
- Apache 安全配置

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。