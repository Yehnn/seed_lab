## 问题描述

根据如下实验，报错。

```bash
shiyanlou:salt/ $ sudo salt '*' state.highstate                                 [10:30:28]
d5ca07d18557:
    Minion did not return. [No response]
```

### 排除

#### 1. master 和 minion 进程是开启的

```bash
ps -ef |grep -v grep|grep salt-master
ps -ef |grep -v grep|grep salt-minion
netstat -luantp  #4505 和 4506 是监听的
```

#### 2. master 和 minion 是连通的

```bash
shiyanlou:salt/ $ sudo salt '*' test.ping                                       [10:28:36]
d5ca07d18557:
    True
```

#### 3. 密钥是接收的

```bash
shiyanlou:salt/ $ sudo salt-key -L                                              [10:43:31]
Accepted Keys:
d5ca07d18557
Denied Keys:
Unaccepted Keys:
Rejected Keys:
```

#### 4. 变量是分发了的

```bash
shiyanlou:salt/ $ sudo salt '*' pillar.items                                    [10:48:26]
d5ca07d18557:
    ----------
    nginx:
        ----------
        HOST:
            192.168.42.4
        PORT:
            8080
```

#### 5. 查看日志

```bash
shiyanlou:salt/ $ sudo tail -f /var/log/salt/master /var/log/salt/minion   
2018-06-25 18:18:13,213 [salt.loaded.int.module.cmdmod                                    :753 ][ERROR   ][9574] Command '[u'runlevel', u'/run/utmp']' failed with return code: 1
2018-06-25 18:18:13,238 [salt.loaded.int.module.cmdmod                                    :755 ][ERROR   ][9574] stdout: unknown
2018-06-25 18:18:13,238 [salt.loaded.int.module.cmdmod                                    :759 ][ERROR   ][9574] retcode: 1
2018-06-25 18:18:13,238 [salt.loaded.int.module.cmdmod                                    :1075][ERROR   ][9574] Command '[u'runlevel', u'/run/utmp']' failed with return code: 1
2018-06-25 18:18:13,238 [salt.loaded.int.module.cmdmod                                    :1080][ERROR   ][9574] output: unknown
```

没看出具体的问题

#### 6. 用 debug 模式运行 minion

```bash
$ sudo salt-minion -l debug
[DEBUG   ] Initializing new AsyncZeroMQReqChannel for (u'/etc/salt/pki/minion', u'd5ca07d18557', u'tcp://127.0.0.1:4506', u'aes')
[DEBUG   ] Initializing new AsyncAuth for (u'/etc/salt/pki/minion', u'd5ca07d18557', u'tcp://127.0.0.1:4506')
[DEBUG   ] Connecting the Minion to the Master URI (for the return server): tcp://127.0.0.1:4506
[DEBUG   ] Trying to connect to: tcp://127.0.0.1:4506
[DEBUG   ] minion return: {u'fun_args': [], u'jid': u'20180626104118658540', u'return': {u'pkgrepo_|-nginx-repo_|-deb http://nginx.org/packages/ubuntu/ trusty nginx_|-managed': {u'comment': u"Package repo 'deb http://nginx.org/packages/ubuntu/ trusty nginx' already configured", u'name': u'deb http://nginx.org/packages/ubuntu/ trusty nginx', u'start_time': '10:41:24.192533', u'result': True, u'duration': 294.91, u'__run_num__': 0, u'__sls__': u'nginx', u'changes': {}, u'__id__': u'nginx-repo'}, u'service_|-nginx-service_|-nginx_|-running': {u'comment': u'The service nginx is already running', u'name': u'nginx', u'start_time': '10:42:02.682633', u'result': True, u'duration': 316.5, u'__run_num__': 4, u'__sls__': u'nginx', u'changes': {}, u'__id__': u'nginx-service'}, u'file_|-nginx-service_|-/etc/nginx/conf.d/shiyanlou.conf_|-managed': {u'comment': u'File /etc/nginx/conf.d/shiyanlou.conf is in the correct state', u'pchanges': {}, u'name': u'/etc/nginx/conf.d/shiyanlou.conf', u'start_time': '10:42:02.457889', u'result': True, u'duration': 219.517, u'__run_num__': 3, u'__sls__': u'nginx', u'changes': {}, u'__id__': u'nginx-service'}, u'archive_|-extract_nginx_|-/home/shiyanlou_|-extracted': {u'comment': u'Path /home/shiyanlou/page exists', u'name': u'/home/shiyanlou', u'start_time': '10:41:24.497514', u'result': True, u'duration': 4.942, u'__run_num__': 1, u'__sls__': u'nginx', u'changes': {}, u'__id__': u'extract_nginx'}, u'pkg_|-nginx-service_|-nginx_|-latest': {u'comment': u'Package nginx is already up-to-date', u'name': u'nginx', u'start_time': '10:41:29.463235', u'result': True, u'duration': 32979.952, u'__run_num__': 2, u'__sls__': u'nginx', u'changes': {}, u'__id__': u'nginx-service'}}, u'retcode': 0, u'success': True, u'fun': u'state.highstate'}
[INFO    ] User sudo_shiyanlou Executing command pillar.items with jid 20180626105058139017
[DEBUG   ] Command details {u'tgt_type': u'glob', u'jid': u'20180626105058139017', u'tgt': u'*', u'ret': u'', u'user': u'sudo_shiyanlou', u'arg': [], u'fun': u'pillar.items'}
```

