---
show: step
version: 1.0
enable_checker: true
---
# Docker 容器与镜像

## 1. 实验介绍

#### 1.1 实验内容

本节内容我们将学习 Docker 容器和镜像的一些管理操作。

#### 1.2 实验知识点

+ Docker 基本命令
+ Docker 容器操作命令
+ Docker 镜像管理及制作

## 2. docker 命令

下面我们将会学习docker 命令。

### 2.1 查看系统信息

除了查看版本信息之外，在 `docker` 的命令组中还有一个较为常用的命令，查看系统的一些相关信息：

```bash
docker system info

或者使用命令

docker info
```

运行截图如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515568022323.png/wm)

### 2.2 help

我们可以直接通过 `help` 或者使用 `man` 手册的方式查看相关命令的详细说明，例如我们直接使用如下命令:

```bash
$ docker --help
```

我们可以看到运行结果如下图所示。如果之前有学习过 docker 相关知识的同学，可能会发现一些不一样的地方。即下图中标出的 `Management commands` 和 `Commands`。在 1.13 版本之前，`docker` 并没有 `Mangement commands`。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515566385566.png/wm)

### 2.3 Management Commands

在 `Docker 1.12 CLI` 中大约有四十个左右的顶级命令，这些命令没有经过任何组织，显得十分混乱，对于新手来说，学习它们并不轻松。

而在 `Docker 1.13` 中将命令进行分组，就得到如上图中所示的 `Management Commands`。例如经常使用的容器的一些相关命令：

```bash
# 创建一个新的容器，下面分别为 Commands 和 Management Commands，作用相同
docker create
docker container create

# 显示容器列表
docker ps
docker container ls

# 在一个新的容器中运行一个命令
docker run
docker container run

...
```

如上所示，对于新的命令而言相比于旧命令明显更具有可读性。并且在实验环境中的 `docker` 版本以及最新版本中两者都是有效的命令，所以在这里我们将一些常用的命令，及其对应的 `Management Commands` 命令都列举出来，方便大家在后续的学习过程中可以进行参考。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517897142154.png/wm)

Docker 基本命令操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week11/2-1.mp4
@`

## 3. 容器

下面我们将会学习容器。

### 3.1 查看容器列表

查看容器列表可以使用如下命令：

```bash
docker container ls [OPTIONS]

或者旧的命令

docker ps [OPTIONS]
```

在使用命令时，我们可以使用一些可选的配置项 `[OPTIONS]`。

+ `-a` 显示所有的容器

+ `-q` 仅显示 `ID`

+ `-s` 显示总的文件大小

> 这些配置项对于上述的两个命令都是有效的，在后面的内容不会再特殊说明。

默认情况下，直接使用该命令仅显示正在运行的容器，如下所示：

```bash
$ docker container ls
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515566501674.png/wm)

此时并没有处于运行中的容器，所以显示为空。我们可以使用 `-a` 参数，来显示所有的容器，并加上 `-s` 选项，显示大小，命令如下：

```bash
$ docker container ls -a -s
```

限于界面大小，为了图片的格式更加友好，这里仅 **截取部分输出结果**：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515566676304.png/wm)

### 3.2 创建一个容器

#### docker run

首先，我们回顾在上一节使用到的 `docker run hello-world` 命令，该命令的格式为：

```
docker run [OPTIONS] IMAGE [COMMAND]
```

对应于 `Management Commands` 的命令为：

```bash
docker container run [OPTIONS] IMAGE [COMMAND]
```

上述两个命令的作用相同，`docker run` 命令会在指定的镜像 `IMAGE` 上创建一个可写的容器（因为镜像是只读的），然后开始运行指定的命令 `[COMMAND]`。

一些常用的配置项为：

+ `-i` 或 `--interactive`， 交互模式

+ `-t` 或 `--tty`， 分配一个 `pseudo-TTY`，即伪终端

+ `--rm` 在容器退出后自动移除

+ `-p` 将容器的端口映射到主机

+ `-v` 或 `--volume`， 指定数据卷

> 关于该命令的详细参数较多，并且大多数参数在很多命令中的意义是相同的，将在后面的内容中使用到时进行相应的介绍。

我们指定 `busybox` 镜像，然后运行命令 `echo "hello shiyanlou"` 命令，如下所示：

