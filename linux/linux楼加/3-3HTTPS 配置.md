---
show: step
version: 1.0
enable_checker: true
---
# HTTPS 配置

## 1.实验介绍

HTTPS 协议已经成为目前绝大多数网站的首选。相比于 HTTP，它们几乎是一样的，除了在传输数据上一个是经过加密的，一个是明文。随着网络上传输的数据越来越多，越来越重要，数据安全性和隐私性变得非常关键。Chrome 浏览器即将在今年（2018）七月份把所有为使用 HTTPS 协议的网站标记为不可信，苹果应用商店已经拒绝上架采用 HTTP 协议跟服务器通信的应用。

本次实验我们将引导大家来为你的网站启用 HTTPS。

## 2.HTTPS 简介

HTTPS，也称作 HTTP over TLS。TLS 的前身是 SSL，SSL 由于存在安全漏洞已经不建议使用。SSL 的版本包括 1.0、2.0 和 3.0，TLS 的版本包括 1.0、1.1、1.2 和 1.3。其中 TLS 1.0 和 SSL 3.0 的差异非常小，但足以排除两者之间的互操作性。

### 2.1 HTTPS 与 HTTP 的区别

HTTPS 相比 HTTP 协议额外提供了数据完整性、数据隐私性和服务端身份认证。具体区别如下：

1. HTTPS 需要到证书颁发机构（Certificate Authority，简称 CA）申请证书。证书有收费的，也有免费的。
2. HTTP 是明文传输，信息很容易被窃听，HTTPS 是加密传输，几乎无法被窃听。
3. HTTP 和 HTTPS 建立连接的方式不同，服务监听端口也不一样。前者使用80端口，后者使用 443端口。

## 3. CA 证书申请

在 HTTPS 未流行之前，CA 证书基本都是要收费的，并且费用比较高昂。随着 HTTPS 的日益普及，不但 CA 机构推出了免费证书服务，还出现了一些专门提供免费服务的 CA 机构，比如本次实验我们要使用的 [Let's Encrypt](https://letsencrypt.org/)。

国内阿里云服务跟 Symantec 合作提供了免费的 CA 证书，有效期一年，每个账号最多可以申请 20 个。虽然说对大多数网站来说已经够用，但毕竟是跟商业公司的合作服务，不能保证服务会一直延续下去。所以建议大家尽量使用免费的 CA 证书服务，比如 Let's Encrypt。

### 3.1 Let's Encrypt CA 证书申请

Let's Encrypt 提供了 [certbot](https://certbot.eff.org/) 工具来简化证书的申请。

#### 3.1.1 选择软件和系统

打开 certbot 主页后，首先需要选择要配置证书的软件（Apache、Nginx等）和服务器操作系统。这里我们以 Nginx 和 Ubuntu 16 为例。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5898timestamp1528190730391.png/wm)

#### 3.1.2 安装 certbot 以及相关插件

在要配置证书的服务器上执行以下命令来安装 certbot 以及相关插件。

```bash
$ cd /usr/lib/python3/dist-packages
$ sudo apt-get update
$ sudo apt-get install software-properties-common

# 执行 add—apt-repository 可能会报错，使用下面这条命令解决
$ sudo ln -s apt_pkg.cpython-34m-x86_64-linux-gnu.so apt_pkg.so
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx
```

```checker
- name: check pkg
  script: |
    #!/bin/bash
	dpkg -l software-properties-common
  error: 没有安装 software-properties-common
- name: check pkg
  script: |
    #!/bin/bash
	dpkg -l python-certbot-nginx
  error: 没有安装 python-certbot-nginx
```

由于我们前面选择的是 nginx，所以这里安装的插件是 `python-certbot-nginx`。这个插件依赖于 certbot，所以 certbot 也会被一并安装。

#### 3.1.3 申请证书和配置 Web 服务

certbot 工具安装好之后就可以用它来申请证书。由于安装了 nginx 插件，我们可以使用 `sudo certbot --nginx` 命令来申请证书并自动配置 nginx，`--nginx` 选项表示要使用 nginx 插件。不过为了更好的了解幕后工作，下面我们不使用 nginx 插件，自己手动来配置。

第一步，执行下面的命令来开启交互界面。

```bash
$ sudo certbot certonly --authenticator standalone --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx"
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5898timestamp1528190731717.png/wm)

`--authenticator` 选项指明我们采用 `standalone` 的方式来跟支持 ACME（自动证书管理环境）协议的 Let's Encrypt CA 服务通信。采用该种方式时，certbot 会自己启动一个临时的 Web 服务器，而不是依赖于系统里正在运行的，可以降低对系统的依赖。

`--pre-hook` 和 `--post-hook` 选项分别指定了在申请开始之前和结束之后要执行的操作。这里在开始之前我们停止了 nginx 服务，结束之后又恢复了它。在申请过程中 certbot 会自己启动一个 web 服务，如果不停止 nginx 服务，那么就会出现 80 和 443 端口冲突。具体要执行的命令依据系统环境而定，如果系统没有运行任何 web 服务，这两个选项可以省略。

第二步，输入和确认信息

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5898timestamp1528190731237.png/wm)

按照命令行提示输入相关信息，其中最重要的是要申请证书的域名（比如 www.shiyanlou.com），多个域名用逗号或空格分隔。Let's Encrypt 暂时不支持泛域名，所以需要为每一个子域名申请一张证书。需要注意的是，在申请过程中需要确保要申请证书的域名解析到执行 certbot 命令的这台服务器，以便 certbot 验证域名所有权。

如果是第一次运行工具，还需要提供邮箱信息，确认用户协议等。这里不再细述，大家可根据界面提示来输入或选择。

如果一切顺利，申请下来的证书将会保存在 `/etc/letsencrypt` 目录的 `live` 子目录下。

第三步，配置 Nginx

以 www.shiyanlou.com 这个域名为例，在 nginx 里可按照如下方式来配置 HTTPS。

```bash
server {
    listen      443 ssl;
    server_name www.shiyanlou.com;

    # 打开 SSL，并配置证书为前面申请下来的
    ssl on;
    ssl_certificate /etc/letsencrypt/live/www.shiyanlou.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.shiyanlou.com/privkey.pem;

    ...
}

server {
    listen      80;
    server_name www.shiyanlou.com;

    # 重定向所有 HTTP 请求为 HTTPS
    return 301 https://$host$request_uri;
}
```

其它 Web 服务的证书配置大家可以查阅官方文档。只要证书申请下来了，配置其实很简单。

### 3.2 Let's Encrypt CA 证书更新

Let's Encrypt 提供的 CA 证书有效期只有 90 天，我们需要在证书到期之前去更新证书。手动更新证书很繁琐，且容易遗忘。好消息是，certbot 默认就已配置好 crontab 定时任务来自动更新证书。

certbot 的定时任务配置在 `/etc/cron.d/certbot` 文件里。其内容如下：

```bash
$ cat /etc/cron.d/certbot
...

0 */12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(3600))' && certbot -q renew
```

可以看到大约每隔 12 个小时，会执行一次 `certbot -q renew` 命令来更新证书。当然只会更新即将或已经到期的，如果所有证书都有效，那就不执行任何操作。另外，执行更新时会记住当初申请证书时的选项，比如我们前面指定过的 --pre-hook 和 --post-hook，这样就不用我们手动去执行前置和后置操作，考虑得非常周到。

## 4. 实验总结

通过本次实验，我们学习到了 HTTPS 是什么和它的重要性，如何申请免费的 Let's Encrypt CA 证书，以及如何配置 Nginx 运行 HTTPS 服务。