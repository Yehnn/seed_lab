# Dockerfile 和 Compose

## 1. 实验介绍

### 1.1 实验内容

本节实验接着之前的实验继续来探讨 Docker 中的 Dockerfile，以及如何在 Dockerfile 中创建镜像和对 Docker Compose 的学习，最后通过一个实战来进行巩固学习。

### 1.2 实验知识点

+ Dockerfile 基本语法
+ Dockerfile 创建镜像流程
+ Docker Compose 使用方法

## 2. Dockerfile

Dockerfile 是一个文本文件，其中包含我们为了构建 Docker 镜像而手动执行的所有命令。Docker 可以从 Dockerfile 中读取指令来自动构建镜像。我们可以使用 `docker build` 命令来创建一个自动构建。

### 2.1 上下文

在 Docker 容器及镜像管理一节中我们有提到构建镜像的一些知识。

构建镜像时，该过程的第一件事是将 `Dockerfile` 文件所在目录下的所有内容递归的发送到守护进程。所以在大多数情况下，最好是创建一个新的目录，在其中保存 `Dockerfile`，并在其中添加构建 `Dockerfile` 所需的文件。而 Dockerfile 文件所在的路径也被称为上下文（context）。

首先创建一个目录，以便开始后面的实验过程：

```
$ mkdir dir1 && cd dir1
```

下面我们简单介绍 Dockerfile 中常用的指令。

### 2.2 FROM

使用 FROM 指令指定一个基础镜像，后续指令将在此镜像的基础上运行：

```
FROM ubuntu:14.04
```

### 2.3 USER

在 Dockerfile 中可以指定一个用户，后续的 `RUN`，`CMD` 以及 `ENTRYPOINT` 指令都会使用该用户去执行，但是该用户必须提前存在。

```
USER shiyanlou
```

### 2.4 WORKDIR

除了指定用户之外，还可以使用 `WORKDIR` 指定工作目录，对于 `RUN`，`CMD`，`COPY`，`ADD` 指令将会在指定的工作目录中去执行。也可以理解为命令执行时的当前目录。

```
WORKDIR /
```

### 2.5 RUN，CMD，ENTRYPOINT

RUN 指令用于执行命令，该指令有两种形式：

+ `RUN <command>`，使用 shell 去执行指定的命令 `command`，一般默认的 `shell` 为 `/bin/sh -c`。

+ `RUN ["executable", "param1", "param2", ...]`，使用可执行的文件或程序 `executable`，给予相应的参数 `param`。

例如我们执行更新命令：

```
RUN apt-get update
```

CMD 的使用方式跟 RUN 类似，不过在一个 Dockerfile 文件中只能有一个 CMD 指令，如果有多个 CMD 指令，则只有最后一个会生效。该指令为我们运行容器时提供默认的命令，例如：

```
CMD echo "hello shiyanlou"
```

在构建镜像时使用了上面的 `CMD` 指令，则可以直接使用 `docker run image`，该命令等同于 `docker run image echo "hello shiyanlou"`。即作为默认执行容器时默认使用的命令，也可在 `docker run` 中指定需要运行的命令来覆盖默认的 `CMD` 指令。

除此之外，该指令还有一种特殊的用法，在 Dockerfile 中，如果使用了 ENTRYPOINT 指令，则 CMD 指令的值会作为 ENTRYPOINT 指令的参数：

```
CMD ["param1", "param2"]
```

ENTRYPOINT 指令会覆盖 CMD 指令作为容器运行时的默认指令，并且不会在 `docker run` 时被覆盖，如下示例：

```
FROM ubuntu:latest
ENTRYPOINT ["ls", "-a"]
CMD ["-l"]
```

上述构建的镜像，在我们使用 `docker run image` 时等同于 `docker run image ls -a -l` 命令。使用 `docker run image -i -s` 命令等同于 `docker run image ls -a -i -s` 指令。即 CMD 指令的值会被当作 ENTRYPOINT 指令的参数附加到 ENTRYPOINT 指令的后面。

### 2.6 COPY 和 ADD