```bash
$ docker container run busybox echo "hello shiyanlou"
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515568678271.png/wm)

在上图中，我们可以看到该命令执行的过程：

1. 对于指定镜像而言，首先会从本地查找，找不到时将会从镜像仓库中下载该镜像

2. 镜像下载完成后，通过镜像启动容器，并运行 `echo "hello shiyanlou"` 命令，输出运行结果之后退出。

在执行命令之后，容器就会退出，如果我们需要一个保持运行的容器，最简单的方法就是给这个容器一个可以保持运行的命令或者应用，比如 `bash`，例如我们在 `ubunutu` 容器中运行 `/bin/bash` 命令：

```bash
$ docker container run -i -t ubuntu /bin/bash
```

对于交互式的进程而言（例如这里的 bash），必须将 `-i` 和 `-t` 参数一起使用，才能为容器进程分配一个伪终端，通常我们会直接使用 `-it`。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516695039457.png/wm)

如上所示，我们已经进入到分配的终端中了，这时如果我们需要退出 `bash`，可以使用以下两种方式，它们的效果完全不同：

1. 直接使用 `exit` 命令，这时候 `bash` 程序终止，容器进入到停止状态

2. 使用组合键退出，容器仍然保持运行的状态，可以再次连接到这个 `bash` 中，组合键是  `ctrl + p` 和 `ctrl +q`。即先同时按下 `ctrl` 和 `p` 键，再同时按 `ctrl` 和 `q` 键。就可以退出

这里我们使用第二种方式，然后使用 `docker container ls` 命令，可以看到该容器仍然处于运行中。

#### docker container create

严格意义上来讲，`docker run` 命令的作用并不是创建一个容器，而是在一个新的容器中运行一个命令。而用于创建一个新容器的命令为

```
docker container create [OPTIONS] IMAGE [COMMAND] [ARG...]

或者使用旧的

docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

该命令会在指定的镜像 `IMAGE` 上创建一个可写容器层，并 **准备** 运行指定的命令。需要着重强调的是，这里是准备运行，并不是立即运行。即该命令只创建容器，并不会运行容器。

一些常见的配置项如下所示：

+ `--name` 指定一个容器名称，未指定时，会随机产生一个名字。

+ `--hostname` 设置容器的主机名

+ `--mac-address` 设置 `MAC` 地址

+ `--ulimit` 设置 Ulimit 选项。

> 关于上述提到的 `ulimit`，我们可以通过其对容器运行时的一些资源进行限制。`ulimit` 是一种 `linux` 系统的内建功能，一些简单的描述，可以参考 https://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/  ，而对于在下面我们将要设置的部分值的含义，可以参考
> https://access.redhat.com/solutions/61334 。

除此之外，关于创建容器，我们还可以设置有关存储和网络的详细内容，将会在下一节的内容中进行介绍。

如下示例，我们指定容器的名字为 `shiyanlou`，主机名为 `shiyanlou`，设置相应的 `MAC` 地址，并通过 `ulimit` 设置最大进程数（`1024:2048` 分别代表软硬资源限制，详细内容可以参考上面的链接），使用 `ubuntu` 的镜像，并运行 `bash`：

```bash
$ docker container create --name shiyanlou --hostname shiyanlou --mac-address 00:01:02:03:04:05 --ulimit nproc=1024:2048 -it ubuntu /bin/bash
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516694843573.png/wm)

此时，容器创建成功后，会打印该容器的 `ID`，这里需要简单说明一下，在 `docker` 中，容器的标识有三种比较常见的标识方式：

+ `UUID` 长标识符，例如 `1f6789f885029dbdd4a6426d7b950996a5bcc1ccec9f8185240313aa1badeaff`

+ `UUID` 短标识符，从长标识符开始，只要不与其它标识符冲突，可以从头开始，任意选用位数，例如针对上面的长标识符，可以使用 `1f`，`1f678` 等等

+ `Name` 最后一种方式即是使用容器的名字

在容器创建成功后，我们可以查看其运行状态，使用如下命令：

```bash
# 此时该容器并未运行，需要使用 -a 参数
$ docker container ls -a
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516695279979.png/wm)

新创建的容器的状态 (`STATUS`) 为 `Created`，并且其容器名被设置为对应的值。

### 3.3 查看容器的详细信息

查看容器的详细信息可以使用如下命令：

```bash
docker container inspect [OPTIONS] CONTAINER [CONTAINER...]

或者旧的

docker inspect [OPTIONS] CONTAINER [CONTAINER...]
```

