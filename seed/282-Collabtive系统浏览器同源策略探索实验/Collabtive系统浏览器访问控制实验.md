---
show: step
version: 0.1
enable_checker: true
---

# Collabtive系统浏览器访问控制实验

## 一、实验简介 

现在的web浏览器的安全模型是基于同源策略，并提供一些基于Web应用程序的保护功能；这个实验的目的是帮助大家对同源策略有一个很好的理解，这将对我们学习跨站脚本攻击和跨站请求伪造有很大帮助。

## 二、实验背景 

Web浏览器本质上是用户代理，它们代表其用户与Web站点/ Web应用交互。通常，用户使用Web浏览器访问一个网站时，网络浏览器代表用户将HTTP请求转发到网站，并显示网站在响应中返回的网页。 Web浏览器使用称为同源策略（SOP）的安全模型来对Web应用程序实施一些访问限制。

同源策略使用其来源识别每个网站，它是协议，域名，端口的独特组合。对于每个来源，web浏览器都会创建一个上下文，并将源自上下文的Web应用程序资源存储在上下文中。不允许来自一个来源的Java程序从另一个来源访问资源。

Cookie和文档对象模型（DOM）对象是应用了同源策略的Web应用程序资源的示例。此外，JavaScript程序可能使用XMLHttpRequest API向Web应用程序发送HTTP请求。SOP也扩展到使用XMLHttpRequest API。首先，我们将提供关于cookie，DOM对象和XMLHttpRequest API的一些背景知识。然后，我们描述将引导学生研究SOP以及它如何影响cookies，DOM对象和XMLHttpRequest API使用的实验室任务。

## 三、预备知识 

### 1、什么是同源策略

同源：如果两个页面使用相同的协议(protocol)，端口和主机（域名），那么这两个页面就属于同一个源。

同源策略：限制了一个源(origin)中加载文本或脚本与来自其他源中资源的交互方式；是客户端脚本（尤其是Javascript）重要的安全度量标准。

**重点：同源策略是浏览器最基本的安全策略，它认为任何站点的信赖内容都是不安全的，所以当脚本运行的时候，只允许脚本访问来自同一站点的资源，而不是那些来自其它站点可能怀有恶意的资源；**

>拓展：单源策略（Single Origin Policy），它是一种用于Web浏览器编程语言（如JavaScript和Ajax）的安全措施，以保护信息的保密性和完整性。同源策略能阻止网站脚本访问其他站点使用的脚本，同时也阻止它与其他站点脚本交互。


### 2、什么是DOM&Cookie？