COPY 和 ADD 都用于将文件，目录等复制到镜像中。使用方式如下：

```
ADD <src>... <dest>
ADD ["<SRC>",... "<dest>"]

COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
```

`<src>` 可以指定多个，但是其路径不能超出上下文的路径，即必须在跟 Dockerfile 同级或子目录中。

`<dest>` 不需要预先存在，不存在路径时会自动创建，如果没有使用绝对路径，则 `<dest>` 为相对于工作目录的相对路径。

COPY 和 ADD 的不同之处在于，ADD 可以添加远程路径的文件，并且 `<src>` 为可识别的压缩格式，如 gzip 或 tar 归档文件等，ADD 会自动将其解压缩为目录。

### 2.7 ENV

ENV 指令用于设置环境变量：

```
ENV <key> <value>
ENV <key>=<value> <key>=<value>...
```

### 2.7 VOLUME

VOLUME 指令将会创建指定的挂载目录，在容器运行时，将创建相应的匿名卷：

```
VOLUME /data1 /data2
```

上述指令将会在容器运行时，创建两个匿名卷，并挂载到容器中的 /data1 和 /data2 目录上。

### 2.8 EXPOSE

EXPOSE 指定在容器运行时监听指定的网络端口，它与 `docker run` 命令的 `-p` 参数不一样，并不实际映射端口，只是将该端口暴露出来，允许外部或其它的容器进行访问。

```
EXPOSE port
```

## 3. 从 Dockerfile 创建镜像

了解了上面一些常用于构建 Dockerfile 的指令之后，可以通过这些指令来构建一个镜像，如下所示，搭建一个 ssh 服务:

```
# 指定基础镜像
FROM ubuntu:14.04

# 安装软件
RUN apt-get update && apt-get install -y openssh-server && mkdir /var/run/sshd

# 添加用户 shiyanlou 及设定密码
RUN useradd -g root -G sudo shiyanlou && echo "shiyanlou:123456" | chpasswd shiyanlou

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
```

首先，我们在之前创建的一个空目录 `dir1` 中编辑 `Dockerfile` 文件，并将上面的内容复制到该文件中，相关的命令如下所示：

```
# 创建目录
$ mkdir dir1 && cd dir1

# 编辑 Dockerfile，将上面的内容写入
$ vim Dockerfile

# 最后执行构建命令
$ docker build -t sshd:test .
```

在上面的命令执行完成之后，该镜像就构建成功了，直接使用该镜像启动一个容器就可以运行一个 ssh 的服务，如下所示：

```
$ docker run -itd -p 10001:22 --rm sshd:test
```

这时就可以通过公网的 IP 地址，以及端口 10001，并且使用用户 `shiyanlou`，密码 `123456`，远程通过 `ssh` 连接到该容器中了。

