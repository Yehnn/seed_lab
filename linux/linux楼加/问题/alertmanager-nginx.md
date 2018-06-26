# 挑战：使用 Alertmanager 来告警 Nginx 服务

## 介绍

上次挑战我们成功使用 Prometheus 来获取到了 Nginx 服务的各项度量指标。本次挑战我们更进一步，使用 Alertmanager 来在 Nginx 服务出现异常时告警。

## 目标

在 Nginx 服务出现 404 响应状态码的时候发送通知。

## 提示语

### 配置 Prometheus 监控 Nginx 服务

按照上次挑战要求配置即可。要求请求地址 `http://localhost:9090/api/v1/query?query=nginx_http_requests_total` 能够得到类似如下的响应结果：

```json
{"status":"success","data":{"resultType":"vector","result":[{"metric":{"__name__":"nginx_http_requests_total","host":"localhost","instance":"localhost:9145","job":"nginx","status":"200"},"value":[1526007699.725,"23"]}]}}
```

除了 value 属性取值可能不同，其它属性和取值应跟上面一致。

### 告警 Nginx 服务

警告的产生是在 Prometheus 里完成的，需要配置告警规则来在适当的时候产生警告。产生警告的条件为 `rate(nginx_http_requests_total{status="404"}[10s]) > 0.01`，也就是过去 10 秒钟平均每秒 404 状态请求数大于 0.01。

另外还要配置告警管理器的地址，以便警告能够发送到 Alertmanager。这可以在 Alertmanager 启动之后完成。

### 配置 Alertmanager 发送通知

警告到达 Alertmanager 之后，需要配置路由来发送通知。接收者的名字需要为 `shiyanlou`，并且接收通知的方式为 webhook。webhook 服务接收到通知后保存到本地文件 `/home/shiyanlou/notifications.txt` 里。

webhook 服务可使用下面的脚本来提供：

```bash
#!/bin/bash

while true; do
  echo -e "HTTP/1.1 200 OK\r\n\r\nOK\r\n" |
  nc -lp 8000 -q 1
  sleep 1
done
```

该脚本对任何 HTTP 请求都会打印其请求内容并响应“OK”。执行命令 `./http_ok_server.sh | tee notifications.txt` 会同时在控制台打印请求内容和保存请求内容到文件。当然你也可以自己实现 webhook 服务，只要能保存通知内容到指定文件里就可以。

## 知识点

1. Prometheus 监控 Nginx 服务
2. Prometheus 产生警告
3. Alertmanager 发送通知

**完成这个首先完成下面这个**

# 监控 Nginx 服务

## 介绍

本次挑战我们将运用前面学习的 Prometheus 知识来完成对 Nginx 服务的监控。

## 目标

通过 Prometheus 的 API 能够获取到 Nginx 服务的度量指标。

```bash
$ curl 'http://localhost:9090/api/v1/query?query=nginx_http_requests_total'
{"status":"success","data":{"resultType":"vector","result":[{"metric":{"__name__":"nginx_http_requests_total","host":"localhost","instance":"localhost:9145","job":"nginx","status":"200"},"value":[1525851831.374,"137"]}]}}
```

## 提示语

