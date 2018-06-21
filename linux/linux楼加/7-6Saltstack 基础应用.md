# Saltstack 基础应用

## 1. 实验介绍

### 1.1 实验内容

本节实验节点的是一种全新的运维管理系统——`Saltstack` ，它非常的简单，能够在几分钟内运行起来，足以用速度来取代复杂性，并且可以扩展来管理数以万计的服务器，同时在几秒内就能和每个系统进行通信。

### 1.2 实验知识点

+ Saltstack 执行命令

+ Saltstack 组件详解

### 1.3 推荐阅读

+ [SaltStack 官方文档](https://docs.saltstack.com/en/latest/contents.html)

+ [SaltStack 中文文档](http://docs.saltstack.cn/)

+ [中国 SaltStack 用户组](http://www.saltstack.cn/kb/cssug-conf-2016-1-beijing/)

## 2. 执行命令

Saltstack 的核心功能之一就是在远程主机上运行预先定义好的命令或者直接远程执行任意命令。

### 2.1 语法

在上一节实验中我们在安装和配置好 Saltstack 之后，通过 `salt` 命令来测试了主从服务器之间的通信，而我们只要确保已经安装了一个 master 和至少一个 minion 就可以通过 `salt` 命令在 minion 上远程执行命令了。

`salt` 命令的语法的三个主要组成部分如下格式：

```bash
salt <target> <module.function> [arguments]
```

示例说明：

例如我们在远程主机上安装一个名为 `cowsay` 的软件，如下命令：

```bash
$ sudo salt '*' pkg.install cowsay
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516848195678.png-wm)

**target**

主要用于确定那些系统应用命令，默认情况下使用主机名匹配，但也有其他很多方式可以选择和过滤。

target 允许过滤哪些 minion 应该来运行后面的功能。默认过滤器是 minion id 上的一个 glob（简化的正则表达式）。如下：

```bash
# 星号（*）匹配零个或任意多个字符

# 表示匹配所有机器
salt '*' test.ping

# 表示匹配主机名以 example.org 结尾的机器
salt '*.example.org' test.ping
```

除了默认的过滤选择以外，还可以有以下几种过滤方式：

+ 通过正则表达式过滤

```bash
salt -E 'virtmach[0-9]' test.ping
```

+ 在列表中指定

```bash
salt -L 'foo,bar' test.ping
```

+ 可以通过 Grains 中的系统信息过滤（后面会讲）

```bash
salt -G 'os:Ubuntu' test.ping
```

+ 以及将上面的组合起来进行过滤选择

```bash
salt -C 'G@os:Ubuntu and webser* or E@database.*' test.ping
```

**module.function**

这个是 salt 命令的核心，主要由一个模块和函数组成，其中 salt 内置了模块来安装软件、复制文件、检查服务以及其他大部分自动执行的任务。（Salt 社区付出巨大的努力后成功创建了数百个简化大多数管理任务的功能，同时在所有支持的平台上一致使用相同的功能）

function 是由 module 提供的一些功能，正如上面提到 salt 社区已经创建了大量可用的模块功能，大家可以参考下官方提供的[完整模块列表信息](https://docs.saltstack.com/en/latest/ref/modules/all/index.html#all-salt-modules)。

比如说，执行 shell 命令，来列出某个路径下的文件。

```bash
$ sudo salt '*' cmd.run 'ls /etc/salt'
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516850832458.png-wm)

显示磁盘信息功能：

```bash
$ sudo salt '*' disk.usage
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516851005224.png-wm)

```bash
$ sudo salt '*' network.interfaces
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516851172626.png-wm)

还有许多其他的模块功能，这里就不一一演示了，大家可以参考官方文档进行练习。

**arguments**

提供调用的函数所需要的其他任何额外的数据参数。

参数可以是用空格划分或者是使用关键字（keyword）以及还可以被格式化为 `YAML` 形式。

+ 利用空格分隔如下：

```bash
salt '*' pkg.file_list ftp postfix
```

+ 使用关键字，通常的形式是：`kwargs=value`，如下所示

```bash
salt '*' pip.install salt timeout=5 upgrade=True
```

+ 参数还可以被格式化为 `YAML` 形式：

```bash
salt '*' cmd.run 'echo "Hello: shiyanlou"'
```

### 2.2 其他常用命令

（这里我们只列出较常用的命令的用法）

+ `salt-key`：主要用于密钥管理

```bash
salt-key [options]
```

```bash
# 查看所有 minion-key
salt-key -L

# 指定接收某台 minion-key
salt-key -a <key-name>

# 指定删除某台 minion-key
salt-key -d <key-name>

# 接受所有的 minion-key
salt-key -A

# 删除所有的 minion-key
salt-key -D
```

+ `salt-run`：主要用于执行 Salt Runners （这个概念后面会讲）的前端命令。通常是在 master 端执行一些简单的模块来执行相关功能。

```bash
salt-run [options] <function> [arguments]
```

```bash
# 查看所有 minion 状态
salt-run manage.status

# 查看所有没在线 minion
salt-run manage.down

# 查看所有在线 minion
salt-run manged.up
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516867802526.png-wm)

+ `salt-call`：通常是在 minion 上执行，minion 自己执行可执行的模块，而不是通过 master 来传递 job。

```bash
salt-call [options]
```

```bash
# 执行test.ping命令
$ sudo salt-call test.ping

# 执行 cmd.run 函数
$ sudo salt-call cmd.run 'ifconfig'
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516868634151.png-wm)