Dockerfile 创建镜像操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week11/4-1.mp4
@`

## 4. Docker Compose

### 4.1 概述

#### Compose

Compose 是定义和运行多容器 `Docker` 应用程序的工具。使用 Compose，可以通过编辑 `YAML` 文件来配置应用程序的服务。它可以用来管理应用程序的生命周期，例如启动，停止以及重构服务。

#### service

在分布式应用程序中，应用程序的不同部分被称为服务（service），例如常见的提供数据库存储的服务。服务实际上只是生产中的镜像。一个服务仅仅运行一个镜像，但它定义了服务运行的方式，例如使用哪个端口，该容器应该运行多少个副本等。

#### 使用过程

使用 Compose 的三个过程如下：

1. 定义应用程序的环境，即 Dockerfile

2. 定义组成应用程序的服务，一般为定义 `docker-compose.yml` 文件

3. 启动整个应用程序

> 关于 docker-compose.yml 文件的详细编写格式可以参考：https://docs.docker.com/compose/compose-file/#reference-and-guidelines

目前有三种版本的 Compose 文件格式：

1. version 1: 最早的版本使用传统格式，将在未来的 Compose 版本中被弃用

2. version 2: 现在使用最多的文件格式

3. version 3: 最新的版本，旨在 Compose 和被集成到 Docker Engine 中的 swarm mode 之间互相兼容（swarm 在下一节的内容会学习相关的知识）。

### 4.2 安装

在 linux 中，Compose 需要单独安装，我们需要从 github 上下载 Docker Compose 二进制文件。但是官网提供的从 github 上下载的链接速度十分缓慢，在实验环境中，我们已提供该文件，直接使用以下命令进行下载：

```
$ wget http://labfile.oss-cn-hangzhou.aliyuncs.com/courses/980/software/docker-compose-Linux-x86_64
```

下载成功后，为了能够直接使用该可执行文件执行命令，一般将其放入 `$PATH` 的环境变量支持的路径中，并添加可执行权限，使用的命令如下：

```
$ sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
```

执行完成后，就能够在终端下直接使用 `docker-compose` 命令了：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517365151690.png/wm)

### 4.3 实例

在这里，我们将创建一个 web 应用程序，该实例参考 Docker Compose 官方文档，有两个服务，并做了一些改变。

在本实验中，我们需要两个容器：

1. web 容器：提供 web 服务，并连接后端的 redis 服务

2. redis 容器：提供 redis 服务，接收 web 容器的连接

其文件目录结构如下所示：

```
app
|----web
|     |----web.py
|     |----requirements.txt
|     |----Dockerfile
|
|----docker-compose.yml
```

1. 首先编辑 `app/web/web.py` 文件，写入下面的内容：

```py
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    redis.incr('number')
    return 'Hello Shiyanlou! # %s' % redis.get('number')

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80, debug=True)
```

上述代码创建一个十分简单的 web 应用程序。该程序会连接 `redis` 服务，在访问 `/` 页面时，会自动将变量 `number` 加一，该 `INCR` 命令也在 `redis 数据类型` 一节中学习过。

2. 编辑 `app/web/requirements.txt` 文件，输入如下内容：

```txt
flask==0.10
redis==2.10.3
```

创建 `requirements.txt` 文件，输入需要使用的 python 依赖包，方便安装。

3. 编辑 `app/web/Dockerfile` 文件，添加如下内容：

```
FROM python:2.7
COPY ./ /web/
WORKDIR /web
RUN pip install -r requirements.txt
CMD python web.py
```

上述 `Dockerfile` 定义了一个镜像，该镜像基于 `python:2.7` 镜像制作，在其基础上安装相应的 python 包，并执行 `CMD` 命令来启动该应用程序。

4. 编辑 `app/docker-compose.yml` 文件：

```txt
services:
  redis:
    image: redis:3.2
  web:
    build:
      context: /home/shiyanlou/app/web
    depends_on:
    - redis
    ports:
    - 8001:80/tcp
    volumes:
    - /home/shiyanlou/app/web:/web:rw
version: '3.0'
```

该 `docker-compose.yml` 文件定义了两个服务，分别为 `web` 和 `redis` 服务。并且我们配置 `web` 服务的端口映射，以及挂载相应的目录。 `depends_on` 定义了依赖关系，被依赖服务的容器需要先创建。

5. 进入 `app` 目录下，执行 `docker-compose up` 命令来启动服务：

```
$ cd app
$ docker-compose up
```

由于此时 `web` 服务的镜像还未构建，所以此时会自动根据 `build`指示，使用 `/home/shiyanlou/app/web/Dockerfile` 文件构建镜像。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517557297094.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517557398547.png/wm)

运行成功后，此时我们可以打开浏览器，输入 `127.0.0.1:8001`，获取到正确的结果：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517368221961.png/wm)

6. 除此之外，也可以使用 `-d` 参数，即 `docker-compose up -d` 在其在后台运行：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517557688428.png/wm)

7. 如果需要暂停以及删除容器，可以直接运行 `docker-compose down` 命令即可

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517558444487.png/wm)


## 5. 总结

本节实验主要介绍了 docker 中的 Dockerfile 和 Docker Compose 的相关内容以及如何用 Dockerfile 创建镜像。