例如我们查看刚刚创建的容器的详细信息就可以使用以下命令：

```bash
# 使用容器名
$ docker container inspect shiyanlou

# 使用 ID ，因生成的 ID 不同，需要修改为相应的 ID
$ docker container inspect 1f6789

$ docker container inspect 1f6
```

例如，我们查看刚刚创建的名为 `shiyanlou` 的容器的 `MAC` 地址，就可以使用如下命令：

```
$ docker container inspect shiyanlou | grep "00:01"
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516695478022.png/wm)

### 3.4 容器的启动和暂停及退出

容器的启动命令为：

```
docker container start [OPTIONS] CONTAINER [CONTAINER...]
```

对于上面我们创建的容器而言，此时处于 `Created` 状态，需要使用如下命令启动它：

```bash
$ docker container start shiyanlou
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516695629395.png/wm)

此时，运行一个容器我们分成了两个步骤，即创建和启动，使用的命令如下：

```bash
# 创建
$ docker container create --name shiyanlou --hostname shiyanlou --mac-address 00:01:02:03:04:05 --ulimit nproc=1024:2048 -it ubuntu /bin/bash

# 启动
$ docker container start shiyanlou
```

上述的两个命令如果我们使用 `docker container run` 只需要一步即可，即此时 `run` 命令同时完成了 `create` 及 `start` 操作：

```
$ docker container run --name shiyanlou --hostname shiyanlou --mac-address 00:01:02:03:04:05 --ulimit nproc=1024:2048 -it ubuntu /bin/bash
```

> 除此之外，上面的 `run` 命令还完成一些其它的操作，例如没有镜像时会 `pull` 镜像，使用 `-it` 参数时完成了 `attach` 操作（后面会学习该操作），使用 `--rm` 参数在容器退出后还会完成 `container rm` 操作。

> `run` 命令是一个综合性的命令，如果能够熟练的使用它可以简化很多步骤，但是其使用方式较为复杂

启动之后，暂停容器可以使用如下命令：

```bash
# 暂停一个或多个容器
docker container stop [OPTIONS] CONTAINER [CONTAINER...]

# 暂停一个或多个容器中的所有进程
docker container pause CONTAINER [CONTAINER...]
```

上述两个命令的区别在于一个是暂停容器中的进程，而另外一个是暂停容器，例如，我们使用 `stop` 停止刚刚启动的容器就可以使用如下命令：

```bash
$ docker container stop shiyanlou

# 查看容器的状态
$ docker container ls -a
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516695730293.png/wm)

如上图所示，容器被暂停后，此时处于 `Exited` 状态。

### 3.5 连接到容器

上述操作我们启动的容器运行于后台，所以，我们需要使用 `attach` 操作将本地标准输入输出流连接到一个运行中的容器，命令格式为：

```bash
docker container attach [OPTIONS] CONTAINER
```

如下示例，我们启动容器，并使用连接命令：

```bash
$ docker container start shiyanlou

$ docker container attach shiyanlou
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516695933022.png/wm)

连接到容器后，查看相应的主机名和 `Mac` 地址，可以判断我们连接到了刚刚创建的容器。

### 3.6 其它命令

除了上面介绍的一些命令之外，还有很多其它的命令，下面简单描述

#### 获取日志

获取容器的输出信息可以使用如下命令：

```bash
docker container logs [OPTIONS] CONTAINER
```

常用的配置项有：

+ `-t` 或 `--timestamps` 显示时间戳

+ `-f` 实时输出，类似于 `tail -f`

如下所示，我们查看刚刚创建的容器的日志，使用如下命令：

```bash
$ docker container logs -tf shiyanlou
```

#### 显示进程

除了获取日志之外，还可以显示运行中的容器的进程信息，例如查看刚刚创建的容器的进程信息：

```bash
$ docker container top shiyanlou
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516696183042.png/wm)

> 需要注意的是，该命令对于并未运行的容器是无效的

#### 查看修改

查看相对于镜像的文件系统来说，容器中做了哪些改变，可以使用如下命令：

```bash
docker container diff shiyanlou
```

例如我们在 `shiyanlou` 容器中创建一个文件，就可以使用 `diff` 命令查看到相应的修改：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516696518553.png/wm)

#### 重启

重启容器可以使用如下命令：

```bash
docker container restart shiyanlou
```

#### 执行命令

除了使用 `docker container run` 执行命令之外，我们还可以在一个运行中的容器中执行命令，使用如下格式：

```bash
docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