+ `salt-cp`：将一个或多个文件复制到一个或多个 minion 中。

```bash
salt-cp '*' [ options ] SOURCE DEST

salt-cp -E '.*' [ options ] SOURCE DEST

salt-cp -G 'os:Arch.*' [ options ] SOURCE DEST
```

```bash
salt-cp '*' testfile /tmp
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517191846957.png-wm)

Saltstack 常用命令操作视频：


`@
http://labfile.oss.aliyuncs.com/courses/980/week10/5-1.mp4
@`

## 3. Saltstack 组件详解

在前面一节我们简单提到了 Saltstack 的组件信息，这里我们对其中的内容进一步来学习理解。

### 3.1 States

States 系统会大量利用远程执行系统，是在远程执行后得到的一个状态。state 的模块和前面的远程执行模块很相似，但也有一个较大的区别，state 模块包含逻辑查看系统是否已经处于正确的状态上。实际上，state 模块通常会简单调用远程执行模块来完成工作。

State 的核心是 `sls（SaLt State file）`文件，`sls` 的默认文件格式是 `YAML` 格式，同时会默认使用 `Jinja` 模板。通常 `state`、和后面会讲到的 `pillar` 以及 `top file` 都会用 sls 文件来编写。

State 主要是用来描述系统，服务，配置文件等的状态，常常被称为配置管理。state 文件我们会默认放在 `/srv/salt` 下，这和我们的 `/etc/salt/master` 配置文件中的 `file_roots` 参数的设置有关。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516877273071.png-wm)

下图是官方给出的有关 sls 文件的示例图片：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516872681151.png-wm)

例如图上的 `apache2.sls` 这个文件的内容，是声明一个叫 apache2 的状态 ID。

```bash
apache2:        # State ID，全文件唯一。如果模块没跟 `-name` 就使用默认用的 ID
    pkg:        # module.function
      - installed
    service:     # module.function
      - running
      - require:      # 表示依赖的系统
        - pkg: apache2 # 表示 id 是 apache2 的 pkg 状态
