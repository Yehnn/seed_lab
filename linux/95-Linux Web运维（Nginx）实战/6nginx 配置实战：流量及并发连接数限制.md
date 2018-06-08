---
show: step
version: 0.1
enable_checker: true
---

# nginx 配置实战：流量及并发连接数限制

## 一、实验介绍
#### 1.1 实验内容
本节实验通过nginx的限制模块对于下载一个mp4文件的下载速度进行限制  

#### 1.2 实验知识点
- 流量限制
- 并发限制

#### 1.3 文件获取
在这之前，我们需要先下载一个 mp4 文件，供后面实验测试使用哦
先建立一个 Documents 目录
```
cd
mkdir Documents
cd Documents
```
再使用命令：
```
git clone https://github.com/shiyanlou/seven
```
```checker
- name: check dir
  script: |
    #!/bin/bash
    ls /home/shiyanlou/Documents
  error: 没有新建 Documents 文件夹
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/Documents/seven
  error: /home/shiyanlou/Documents/ 目录下没有 seven 文件夹
```

## 二、实验原理

下面介绍实验有关原理。

### 2.1 实现流量限制具体配置

####（1）指令使用及配置修改

首先要清楚，我们的配置是基于 `ngx_http_core_module`  模块的。要简单完成流量限制，我们必须做一点前期准备。我们刚才已经在下载测试使用包，现在要做的，就是安装 nginx，上几章的讲解已经很详细了，这里我们选择源码安装。（前面的实验中我们已经编译安装了）

```
wget http://labfile.oss.aliyuncs.com/nginx-1.7.9.tar.gz  
```

接下来就是解压，编译，安装，前两章着重讲过。

下面我们就修改配置文件，前几章我们讲过对于 nginx 来说，最重要的莫过于他自己的配置文件 `nginx.conf`。文件路径为`/usr/local/nginx/conf/nginx.conf`，我们需要在 `http` 的 `server` 中加一个 `location`,就好比之前我们做的简单的 `echo` 模块，格式为：

```
location /刚下好的测试文件夹/ {
    root /刚才下载测试文件的目录/;
    ...
    ...
    ...
    }
```

接下来就是调用命令实现流量限制。nginx 的模块中早就写好了对应的命令，我们只需要调用就好。

实现流量限制由两个指令 limit_rate 和 limit_rate_after 共同完成：

- **limit_rate**

语法：limit_rate rate;

默认值：limit_rate 0;

作用域：http, server, location, if in location

命令概述：限制向客户端传送响应的速率限制。参数 rate 的单位是字节/秒，设置为 0 将关闭限速。 nginx 按连接限速，所以如果某个客户端同时开启了两个连接，那么客户端的整体速率是这条指令设置值的 2 倍。

- **limit_rate_after**

语法：limit_rate_after size;

默认值：limit_rate_after 0;

作用域：http, server, location, if in location


设置不限速传输的响应大小。当传输量大于此值时，超出部分将限速传送。

作用范围：http，server，location，if inlocation   

要是想了解更多，这里有更加详细的命令信息
  http://wiki.nginx.org/HttpCoreModule#limit_rate 

讲了要使用的指令。我们接下来就实战配置

**这是实验的具体配置（传输量限制为 3m，速率限制为 20k/s）：**

```nginx
location /seven/{
                root /home/shiyanlou/Documents;
                limit_rate_after 3m;
                limit_rate 20k;
}
```

代码截图如下：

![enter image description here](https://doc.shiyanlou.com/userid20406labid443time1422698113007/wm)

```checker
- name: check content
  script: |
    #!/bin/bash
    grep -E "seven|20k|3m" /usr/local/nginx/conf/nginx.conf
  error: 没有配置 /usr/local/nginx/conf/nginx.conf
```

配置完以后记得重新启动 nginx（注意这是编译安装的）

```
cd /home/shiyanlou/nginx-1.7.9/objs
sudo ./nginx
```

####（2）配置之前与配置之后的测试
接下来就是测试

```
cd
wget http://本机地址/seven/seven.mp4
```

本机地址用 ifconfig 来查看

修改配置之前，速率没有限制：

![enter image description here](https://doc.shiyanlou.com/userid20406labid443time1422698167855)

修改配置之后，可见由于传输量大于 `3m`，超出部分的传输速率已经被限制在 `20k/s`：

![enter image description here](https://doc.shiyanlou.com/userid20406labid443time1422698187952)

也许你已经发现配置之后**刚开始**的时候传输速度很高，因为，传输量大于设定值的部分才会受到限制。这就说明，我们两个命令都发挥了作用，这也就完成了我们第一个实战。


### 2.2 实现并发连接数限制的具体配置

####（1）指令使用及配置文件修改

这个的配置是基于 ngx_http_limit_zone_module 模块的,要简单完成并发限制，我们要涉及到 limit_conn_zone 和 limit_conn 这两个指令：

- **limit_conn_zone**

语法： limit_conn_zone zone_name $variable the_size 

默认值： no 

作用域： http 

本指令定义了一个数据区，里面记录会话状态信息。
variable 定义判断会话的变量；the_size 定义记录区的总容量。

- **limit_conn**

语法： limit_conn zone_name the_size 

默认值： no 

作用域： http, server, location 

指定一个会话最大的并发连接数。 当超过指定的最发并发连接数时，服务器将返回 "Service unavailable" (503)。 

配置示例：

```
http {
    limit_conn_zone   $binary_remote_addr  zone=one:10m;
    ...
    server {
        ...
        location /seven/ {
            limit_conn   one  1;
            .....
        }

```


- 定义一个叫“one”的记录区，总容量为 `10M`，以变量`$binary_remote_addr `作为会话的判断基准（即一个地址一个会话）。 限制 /seven/ 目录下，一个会话只能进行一个连接。 简单点，就是限制 /seven/ 目录下，一个 `IP` 只能发起一个连接，多过一个，一律 `503`。 

- 你可以注意到了，在这里使用的是 `$binary_remote_addr` 而不是 `$remote_addr`。`$remote_addr` 的长度为 7 至 `15 bytes`，会话信息的长度为 `32` 或 `64 bytes`。 而 
  `$binary_remote_addr` 的长度为 4 bytes，会话信息的长度为 32 bytes。 
  当 `zone` 的大小为 1M 的时候，大约可以记录 32000 个会话信息（一个会话占用 32 bytes）。 


- 由于环境的原因这里没有办法测试只是让大家了解，限制并发连接数 nginx 的对应模块的配置，如果大家有需要，可以进一步参照 http://wiki.nginx.org/HttpLimitZoneModule 这里有更详细的讲解。


## 三、实验总结

`如果提示404 not found怎么处理`
404 not found 表明文件没有找到，你看看seven.mp4是放在哪的

如果你的配置文件是这样写的话
```
        location /seven/{
            root /home/shiyanlou/Documents;
            limit_rate_after 3m;
            limit_rate 10k;
        }
```
wget是这样
```
wget http://localhost/seven/seven.mp3
```
那么当你访问`seven/`，`nginx`以`/home/shiyanlou/Documents`作为根目录去寻找`seven/seven.mp3`  

所以mp3文件应该存放在  

`/home/shiyanlou/Documents/seven/seven.mp3`

在这一节中，我们了解到要想实现流量及并发连接数限制，我们只需要清楚必须对应哪些模块，添加哪些命令，而这些命令早就已经写好。我们只需调用，如果有必要再深究这些命令的的实现方法。

####练习

按照所给步骤实现流量限制。