在 Nginx 服务里加载导出器来提供 Prometheus 度量指标服务端点，然后配置 Prometheus Server 从该服务端点抓取度量指标。导出器请使用这个 [nginx-lua-prometheus](https://github.com/knyar/nginx-lua-prometheus)。

### 安装支持运行 Lua 脚本的 Nginx

因为导出器是使用 Lua 编写的，需要 Nginx 支持运行 Lua 脚本。大部分发行版里默认安装的 Nginx 都不支持，需要自己编译安装。编译的时候需要指定编译模块 `nginx-lua`。Nginx 源码可以从 [官网](http://nginx.org/en/download.html) 下载，也可以从实验楼下载 [nginx-1.14.0.tar.gz](http://labfile.oss.aliyuncs.com/courses/980/05/assets/nginx-1.14.0.tar.gz)。

可通过 `nginx -V` 命令来查看编译参数，如果输出里包含 nginx-lua 字样则证明支持。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5955timestamp1528873097717.png/wm)

Ubuntu 系统可以安装 `nginx-extras` 包，这个版本的 Nginx 带有许多额外功能，包括运行 Lua 脚本。

```bash
$ sudo apt install nginx-extras
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5955timestamp1528873097368.png/wm)

### 配置 Nginx 加载导出器

参照导出器官方文档来配置 Nginx，注意 Lua 相关配置需要放在 `http` 配置块下。

### 配置 Prometheus 抓取 Nginx 度量指标

在 Prometheus 的配置里添加一个 job 来抓取 Nginx 度量指标。Prometheus Server 需要在默认的 9090 端口监听，以便检查结果。

## 知识点

1. Nginx Lua 脚本运行支持
2. Nginx Prometheus 度量指标导出器配置
3. Prometheus 抓取配置



# 解决方案

## 我的环境

- 系统：ubuntu18.04
- 软件源：阿里云

```bash
kinglion@ubuntu:~$ cat /etc/apt/sources.list
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic stable
# deb-src [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic stable
# deb-src [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic stable
deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu bionic stable
# deb-src [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu bionic stable
deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu zesty stable
# deb-src [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu zesty stable
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

$ sudo apt update
```

- prometheus：2.2.1
- alertmanager：0.15.0
- nginx：1.14.0

## 需要的软件

首先把需要准备的软件准备好：

- `prometheus`
- `alertmanager`
- `nginx`
- `nginx-extras` 
- 第三方 nginx 导出器 [nginx-lua-prometheus](https://github.com/knyar/nginx-lua-prometheus)

### prometheus

这里采用的二进制安装包方式

```bash
wget http://labfile.oss.aliyuncs.com/courses/980/05/assets/prometheus-2.2.1.linux-amd64.tar.gz
tar xvfz prometheus-2.2.1.linux-amd64.tar.gz
cd prometheus-2.2.1.linux-amd64
./prometheus
```

> *它的官网 https://prometheus.io/download/* 

上面的命令会在默认的 9090 端口启动 Prometheus 服务，打开地址 http://localhost:9090/ 即可看到其 Web 界面。

### alertmanager

同样采用二进制。

```bash
wget http://labfile.oss.aliyuncs.com/courses/980/05/assets/alertmanager-0.15.0-rc.1.linux-amd64.tar.gz
tar -zxvf alertmanager-0.15.0-rc.1.linux-amd64.tar.gz
cd alertmanager-0.15.0-rc.1.linux-amd64
cp simple.yml alertmanager.yml
./alertmanager
```

> *它的官网 https://prometheus.io/download/* 

`alertmanager` 命令默认会加载同目录下的配置文件 `alertmanager.yml`，如果找不到配置文件会报错。也可以通过 `--config.file` 参数来显示指定要使用的配置路径，支持相对路径。

启动之后，可通过地址 http://localhost:9093/ 来访问告警管理器的 Web 界面。

### nginx

```
sudo apt install -y nginx
```

### nginx-extras

```bash
sudo apt install -y nginx-extras
```

`nginx -V 2>&1 |grep luna` 查看可以看到就对了。

### nginx-lua-prometheus

从 github 克隆。

```bash
 git clone https://github.com/knyar/nginx-lua-prometheus.git
```

## 配置监控

- [prometheus.yml](#prometheus.yml)
- [nginx.conf](#nginx.conf)
- [配置 prometheus](#配置-prometheus)
- [测试](#测试)

### prometheus.yml

配置 Prometheus.yml 增加一个 nginx 的 job。 `~/prometheus-2.2.1.linux-amd64/prometheus.yml`

```
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'nginx'         
    static_configs:
      - targets: ['localhost:9145']
```

job 名为 nginx ，配置的端口为 9145

### nginx.conf

根据导出器 [nginx-lua-prometheus](https://github.com/knyar/nginx-lua-prometheus) 说明文档。 `/etc/nginx/nginx.conf` 

**注意是添加到 `http` 模块中的。** 

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
	worker_connections 768;
}

http {
	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_prefer_server_ciphers on;

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	gzip on;
	gzip_disable "msie6";

    lua_shared_dict prometheus_metrics 10M;
    lua_package_path "/home/shiyanlou/nginx-lua-prometheus/?.lua";
    init_by_lua '
        prometheus = require("prometheus").init("prometheus_metrics")
        metric_requests = prometheus:counter(
        "nginx_http_requests_total", "Number of HTTP requests", {"host", "status"})
        metric_latency = prometheus:histogram(
        "nginx_http_request_duration_seconds", "HTTP request latency", {"host"})
        metric_connections = prometheus:gauge(
        "nginx_http_connections", "Number of HTTP connections", {"state"})
    ';
    log_by_lua '
        local host = ngx.var.host:gsub("^www.", "")
        metric_requests:inc(1, {host, ngx.var.status})
        metric_latency:observe(tonumber(ngx.var.request_time), {host})
    ';

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```

###配置 prometheus

根据导出器说明文档。

新建 `/etc/nginx/sites-enabled/prometheus` 

```nginx
server {
  listen 9145;
  allow 127.0.0.1;
  deny all;
  location /metrics {
    content_by_lua '
      metric_connections:set(ngx.var.connections_reading, {"reading"})
      metric_connections:set(ngx.var.connections_waiting, {"waiting"})
      metric_connections:set(ngx.var.connections_writing, {"writing"})
      prometheus:collect()
    ';
  }
}
```

>  重启 nginx 是可以看到在监听 9145 端口的。
>
> ```bash
> kinglion@ubuntu:~$ netstat -lunat
> 激活Internet连接 (服务器和已建立连接的)
> Proto Recv-Q Send-Q Local Address           Foreign Address         State      
> tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
> tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN     
> tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
> tcp        0      0 0.0.0.0:9145            0.0.0.0:*               LISTEN     
> tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN     
> tcp        0      0 127.0.0.1:9145          127.0.0.1:60138         ESTABLISHED
> tcp        0      0 127.0.0.1:40328         127.0.0.1:9090          ESTABLISHED
> tcp        0      0 127.0.0.1:60138         127.0.0.1:9145          ESTABLISHED
> tcp        0      0 192.168.79.131:34970    52.34.196.165:443       TIME_WAIT  
> tcp        0      0 127.0.0.1:40334         127.0.0.1:9090          ESTABLISHED
> tcp6       0      0 :::9090                 :::*                    LISTEN     
> tcp6       0      0 :::9093                 :::*                    LISTEN     
> tcp6       0      0 :::9094                 :::*                    LISTEN     
> tcp6       0      0 :::80                   :::*                    LISTEN     
> tcp6       0      0 ::1:631                 :::*                    LISTEN     
> tcp6       0      0 127.0.0.1:9090          127.0.0.1:40328         ESTABLISHED
> tcp6       0      0 127.0.0.1:9090          127.0.0.1:40334         ESTABLISHED
> udp    42240      0 127.0.0.53:53           0.0.0.0:*                          
> udp     6528      0 0.0.0.0:68              0.0.0.0:*                          
> udp        0      0 0.0.0.0:631             0.0.0.0:*                          
> udp        0      0 0.0.0.0:52367           0.0.0.0:*                          
> udp    11968      0 0.0.0.0:5353            0.0.0.0:*                          
> udp6       0      0 :::39200                :::*                               
> udp6       0      0 :::9094                 :::*                               
> udp6   11008      0 :::5353                 :::*            
> ```
>
> nginx 配置可以用 sudo nginx -t 检查
>
> ```bash
> kinglion@ubuntu:~$ sudo nginx -t
> nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
> nginx: configuration file /etc/nginx/nginx.conf test is successful
> ```
>
> 这就是对的。
>
> ```
> sudo service nginx restart
> ```
>
> *补充：sudo service nginx reload* 重新加载配置 

### 测试

测试

```bash
sudo service nginx restart
cd ~/prometheus-2.2.1.linux-amd64
```

访问 nginx -> `localhost:80` 会出现 nginx 主页。

访问 `localhost:9090/metrics` 能看到一些内容。无法访问说明 nginx 的 Prometheus allow 配错了。或者用  `curl http:localhost:9090/metrics` 

![](http://ww1.sinaimg.cn/large/8f2bdb7fgy1fsiv2gys8oj20zh0i4q5n.jpg)

通过命令可以看到返回信息。

```
curl 'http://localhost:9090/api/v1/query?query=nginx_http_requests_total' | grep 
```

![](http://ww1.sinaimg.cn/large/8f2bdb7fgy1fsiv4v3qxtj20sz07lgn5.jpg)

通过页面 `localhost:9090` 查询语句 `nginx_http_connections` 

![](http://ww1.sinaimg.cn/large/8f2bdb7fgy1fsiv6vk4npj20zm0ip41k.jpg)

这样就是说明能监控成功，获取信息是对的。

## 配置告警

使用 alertmanager 配置警告。

- prometheus.yml
- alertmanager.yml
- webhook
- alerting.rules.yml
- 测试

### prometheus.yml

```yaml
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'nginx'
    static_configs:
      - targets: ['localhost:9145']

rule_files:
  - /home/kinglion/alerting.rules.yml

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```

> - 警告规则文件位置
> - alertmanagers 端口

### alertmanager.yml

位置 `~/alertmanager-0.15.0-rc.1.linux-amd64/alertmanager.yml ` 

```yaml
global:
... #意思是省略一些默认的内容，不输入这三个点
route:
  group_wait: 1s
  group_interval: 1s
  receiver: shiyanlou
  routes:
  - match:
      job: nginx
    group_by: ['job']
    receiver: shiyanlou

receivers:
- name: shiyanlou
  webhook_configs:
  - url: http://localhost:8000/
```

> 配置接收者和路由。这里采用的 webhook 接收通知。端口配置为 8000 。

### webhook

用一个简单脚本来提供 webhook 服务 `http_ok_server.sh` 

```bash
#!/bin/bash

while true; do
  echo -e "HTTP/1.1 200 OK\r\n\r\nOK\r\n" |
  nc -lp 8000 -q 1
  sleep 1
done
```

> 端口 8000 

### 规则 alerting.rules.yml

位置 `~/alerting.rules.yml` 也就是 `/home/kinglion/alerting.rules.yml` 

```yaml
groups:
- name: nginx
  rules:
  - alert: NginxDown
    expr: rate(nginx_http_requests_total{status="404"}[10s]) > 0.01
    for: 10s
```

### 测试

```
sudo service nginx restart
./alertmanager
./prometheus
```

会打印东西不中断，不要关闭窗口。

![](http://ww1.sinaimg.cn/large/8f2bdb7fgy1fsivrui2urj21fh0j6akk.jpg)

没有中断说明是正常的。

再来运行 shell 脚本 http_ok_server.sh 。

```bash
./http_ok_server.sh | tee notifications.txt
```

用浏览器访问一个访问不到的网页，如 `localhost/good.html` 。就模拟了 404 报错的情况。

等待片刻，再来观察。会打印出一些结果。这就说明我们的告警配置成功，成功接收到通知。

```bash
kinglion@ubuntu:~$ ./http_ok_server.sh | tee notifications.txt 
POST / HTTP/1.1
Host: localhost:8000
User-Agent: Alertmanager/0.15.0-rc.1
Content-Length: 675
Content-Type: application/json
Accept-Encoding: gzip
Connection: close

{"receiver":"kinglion","status":"firing","alerts":[{"status":"firing","labels":{"alertname":"NginxDown","host":"_","instance":"localhost:9145","job":"nginx","status":"404"},"annotations":{},"startsAt":"2018-06-20T22:39:35.127773583-07:00","endsAt":"0001-01-01T00:00:00Z","generatorURL":"http://ubuntu:9090/graph?g0.expr=rate%28nginx_http_requests_total%7Bstatus%3D%22404%22%7D%5B10s%5D%29+%3E+0.01\u0026g0.tab=1"}],"groupLabels":{"job":"nginx"},"commonLabels":{"alertname":"NginxDown","host":"_","instance":"localhost:9145","job":"nginx","status":"404"},"commonAnnotations":{},"externalURL":"http://ubuntu:9093","version":"4","groupKey":"{}/{job=\"nginx\"}:{job=\"nginx\"}"}

```

