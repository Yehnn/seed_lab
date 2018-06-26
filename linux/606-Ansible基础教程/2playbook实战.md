---
show: step
version: 1.0
enable_checker: true
---
# Playbook 实战

## 一、实验介绍
#### 1.1 实验内容  
Ansible 的 Ad-Hoc让我们体验到了 Ansible 的便捷，但是只是针对只有一两个操作的时候，当我们有多个操作时，如部署环境，修改大量的配置文件时 Ad-Hoc 便不太合适，本实验我们将介绍可以多任务执行的 Playbook,`并完成一个利用plybook安装docker的实验`



#### 1.2 实验涉及的知识点

- playbook 的认识
- ansible 模块的介绍
- playbook 实战

#### 1.3 适合人群
适合想学习和使用ansible的同学，这个playbook的使用高效而且快捷。


## 二、 实验原理

下面我们将会学习实验原理。

### 2.1 playbook 的认识

Ad-Hoc 是 ansible 的执行命令模式，而 playbook 是一种与 Ad-Hoc 完全不同的一种方式，playbook 功能将更加的强大。

因为 ansible 的 Ad-Hoc 这样的命令模式，他只是一个简单的命令，只能执行一个 task 任务，不能给其赋予过多的任务指令，并且多指令的情况下若使用命令，这样没有记录下来，以后又有同样的操作，又全部重新输入一遍，这样是非常的低效率的

而 playbook 就是一个任务列表，这个列表中的可以包含一个或者多个 'plays'，所有的操作放在这么一个文件中，然后一次性的执行。


playbook 是使用的 yaml 的语法格式来书写的，为的就是将语法做到最小化，避免 playbook 做成一种编程语言或者做成脚本，但是同时它也不是一个配置模型或者是过程模型。

简单来说便是官方使用 yaml 来书写 playbook，就是不要过多的语法，只是简单的格式要求，它像 XML 或 JSON 是一种利于人们读写的数据格式，将 playbook 的书写要求，学习代价降到最低。并且使用 YAML 也是因为大多数编程语言都有使用 YAML 的库.


### 2.2 YAML 的语法

>**YAML**是一个可读性高，用来表达资料序列的格式

>YAML是"YAML Ain't a Markup 
>在 Ansible 中，几乎每个 YAML 文件的是以一个列表开始的，在列表中的每一个项都是列表中的 key/value 这样的键值对。通常称之为 "hash"、"dictionary"

整体的结构是通过空格来展示。列表里的同级别的项以 `-` 开头来代表，Map 里的键值对用 `:` 分隔也就是 `key: value`。这几乎是 YAML 的所有要求了。

就是这么的简单，我们就会了 YAML 的基本使用。

在 Ansible 的官方文档中作者写到他有这样的一个习惯，而这习惯让人感觉有头有尾，有结构的，当然是可选的，不是必须这么做的。

这个习惯就是在每个 YAML 文件的开头加上 `---` 来表示从这里开始整个文件，然后再结尾上加上 `...` 来表示整个文件的结束。

如下面这个例子：

```
---
- hosts: test
  sudo: yes
  vars: 
      apt_packages_ca: 
         - apt-transport-https
         - ca-certificates

  tasks:

    - name: write deb url of docker to docker.list
      blockinfile:
          dest: /etc/apt/sources.list.d/docker.list
          marker: ""
          block: |
             deb https://apt.dockerproject.org/repo ubuntu-trusty main  

...



```
### 2.3 playbook基本语法


- `主机与用户`
 - hosts 行的内容是一个或多个组或主机，以逗号为分隔符
- `Tasks 列表`  
   -每一个 play 包含了一个 task 列表（任务列表）.一个 task 在其所对应的所有主机上（通过 host pattern 匹配的所有主机）执行完毕之后,下一个 task 才会执行.  

----------------

每个 playbook 中必有 `hosts` 来指定你需要操作的机器，`tasks` 来开始你的任务，其中 `name` 是为了格式，也是为了方便调试，将各个任务分隔开，方便若是在出现错误的时候知道是停止，报错的，易于调试，需要注意的有:

- `key: value` 的形式中，`:` 后面必定有一个空格，不然会有格式错误
- 当内容太多需要分行输入但是内容本来又应该属于一行的时候，可以使用 `|` 来分割
- 在使用一些变量的时候需要像这样 `"{{ item }}"` 来括起来
- 在使用这样一些大段语句时使用 `""` 括起来，例如 `foo: "somebody said I should put a colon here: so I did"`

在 playbook 中可以使用变量，变量的独立出来，方便后期的修改、维护。也可以供其它的任务使用，以实现代码的重用。变量的定义可以在：

- Inventory 中
- playbook 全局中或者某个任务中
- 使用 roles 结构放在一个单独的文件中，
- 使用 registered 模块，注册变量。（主要用于调试，和判断）

在上面的那个例子中，便是设置的当前 playbook 的全局变量，在任务之前使用 `vars: `，在里面声明变量，以及变量的值，然后再任务中以循环来读取使用变量，更多变量的高级用法可以查看[官方文档](http://docs.ansible.com/ansible/playbooks_variables.html#advanced-syntax)

当然在 playbook 中也少不了条件判断与循环的使用，如这个例子：

```
---
- hosts: test

  tasks:
     - name: test Loop and condition
       command: echo {{ item }}
       with_items: [ 0, 2, 4, 6, 8, 10 ]
       when: item > 5

...
```

用 `item` 放在需要使用循环读取值得变量位置，用 `with_items` 来指定需要循环读取出来的值，当然这里的值列表也可以像上个例子一样定义一个全局的变量，使用 `when` 来列出判断的条件，条件为真则执行上面的命令，条件为假则 skipping 略过，上面的例子便是只输出大于5的值。

![2.3](https://dn-simplecloud.shiyanlou.com/1135081470720445826-wm)

还记得我们在上一个试验中提到过的，playbook 中是一个任务的列表，有一个或者多个任务，当多个任务的时候若是中间的某个任务出错，出现 Failed 的情况，后面的任务将不会继续执行，任务列表将终止在这个任务

而在 Ansible 2.0 的版本出了新的特性，`Blocks` 将可以解决这样的问题出现，借用[官网](http://docs.ansible.com/ansible/playbooks_blocks.html)中的一个例子给大家讲解下：

```
 tasks:
  - block:
      - debug: msg='i execute normally'
      - command: /bin/false
      - debug: msg='i never execute, cause ERROR!'
    rescue:
      - debug: msg='I caught an error'
      - command: /bin/false
      - debug: msg='I also never execute :-('
    always:
      - debug: msg="this always executes"
```

在这个任务中，`block` 部分的指令将正常的执行，若是在执行的时候出现了错误那么 `rescue` 部分将开始执行，补救前面的错误等等，而 `always` 部分便是无论前面发生什么情况，有出现错误，还是成功的执行了都会执行，就像在 C++ 中的 do while 一样，无论如何都会执行。

在例子中便是，一开始 `block` 部分会正常的执行，若是没有错误将跳过 `rescue` 直接执行 `always`，而不幸的是这里错误了，然后他将去执行 `rescue` 看看有没有什么补救的方法，不幸的是这个补救方法也是不可行的，那么 `always` 就是一个必须可行的方案。

这里只是简单的介绍了 playbook 中的一些变量，循环，判断的一些用法，在[官网的文档](http://docs.ansible.com/ansible/playbooks.html)中，还有许多让你耳目一新的用法,希望大家灵活使用，多多查阅文档。

执行 playbook 的方式是：

1. vim xxxx.yaml (首先编写一个 yaml 的 playbook)
2. ansible-playbook xxxx.yaml （使用 ansible-playbook 命令来执行）

### 2.4 ansible 模块的介绍

前人栽树，后人乘凉。在我们使用 ansible 之前，已经有无数的前辈写了各种常用的模块来供我们使用，[官方文档](http://docs.ansible.com/ansible/list_of_all_modules.html)中列有迄今为止所有的模块，以及功能描述和用法

这里将简单介绍常用的模块以及其参数

首先就是我们最常用的 `shell` 模块了,用法非常的简单，就像直接输入执行命令一般，其实一般的命令已经被做成了模块，可以直接使用了，只是有些模块功能并没有做的那么全面，命令的部分参数没有完全转化成模块的参数所以在这种万不得已的情况下，我们会用 shell 模块来补救。

例如我们并没有在模块列表中找到较好的方法来编译安装一个程序的时候，我们就会使用的 shell 模块来补救一下：

```
    - name: make install gucad
      shell: ./configure --with-init-dir=/etc/init.d && make && make install && ldconfig && update-rc.d guacd defaults
      args:
          chdir: /home/ubuntu/src/guacamole-server-0.9.9/
```

比如一些简单的功能，如创建一个文件，复制一个文件都是有专门的模块来执行的：

```
  - name: add docker source list file for  install docker
    file:
         path: /etc/apt/sources.list.d/docker.list 
         state: touch
         owner: root
         mode: 'u+r,g+rw' 


  - name: add 163 source to speed up apt-get
    copy:
         src: /Users/richard/Downloads/Github/test-ansible/test-ansible/sources.list
         dest: /etc/apt/sources.list
```

例如在安装 docker 的时候，在 `sources.list.d` 中并没有 docker.list 这个文件，这个时候我们就可以使用 file 模块来穿件这个文件了。

当我们想将源列表直接清空，添加上我们想要的源，但是又不想在 YAML 里面书写过多，而且我们本地有这么一个文件，就可以使用 copy 这个模块，将我们在本地想要的文件复制到远程的主机中

像在 ubuntu 中用的最多的莫过于 `apt-get update`、`apt-get install` 等等，这当然也是逃不出变成一个模块的命运啦

```
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

当我们需要修改文本中的某个单词的时候，我们可以使用 replace 模块，我们需要修改某行文字的时候，我们可以使用 lineinfile 模块，当我们需要添加大段文字的时候我们可以使用 blockinfile 模块，等等。善用官方文档[Ansible All Modules](http://docs.ansible.com/ansible/list_of_all_modules.html)

就像一个字典一样，随你翻阅，所有现有的模块，参数都在这里面，有新的模块加入其中，这个文档都会随时更新的。

当然若是实在找不到我们需要的模块，但是我们有经常使用到，我们可以自定义模块使用。

## 三、实验步骤
下面我们将给大家讲解一个实例，安装一个docker的例子。
### 3.1  playbook 的实战

说的在多也不如来一个实例看的明白，这里给大家一个安装 docker 的例子

- 首先我们创建一个yaml。

```
vim install-docker.yaml
```

- 然后书写安装 docker 的 yaml

```
- hosts: test #对 test 组执行下面的任务
  sudo: yes  #下面的任务将有 sudo 的执行权限
  vars: 
      apt_packages_ca: 
         - apt-transport-https
         - ca-certificates
#定义一个变量，安装多个包，以免增加多个类似的代码
  tasks:

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

    - name: add CA certificates and ensure installed.
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


```

- 然后使用 ansible-playbook 运行

```
ansible-playbook install-docker.yaml
```

playbook 的运行（docker 的安装比较缓慢，请大家耐心等待一下，大概10分钟）

![3.1-1](https://dn-simplecloud.shiyanlou.com/1135081470723646650-wm)

![3.1-2](https://dn-simplecloud.shiyanlou.com/1135081470724897133-wm)

成功安装上 docker 的验证

![3.1-3](https://dn-simplecloud.shiyanlou.com/1135081470723686727-wm)

从图中我们可以看到，我们成功的是用 Ansible 的 playbook 在本机上安装上了 docker，设想一下若是我们有 100 台，主机我是一台一台的登陆上去执行命令安装，或者执行 shell 脚本安装，那得装到什么时候呀，而使用 playbook 将一键搞定。

## 五、实验总结

如果在安装docker的过程中觉得很慢，可以将docker官方的源更换为阿里的源
```
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
```
通过本实验我们初步的接触到 playbook 的优势，以及书写 playbook 所该注意的地方，最后通过一个例子真正的使用了 playbook，整个实验中知识点只是粗略的介绍，更多的详细介绍，以及高级使用多多查看[官方文档](http://docs.ansible.com/ansible/playbooks.html)，若是介绍的英文看起来吃力可以查看[中文文档](http://ansible-tran.readthedocs.io/en/latest/)

## 六、参考资料
[1] Ansible Document:<http://docs.ansible.com/ansible/playbooks_intro.html>

[2] Ansible All Modules:<http://docs.ansible.com/ansible/list_of_all_modules.html>

[3] Ansible 中文文档：<http://ansible-tran.readthedocs.io/en/latest/>