```

> 说明：

> `pkg` 代表 pkg 模块，`installed` 是模块下的一个函数，描述的是状态，该函数来表示 apache2 是否部署，返回值为 `True` 或者 `False`，为真时，表示状态 `OK`，否则就会去满足该状态(即下载安装 apache2 来满足)，如果满足不了会提示 `error`，在该模块上面省略了参数 `-name: apache2`，因为 ID 已经定为 apache2。
>
> `service` 模块主要是描述 service 状态的函数，`running` 状态函数表示 apache2 在运行，省略 -name 不再赘述，-require 表示依赖系统，依赖系统是 state system 的重要组成部分，这里表示了 apache2 服务的运行需要依赖 apache2 软件的部署。

这里我们可以在 `/srv/salt` 下创建一个 `apache2.sls` 的 states 文件，如下：

```bash
$ cd /srv/salt
$ sudo vim apache2.sls

 apache2:
     pkg:
      - installed
     service:
         - running
         - require:
           - pkg: apache2
```

保存退出即可。

### 3.2 Top File

`Top File` 主要用于将 states files 和 pillar 数据匹配到 salt minions。应用于每个系统的状态（state）是由 top file 中指定的 target 决定。

在创建一个 Top 文件时，需要考虑你要配置什么样的系统，以及每个系统的通用和独特之处，同时每个系统可以接受多种配置，我们可以从最一般的进行操作。

如下面的官方示例，左边的列表就是一个 top 文件，列表中的每一项就是一个 state，通过使用 target 来定义每个 minion 上应用的状态。右边用 YAML 格式显示了相应的 top 文件。如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4131timestamp1516950790320.png/wm)

这里我们创建一个应用于之前创建的 apache2 的 top 文件，如下：

```bash
$ sudo vim top.sls

base:
    '*':
      - apache2
```

保存退出，然后通过下面这个命令来应用 apache2 的状态

```bash
$ sudo salt '*' state.apply apache2
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517192662430.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517192601255.png-wm)

或者可以使用 `salt '*' state.sls apache2` 这个命令来运用也是类似的效果，不过 saltstack 和之前学到的 ansible 类似也是幂等性，再次运行没有什么变化。

### 3.3 Runners

Runner 提供在 Salt master 上执行支持任务的模块。Salt runner 会报告工作状态，连接状态，从外部 API 读取数据，查询连接的 minions 等等。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4131timestamp1516879175673.png/wm)

（图片来自官方）

+ Runner 使用 `salt-run` 命令进行调用。（前面已经讲到了这个命令）

+ 将参数传递给 Salt runners 的语法与用于将参数传递给 Salt 执行模块的语法相同。

Runner 提供了 salt 的核心功能之一，按照定义的顺序运行命令并且跨多个应用程序进行配置，通过协调 Runner 就可以获取自己的部分。例如：使用 Orchestrate 在许多系统中协调配置部署等。

### 3.4 Grains

`Grains` 是用来获取有关系统的数据，也是一个关于底层管理系统的静态信息，包括操作系统（OS），内存和许多其他系统属性。也可以为任何系统定义自定义 Grains，这是一种系统变量。

结构图如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4131timestamp1516879993460.png/wm)

（此图来自官方文档）

Grains 数据是相对静态的，就比如说 OS 是 Ubuntu ，除非是重装系统才会改变。但是如果系统信息发生变化(例如，网络设置发生了变化，或者新的值被分配给定义的 Grain)，Grain 数据也会被刷新。

列出 Grains：

1. 通过 `grains.ls` 模块可以列出可用 grains：

```bash
$ sudo salt '*' grains.ls
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517192980595.png-wm)

2. 通过 `grain.items` 模块可以列出 grain 的数据：

```bash
$ sudo salt '*' grains.items
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517192986883.png-wm)

在 minion 配置文件（`/etc/salt/minion`）中静态分配 grain，其中文件格式也是使用的 YAML：