**等待片刻可以看到 minion 是有响应信息的，然而 master 却提示没有，以上所有说明一个问题。这个时间超过了 master 设置的 timeout。 所以需要增大 timeout。** 

```bash
$ sudo vi /etc/salt/master
timeout: 300 # 改的长一些，最好超过 1 分钟
```

更改过后重启 `sudo service salt-master restart` 。

```bash
shiyanlou:salt/ $ sudo salt '*' state.highstate                                 [10:40:48]
d5ca07d18557:
----------
          ID: nginx-repo
    Function: pkgrepo.managed
        Name: deb http://nginx.org/packages/ubuntu/ trusty nginx
      Result: True
     Comment: Package repo 'deb http://nginx.org/packages/ubuntu/ trusty nginx' already configured
     Started: 10:41:24.192533
    Duration: 294.91 ms
     Changes:   
----------
          ID: extract_nginx
    Function: archive.extracted
        Name: /home/shiyanlou
      Result: True
     Comment: Path /home/shiyanlou/page exists
     Started: 10:41:24.497514
    Duration: 4.942 ms
     Changes:   
----------
          ID: nginx-service
    Function: pkg.latest
        Name: nginx
      Result: True
     Comment: Package nginx is already up-to-date
     Started: 10:41:29.463235
    Duration: 32979.952 ms
     Changes:   
----------
          ID: nginx-service
    Function: file.managed
        Name: /etc/nginx/conf.d/shiyanlou.conf
      Result: True
     Comment: File /etc/nginx/conf.d/shiyanlou.conf is in the correct state
     Started: 10:42:02.457889
    Duration: 219.517 ms
     Changes:   
----------
          ID: nginx-service
    Function: service.running
        Name: nginx
      Result: True
     Comment: The service nginx is already running
     Started: 10:42:02.682633
    Duration: 316.5 ms
     Changes:   

Summary for d5ca07d18557
------------
Succeeded: 5
Failed:    0
------------
Total states run:     5
Total run time:  33.816 s
```

可见已经有响应了。

## 3. Saltstack 实战

下面我们将会开始 Saltstack 实战。

### 3.1 实战需求

在已经部署好 Saltstack 的 master 与 minion 环境中，此时我们需要在每个 minon 中安装最新版本的 Nginx，并同时部署好 page 项目（page 项目源码在：<http://labfile.oss.aliyuncs.com/courses/980/files/week10/page.tar> 链接中，需要部署在 /home/shiyanlou/ 目录中），并且所有的项目服务于 8080 端口，同时服务的 ServerName 为当前节点的 IP 地址。

### 3.2 需求分析

从需求中我们主要得到了这样的信息：

- Nginx 需要是最新版本，安装最新版本的方式无外乎两种
  + 通过 Nginx 提供的 apt 源安装（使用）
  + 通过 Nginx 源码来编译安装（舍弃）
