# Playbook 实战

## 1. 实验介绍

### 1.1 实验内容

在上一节的实验中我们学习了用 `AD-HOC` 临时命令来快速执行任务，但是 Ansible 真正的重点在于 playbook 的编写，因为在自动化运维中批量进行系统配置和部署服务才是自动化的核心用法，所以在这一节实验中，我们一起来学习如何编写一个完整的 `playbooks`。

### 1.2 实验知识点

+ Playbook 介绍

+ YAML 语法格式

+ Playbook 语法结构

+ Ansible 模块介绍

+ Playbook 执行控制

+ Playbook 示例

### 1.3 推荐阅读

+ [Playbooks 官方文档](http://docs.ansible.com/ansible/latest/playbooks_intro.html)

+ [[Playbooks 官方中文文档](http://ansible-tran.readthedocs.io/en/latest/docs/playbooks.html)

## 2. Playbook 介绍

### 2.1 Playbook 概述

`Playbook` 是一种非常简单的配置管理系统以及是多机器部署系统的基础，十分适合复杂的应用部署。同时，`Playbook` 还可以用于声明配置，以及编排有序的执行过程，使得在多组机器之间有序的执行指定步骤，或者同步或异步的发起任务。

`Playbook` 也是一个任务列表，这个列表中的可以包含一个或者多个 `plays`，所有的操作放在这么一个文件中，然后一次性的执行。而它与 `AD-HOC` 最大的不同之处就在于它是把这些任务放在源码中进行控制。

### 2.2 YAML 语法格式

在上一个实验中我们尝试了用 `INI` 格式的语言来执行 `AD-HOC` 命令，这里 Ansible 官方默认使用 `YAML` 这种格式来书写 Playbook。（只是为了简化语法，避免做成一种编程语言，就像是 XML 或 JSON 一样是一种利于读写的数据格式，从而降低学习难度。此外，在大多数编程语言中都有用于使用 YAML 的库。）

对于 Ansible 来说，几乎每个 `YAML` 文件都以一个列表开始。列表中的每个项都是 `key/value` 对的列表，通常称为 `Hash` 或 `Dictionary`。

**YAML 格式说明**

+ 文件均以 `---` 作为开始，以 `...` 结束。

+ 列表中同级别的项使用相同缩进以及短横线加空格（`- `）开始。

+ 字典（dictionary）用 `key: value` 的简单形式表示，其中冒号后面必须有个空格。value 部分可以用几种形式（`yes/no/true/false`）来指定布尔值。

+ 换行符：value 部分还可以使用 `|` 或 `>`，使用 `|` 可以将内容进行分行处理，而使用 `>` 将忽略换行符，任何情况下缩进都会被忽略。

+ 语句过长可以使用 `""` 括起来，例如：`foo: "somebody said I should put a colon here: so I did"`。

如下一个简单的示例大家可以初步认识下 YAML 的语法格式：

```bash
---
Name: Linux louplus
Label：Linux
Start: True
Courses:
    - linux
    - Bash
    - python
    - MySQL
    - Ansible
Describe: |
    100 lab
    12 weeks
    50 challenges
...
```

## 3. Playbook 语法结构

我们知道 Playbooks 是有一个或多个 plays 组成，也就是它的内容是以 plays 为元素的列表。play 的内容也被称为 task （任务），执行一个任务就是对模块的调用。

下面我们通过这个 playbook 的简单示例来学习它的语法结构：

```bash
---
- hosts: test
  remote_user: root
  vars:

  tasks:
    - name: Install the package "bc"
      apt:
        name: bc
        state: present
  handlers:
...
```

**结构说明：**

+ 主机和用户（hosts）

`hosts` 参数表示一个或多个组或主机，多个时用逗号分隔。

`remote_user` 表示用户名，这里还可用 `sudo: yes` 来表示使用 sudo 的权限执行操作。

+ vars（变量）

在任务行中可以使用变量来定义，像这样 `"{{item}}"` 括起来。将变量独立出来，可以方便后期的修改和维护等，也可以在其他任务中使用。

变量的定义可以放在如下几处：

1. Inventory 中

2. 全局中（`var: `）或者某个任务（task）中

3. 在 [roles](http://docs.ansible.com/ansible/latest/playbooks_reuse_roles.html) 结构中用于存放单独的文件

4. 在 registered 模块中注册变量，主要用于调试和判断

更多高级变量的用法可以参考[官方文档](http://docs.ansible.com/ansible/latest/playbooks_variables.html#advanced-syntax)

+ tasks（任务）

在运行 playbook 时是从上到下依次执行，并且一个 task 在其对应的所有主机上执行完毕之后才会执行下一个 task。如果一个 host 执行 task 失败，那么这个 host 将会从整个 playbook 的 rotation 中移除。如果发生执行失败的情况，需要修正 playbook 中的错误，然后再重新执行。

每一个 task 必须有一个 `name`，这样在输出任务时才可以清楚地辨别出属于哪个任务，若没有定义将会被特定标记。

+ handlers

Handlers （可选项）和一般的 task 没有什么区别，也是一个列表项，只是通过名字来对它进行引用。Handlers 是由通知者进行 notify，如果没有被通知handlers 不会被执行，不管有多少个通知者进行了 notify，等到所有 task 执行完成之后，handlers 也只会被执行一次。同时 handlers 也会按照声明的顺序执行。

Handlers 最佳的应用场景是用来重启服务，或者触发系统重启操作，除此以外很少用到了。

## 4. Ansible 模块

Ansible 的模块（modules），也被称为 `task plugins` 或者 `library plugins`，是在 Ansible 中真正进行实际工作方式，在每个任务中执行相应的内容。

模块（modules）具有幂等性，即在一个序列中多次运行一个模块和运行一次的效果相同，换句话说，当你再次执行模块时，模块只会执行必要的改动，所以要实现幂等性的一个办法就是让模块检测出已经到达了期望的最终状态，那么退出时就不会再执行任何的动作了。因而重复多次执行 playbook 也会是安全的。

下面我们简单介绍几个常用的模块，更全面的模块介绍可以参考[官方文档](http://docs.ansible.com/ansible/latest/list_of_all_modules.html)中的描述。

+ service 模块

这是一个比较基本的模块定义，主要用于管理服务。使用 key=value 这种参数格式来书写。

如下示例：

```bash
# 开启服务
- name: make sure apache is running
    service: name=httpd state=started

# 也可以用下面这种 key：value 参数格式来书写
- service:
    name: httpd
    state: started
```

+ shell 模块

shell 模块用法很简单也非常常用，就像直接输入执行命令一样。不过，shell 模块（和 [command 模块](http://docs.ansible.com/ansible/latest/list_of_commands_modules.html)）是唯一不会使用 `key=value` 参数格式的模块，它们只取得参数的列表。例如编译安装一个程序，我们可以编写如下语句：

```bash
- name: make install gucad
      shell: ./configure --with-init-dir=/etc/init.d && make && make install && ldconfig && update-rc.d guacd defaults
      args:
          chdir: /home/ubuntu/src/guacamole-server-0.9.9/
```

不过在 shell 命令中大多数的常用命令已经被做成了一个模块，可以直接使用而不用采用 shell 命令来执行，只是有些命令的参数还未完全转化成模块，这种情况下才采用 shell 模块来实现。

在 Ubuntu 中我们常用的命令像 `apt-get update`、`apt-get install` 这样的命令，以及创建文件（`file`）和复制文件（`copy`）这些命令操作都是有专门的模块来实现，如下所示：

```bash
#  在指定目录下创建一个文件，并赋予权限
- name: create a file
    file:
         path: /home/shiyanlou/file
         state: touch
         owner: shiyanlou
         mode: 'u+rw,g+rw'

# 复制一个文件到指定目录
  - name: copy a file
    copy:
         src: /etc/ansible/ansible.cfg
         dest: /home/shiyanlou/file

# 安装一个软件包
- hosts: test
  sudo: yes
  vars:
      apt_packages_ca:
         - apt-transport-https
         - ca-certificates
         - apparmor-utils

  tasks:
    - name: add CA certificates are installed.
      apt:
          name: "{{ item }}"
          update_cache: yes
      with_items: apt_packages_ca
```

## 5. Playbook 执行控制

和其他语言相似，Ansible 也提供了很多不同的选项方式来控制执行流，下面我们就来学习几种常见的方式。（注：部分示例来自于官方文档）

+ 条件（condition）

通常一个任务的结果会取决于变量的值、事件、或者之前任务的结果，然后通过某些条件是否满足来判断如何控制执行。

一个条件执行可以像如下语句所示：

```bash
tasks:
    - shell: echo "This certainly is epic!"
      when: epic
```

`when` 语句在这里作为一个条件来进行判断执行。

+ When 语句

有时可能会需要某个主机跳过某个特定的步骤，这时使用 `when` 子句就很容易实现了，其中包含没有花括号的原始 Jinja2 表达式。

如下示例，尝试使用 Jinja2 的 `defined` 命令:

```bash
tasks:
        - shell: echo "I've got '{{ foo }}'"
          when: foo is defined

        - fail: msg="Bailing out. this play requires 'bar'"
          when: bar is not defined
```

+ 循环（Loop）

若在一个任务中想要创建大量用户或安装许多包，或者重复轮询步骤，这是通过循环（loop）就能高效的实现结果。和一般循环相似，playbook 也有标准循环、嵌套循环、哈希循环、Do-Until 循环等等，这里我们只讲解几个简单的循环语句，详细的可以参考官方的 [Loop 部分](http://docs.ansible.com/ansible/latest/playbooks_loops.html)。

如下示例：

1. 标准循环

```bash
# 添加多个用户
- name: add several users
  user:
    name: "{{ item }}"
    state: present
    groups: "shiyanlou"
  with_items:
     - testuser1
     - testuser2
```

`item`：用于放置需要读取的变量的位置

`with_items`：用来指定需要读取出来的值

上面的语句相当于如下写法：

```bash
- name: add user testuser1
  user:
    name: "testuser1"
    state: present
    groups: "shiyanlou"
- name: add user testuser2
  user:
    name: "testuser2"
    state: present
    groups: "shiyanlou"
```

2. 嵌套循环

如下示例：

```bash
- name: give users access to multiple databases
  mysql_user:
  name: "{{ item[0] }}"
  priv: "{{ item[1] }}.*:ALL"
  append_privs: yes
  password: "shiyanlou"
  with_nested:
    - [ 'Jay', 'Chou' ]
    - [ 'studentdb', 'coursedb', 'classdb' ]
```

前面我们介绍了 playbook 的许多语法结构、控制流和模块等，而它的控制执行方式也比较简单，如下两个步骤：

+ 1. 编写一个 `yaml` 的 playbook 文件：`vim *.yaml`

+ 2. 使用 `ansible-playbook` 命令执行即可：`ansible-playbook *.yaml`

下面这个简单的示例，大家可以来尝试写一个简单的 playbook。

```bash
---
- hosts: test

  tasks:
      - name: test condition
        command: echo {{ item }}
        with_items: [ 0, 2, 4, 6, 8, 10 ]
        when: item > 5
...
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516266592503.png-wm)

`when`：列出判断的条件，条件为真则执行命令，为假则 skipping 略过

执行一下这个 `playbook`，如下所示：

```bash
$ sudo vim test.yaml
$ ansible-playbook test.yaml
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516266567018.png-wm)

## 6. Playbook 示例

经过上面这么多知识点的铺垫后，下面我们来实操一下。

我们先用安装 Apache2 服务并启动为例，做如下操作：

以下是安装 Apache2 服务的一个简单脚本：

```bash
#!/bin/bash

# install apache2
apt-get install apache2

# start-up apache
service apache2 start

```

我们将其转换为 playbook ：

```bash
---
 - hosts: test
   sudo: yes

   tasks:
       - name: "Install Apache"
         apt:
             name: apache2
            state: present
       - name: "Startup Apache"
          service:
              name: apache2
              state: started
              enabled: yes
...
```

```bash
$ sudo vim test_apache.yaml
$ ansible-playbook test_apache.yaml
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516267533726.png-wm)

上面这个例子相对比较简单，执行步骤也较少。

下面我们来尝试下如何安装 docker 这个相对复杂的软件（在下一周我们会重点学习 docker 的详细内容，大家可以把这个实验当作是后面学习的一个参考）。

首先，还是建立一个 yaml 文件。

```
$ sudo vim test_docker.yaml
```

然后，编写安装 docker 的 playbook。

```bash
---
- hosts: test # 对 test 组执行下面的任务
  sudo: yes  # 下面的任务将有 sudo 的执行权限
  vars:      # 定义一个变量，安装多个包，以免增加多个类似的代码
      apt_packages_ca:
         - apt-transport-https
         - ca-certificates

  tasks:    # 定义任务列表

    - name: add docker source list file for  install docker
      file:
          path: /etc/apt/sources.list.d/docker.list
          state: touch
          owner: root
          mode: 'u+r,g+rw'

    - name: write deb url of docker to docker.list
      blockinfile:
          dest: /etc/apt/sources.list.d/docker.list
          marker: ""
          block: |
            deb https://apt.dockerproject.org/repo ubuntu-trusty main

    - name: add CA certificates and ensure installed
      apt:
          name: "{{ item }}"
          update_cache: yes
      with_items: "{{ apt_packages_ca }}"

    - name: add apt-key of dockers
      apt_key:
          keyserver: p80.pool.sks-keyservers.net
          id: 58118E89F3A912897C070ADBF76221572C52609D

    - name: install docker-engine
      apt:
          name: docker-engine
          state: latest
          force: yes
...
```

```bash
$ ansible-playbook test_docker.yaml
```

（安装的过程可以会有点久）

![实验楼](https://dn-simplecloud.shiyanlou.com/2767331516268095128-wm)

最后，可以通过 `docker --version` 来查看安装的版本信息，验证是否安装成功。这里着重要大家掌握的是 playbook 如何编写，而安装 docker 的方式大家可以当作是后面学习 docker 内容的一个参考。

Playbook 示例操作视频：


`@
http://labfile.oss.aliyuncs.com/courses/980/week10/2-1.mp4
@`

## 7. 总结

本节实验主要讲解了 ansible 中 playbook 的编写过程，以及它的详细语法结构等。