```bash
grains:
  roles:
    - webserver
    - memcache
  deployment: datacenter4
  cabinet: 13
  cab_u: 14-15
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517192992402.png-wm)

选学部分：

我们除了可以通过执行模块中的 public 功能来静态分配 grain 以外，还可以通过 python 来自定义一个 grain 模块的函数功能，不过模块中的函数返回的必须是一个 `python dict`，字典中的 key 是 grains 的名称，value 就是 grain 的值。

通常我们会将定义的模块放在 `_grains` 的子目录下，这个目录的默认路径有配置文件 `/etc/salt/master` 下的 `file_root` 参数来指定，默认一般是放在 `/srv/salt/_grains` 下（默认没有这个目录需要我们创建）。

> 虽然说在 salt 中基本不用编写代码，直接像之前那样调用一些模块或者 YAML 语句，但是如果可以自定义一些模块代码来实现功能也是一件不错的事。这里我们就简单的来写点 python 的代码。

Grains 模块的 python 代码非常简单书写，因为只用返回一个字典即可，python 代码的官方格式如下：

```bash
def yourfunction():
     # 初始化一个 grains 字典
     grains = {}
     # 设置 grains 的逻辑代码
     grains['yourcustomgrain'] = True
     grains['anothergrain'] = 'somevalue'
     # 返回 grains
     return grains
```

下面我们就用 python 来编写一个 grains 模块，然后同步到 minion 中去。

1. 首先需要建立一个 `_grains` 目录

```bash
$ sudo mkdir -p /srv/salt/_grains
```

2. 编写 grains 文件，返回一个字典。

```bash
$ sudo vim salt1.py

def hello():
   grains = {}
   grains['hello shiyanlou'] = 'saltstack'
return grains
```

保存退出

3. 同步 grains 到 minion 上去

```bash
$ sudo salt '*' saltutil.sync_grains
$ sudo salt '*' saltutil.sync_all
$ sudo salt '*' state.highstate
```

> `saltutil.sync_grains`：将 grains 模块同步到 minion 上。

> `saltutil.sync_all`：将服务器中的所有动态模块同步到特定环境。该函数会同步自定义模块，states，beacons，grains，returners，output modules，renderers 和 utils。

> `state.highstate`：从这个 minion 的 master 上检索 state 数据并执行。

![实验楼](https://dn-simplecloud.shiyanlou.com/2767331517193461360-wm)

4. 验证

```bash
$ sudo salt '*' grains.items
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517193537639.png-wm)

可以看到我们自定义的 grains 模块已经同步上去了。

上面我们学习了如何自定义 grains ，并且我们也知道了几种配置 grain 的方法，其中 core grain 会被我们自定义的 grains 同名覆盖掉，所以这里再补充一个知识就是配置 grains 的优先顺序，如下所示：

+ 1. Core grains

+ 2. 在 `/etc/salt/minion` 中配置定义 grains

+ 3. 在 `_grains` 目录下定义 grains ，并同步到 minion 上


### 3.5 Pillar

`Pillar` 是一种用户定义的变量。这些安全变量被定义并存储在 Salt Master 上，然后使用 target 分配到一个或多个 minion 上。pillar 存储的数据，包括：端口，文件路径，配置参数和密码等。

Pillar 数据使用公钥进行加密，并通过安全通道发送，所以 Pillar 也非常适合分发安全数据，如密码和 ssh 密钥，因为它只能由目标 minion 解密。并且 pillar 数据永远不会写在 minion 的磁盘上。

结构图如下所示：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516870372175.png-wm)

（此图来自官方文档）

在 salt 中 pillar 是非常重要的组件之一，它不同于 grains 一样将会分发配置到所有的 minions 上，而可以指定一些信息到指定的 minion 上。除此以外，它保存的数据是动态的，非常适合较敏感的数据，如密码、key 等。同时，pillar 也是用 sls 文件来编写的，主要格式采用键值对。

声明 pillar 也是在 salt 服务器中使用和 `file_root` 结构相匹配的 `pillar_root` 设置。`pillar_roots` 选项会将环境映射到目录。然后根据与 top 文件将 pillar 数据映射到 minion 上。

如下为默认的 `/etc/salt/master` 配置文件中的参数设定：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4131timestamp1516962925994.png/wm)