例如，我们在刚刚创建的容器中执行 `echo "test_exec"` 命令，就可以使用如下命令：

```bash
$ docker container exec shiyanlou echo "test_exec"
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516696614726.png/wm)

#### 删除容器

删除容器的命令：

```bash
docker container rm [OPTIONS] CONTAINER [CONTAINER...]
```

> 需要注意的是，在删除容器后，在容器中进行的操作并不会持久化到镜像中

Docker 容器命令操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week11/2-2.mp4
@`

## 4. 镜像

如果能够熟练的使用上述命令，那么这里关于镜像的部分操作，类比 `Management Commands` 的特性，可以很容易的学习相应的命令。

镜像存储中的核心概念仓库（Repository）是镜像存储的位置。Docker 注册服务器（Registry）是仓库存储的位置。每个仓库包含不同的镜像。

比如一个镜像名称 `ubuntu:14.04`，冒号前面的 `ubuntu` 是仓库名，后面的 `14.04` 是 TAG，不同的 TAG 可以对应相同的镜像，TAG 通常设置为镜像的版本号。

`Docker Hub` 是 `Docker` 官方提供的公共仓库，提供大量的常用镜像，由于国内网络原因经常连接 `Docker Hub` 会比较慢。

并且 `Docker` 的镜像是分层存储，每一个镜像都是由很多层组成的。而一些镜像会共享一些相同的层。对于实验环境中的 `docker` 来说，其使用的存储驱动是 `aufs`，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516930473464.png/wm)

> 图片中显示的是 aufs，但是对于如果是在自己的 Linux 环境下安装的 docker，其版本是高于 17.05，显示的有可能是 overlay2，其基本原理和 aufs 类似。

