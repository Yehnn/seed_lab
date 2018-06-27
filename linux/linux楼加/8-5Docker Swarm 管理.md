---
show: step
version: 1.0
enable_checker: true
---
# Docker Swarm 管理

## 1. 实验介绍

#### 1.1 实验内容

本节实验主要介绍 Docker 的 Swarm 相关管理。

#### 1.2 实验知识点

+ Docker Swarm 基本概念
+ Docker Swarm 常用命令

#### 1.3 推荐阅读

+ [Docker 集群：工作原理](http://www.subond.com/pages/2017/04/26/docker-swarm-gong-zuo-yuan-li.html)

## 2. 概述

#### 2.1 Swarm

Swarm 是 Docker 发布的管理集群的工具，一个集群由多个运行 `Docker` 的主机组成。下面我们简单介绍 Swarm 中的一些关键概念。

Swarm 在 Docker 1.12 之后被集成到 Docker Engine 中，又被称为 `swarm mode`，后面我们所说的 swarm 都是指 swarm mode。

#### 2.2 关键概念

##### Role

一个集群由多个运行 Docker 的主机组成，分别作为管理者（Manager）和工作者（Worker）两个角色。管理者管理集群中的成员，而工作者运行集群服务。给定的 Docker 主机可以是一个管理员，也可以是一个工作者，或者同时具备这两个角色。

##### Node

一个节点（Node）是参与到 Swarm 集群中的一个实例。一般表现为运行 Docker 的主机。

##### 服务与任务

一个服务是任务在管理节点或工作节点执行的定义，服务中运行的单个容器被称为任务。

使用集群模式运行服务时，一般有两种选项：

1. replicated services，复制服务，根据设定的值，swarm 调度在节点之间运行指定的副本任务。

2. global services，全局服务，集群在每个可用节点上运行一项任务。

一个服务的多个任务之间没有什么不同，但是对于一些特殊的服务而言，例如涉及到端口映射的服务，即便设定了多个任务，也只能启动一个。

##### 堆栈

堆栈（stack）是一组相互关联的服务，即一个堆栈能够定义和协调整个应用程序的功能（但是一些非常复杂的应用程序可能需要使用多个堆栈）。

对于在上一节我们学习的 Docker Compose 定义的应用程序来说，从技术角度来讲，就可以说我们一直在使用堆栈。但是 Docker Compose 是运行在单个主机上，而这里我们所说的堆栈可以运行在一个集群中，即是一个分布式的应用程序。

## 3. Docker Swarm

下面我们将会学习 Docker Swarm。

### 3.1 环境搭建

#### 创建一个 swarm

在 Docker 1.12 版本之后，Swarm 被集成到 Docker Engine 中（即 Swarm mode）。我们可以直接运行以下命令来创建一个集群：

```
$ docker swarm init --advertise-addr <IP>
```

对于单网卡，单 IP 来说，我们可以直接使用 `docker swarm init` 命令，但是实验环境中有两张网卡，用于访问外部网络的网卡为 `eth1`，所以需要指定 `<IP>` 为 `eth1` 上的地址，因为其它节点必须能够访问管理者的 IP 地址。其运行结果中包含将新节点加入到集群中的命令：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517280941602.png/wm)

这时，我们可以使用 `docker info` 命令查看 swarm 的状态：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517281019192.png/wm)

除此之外，还可以使用 `docker node ls` 命令查看有关节点的信息：

```
$ docker node ls
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517281062552.png/wm)

#### 向 swarm 中添加节点

在运行 `docker swarm init` 时会输出向 swarm 中添加节点的命令。除此之外，我们也可以使用如下两个命令分别获取向 swarm 中添加管理节点和工作节点的命令，获取到的命令需要在被添加的节点上运行：

```
# 管理节点
$ docker swarm join-token manager

# 工作节点
$ docker swarm join-token worker
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517281108402.png/wm)

由于实验环境中，我们只有一台服务器，如果自己本地搭建的有 docker 环境，可以尝试在自己的机器上运行上图中的命令，如下所示，为作者在本机 windows 安装的 docker ，将其加入到 swarm 中：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517281271834.png/wm)

这时，再次使用 `docker node ls` 命令，就可以看到新增加的 `worker` 节点了，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517562812869.png/wm)

#### 移除节点

如果需要在集群中移除一个节点，首先该节点的状态必须为 `down`，即该节点不可用，之后就可以正常的删除该节点了，在管理节点使用如下命令：

```
# NODE 为该节点的 ID
$ docker node rm NODE
```

#### 提权或撤销权限

对于一个 `worker` 节点来说，我们可以将其提升为一个 `manager` 节点，也可以将一个管理节点变为 `worker` 节点。在管理节点使用下面的命令即可：

