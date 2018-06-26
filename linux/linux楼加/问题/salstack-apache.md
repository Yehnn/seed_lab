# 挑战：使用 Saltstack 部署 Apache

## 介绍

在上节实验中我们通过 Saltstack 部署 Nginx，并能成功访问，同样的需求但是我们通过 Apache 来作为我们的 web 服务工具，Saltstack 的目录结构与实验中相同（目录名为 apache2 不再为 nginx）。需求如下：

- Apache2 直接安装（可以不用最新版）
- 部署 page 项目，需要的操作：
  + 1.下载该源码（地址：http://labfile.oss.aliyuncs.com/courses/980/files/week10/page.tar）
  + 2.解压该源码至 `/var/www/html` 中
  + 3.配置文件直接覆盖原有 `000-default.conf` 配置文件（配置文件中无变量，不是模版，地址：http://labfile.oss.aliyuncs.com/courses/980/files/week10/000-default.conf）
- 工作于 80 端口即可

## 目标

1. 正确的配置 Saltstack 配置文件
2. 获取并替换 `000-default.conf` 配置文件

## 提示

- 需要自己下载之后计算 hash 值（使用 md5sum 命令）

## 知识点

- Saltstack 的 topfile

## 解决方案

### 安装 saltstack

```bash
wget -O - https://repo.saltstack.com/apt/ubuntu/14.04/amd64/latest/SALTSTACK-GPG-KEY.pub | sudo apt-key add -
echo "deb http://repo.saltstack.com/apt/ubuntu/14.04/amd64/latest trusty main" | sudo tee /etc/apt/sources.list.d/saltstack.list
sudo apt-get update
sudo apt-get install salt-master salt-minion -y
```

### 修改 hosts

```bash
sudo vim /etc/hosts
输入一下内容

127.0.0.1 localhost salt
```

### 启动 saltstack

```bash
sudo service salt-master start
sudo service salt-minion start
```

### 通过密钥

```bash
sudo salt-key -L
sudo salt-key -A
```

#### 创建目录

```
sudo mkdir -p /srv/salt
```

#### 创建 top 文件

```
vim /srv/salt/top.sls
```

输入下面的内容

```
base:
  '*':
  - apache2
```

#### 创建执行文件

```
vim /srv/salt/apache2.sls
```

输入下面的内容

```
apache2-service:
    pkg.latest:
        - name: apache2
        - refresh: True
    file.managed:
        - name: /etc/apache2/sites-available/000-default.conf
        - source: http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/files/week10/000-default.conf
        - source_hash: 61f92b16d9e1eded1e564a64506cc5e5
        - user: root
        - group: root
        - mode: 644
        - require:
            - pkg: apache2-service
    service.running:
        - name: apache2
        - enable: True
        - reload: True
        - watch:
            - file: apache2-service

extract_project:
    archive.extracted:
        - name: /var/www/html
        - source: http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/files/week10/page.tar
        - skip_verify: True
        - user: root
        - group: root
        - options: v
        - if_missing: /var/www/html/page
```

#### 执行

```
salt '*' state.apply
```

### 我的错误

在一开始我使用下面命令，都有错误。

```bash
sudo salt '*' state.apply
sudo salt '*' state.highstate
```

```
856dfa19e6a0:
    Data failed to compile:
----------
    The function "state.highstate" is running as PID 8993 and was started at 2018, Jun 26 16:20:34.495459 with jid 20180626162034495459
```

> 这个解决用 kill 杀掉进程 sudo kill -9 8993

然后还是有错误。有 3 个步骤都是红的，就是错误的。

