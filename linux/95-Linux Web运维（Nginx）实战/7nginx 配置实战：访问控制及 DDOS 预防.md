---
show: step
version: 0.1
enable_checker: true
---

# nginx 配置实战：访问控制及 DDOS 预防

##一、实验介绍
#### 1.1 实验内容
本节实验讲解了nginx的访问控制，可以有效的防治ddos工具，并有ab命令做压力测试,大家不要觉的难，其实我们要改的东西比较简单，只是前面在进行原理讲解，可以直接看实验步骤，然后回头看指令的用法

#### 1.2 实验知识点
- nginx访问控制配置
- ab命令
- log排查

#### 1.3 适合人群
本节实验设计了更多的关于nginx的配置内容，更加深入的了解了nginx的功能和配置，适合运维人员或者感兴趣的同学学习

## 二、实验原理

下面介绍有关实验的原理。

### 2.1 访问控制配置

基于各种原因，我们要进行访问控制。比如说，一般网站的后台都不能让外部访问，所以要添加 IP 限制，通常只允许公司的 IP 访问。访问控制就是指只有符合条件的 IP 才能访问到这个网站的某个区域。

涉及模块：ngx\_http\_access\_module

模块概述：允许限制某些 IP 地址的客户端访问。

对应指令：

- **allow**

语法:     allow address | CIDR | unix: | all;

默认值:     —

作用域:     http, server, location, limit_except

允许某个 IP 或者某个 IP 段访问。如果指定 unix，那将允许 socket 的访问。注意：unix 在 1.5.1 中新加入的功能，如果你的版本比这个低，请不要使用这个方法。

- **deny**

语法:     deny address | CIDR | unix: | all;

默认值:     —

作用域:     http, server, location, limit_except

禁止某个 IP 或者一个 IP 段访问。如果指定 unix，那将禁止 socket 的访问。注意：unix 在 1.5.1 中新加入的功能，如果你的版本比这个低，请不要使用这个方法。

配置范例：

```
location / {
    deny  192.168.1.1;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    allow 2001:0db8::/32;
    deny  all;
}
```

规则按照顺序依次检测，直到匹配到第一条规则。 在这个例子里，IPv4 的网络中只有 10.1.1.0/16 和 192.168.1.0/24 允许访问，但 192.168.1.1 除外；对于 IPv6 的网络，只有 2001:0db8::/32 允许访问。 

