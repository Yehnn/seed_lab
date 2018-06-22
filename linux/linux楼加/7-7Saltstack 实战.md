---
show: step
version: 1.0
enable_checker: true
---
# Saltstack 实战

## 1. 实验介绍

#### 1.1 实验内容

在本实验中，将带领大家通过使用 Saltstack 安装 Nginx，并配置 Nginx 的方式进一步熟悉使用 Saltstack，同时补充上一章节我们所未能讲解的知识点。

#### 1.2 实验知识点

+ Jinja2 介绍
+ Saltstack 实战

#### 1.3 推荐阅读

+ [SaltStack 官方文档](https://docs.saltstack.com/en/latest/contents.html)

+ [Jinja2 docs](http://jinja.pocoo.org/docs/dev/)

## 2. Jinja2 介绍

当我们批量部署服务的时候，不仅仅是安装亦或者是查看信息，我们还会修改相关的文件，当我们配置的文件大体相同但是其中只有个别的值是个性化的时候我们就会使用到模板了，

Salt 支持了 Jinja2 的模板引擎，它可以用于 Salt states 文件、Salt pillar 文件和其他管理的文件。Salt 允许使用 Jinja 来访问 minion 的配置值、grains 和 pillar 数据，并调用 salt 来执行模块。

### 2.1 Jinja2 简介

Jinja2 是一个基于 Python 的全功能模板引擎。它能完全支持 unicode，并具有集成的沙箱执行环境，应用广泛。Jinja2 使用 BSD 授权。

Jinja2 是 Python 中最常用的模板引擎之一。它的灵感来自 Django 的模板系统，但是它扩展了一种表达性的语言，给模板提供了更强大的工具集。除此之外，它还添加了沙箱执行和可选的自动转接程序，以确保安全性。其中，Jinja2 和 Python 2.6.x， 2.7.x 和 >= 3.3 的版本兼容。

### 2.2 Jinja2 语法

Jinja2 是一个比较友好的 Python 模板语言，模仿 Django 的模板。（如果你熟悉像 Django 或 Smarty 这类基于文本的模板语言，那么 Jinja2 的学习就会更加轻松）。

Jinja2 中主要有这样的一些 Delimiters:

- {%...%}：在这样的符号中一般放置语句，如模版的继承，流程控制等等
- {{...}}：在这样的符号中一般放置表达式，如变量名，表达式等等
- {#...#}：在这样的符号中一般放置需要注释的内容

在 Saltstack 使用 Jinja2 模版中非常少、非常简单的部分，主要掌握在其中使用流程控制与变量名、表达式使用即可，我们通过以下的例子来做进一步的认识。

### 2.3 Jinja 的控制结构

+ 条件语句

Jinja 最常见的就是将条件语句插入到 Salt pillar 文件中。例如不同发行版的软件包的就可以使用 os grain 来设置特定的平台路径，软件包名称和其他值等，如下示例：

```bash
# 使用 Jinja 中的判断语句 if，通过 endif 来闭合，判断操作系统的类别
{% if grains['os_family'] == 'RedHat' %}

# 在红帽系列中 Apache 包名为 httpd
apache: httpd
git: git
{% elif grains['os_family'] == 'Debian' %} 

# 在 Debian 发行版中 Apache 包名为 apache2
apache: apache2

# 在非常早期的 Debian 中有一个工具集名 git，而版本控制的 git 只好名为 git，但是后来 git 太火便更正了
git: git-core

# 通过 endif 来闭合 if
{% endif %}
```

+ 循环语句

循环结构在 salt states 下创建用户或文件夹十分有用。如下示例：

批量创建用户：

```bash
# 在 {% %} 中使用 for 语句
{% for usr in ['shi','yan','lou'] %}

# 在 {{ }} 中使用变量
{{ usr }}:

  # 使用 saltstack 的 user 模块来创建用户，name 默认等于 ID，所以省略
  user.present

# 使用 endfor 语句来闭合 for 循环
{% endfor %}
```

批量创建文件夹：

```bash
{% for DIR in ['/dir1','/dir2','/dir3'] %}
{{ DIR }}:
  # 使用 saltstack 的 file 模块来创建文件夹
  file.directory:
    - user: root
    - group: root
    - mode: 774
{% endfor %}
```

注：一般来说，尽可能保证 salt state 简单，如果编写的 Jinja 过于复杂可以考虑将任务分解成多个 Salt state，或者为任务编自定义的执行模块。

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

我们可以看到模版中需要使用变化的部分都是用变量来代替了，若 listen 的值使用 `PORT` 变量，若 server_name 的值使用 `HOST` 变量。

5.为了能够让我们创建的自定义变量分发我们需要创建 top.sls 文件

```bash
sudo vim /srv/salt/pillar/top.sls
```

我们当前 pillar 当前目录中并没有 sls 文件执行，所以我们不需要指定 base 目录，直接指定 nginx 中的文件即可，所以内容为：

```bash
nginx:
	'*':
		- nginx
```

表示 nginx 目录中的 nginx 文件执行与所有的 minion 节点。

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081517731224505-wm)

6.分发我们所创建的自定义变量：

```bash
salt '*' saltutil.refresh_pillar 
```

分发之后我们一定验证一下我们的操作是否成功，查看 pillar 变量：

```bash
salt '*' pillar.items
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

## 4. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

+ Jinja2 介绍
+ Saltstack 实战

Jinja2 的简单介绍，与通过 Saltstack 编写一个部署的脚本来熟悉应用我们之前章节中所学到的内容。