- 部署 page 项目，需要的操作：
  + 1.下载该源码
  + 2.解压该源码
  + 3.Nginx 该项目的配置文件的 root 需要为 /home/shiyanlou/page
- 服务于 8080，项目名为 IP 地址，也就是其他的配置项相同，只有 IP 不同，考虑用模版的方式(模版已经为大家提供于 <http://labfile.oss.aliyuncs.com/courses/980/files/week10/shiyanlou.conf>)

### 3.3 目录规划

因为此次的批量化操作主要是对 Nginx 的部署，所以我们以功能来创建目录，以这样的方式来规划：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517729435159-wm)

- top.sls：用于 sls 文件与节点和存放位置的对应
- nginx：用于存放该项目所使用的 sls 文件
- pillar：用于存放我们需要自己创建变量的 sls
  + top 文件同样用于对应
  + nginx：用于存放此次我们需要分发的变量文件

### 3.4 实际操作

以下的所有操作都是建立与我们已经安装好 Saltstack 的 master 与 minon 的前提下。具体的安装步骤参照之前的章节（同时别忘记通过密钥的链接请求 `salt-key -A`）。

根据我们的分析我们将做这样的一些操作：

- 根据我们的目录规划修改配置文件
- 创建相关的目录与文件
- 创建 pillar 的相关文件
- 分发 pillar 变量
- 创建部署的 sls 文件
- 实施部署

根据我们的规划，我们开始我们的操作：

1.首先按照我们之前所规划的目录修改 saltstack 配置文件

```bash
sudo vim /etc/salt/master
```

找到配置文件中用于配置目录结构的 file_roots 部分，我们可以看到默认注释的内容，我们添加这样的一些内容：

```bash
file_roots:
  base:
    - /srv/salt
  nginx:
    - /srv/salt/nginx
pillar_roots:
  base:
      - /srv/salt/pillar
  nginx:
      - /srv/salt/pillar/nginx
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517730228680-wm)

保存退出

2.根据我们的目录结构创建相关的文件夹

```bash
sudo mkdir -p /srv/salt/nginx
sudo mkdir -p /srv/salt/pillar/nginx
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517730338557-wm)

3.重启 master 节点，使得我们的配置文件生效

```bash
sudo service salt-master restart
```

4.开始创建 pillar 文件，需要自定义的变量

```bash
sudo vim /srv/salt/pillar/nginx/nginx.sls
```

因为我们配置文件中有一个值是自定义的，每台机器都不同，那就是 ServerName（为 IP 地址），所以我们的摹本中需要该变量，同时为了应对需求的变化我们会将端口也通过变量来控制，所以该文件的内容为（当然两个变量是因为我们在配置文件模版中使用的这样两个变量，若是修改了，还需要修改配置文件模版中的变量）：

```bash
nginx:
	HOST: {{ grains['fqdn_ip4'][0]}}
	PORT: 8080
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517733183166-wm)

我们可以通过上面的链接来查看我们的模版：

```
wget http://labfile.oss.aliyuncs.com/courses/980/files/week10/shiyanlou.conf
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517731756997-wm)

我们可以看到模版中需要使用变化的部分都是用变量来代替了，listen 的值使用 `PORT` 变量， server_name 的值使用 `HOST` 变量。

5.为了能够让我们创建的自定义变量分发,我们需要创建 top.sls 文件

```bash
sudo vim /srv/salt/pillar/top.sls
```

我们当前 pillar 当前目录中并没有 sls 文件执行，所以我们不需要指定 base 目录，直接指定 nginx 中的文件即可，所以内容为：

```bash
nginx:
	'*':
		- nginx
```

表示 nginx 目录中的 nginx 文件执行于所有的 minion 节点。

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517731224505-wm)

6.分发我们所创建的自定义变量：

```bash
salt '*' saltutil.refresh_pillar 
```

分发之后我们一定验证一下我们的操作是否成功，查看 pillar 变量：

```bash
salt '*' pillar.items
```

