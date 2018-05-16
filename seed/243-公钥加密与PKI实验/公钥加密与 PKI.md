#公钥加密与 PKI

##一、实验描述

本实验的学习目的是让学生熟悉公钥加密与 PKI 的概念，本课实验包含公钥加密，数字签名，公钥认证，认证授权，基于 PKI 授权等内容。学生将以使用工具或者写程序的方式建立基于 PKI 的安全信道。

##二、实验环境

本实验中，我们将使用 openssl 命令行工具及其库。实验环境中已自带命令行工具，需安装 openssl 开发库。
```python
$ sudo apt-get update
$ sudo apt-get install libssl-dev
```
编辑器使用 bless 十六进制编辑器,需预先安装。
```python
$ sudo apt-get install bless
```
系统用户名:  shiyanlou

一些有用的帮助文档：
1. [OpenSSL Command-Line HOWTO](http://www.madboa.com/geek/openssl/)
2. [OpenSSL Examples (client/server examples) with detailed explanation](http://www.rtfm.com/openssl-examples/)
3. [An Introduction to OpenSSL Programming (Part I)](http://www.rtfm.com/openssl-examples/part1.pdf)
4. [An Introduction to OpenSSL Programming (Part II)](http://www.rtfm.com/openssl-examples/part2.pdf)

##三、实验内容

###实验 1:成为数字证书认证机构（CA）
>数字证书认证机构（英语：Certificate Authority，缩写为 CA），也称为电子商务认证中心、电子商务认证授权机构，是负责发放和管理数字证书的权威机构，并作为电子商务交易中受信任的第三方，承担公钥体系中公钥的合法性检验的责任。

>数字证书的作用是证明证书中列出的用户合法拥有证书中列出的公开密钥。CA 机构的数字签名使得攻击者不能伪造和篡改证书。它负责产生、分配并管理所有参与网上交易的个体所需的数字证书，因此是安全电子交易的核心环节。

想要从商业 CA 获取数字证书需要向其支付一定的金钱，不过我们不用那么破费，可以自己成为 root CA，为自己发布证书。

在本实验中，我们将成为 root CA，并为该 CA 生成证书。不像其他 CA 需要被另外的 CA 认证， root CA 的证书是自己为自己认证的，一般 Root CA 的证书都已事先加载在大多数操作系统，浏览器或者依赖 PKI 的软件之中了。Root CA 的证书是被无条件信任的。

**配置文件：openssl.conf**

为了使用 openssl 生成证书，我们首先需要进行配置，配置文件扩展名为.cnf。openssl 的 ca， req， x509 命令常常会用到这份配置文件。你可以从/usr/lib/ssl/openssl.cnf 获得一份拷贝。将文件拷贝到工作目录后，创建以下在配置文件中指定的子文件夹（详情查看配置文件[CA default]处）
```
dir = ./demoCA # Where everything is kept
certs = $dir/certs # Where the issued certs are kept
crl_dir = $dir/crl # Where the issued crl are kept
new_certs_dir = $dir/newcerts # default place for new certs.
database = $dir/index.txt # database index file.
serial = $dir/serial # The current serial number
```
![图片描述信息](https://doc.shiyanlou.com/userid8834labid777time1427765226433/wm)

这里给出文件目录树：
![图片描述信息](https://doc.shiyanlou.com/userid8834labid777time1427765611521/wm)

index.txt 只要创建空文件就行，至于 serial 文件，内容必须是字符串格式的数字（比如 1000）

一旦你设置好 openssl.cnf 就可以创建和发布证书了。


**数字证书认证机构（CA）**
我们需要为自己的 CA 生成自签名证书。这意味着该机构是被信任的，而它的证书会作为 root 证书。你可以运行以下命令为 CA 生成自签名证书：

    $ openssl req -new -x509 -keyout ca.key -out ca.crt -config openssl.cnf

它要求你提供信息和密码，可千万别忘记密码了，因为每次为别人签证书的时候你都需要用到密码。信息包括城市名，通用名等等。命令的输出存储在两个文件中：ca.key 与 ca.crt 中。文件 ca.key 包括 CA 的私钥，而 ca.crt 包含了公钥证书

###实验 2: 为 PKILabServer.com 生成证书


现在，我们是 root CA 了，我们已经准备好为我们的客户签数字证书，我们的第一个客户是叫作 PKILabServer.com 的公司，这家公司从 CA 获取数字证书需要 3 个步骤：

**Step 1：生成公开/私有密钥对**

该公司首先需要生成它自己的公开/私有密钥对。我们运行以下命令来生成 RSA 密钥对。你同时需要提供一个密码来保护你的密钥。密钥会被保存在 server.key 文件中。

    $ openssl genrsa -des3 -out server.key 1024

**Step 2:   生成证书签名请求**
一旦公司拥有了密钥文件，它应当生成证书签名请求（CSR）。CSR 将被发送给 CA，CA 会为该请求生成证书（通常在确认 CSR 中的身份信息匹配后）。**请将 PKILabServer.com 作为证书请求的通用名，并且请记住自己都输了些啥**。

    $ openssl req -new -key server.key -out server.csr -config openssl.cnf


**Step 3: 生成证书**
生成证书。CSR 文件需要拥有 CA 的签名来构成证书。在现实世界中，CSR 文件常常被发送给可信任的 CA 签名。本实验中，我们将使用我们自己的 CA 来生成证书：

    $ openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key -config openssl.cnf

如果 OpenSSL 拒绝生成证书，那很可能是因为你请求中的名字与 CA 所持有的不匹配。匹配规则在配置文件中指定([policy match]处)，你可以更改名字也可以更改规则。都做到这了，就改规则吧。

![图片描述信息](https://doc.shiyanlou.com/userid8834labid777time1427766650462/wm)


###实验 3：在网站中使用 PKI

在本节实验中，我们将探索公钥证书是如何保障网络浏览的安全的。首先我们需要一个域名，比如 PKILabServer.com 为了让计算机识别该域名，在/etc/hosts 文件中添加以下内容：
    
    127.0.0.1 PKILabServer.com

该文件将指定域名映射到指定 ip 地址上，这里我们将它映射到本地 IP。接下来，让我们启动一个拥有我们之前生成的证书的简单的 web 服务器。使用以下命令开启一个 web 服务器：
    
    # Combine the secret key and certificate into one file
    $ cp server.key server.pem
    $ cat server.crt >> server.pem
    # Launch the web server using server.pem
    $ openssl s_server -cert server.pem -www


默认情况下，服务器会监听 4433 端口。所以你现在可以通过 https://PKILabServer.com:4433/ 访问服务器了。你很可能从浏览器收到一条错误消息 

![图片描述信息](https://doc.shiyanlou.com/userid8834labid777time1427768852727/wm)

如果该证书是被 VeriSign 之类的 CA 授权的话，我们就不会收到错误信息了，因为火狐内置了该 CA 的证书。可惜 PKILabServer.com 的证书是被我们自己的 CA 签名，火狐不认识我们的 CA。有两种方法能够让火狐接受我们的自签名证书。


1. 我们可以要求 Mozilla 在火狐里加入我们的证书，这样每一个用火狐的人都能够识别我们的 CA 了。这就是一般 CA 使用的方法，对我们来说自然是行不通的。
2. 我们可以手动加载我们 CA 的证书通过以下步骤：
  Edit -> Preference -> Advanced -> View Certificates. 
  你将看到一列证书列表，在这里我们导入我们的证书。 导入 ca.crt 并且选择 “Trust this CA to identify web sites” 。

![图片描述信息](https://doc.shiyanlou.com/userid8834labid777time1427769016477/wm)

现在，浏览 https://PKILabServer.com:4433 请描述并解释你的观察。如果仍旧出现错误页面，请使用 opera 浏览器。

接下来做这两件事：
1.  修改 server.pem 的一个字节并且重启服务器，访问 https://PKILabServer.com:4433 ，你观察到了些什么？确保之后恢复 server.pem.服务器可能无法重启，那样的话，换一个字节做修改。
2.  既然 PKILabServer.com 指向 localhost，如果我们使用 https://localhost:4433 替代域名访问，就会连接到同一个服务器。请尝试这么做并解释你的观察。

###实验 4：使用 PKI 与 PKILabServer.com 建立安全 TCP 链接

在本实验中，我们将实现 TCP 客户端，服务器端及其安全连接。也就是说，客户端与服务器端使用一个只有他们自己知道的会话密钥对通信内容进行加密。另外，客户端需要确保连接的是目标服务器（PKILabServer.com）也就是是说客户端需要给服务器端授权。服务器授权需要公钥证书，OpenSSL 已经实现了 SSL 协议来做服务器授权。OpenSSL 的 SSL 函数直接在客户端与服务器端之间建立连接，证书的认证由 SSL 函数自动完成。可以先参考以下教程学习 SSL 函数。
+ OpenSSL examples: http://www.rtfm.com/openssl-examples/
+ http://www.ibm.com/developerworks/linux/library/l-openssl.html


我们提供两个示例程序，cli.cpp 与 serv.cpp 帮助你理解。该程序演示了如何建立 SSL 连接，如何得到对方的证书，如何认证证书，如何从证书中获取信息等等。为了让程序能够运行，你需要先解包然后运行 make 命令，README 文件中记录了客户端和服务器端的密码。你可以以我们提供的文件为基础完成实验。

![图片描述信息](https://doc.shiyanlou.com/userid8834labid790time1427791010367/wm)

![图片描述信息](https://doc.shiyanlou.com/userid8834labid790time1427791033357/wm)

> 文件地址：
> http://labfile.oss.aliyuncs.com/courses/243/demo_openssl_api.zip
> 可以使用 wget 进行下载

###实验 5：性能比较 RSA vs AES

本节实验中，我们将考察公钥加密算法的性能。请准备一个文件（message.txt），该文件包含 16 字节的信息，再生成一个 1024 位的 RSA 密钥对，进行如下步骤：
1. 使用公钥加密 message.txt，将其保存在 message enc.txt 中
2. 使用私钥对 message enc.txt 进行解密。
3. 使用 AES-128 对文件加密
4. 比较每一步所用的时间，描述你的观察。如果其中一步耗时太短，需要多次实验求平均值。

    % openssl genrsa -out mykey.pem 1024
    % openssl rsa -in mykey.pem -pubout -out mypubkey.pem
    % time openssl rsautl -encrypt -in message.txt -pubin -inkey mypubkey.pem -out message.enc
    % time openssl rsautl -decrypt -in message.enc -inkey mykey.pem -out message_decryped.txt
    % time openssl enc -aes-128-cbc -in message.txt -out message.aes

在你完成了上述练习后，可以使用 openssl 的 speed 命令做基准测试，请描述你之前的观察是否与 speed 的结果相近。下面给出基准测试的方法：

    % openssl speed rsa
    % openssl speed aes

###实验 6：创建数字签名
本节实验中，我们将使用 OpenSSL 生成数字签名。请准备一个任意大小的 example.txt 文件。并准备 RSA 密钥对。步骤如下
1. 对 example.txt 产生的数字摘要使用 sha256 进行签名；输出文件 example.sha256。
2. 验证 example.sha256 的数字签名。
3. 稍稍修改 example.txt 再一次验证数字签名

请描述你的具体操作（比如用到哪些命令）解释你的观察。并解释为什么数字签名有效。

```
$ openssl genrsa -des3 -out myrsaCA.pem 1024
$ openssl rsa -in myrsaCA.pem -pubout -out myrsapubkey.pem
$ openssl dgst -sha256 -out example.sha256 -sign myrsaCA.pem example.txt
$ openssl genrsa -des3 -out myrsaCA.pem 1024
$ openssl dgst -sha256 -signature example.sha256 -verify myrsapubkey.pem example.txt
```


##四、作业
###按要求完成实验内容并回答每节实验给出的问题。

##版权声明
本课程所涉及的实验来自 Syracuse SEED labs，并在此基础上为适配实验楼网站环境进行修改，修改后的实验文档仍然遵循 GNU Free Documentation License。

本课程文档 github 链接：https://github.com/shiyanlou/seedlab

附 Syracuse SEED labs 版权声明：
>Copyright Statement Copyright 2006 – 2014 Wenliang Du, Syracuse University. The development of this document is funded by the National Science Foundation’s Course, Curriculum, and Laboratory Improvement (CCLI) program under Award No. 0618680 and 0231122. Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.2 or any later version published by the Free Software Foundation. A copy of the license can befound at http://www.gnu.org/licenses/fdl.html.