`aufs` 是一种联合文件系统(`UnionFS`)，理解其原理对于我们理解 Docker 镜像有很大的帮助，有兴趣的同学可以尝试学习 [Linux 文件系统之 aufs](https://segmentfault.com/a/1190000008489207)

### 4.1 查看镜像列表

我们查看镜像可以使用如下命令：

```bash
$ docker image ls
```

也可以查看指定仓库的镜像，例如。查看 `ubuntu` 仓库的镜像：

```bash
$ docker image ls ubuntu
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516697042412.png/wm)

### 4.2 查看镜像的详细信息

查看镜像的详细信息使用如下命令：

```bash
docker image inspect ubuntu
```

### 4.3 拉取镜像

上面的内容中描述了仓库和注册表的内容，这里，我们学习从注册表中获得镜像或者仓库的命令，使用如下命令：

```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

比较常用的配置参数为 `-a`，代表下载仓库中的所有镜像，即下载整个存储库。

如下所示，我们下载 `ubuntu:14.04` 镜像，使用如下命令：

```bash
$ docker image pull ubuntu:14.04
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516697216997.png/wm)

对于 `pull` 下来的镜像来说，其具体的保存路径为 `/var/lib/docker`。因为这里的存储驱动为 `aufs`，所以具体路径为 `/var/lib/docker/aufs`

### 4.4 构建镜像

#### commit

此时，对于我们 `pull` 的新镜像 `ubuntu:14.04` 来说，如果我们需要对其进行更新，可以创建一个容器，在容器中进行修改，然后将修改提交到一个新的镜像中。

提交修改使用如下命令：

```bash
docker container commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

该命令的解释为从一个容器的修改中创建一个新的镜像。例如，我们运行一个容器，然后在其中创建一个文件，最后使用  `commit` 命令：

```bash
# 使用 run 创建运行一个新命令
$ docker container run -it --name shiyanlou001 busybox /bin/sh

# 在运行的容器中创建两个文件，test1 和 test2
touch test1 test2

# 使用 ctrl + p  及  ctrl+q 键退出

# 使用提交命令，提交容器 shiyanlou001 的修改到镜像 busybox:test 中
$ docker container commit shiyanlou001 busybox:test

# 查看通过提交创建的镜像
$ docker image ls busybox
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516697682918.png/wm)

> 通过上述操作我们创建了一个新的镜像，但是本方法不推荐用在生产系统中，未来会很难维护镜像。最好的创建镜像的方法是 `Dockerfile`，修改镜像的方法是修改 `Dockerfile`，然后重新从 `Dockerfile` 中构建新的镜像。

#### BUILD

`docker` 可以从一个 `Dockerfile` 文件中自动读取指令构建一个新的镜像。 `Dockerfile` 是一个包含用户构建镜像命令的文本文件。在
创建该文件后，我们可以使用如下命令构建镜像：

```bash
docker image build [OPTIONS] PATH | URL
```

> 构建镜像时，该过程的第一件事是将 `Dockerfile` 文件所在目录下的所有内容递归的发送到守护进程。所以在大多数情况下，最好是创建一个新的目录，在其中保存 `Dockerfile`，并在其中添加构建 `Dockerfile` 所需的文件。

对于一个 `Dockerfile` 文件内容来说，基本语法格式如下所示：

```bash
# Comment
INSTRUCTION arguments
```

使用 `#` 号作为注释，指令（`INSTRUCTION`）不区分大小写，但是为了可读性，一般将其大写。而 `Dockerfile` 的指令一般包含下面几个部分：

1. 基础镜像：以哪个镜像为基础进行制作，使用 `FROM` 指令来指定基础镜像，一个 `Dockerfile` 必须以 `FROM` 指令启动。

2. 维护者信息：可以指定该 `Dockerfile` 编写人的姓名及邮箱，使用 `MAINTAINER` 指令。

3. 镜像操作命令：对基础镜像要进行的改造命令，比如安装新的软件，进行哪些特殊配置等，常见的是 `RUN` 命令。

4. 容器启动命令：基于该镜像的容器启动时需要执行哪些命令，常见的是 `CMD` 命令或 `ENTRYPOINT`

例如一个最基本的 `Dockerfile`：

```txt

# 指定基础镜像
FROM ubuntu:14.04

# 维护者信息
MAINTAINER shiyanlou/shiyanlou001@simplecloud.cn

# 镜像操作命令
RUN apt-get -yqq update && apt-get install -yqq apache2

# 容器启动命令
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

通过阅读上述内容中我们熟悉的一些 `linux` 指令，可以很容易的得出该命令创建了一个 `apache` 的镜像。包含了最基本的四项信息。

其中 `FROM` 指定基础镜像。`RUN` 命令默认使用 `/bin/sh`，并使用 `root` 权限执行。`CMD` 命令也是默认在 `/bin/sh` 中执行，但是只能有一条 `CMD` 指令，如果有多条则只有最后一条会被执行。

下面我们创建一个空目录，并在其中编辑 `Dockerfile` 文件，并基于此构建一个新的镜像，使用如下操作：

```bash
# 首先创建目录并切换目录
$ mkdir /home/shiyanlou/test1 && cd /home/shiyanlou/test1

# 编辑 Dockerfile 文件，默认文件名为 `Dockerfile`，也可以使用其它值，使用其它值需要在构建时通过 `-f` 参数指定，这里我们使用默认值。并在其中添加上述示例的内容
$ vim Dockerfile

# 使用 build 命令，`-t` 参数指定新的镜像
$ docker image build -t shiyanlou:1.0 .
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516700296627.png/wm)

在执行构建命令后，需要花费一些时间来完成构建。在运行结束后，最后查看新创建的镜像：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516700483506.png/wm)

在构建完成后，我们可以使用该镜像启动一个容器来运行 `apache` 服务，运行如下命令：

```
# 使用 -p 参数将本机的 8000 端口映射到容器中的 80 端口上。
$ docker container run -d -p 8000:80 --name shiyanlou002 shiyanlou:1.0
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516700663701.png/wm)

此时，容器启动成功后，并且配置了端口映射，我们就可以通过本机的 `8000` 端口访问容器 `shiyanlou002` 中的 `apache` 服务了。我们打开浏览器，输入 `localhost:8000`，显示结果如下图：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516700754471.png/wm)

> 更多有关于 Dockerfile 文件格式的信息可以参考官方文档 https://docs.docker.com/engine/reference/builder/

### 4.5 删除

我们删除 `ubuntu:latest` 镜像就可以使用如下命令：

```bash
# 删除镜像
$ docker image rm ubuntu
```

需要注意的是，如果该镜像正在被一个容器所使用，需要将容器删除才能成功的删除镜像。


Docker 镜像命令操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week11/2-3.mp4
@`

## 5. 总结

本节主要讲解了 docker 容器和镜像的操作。