```
# 提权
docker node promote NODE

# 撤权
docker node demote NODE
```

Docker Swarm 基本操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week11/5-1.mp4
@`

### 3.2 Docker Compose 与 Docker Swarm

对于 Docker Compose 来说，即使我们可以定义多容器的应用程序，但是多个容器依然只能在单个主机上工作。

这里我们以上一节定义的应用程序为例。进入 `app` 目录下，执行 `docker-compose up -d` 命令来启动服务。

`docker-compose up` 命令的运行结果如下图所示，图中标注的提示信息提示我们 Compose 并没有使用集群模式去部署服务，并将所有的容器调度到当前节点：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517561367768.png/wm)

此时运行成功后打开浏览器，输入 `127.0.0.1:8001`，依旧可以获取到正确的结果：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517368221961.png/wm)

但是如果需要使用集群模式部署服务，需要使用 `docker stack deploy` 命令，并且服务所需的镜像需要提前构建好。所以需要修改 `app/docker-compose.yml` 文件，修改后如下所示：

```
services:
  redis:
    image: redis:3.2
  web:
    image: app_web:latest
    depends_on:
    - redis
    ports:
    - 8001:80/tcp
    volumes:
    - /home/shiyanlou/app/web:/web:rw
version: '3.0'
```

> swarm 只支持 version: '3.0' 版本的 docker-compose.yml 文件

这时首先使用 `docker-compose down` 命令暂停之前使用 `docker-compose` 启动的容器，然后使用 `docker stack deploy` 命令部署该应用程序，使用的命令如下：

```
$ docker stack deploy -c docker-compose.yml app
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517369570872.png/wm)

上述命令会创建两个服务，不过与直接使用 `docker-compose` 不同的是，该命令部署的是一个堆栈（stack），即一组相互关联的服务，服务会被部署在集群中，而不是单个节点。并且此时我们创建的是服务，而不是单个容器，即便我们暂停或删除运行副本任务的容器，该容器也会再次启动。

> 如果你添加了自己运行 docker 的主机到 swarm 中，就有可能发现上述部署的服务的部分容器运行在 swarm 中的不同节点上，即分布式应用程序。

如果有添加自己运行 docker 的主机到集群中，就会发现上述两个服务，其中某一个服务运行在自己的节点上，如下所示，`app_redis` 服务运行在作者的 windows 节点上：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517561818075.png/wm)

而实验环境中运行的则是 `app_web` 服务，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517562145047.png/wm)

### 3.3 管理堆栈和服务

在创建一个集群之后，`docker stack` 和 `docker service` 两个命令集就变得可用了，他们分别用于管理堆栈和管理服务。

在上面部署了一个堆栈到集群中，我们可以使用下面的命令来查看所有的堆栈：

```
$ docker stack ls
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517563035483.png/wm)

如图所示，该 `app` 堆栈有两个服务，具体的服务可以使用下面的命令来查看：

```
$ docker stack services app
```

查看所有的服务则可以使用 `docker service ls` 命令，由于总共只有两个服务，所以下面两个命令的运行结果是一样的：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517563385194.png/wm)

除此之外，对于我们之前引入的任务的概念，我们也可以查看相应的任务，来确定该任务的状态以及具体运行于某个节点，如下所示：

```
# 查看 app 堆栈的任务
$ docker stack ps app

# 查看服务 app_redis 的任务
$ docker service ps app_redis
```

由于会显示启动失败的任务，并且集群中是否有添加自己的节点，所以显示结果可能并不一致，这里没有给出具体的命令截图。

并且由于此时实在集群中部署的服务，所以手动暂停或删除容器之后，还会自动启动新的容器，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517564302740.png/wm)

因此对于服务的管理，我们需要直接操作该堆栈或者操作服务，而不是容器。

例如，我们停止服务可以使用如下的命令：

```
# 移除一个堆栈，将会移除堆栈中的所有服务
$ docker stack rm app

# 移除一个或多个服务
$ docker service rm app_redis app_web
```

在移除了一个堆栈里的所有服务之后，该堆栈也被自动移除了，如下图所示，移除掉 `app` 堆栈里的所有服务后，该堆栈也被自动移除了：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517565257416.png/wm)


## 4. 总结

对于服务（service）或堆栈（stack）来说，docker 中也有一套专属的命令集，即 `docker service COMMAND` 和 `docker stack COMMAND`，以及集群和集群中节点的命令集 `docker swarm COMMAND` 和 `docker node COMMAND`。在本节内容中将不再单独介绍，希望大家在经过前面的学习过后，能够很容易的去学习理解这些命令的使用方法。