ngx\_http\_access\_module 配置允许的地址能访问，禁止的地址被拒绝。这只是很简单的访问控制，而在规则很多的情况下，使用 ngx\_http\_geo\_module 模块变量更合适。这个模块大家下来可以了解 : [ngx_http_geo_module](http://nginx.org/en/docs/http/ngx_http_geo_module.html)

### 2.2 DDOS 预防配置

DDOS 的特点是分布式，针对带宽和服务攻击，也就是四层流量攻击和七层应用攻击，相应的防御瓶颈四层在带宽，七层的多在架构的吞吐量。对于七层的应用攻击，我们还是可以做一些配置来防御的，使用 nginx 的 http\_limit\_conn 和 http\_limit\_req 模块通过限制连接数和请求数能相对有效的防御。

ngx_http_limit_conn_module 可以限制单个 IP 的连接数

ngx_http_limit_req_module 可以限制单个 IP 每秒请求数

**配置方法：** 

#### **2.2.1 限制每秒请求数** 

ngx_http_limit_req_module模块通过漏桶原理来限制单位时间内的请求数，一旦单位时间内请求数超过限制，就会返回503错误。配置需要在两个地方设置：

nginx.conf的http段内定义触发条件，可以有多个条件
在location内定义达到触发条件时nginx所要执行的动作

```
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
    ...
    server {
        ...
        location  ~ \.php$ {
            limit_req zone=one burst=5 nodelay;  
               }
           }
     }
```
`参数说明`  
- `$binary_remote_addr`  二进制远程地址,这个参数就些这个就好了，不需要改  
- `zone=one:10m`    定义zone名字叫one，并为这个zone分配10M内存，用来存储会话（二进制远程地址），1m内存可以保存16000会话
- `rate=10r/s;`     限制频率为每秒10个请求
- `burst=5`   允许超过频率限制的请求数不多于5个，假设1、2、3、4秒请求为每秒9个，那么第5秒内请求15个是允许的，反之，如果第一秒内请求15个，会将5个请求放到第二秒，第二秒内超过10的请求直接503，类似多秒内平均速率限制。
- `nodelay`         超过的请求不被延迟处理，设置后15个请求在1秒内处理。

#### **2.2.2 限制 IP 连接数**

上一章讲过，我们就直接写出来

```
http {
    limit_conn_zone $binary_remote_addr zone=addr:10m; //触发条件
    ...
    server {
        ...
        location /download/ {
            limit_conn addr 1;    // 限制同一时间内1个连接，超出的连接返回503
                }
           }
     }
```
####  **2.2.3 白名单设置** 

http\_limit\_conn 和 http\_limit\_req 模块限制了单 IP 单位时间内的连接和请求数，但是如果 Nginx 前面有 lvs 或者 haproxy 之类的负载均衡或者反向代理，nginx 获取的都是来自负载均衡的连接或请求，这时不应该限制负载均衡的连接和请求，就需要 geo 和 map 模块设置白名单：

```
geo $whiteiplist  {
        default 1;
        10.11.15.161 0;
    }
map $whiteiplist  $limit {
        1 $binary_remote_addr;
        0 "";
    }
limit_req_zone $limit zone=one:10m rate=10r/s;
limit_conn_zone $limit zone=addr:10m;

```

geo 模块定义了一个默认值是 1 的变量 whiteiplist，当在 ip 在白名单中，变量 whiteiplist 的值为 0，反之为 1

- 下面是设置的逻辑关系解释：

如果在白名单中--> whiteiplist=0 --> $limit="" --> 不会存储到 10m 的会话状态（one 或者 addr）中 --> 不受限制；

反之，不在白名单中 --> whiteiplist=1 --> $limit=二进制远程地址 -->存储进 10m 的会话状态中 --> 受到限制。


## 三、实验步骤
`动手`测试 DDOS 预防配置

下面我们就来测一下刚刚的配置是否起到了作用。

### 3.1 安装所有测试所需的软件
（apt 安装 php5-fpm，apt 安装 apache2-utils）

```
sudo apt-get update # 更新环境
sudo apt-get install apache2-utils
```

```
sudo apt-get install php5-fpm
```

安装好以后 分别启动 php5 和 nginx。

```
sudo service nginx start
sudo service php5-fpm start
```

```checker
- name: check service
  script: |
    #!/bin/bash
    ps -ef |grep -v grep|grep nginx
  error: 没有启动 nginx
- name: check php
  script: |
    #!/bin/bash
    cat /etc/nginx/sites-available/default|grep -v '#'|grep php
    nginx -t
  error: /etc/nginx/sites-available/default 没有配置 php
- name: check service
  script: |
    #!/bin/bash
    ps -ef|grep -v grep|grep php5
  error: 没有启动php5-fpm
```

### 3.2 测试访问

写一个测试的 php 文件，修改 nginx 配置文件，使其能正常访问。

在`/usr/share/nginx/html`目录下写一个 test.php,内容如下：

![图片描述信息](https://doc.shiyanlou.com/userid20406labid453time1422935624797/wm)

![图片描述信息](https://doc.shiyanlou.com/userid20406labid453time1422935650428/wm)

> 如果显示 `404 notfound` 则是因为之前配置过端口号 9000，在访问时输入 `localhost:9000/test.php` 即可。后文同理。

### 3.3 修改配置


`修改/etc/nginx/nginx.conf`
```
http {

        ##
        # Basic Settings
        ##
        limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s; //加入这行
        ....
        ....
        
```

`修改/etc/nginx/sites-available/default`
```
        location ~ \.php$ {
                limit_req zone=one burst=5 nodelay;   //加入这行
                root /usr/share/nginx/html;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                # NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini

                # With php5-cgi alone:
                #fastcgi_pass 127.0.0.1:9000;
                # With php5-fpm:
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                include fastcgi_params;
        }

```

### 3.4 测试

修改配置文件后，使用命令 ab 测试，。

```
shiyanlou:/$ ab -n 10 -c 10  http://localhost/test.php
```
其中-n代表每次并发量，-c代表总共发送的数量。这条命令表示10个请求一次并发出去

测试结果：
```
Benchmarking localhost (be patient).....done


Server Software:        nginx/1.4.6
Server Hostname:        localhost
Server Port:            80

Document Path:          /test.php           ###请求的资源
Document Length:        16 bytes            ###文档返回的长度，不包括相应头

Concurrency Level:      10                   ###并发个数
Time taken for tests:   0.006 seconds        ###总请求时间
Complete requests:      10               ###总请求数
Failed requests:        4                ###失败的请求数
   (Connect: 0, Receive: 0, Length: 4, Exceptions: 0)
Non-2xx responses:      10
Total transferred:      2744 bytes
HTML transferred:       980 bytes
Requests per second:    1645.55 [#/sec] (mean)   ###平均每秒的请求数
Time per request:       6.077 [ms] (mean)        ###平均每个请求消耗的时间
Time per request:       0.608 [ms] (mean, across all concurrent requests) ###上面的请求除以并发数
Transfer rate:          440.96 [Kbytes/sec] received  ###传输速率

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        1    4   1.7      5       5
Processing:     1    2   1.7      1       5
Waiting:        0    1   1.3      1       5
Total:          5    6   0.0      6       6

Percentage of the requests served within a certain time (ms)
  50%      6            ###50%的请求都在6Ms内完成
  66%      6
  75%      6
  80%      6
  90%      6
  95%      6
  98%      6
  99%      6
 100%      6 (longest request)
```

然后我们查看log,可以看出我们并发了10条请求，有4条访问失败
```
sudo tail /var/log/nginx/error.log  

2017/03/08 11:28:50 [error] 1430#0: *161 limiting requests, excess: 5.995 by zone "one", client: ::1, 
2017/03/08 11:28:50 [error] 1430#0: *162 limiting requests, excess: 5.995 by zone "one", client: ::1, 
2017/03/08 11:28:50 [error] 1430#0: *163 limiting requests, excess: 5.995 by zone "one", client: ::1, 
2017/03/08 11:28:50 [error] 1430#0: *164 limiting requests, excess: 5.995 by zone "one", client: ::1, 
```


## 四、实验总结
在这一章中我们讲述了要想实现访问控制和 DDOS 的防御，我们就要学会两个相应模块的配置及指令的使用，这些都是现成写好的模块与指令，我们之需要了解什么需求对应该的什么模块，运用哪些指令就好。

在我们遇到问题的时候，最重要的就是错误的排查，也就是查看log，log是对于一个运维人员非常非常重要的。

####练习
依照实验步骤实现访问限制配置