下面我们还是通过一个实际例子来学习一下怎么使用 pillar。

1. 指定 pillar_root，创建目录

```bash
$ sudo mkdir -p /srv/pillar
$ cd /srv/pillar
```

2. 编辑 pillar 文件

```bash
$ sudo vim packages.sls

bind: bind9
```

3. 创建 top file

```bash
$ sudo vim top.sls

base:
    '*':
      - packages
```

4. 刷新 pillar 数据

```bash
$ sudo salt '*' saltutil.refresh_pillar
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517194033419.png-wm)

```bash
$ sudo salt '*' pillar.items
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1517194161430.png-wm)

Pillar 实例操作视频：


`@
http://labfile.oss.aliyuncs.com/courses/980/week10/5-2.mp4
@`

## 4. Python

虽然 saltstack 不需要去编写 python 或其他代码来使用 salt，但是，能都读懂 python 或者至少能够知道 python function 的文档也是对我们很有用的。下面我们学习一下在 salt 中的一些 python 基础知识。

### 4.1 Modules

所有的 module 都是存放在 `salt source` 文件夹中，每个子系统都有一个单独的文件夹，每个模块也都是一个单独的 `file.py`。同时，在 salt 中，之前提到的插件（plug-in）也都是一个 python 模块，可以将模块看作是管理应用程序（mysql，docker），系统组件（磁盘，文件）或与外部系统（gitfs）交互的一组函数。由此可以看出，模块的种类也就十分繁多，但是这里我们通常直接使用的只有两种类型的模块：执行模块（`salt.module`）和状态模块（`salt.states`）。

模块名称的格式为：`salt.subsystem.module`，例如：文件执行模块（salt.modules.file）、uwsgi stats 服务器执行模块（salt.modules.uwsgi）等。不过可能会混淆的就是执行模块，他们的开始是 salt.module ，这是因为它们是 salt 的初始版本中的第一个也是唯一的模块。

更多关于模块的内容这里不做详解，例如[模块的编写](https://docs.saltstack.com/en/latest/ref/modules/index.html#modules-are-easy-to-write)，大家可以参考官方文档中的说明。

### 4.2 function

在前面的学习中，我们都已经运用了很多 salt 的模块功能，所以这里可以简单就将功能看作是调用的模块中的特定命令来管理和配置系统。例如：`salt.modules.pkg.install`、`salt.modules.network.interfaces` 、`salt.modules.user.add` 等等都是常见的执行的功能。（这里就不再赘述了）

### 4.3 argument

在前面我们讲到了执行命令中调用参数的几种方式，但是在参数语法上对于远程执行命令和状态（states）之间会有所不同，这里我们对这两种做进一步的讲解。

**执行函数参数**

在命令行上调用 salt 时，执行参数往往是作为一个附加值进行传递，即：`argument=value`。只需空格进行分隔，然后以特定的顺序传递即可。就如之前列举的几种方法：常规参数（argument）、关键字参数（kwargs）、列表以及字典等形式。（前面已经介绍了这里就不做重复）

**state 函数参数**

因为状态函数是有状态的，所以需要先进行判断状态的情况，再进行函数的调用执行。而状态函数主要是使用 YAML 语法格式在 sls 文件中调用，同样它也可以通过常规参数、关键字参数、列表以及字典的形式在 YAML 中使用。

+ 关键字参数

```bash
user.present:
    - name: shiyanlou
    - shell: /bin/zsh
```

+ 列表

```bash
 pkg.installed:
    - pkgs:
      - ftp
      - postfix
      - samba
```

+ 字典

```bash
pkg.installed:
    - name: mypkgs
    - sources:
      - foo: salt://foo.deb
      - bar: http://somesite.org/bar.deb
```

## 5. 总结

本节实验主要对 saltstack 的运用做了进一步的讲解，通过执行命令的操作和各个重要组件的结构，让我们能够更好的了解 saltstack ，在后一个实验我们就将知识进行实战练习。