DOM：文档对象模型(Document Object Module)，是处理可扩展标志语言的标准编程接口。
详细请参考[DOM—百度百科](http://baike.baidu.com/link?url=vTFAJ02ScpoGHdNs9sDSuyWsnmDxWFtb0YCd32u1IqsnXYz0-SjoIMsuu9eYGye5U0Xrpe34QCOBDdGYn4V3Qcr0doxM3N4nlC55KnkOW0m)

Cookie：为了辨别用户身份，进行session跟踪而存储在用户本地终端上的数据(通常经常加密)。
详细请参考[Cookie—百度百科](http://baike.baidu.com/link?url=4Of8ifjwfzzNtWmrjGN1B4tBho2pogxWwys9wgYlREX6ehbQIIdvQPCVIvrI36EDgSAhMZF35shi5vPLUZIg__f8eckSFVfQNdkc9lmdlQi)

### 3、什么是 XMLHttpRequest

[XMLHttpRequest—百度百科](http://baike.baidu.com/link?url=mEnP_BFD77g3L_dGEhV7rDTJNM9D_m-vNkEzFS0gD2f4hnRod-nHuirKC_n_oSThYUbd1hGlO8vWH4H2vb3VXq)

应用对象：后台与服务器交换数据；

>作用：

>1. 在不重新加载页面的情况下更新网页。

>2. 在页面已加载后从服务器请求数据。

>3. 在页面已加载后从服务器接受数据。

>4. 在后台向服务器发送数据。


### 4、环境搭建

在菜单中打开Terminal，启动服务：

```
sudo service apache2 start
```

```
sudo vim /etc/hosts
```

添加如下内容：

```
127.0.0.1 www.soplab.com
127.0.0.1 www.soplabattacker.com
127.0.0.1 www.soplabcollabtive.com
```

>按 `i` 进入编辑模式
>按`Esc`退出编辑
>输入`:wq `退出并保存

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid8797labid877timestamp1525863683439.png)

每个网址对应的网站根目录如下表：

| URL                             | Description | Directory                      |
| ------------------------------- | ----------- | ------------------------------ |
| http://www.soplab.com           |             | /var/www/SOP/                  |
| http://www.soplabattacker.com   | Attacker    | /var/www/SOP/attacker/         |
| http://www.soplabcollabtive.com | Collabtive  | /var/www/SOP/soplabCollabtive/ |

```checker
- name: check hosts
  script: |
    #!/bin/bash
    grep soplab /etc/hosts
    grep soplabattacker /etc/hosts
    grep soplabcollabtive /etc/hosts
  error: 没有配置/etc/hosts文件
- name: check apache2
  script: |
    #!/bin/bash
    ps -ef |grep -v grep|grep apache2
  error: 没有启动apache2
```

## 四、实验内容 

### lab1 理解DOM和Cookie 

本实验中，我们通过编写代码来了解DOM和Cookie

#### 使用DOM API来显示html的子节点h1的内容  

```
sudo vim /var/www/SOP/first.html
```

```
<html>

<head>
    <title>Self-modifying HTML</title>
    <script>
        function appendp() //添加h1内容
        {
            var h1_node = document.createElement("h1");
            h1_node.innerHTML = "Self-modifying HTML Document";
            document.childNodes[0].childNodes[2].appendChild(h1_node);
            var p_node = document.createElement("p");
            p_node.innerHTML = "This web page illustrates how DOM API can be used to modify a web page";
            document.childNodes[0].childNodes[2].appendChild(p_node);
        }

        function gethtmlchildren() //获取节点
        {
            var entiredoc = document.childNodes[0];
            var docnodes = entiredoc.childNodes;
            for (i = 0; i < docnodes.length; i++)
                alert(docnodes[i].nodeName);
        }
    </script>
</head>

<body name="bodybody">
    <script>
        appendp();
    </script>
    <input type="button" value="Display children of HTML tag" onclick=gethtmlchildren()>
</body>

</html>
```

使用浏览器访问 www.soplab.com/first.html 网址：

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid9094labid877time1429163411514?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

点击 Display children of HTML tag 按钮，显示html子节点：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971525932276053-wm)

[常见JavaScript获取DOM节点](http://www.cnblogs.com/seamar/archive/2011/07/25/2116197.html)

#### 通过LiveHttpHeader抓取Cookie

首先需要安装 livehttpheader 插件：点击浏览器的 tools-》add-ons ，在搜索框输入 live http header 就可以找到，点击 install 即可安装，安装后重启浏览器就可以了。我这里是已经安装了，截图如下：

![3.4-1](https://dn-simplecloud.shiyanlou.com/uid/8797/1524647286699.png-wm)

启动服务器（注意不要关闭启动服务的窗口）：

```
	sudo service apache2 restart
	sudo mysqld_safe
```

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid9094labid877time1429163896638?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

首先打开Tools内的的Live HTTP headers：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1525936693420.png-wm)

然后用firefox访问www.soplabcollabtive.com，再切换到 livehttpheaders 窗口查看 cookie：

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid9094labid877time1429163950470?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

#### 通过Cookie获取页面的访问量

访问：www.soplab.com/cookie.html 会提示输入你的信息，第二次进入以后就会判断你的Cookie并返回相应的网页；

![实验楼](https://dn-simplecloud.shiyanlou.com/87971525936942571-wm)![实验楼](https://dn-simplecloud.shiyanlou.com/87971525936960886-wm)![实验楼](https://dn-simplecloud.shiyanlou.com/87971525936977465-wm)

你可以通过查看源代码进行理解如何存储并处理Cookie。

下面我们来通过代码来实现“通过cookie获取页面访问次数”，这里我们新建一个count_cookie.html 在 /var/www/SOP 目录下：

```
sudo vim /var/www/SOP/count_cookie.html
```
输入下面代码：
```
    <script type="text/javascript"> 
    if(getCookie("num")){ 
    var nn=parseInt(getCookie("num")); 
    setCookie("num",++nn); 
    }else{ 
    setCookie("num",1); 
    } 
    function getCookie(name){ 
    var str=document.cookie.split(";"); 
    for(i=0;i<str.length;i++){ 
    var str2=str[i].split("=") 
    if(str2[0].replace(/\s(.*)\s/,"$1")==name){ 
    return str2[1]; 
    } 
    } 
    } 
    function setCookie(name,value){ 
    var Days=30; 
    var exp = new Date(); 
    exp.setTime(exp.getTime() + Days*24*60*60*1000); 
    document.cookie = name + "="+ escape(value) +";expires"+ exp.toGMTString(); 
    } 
    alert("您是第"+getCookie("num")+"次访问"); 
    </script>
```

再访问 `www.soplab.com/count_coolie.html` ，就会弹出访问次数了：

![实验楼](https://dn-simplecloud.shiyanlou.com/87971525939312626-wm)

```checker
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
 - name: check /var/www/SOP/first.html
   script: |
     #!/bin/bash
     ls /var/www/SOP/first.html
     grep appendp /var/www/SOP/first.html
   error: 不存在 /var/www/SOP/first.html 或者配置错误
- name: check /var/www/SOP/count_cookie.html
  script: |
     #!/bin/bash
     ls /var/www/SOP/count_cookie.html
     grep getCookie /var/www/SOP/count_cookie.html
   error: 不存在 /var/www/SOP/count_cookie.html 或者配置错误
```

### lab2 SOP的DOM与Cookie

本次实验是为了说明Web浏览器如何识别Web应用程序的来源，以及DOM对象和Cookie的访问限制；

#### 查看frame以及Cookie

访问 www.sqllab.com 网页，通过页面上的url地址栏访问www.soplab.com查看frame(是否同一个源，允许访问)

>同源：协议相同(ftp,http,https等等)；主机相同(www.xxx.cn/com等等)；端口相同(默认情况下都是80端口，但是有些域名绑定的不是80端口，这种情况属于不同源)；

先点击 go 按钮，然后点击 view source 按钮：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1525939880534.png-wm)

点击read cookie 按钮访问Cookie(允许访问，但是这个时候Cookie为空)

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid9094labid877time1429164220677?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

#### 从不同源的的url进行访问

在 url 地址栏输入 www.baidu.com ，然后点击 go 按钮，再点击 view source 按钮查看frame(因为主机名不同，所以同源策略不允许访问)

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid9094labid877time1429164390498?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

查看Cookie(因为主机名不同，所以同源策略不允许访问)

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid9094labid877time1429164592090?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

#### 设置不同端口进行访问

访问frame(因为端口不同，所以同源策略不允许访问)

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid9094labid877time1429164650461?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

查看Cookie(因为端口不同，所以同源策略不允许访问)

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid9094labid877time1429164713508?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

#### 不仅仅是Cookie和frame受到SOP约束，还有历史对象

不同源：历史对象同样受到限制

**最下面两幅图是有区别的（大家好好体会一下！）**

点击 back 发现同样不可访问同源策略。

![图片描述信息](https://dn-anything-about-doc.qbox.me/userid9094labid877time1429164806536?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

### SOP XMLHttpRequest

#### SOP是否扩展到HTTP请求的目标URL

在 url 栏输入 http://www.soplab.com/navigation.html 后再点击 go ，send request ，得到正确结果。

![实验楼](https://dn-simplecloud.shiyanlou.com/87971525940966369-wm)

通过验证可以表示，同源策略没有扩展到请求http url；

#### 描述通过XMLHttpRequest进行绕过同源策略

[科普](http://blog.csdn.net/shimiso/article/details/21830313)

#### SOP的例外

[科普](http://www.91ri.org/7330.html)

## 作业

你需要提交一份详细的实验报告，描述你所做的和你所观察到的。

使用的LiveHTTPHeaders的截图，请提供详细资料。

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