```checker
- name: check dir
  script: |
    #!/bin/bash
	ls /srv/salt/nginx
  error: 没有 /srv/salt/nginx 目录
- name: check dir2
  script: |
    #!/bin/bash
	ls /srv/salt/pillar/nginx
  error: 没有 /srv/salt/pillar/nginx 目录
- name: check service
  script: |
    ps -ef |grep -v grep |grep salt-master
  error: 没有启动 salt-master
- name: check service
  script: |
    ps -ef |grep -v grep |grep salt-minion
  error: 没有启动 salt-minion
- name: check pillar
  script: |
    #!/bin/bash
	salt '*' pillar.items|grep -e 'HOST' -e 'PORT'
  error: 分发变量不成功
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517733247392-wm)

7.创建我们部署的操作：

```bash
sudo vim /srv/salt/nginx/nginx.sls
```

部署操作的内容我们上文有分析过：

（1）通过源的方式安装最新的 nginx，所以添加源

```bash
nginx-repo:
    pkgrepo.managed:
        - humanname: Nginx repo
        - name: deb http://nginx.org/packages/ubuntu/ trusty nginx 
        - disk: trusty
        - file: /etc/apt/sources.list.d/nginx.list
        - keyid: ABF5BD827BD9BF62
        - keyserver: keyserver.ubuntu.com
        - require_in:
            - pkg: nginx
```

（2）我们将下载项目文件的源码并解压好（遇到未使用过的模块，希望大家到官网查看其作用与具体参数、使用方式）：

```bash
# 步骤 ID
extract_nginx:
    # 使用 archive 模块的 extracted 方法
    archive.extracted:
        # 放置的目录
        - name: /home/shiyanlou
        # 文件的来源，来源可以是本地也可以是网络
        - source: http://labfile.oss.aliyuncs.com/courses/980/files/week10/page.tar
        # 为了保证文件没有被篡改过，所以必须提供 hash 值，该值通过 md5sum 命令即可获得
        - source_hash: 749ecdeaff0733d84b0271f7fc850b99
        # 解压之后文件的所属者与组
		- user: nginx
		- group: nginx
        # 若是不存在该目录便创建
        - if_missing: /home/shiyanlou/page

```

(3)紧接着就是安装好 nginx，同时放好配置文件，并启动 nginx：

```bash
nginx-service:
    # 安装 nginx
    pkg.latest:
        - name: nginx
        # 安装之前通过 apt-get update 更新源信息
        - refresh: True
    # 配置文件操作
    file.managed:
        - name: /etc/nginx/conf.d/shiyanlou.conf
        # 从网络地址获取模版文件
        - source: http://labfile.oss.aliyuncs.com/courses/980/files/week10/shiyanlou.conf
        - source_hash: 4a68711fe5d4eda22a0cf128f640a64a
        - user: root
        - group: root
        - mode: 644
        # 使用的模版引擎
        - template: jinja
        # 需要替换的变量值
        - defaults:
            HOST: {{ pillar['nginx']['HOST']}}
            PORT: {{ pillar['nginx']['PORT']}}
        - require:
            - pkg: nginx-service
    # 启动 nginx 服务
    service.running:
        - name: nginx
        - enable: True
        - reload: True
        - watch:
          - file: nginx-service
```

将上述内容都写入我们的操作文件中：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517732533297-wm)

8.创建 top 文件：

```bash
sudo vim /srv/salt/top.sls
```

同样在当前目录中我们并没有相关的执行文件，我们的文件存放于 nginx 目录中，所以我们的内容如下：

```bash
nginx:
	'*':
		- nginx
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517732919250-wm)

9.执行部署：

然后执行我们的脚本：

```bash
sudo salt '*' state.highstate
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517734469097-wm)

从执行的输出信息中，我们看到我们 sls 文件中的五个操作都执行成功了。

### 3.5 验证

我们看到我们的操作执行成功了，但是执行成功并不代表真的没有问题，我们需要通过浏览器来验证我们的操作是否真的没有问题：

通过 `ifconfig eth0` 查看当前的 IP，然后访问其 8080 端口：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517734906601-wm)

由此证明我们不仅执行没有出错，我们的结果也是没有问题的。