```
shiyanlou:salt/ $ sudo salt '*' state.apply                                     [14:19:50]

856dfa19e6a0:
----------
          ID: apache2-service
    Function: pkg.latest
        Name: apache2
      Result: False
     Comment: An error was encountered while checking the newest available version of package(s): E: Could not get lock /var/lib/apt/lists/lock - open (11: Resource temporarily unavailable)
              E: Unable to lock directory /var/lib/apt/lists/
     Started: 14:22:44.853782
    Duration: 15281.054 ms
     Changes:   
----------
          ID: apache2-service
    Function: file.managed
        Name: /etc/apache2/sites-available/000-default.conf
      Result: False
     Comment: One or more requisite failed: apache2.apache2-service
     Started: 14:23:00.179850
    Duration: 0.023 ms
     Changes:   
----------
          ID: apache2-service
    Function: service.running
        Name: apache2
      Result: False
     Comment: One or more requisite failed: apache2.apache2-service
     Started: 14:23:00.203790
    Duration: 0.02 ms
     Changes:   
----------
          ID: extract_apache2
    Function: archive.extracted
        Name: /var/www/html
      Result: True
     Comment: Path /var/www/html/page exists
     Started: 14:23:00.228665
    Duration: 19.937 ms
     Changes:   

Summary for 856dfa19e6a0
------------
Succeeded: 1
Failed:    3
------------
Total states run:     4
Total run time:  15.301 s
ERROR: Minions returned with non-zero exit code
```

### 排除

1. master 和 minion 是连通的

```
shiyanlou:salt/ $ sudo salt '*' test.ping                                       [14:05:23]
856dfa19e6a0:
    True
```

2. 密钥是接收的 sudo salt-key -L
3. 文件位置什么的都没问题
4. 配置文件的错

我的 apache2.sls 是

```yaml
apache2-service:
    pkg.latest:
        - name: apache2
        - refresh: True
    file.managed: 
        - name: /etc/apache2/sites-available/000-default.conf
        - source: http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/files/week10/000-default.conf
        - source_hash: 61f92b16d9e1eded1e564a64506cc5e5
        - user: root
        - group: root
        - mode: 644
        - require:
            - pkg: apache2-service
    service.running:
        - name: apache2
        - enable: True
        - reload: True
        - watch:
            - file: apache2-service

extract_apache2:
    archive.extracted:
        - name: /var/www/html
        - source: http://labfile.oss.aliyuncs.com/courses/980/files/week10/page.tar
        - skip_verify: True
        - user: root
        - group: root
        - options: v
        - if_missing: /var/www/html/page
```

正确的 apache2.sls 是

```yaml
apache2-service:
    pkg.latest:
        - name: apache2
        - refresh: True
    file.managed:
        - name: /etc/apache2/sites-available/000-default.conf
        - source: http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/files/week10/000-default.conf
        - source_hash: 61f92b16d9e1eded1e564a64506cc5e5
        - user: root
        - group: root
        - mode: 644
        - require:
            - pkg: apache2-service
    service.running:
        - name: apache2
        - enable: True
        - reload: True
        - watch:
            - file: apache2-service

extract_project:
    archive.extracted:
        - name: /var/www/html
        - source: http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/files/week10/page.tar  #####这里不一样
        - skip_verify: True
        - user: root
        - group: root
        - options: v
        - if_missing: /var/www/html/page
```

总结：

发现修改的 /etc/salt/minion 加一个 127.0.0.1 和 /etc/hosts 中的 127.0.0.1 localhost salt 和结果没关系。

> /etc/salt/master 文件中的 timeout 设置大些，比如 timeout=300

正确结果如下：

```
shiyanlou:salt/ $ sudo salt '*' state.apply                                     [16:53:30]
856dfa19e6a0:
----------
          ID: apache2-service
    Function: pkg.latest
        Name: apache2
      Result: True
     Comment: Package apache2 is already up-to-date
     Started: 16:54:10.648966
    Duration: 30446.877 ms
     Changes:   
----------
          ID: apache2-service
    Function: file.managed
        Name: /etc/apache2/sites-available/000-default.conf
      Result: True
     Comment: File /etc/apache2/sites-available/000-default.conf is in the correct state
     Started: 16:54:41.104678
    Duration: 89.851 ms
     Changes:   
----------
          ID: apache2-service
    Function: service.running
        Name: apache2
      Result: True
     Comment: The service apache2 is already running
     Started: 16:54:41.195787
    Duration: 208.798 ms
     Changes:   
----------
          ID: extract_project
    Function: archive.extracted
        Name: /var/www/html
      Result: True
     Comment: Path /var/www/html/page exists
     Started: 16:54:41.406458
    Duration: 2.82 ms
     Changes:   

Summary for 856dfa19e6a0
------------
Succeeded: 4
Failed:    0
------------
Total states run:     4
Total run time:  30.748 s
```



