---
show: step
version: 1.0
enable_checker: true
---
# Docker 存储与网络

## 1. 实验介绍

#### 1.1 实验内容

在本节内容中，我们将讨论 `Docker` 中管理数据的几种方式。

#### 1.2 实验知识点

+ Docker 存储概念
+ Docker volumes/bind mounts/tmpfs
+ Docker 网络管理

## 2. 存储

下面我们将会学习存储。

### 2.1 概述

通过之前的学习，我们学习了有关于容器和镜像的一些知识。对于数据来说，我们可以将其保存在容器中，但是会存在一些缺点：

+ 当容器不再运行时，我们无法使用数据，并且容器被删除时，数据并不会被保存。

+ 数据保存在容器中的可写层中，我们无法轻松的将数据移动到其他地方。

针对上述的缺点而言，有些数据信息，例如我们的数据库文件，我们不应该将其保存在镜像或者容器的可写层中。Docker 提供三种不同的方式将数据从 Docker 主机挂载到容器中，分别为卷（`volumes`），绑定挂载（`bind mounts`），临时文件系统（`tmpfs`）。很多时候，`volumes` 总是正确的选择。

+ `volumes`， 卷存储在 Docker 管理的主机文件系统的一部分中（`/var/lib/docker/volumes/`） 中。完全由 `Docker` 管理

+ `bind mounts`， 绑定挂载，可以将主机上的文件或目录挂载到容器中

+ `tmpfs`， 仅存储在主机系统的内存中，而不会写入主机的文件系统

无论使用上述的哪一种方式，数据在容器内看上去都是一样的。它被认为容器文件系统中的目录或单个文件。

### 2.2 卷列表

对于三种不同的存储数据的方式来说，卷是唯一完全由 `Docker` 管理的。它更容易备份或迁移，并且我们可以使用 `Docker CLI` 命令来管理卷。

列出本地可用的卷列表可以使用如下命令：

```
$ docker volume ls
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516701361387.png/wm)

由于此时我们并未创建有相应的卷，所以显示为空。

#### 2.2.1 创建卷

创建卷我们可以直接使用如下命令：

```
$ docker volume create
```

上述命令会创建一个数据卷，并且会随机生成一个名称。创建之后我们可以查看卷列表：

```
$ docker volume ls
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516701446377.png/wm)

这种由系统随机生成名称的创建卷的方式被称为**匿名卷**，直接使用该卷需要指定卷名，即自动生成的 `ID`，所以创建卷时一般手动指定其 `name`，例如我们创建一个名为 `volume1` 的卷。

```
$ docker volume create volume1
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516701557988.png/wm)

#### 2.2.2 用卷启动一个容器

创建卷之后，我们可以用卷来启动一个容器，这里首先需要学习 `docker container run` 命令的两个参数：

+ `-v` 或 `--volume`

    - 由三个由冒号（:）分隔的字段组成，`[HOST-DIR:]CONTAINER-DIR[:OPTIONS]`。

    - `HOST-DIR` 代表主机上的目录或数据卷的名字。省略该部分时，会自动创建一个匿名卷。如果是指定主机上的目录，需要使用绝对路径。

    - `CONTAINER-DIR` 代表将要挂载到容器中的目录或文件，即表现为容器中的某个目录或文件

    - `OPTIONS` 代表配置，例如设置为只读权限(`ro`)，此卷仅能被该容器使用（`Z`），或者可以被多个容器使用 （`z`）。多个配置项由逗号分隔。

    - 例如，我们使用 `-v volume1:/volume1:ro,z`。代表的是意思是将卷 `volume1` 挂载到容器中的 `/volume1` 目录。`ro,z` 代表该卷被设置为只读（`ro`），并且可以多个容器使用该卷（`z`）

+ `--mount`

    - 由多个键值对组成，键值对之间由逗号分隔。例如： `type=volume,source=volume1,destination=/volume1,ro=true`。

    - `type`，指定类型，可以指定为 `bind`，`volume`，`tmpfs`。

    - `source`，当类型为 `volume` 时，指定卷名称，匿名卷时省略该字段。当类型为 `bind`，指定路径。可以使用缩写 `src`。

    - `destination`，挂载到容器中的路径。可以使用缩写 `dst` 或 `target`。

    - `ro` 为配置项，多个配置项直接由逗号分隔一般使用 `true` 或 `false`。

针对上述创建的卷 `volume1`，用其来运行一个容器就可以使用如下命令：

```
$ docker container run -it --name shiyanlou003 -v volume1:/volume1 --rm ubuntu bash
```

或者我们也可以使用 `--mount`，其语法格式如下：

```
$ docker run -it --name shiyanlou004 --mount type=volume,src=volume1,target=/volume1 --rm ubuntu bash
```

> 从命令中，可以很明显的得出，`--mount` 的可读性更好。所以，推荐大家使用 `--mount`

上述操作，我们分别运行了两个容器，并分别挂载了一个卷，还可多次使用该参数挂载多个卷或目录。并且对于这两个容器来说，由于我们使用的是同一个卷，所以他们将共享该数据卷，但是对于多个容器共享数据卷时，需要注意并发性。大家可以分别连接到两个容器中，操作数据，验证其是同步的，这里就不再详细演示了。

### 2.3 bind-mounts

对于数据卷来说，其优点在于方便管理。而对于绑定挂载（`bind-mounts`）来说，通过将主机上的目录绑定到容器中，容器就可以操作和修改主机上该目录的内容。这既是其优点也是其缺点。

例如，我们将 `/home/shiyanlou` 目录挂载到容器中的 `/home/shiyanlou` 目录下，使用的命令如下：

```bash
$ docker run -it -v /home/shiyanlou:/home/shiyanlou --name shiyanlou005 --rm ubuntu bash
```

而如果使用的是 `--mount`，相应的语句如下：

```bash
$ docker run -it --mount type=bind,src=/home/shiyanlou,target=/home/shiyanlou --name shiyanlou006 --rm ubuntu bash
```

> 如果绑定挂载时指定的容器目录是非空的，则该目录中的内容将会被覆盖。并且如果主机上的目录不存在，会自动创建该目录。

上述两个操作针对的是目录，而对于挂载文件来说，可能会出现一些特殊情况，涉及到绑定挂载和使用卷的区别。下面我们重现这一操作：

1. 首先在当前目录，即 `/home/shiyanlou` 目录下，创建一个 `test.txt` 文件。并向其中写入文本内容 "test1"：

```
$ echo "test1" > test.txt
```

2. 接着创建一个容器 `shiyanlou007`，将 `test.txt` 文件挂载到容器中的 `/test.txt` 文件，并查看容器中 `/test.txt` 文件的内容：

```bash
$ docker run -it -v /home/shiyanlou/test.txt:/test.txt --name shiyanlou007 ubuntu /bin/bash
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516934638946.png/wm)

3. 这时新打开一个终端，通过 `echo` 命令向 `/home/shiyanlou/test.txt` 文件追加内容 "test2"，并在容器中查看 `/test.txt` 文件的内容:

```
$ echo "test2" >> test.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516934866508.png/wm)

4. 这时无论是在容器中还是主机上都能查看到该文件的内容。接下来在主机上查看 `test.txt` 的 `inode` 号，并使用 `vim` 编辑该文件，添加 "test3"，并查看该文件的内容：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516935268507.png/wm)

如上图所示，在主机上使用 `vim` 编辑后，通过 `vim` 做出的修改不能在容器中查看到。这是因为 `vim` 编辑保存文件的时候，会将文件内容写入到一个新的文件中，保存好后，删除掉原来的文件，并将新文件重命名，从而完成保存的操作。但是我们标识文件是通过 `inode`，这在第一周的内容中有讲解到，因此 Docker 绑定的主机文件，依旧是 `vim` 编辑之前的 `inode`，即旧文件。所以容器中看到的，依然是旧的内容。

对于数据卷来说，由 `docker` 完全管理，而绑定挂载，则需要我们自己去维护。我们需要自己手动去处理这些问题，这些问题并不仅仅是上面演示的内容，还可能有用户权限，`SELINUX` 等问题。

### 2.3 使用 tmpfs 挂载数据

`tmpfs` 只存储在主机的内存中。当容器停止时，相应的数据就会被移除。

```
$ docker run -it --mount type=tmpfs,target=/test --name shiyanlou008 --rm ubuntu bash
```

Docker 存储操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week11/3-1.mp4
@`

## 3. 网络

> 在开始下面的内容之前，为了不出现命名上的冲突，也为了显示更为直观并且方便演示示例，首先需要将前面创建或启动的容器全部删除。可以使用下面两条命令达到这一效果：

```
# 暂停所有运行中的容器
$ docker container ls -q | xargs docker container stop

# 删除所有的容器
$ docker container ls -aq | xargs docker container rm
```

在我们安装 `Docker` 后，会自动创建三个网络。我们可以使用下面的命令来查看这些网络：

```bash
$ docker network ls
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516712155589.png/wm)

如上图所示，三种默认的网络，分别为 `bridge`，`host`，`none`。

### 3.1 bridge

`bridge`，即桥接网络，在安装 `docker` 后会创建一个桥接网络，该桥接网络的名称为 `docker0`。我们可以通过下面两条命令去查看该值。

```bash
# 查看 bridge 网络的详细信息，并通过 grep 获取名称项
$ docker network inspect bridge | grep name

# 使用 ifconfig 查看 docker0 网络
ifconfig
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516775691321.png/wm)

在上图中，我们可以查看到对应的值。默认情况下，我们创建一个新的容器都会自动连接到 `bridge` 网络。其详细信息如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516778041875.png/wm)

我们可以尝试创建一个容器，该容器会自动连接到 `bridge` 网络，例如我们创建一个名为 `shiyanlou001` 的容器：

```
$ docker container run -itd --name shiyanlou001 ubuntu /bin/bash

上述命令中默认使用 --network bridge ，即指定 bridge 网络，与下面的命令等同
$ docker container run -itd --name shiyanlou001 --network bridge ubuntu /bin/bash
```

创建后，再次查看 `bridge` 的信息：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516778620559.png/wm)

这时可以查看到相应的容器的网络信息，该容器在连接到 `bridge` 网络后，会从子网的地址池中获得一个 IP 地址，即上图中的 `192.168.0.2`。

使用 `docker container attach shiyanlou001` 命令，也可查看相应的地址信息：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516778879129.png/wm)

并且对于连接到默认的 `bridge` 之间的容器可以通过 IP 地址互相通信。例如我们启动一个 `shiyanlou002` 的容器，它可以与 `shiyanlou001` 通过 IP 地址进行通信。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516877067391.png/wm)

> 其具体的实现原理可以参考链接 [Linux 上的基础网络设备](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516877067391.png/wm)，以及涉及到[网桥的工作原理](https://segmentfault.com/a/1190000009491002)

上述的操作我们通过 `ping` 命令演示了 `IP` 相关的内容。但是对于应用程序来讲，如果需要在外部进行访问，我们还会涉及到端口的使用，而 `Docker` 对于 `bridge` 网络使用端口的方式为设置端口映射，通过 `iptables` 实现。

下面我们通过 `iptables` 来为大家演示 docker 实现端口映射的方式，主要针对 nat 表和 filter 表：

1. 首先删除掉上面创建的两个容器。这里不再给出具体的命令

2. 这时，我们查看 `nat` 表的转发规则，使用如下命令：

```bash
$ sudo iptables -t nat -nvL
```

3. 由于此时并未创建 docker 容器，nat 表中没有什么特殊的规则。接下来，我们使用上一节构建的 `shiyanlou:1.0` 镜像创建一个容器 `shiyanlou001`，并将本机的端口 `10001` 映射到容器中的 `80` 端口上，在浏览器中可以通过 `localhost:10001` 访问容器 `shiyanlou001` 的 `apache` 服务，命令如下：

```
$ docker run -d -p 10001:80 --name shiyanlou001 shiyanlou:1.0
```

> `docker run` 命令的 `-p` 参数是通过端口映射的方式，将容器的端口发布到主机的端口上。其使用格式为 `-p ip:hostPort:containerPort`。并且还可以指定范围，例如 `-p 10001-10100:1-100`，代表将容器 `1-100` 的端口映射到主机上的 `10001-10100`端口上，两者一一对应。

4. 创建成功后，我们可以在浏览器中输入 `localhost:10001` 访问到容器`shiyanlou001` 的 `apache` 服务，并查看此时 `iptables` 中 `nat` 表和 `filter` 表的规则，其中分别新增了一条比较重要的内容，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517189337316.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517189420438.png/wm)

5. 接下来，再次使用镜像 `shiyanlou:1.0` 来启动一个容器 `shiyanlou002`，这次我们不指定端口映射，通过手动修改 `nat` 表的方式来模拟实现：

```bash
$ docker run -d --name shiyanlou002 shiyanlou:1.0
```

6. 获取容器 `shiyanlou002` 的 ip 地址，如果按步骤操作此 ip 为 `192.168.0.3`。此时我们想通过主机的 `10002` 端口访问容器 `shiyanlou002` 的 `80` 端口，就可以添加一条规则：

```
# 添加一条规则，大致解释为将从非 docker0 接口上，目的端口为 10002 的 tcp 报文，修改其目的地址为 192.168.0.3:80

$ sudo iptables -t nat -A DOCKER ! -i docker0 -p tcp --dport 10002 -j DNAT --to-destination 192.168.0.3:80
```

7. 添加成功后我们在本地发出的 `localhost:10002` 请求会被定位到 `192.168.0.3:80` 上，但是在将请求转发到 `docker0` 网桥上时，对于默认的 `filter` 表中的 `FOEWARD` 链的规则是 `DROP`，因此我们还需要在 `filter` 表中设置相应的规则：

```bash
$ sudo iptables -t filter -A FORWARD ! -i docker0 -o docker0 -p tcp -d 192.168.0.3 -j ACCEPT --dport 80

或者你也可以选择将其加到由 docker 定义的 DOCKER 链中，上面的命令和下面的命令选择其中的一个即可

$ sudo iptables -t filter -A DOCKER ! -i docker0 -o docker0 -p tcp -d 192.168.0.3 -j ACCEPT --dport 80
```

9. 此时我们就能够通过 `192.168.0.3:80` 访问容器 `shiyanlou002` 中的 `apache` 服务了。 即通过 `iptables` 的方式实现了容器 `shiyanlou002` 上 `80` 端口到主机 `10002` 端口的映射。

10. 最后，为了不影响后面实验的进行，这里我们删除掉手动添加的规则，并删除容器。

### 3.2 自定义网络

对于默认的 `bridge` 网络来说，使用端口可以通过端口映射的方式来实现，并且在上面的内容中我们也演示了容器之间通过 `IP` 地址互相进行通信。但是对于默认的 `bridge` 网络来说，每次重启容器，容器的 `IP` 地址都是会发生变化的，因为对于默认的 `bridge` 网络来说，并不能在启动容器的时候指定 ip 地址，在启动单个容器时并不容易看到这一区别。

#### 旧版的容器互联

容器间都是通过在 `/etc/hosts` 文件中添加相应的解析，通过容器名，别名，服务名等来识别需要通信的容器。

这里，我们启动两个容器，来演示旧的容器互联：

1. 首先启动一个名为 `shiyanlou001` 的容器，使用镜像 `busybox`：

```
$ docker run -it --rm --name shiyanlou001 busybox /bin/sh
```

2. 这时打开一个新的终端，启动一个名为 `shiyanlou002` 的容器，并使用 `--link` 参数与容器 `shiyanlou001` 互联。

```
$ docker run -it --rm --name shiyanlou002 --link shiyanlou001 busybox /bin/sh
```

> docker run 命令的 `--link` 参数的格式为 `--link <name or id>:alias`。格式中的 `name` 为容器名，`alias` 为别名。即可以通过 `alias` 访问到该容器。

如下图所示，左侧为 `shiyanlou001`，右侧为 `shiyanlou002`：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517193383367.png/wm)

3. 如果此时 `shiyanlou001` 容器退出，这时我们启动一个 `shiyanlou003`，再次启动一个 `shiyanlou001`：

```
$ docker run -itd --name shiyanlou003 --rm busybox /bin/sh

$ docker run -it --name shiyanlou001 --rm busybox /bin/sh
```

按照顺序分配的原则，此时 `shiyanlou003` 的 IP 地址为 `192.168.0.2`，容器 `shiyanlou001` 的 IP 地址为 `192.168.0.4`。并且此时容器 `shiyanlou002` 中 `/etc/hosts` 文件的解析依旧不变，所以不能获取到正确的解析：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517194194401.png/wm)

如上所示，旧的容器 `shiyanlou002` 通过 `--link` 连接到 `shiyanlou001`。而在 `shiyanlou001` 重启后，由于 IP 地址的变化，此时 `shiyanlou002` 并不能正确的访问到 `shiyanlou001`。

除了使用 `--link` 链接的方式来达到容器间互联的效果，在 `docker` 中，容器间的通信更应该使用的是自定义网络。

#### 自定义网络

docker 在安装时会默认创建一个桥接网络，除了使用默认网络之外，我们还可以创建自己的 `bridge` 或 `overlay` 网络。

如下所示，我们创建一个名为 `network1` 的桥接网络，简单命令如下：

```
$ docker network create network1

$ docker network ls
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517195281317.png/wm)

创建成功后，可以使用 `ifconfig` 或者 `ip addr show` 命令查看该桥接网络的网络接口信息，如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517195572337.png/wm)

而对于该网络的详细信息可以通过 `docker network inspect network1` 命令来查看，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517195691681.png/wm)

其相应的网络接口名称和子网都是由 docker 随机生成，当然，我们也可以手动指定：

```
# 首先删除掉刚刚创建的 network1 
$ docker network rm network1

# 再次创建 network1，指定子网
$ docker network create -d bridge --subnet=192.168.16.0/24 --gateway=192.168.16.1 network1
```

此时，我们可以运行一个容器 `shiyanlou001`，指定其网络为 `network1`，使用 `--network network1`：

```
$ docker run -it --name shiyanlou001 --network network1 --rm busybox /bin/sh
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517196736819.png/wm)

使用 `exit` 退出该容器使其自动删除，这时我们再次创建该容器，但是不指定其 `--network`：

```
$ docker run -it --name shiyanlou001 --rm busybox /bin/sh
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517196983288.png/wm)

此时，该容器连接到默认的 `bridge` 网络，这时，可以新打开一个终端，在其中运行如下命令，将 `shiyanlou001` 连接到 `network1` 网络中：

```
# 在新打开的终端中运行，将容器 shiyanlou001 连接到 network1 网络中
$ docker network connect network1 shiyanlou001

# 这时再次在容器 `shiyanlou001` 中使用 `ifconfig` 命令
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517197247336.png/wm)

如上图中所示，出现了一个 `eth1` 接口，此时，`eth0` 连接到默认的 `bridge` 网络，`eth1` 连接到 `network1` 网络。

对于自定义的网络来说，docker 嵌入的 `DNS` 服务支持连接到该网络的容器名的解析。这意味着连接到同一个网络的容器都可以通过容器名去 `ping` 另一个容器。

如下所示，启动两个容器，连接到 `network1`：

```
$ docker run -itd --name shiyanlou_1 --network network1 --rm busybox /bin/sh

$ docker run -it --name shiyanlou_2 --network network1 --rm busybox /bin/sh
```

启动之后，由于上述的两个容器都是连接到 `network1` 网络，所以可以通过容器名 `ping` 通：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517198037471.png/wm)

除此之外，在用户自定义的网络中，是可以通过 `--ip` 指定 IP 地址的，而在默认的 `bridge` 网络不能指定 IP 地址：

```
# 连接到 network1 网络，运行成功
$ docker run -it --network network1 --ip 192.168.16.100 --rm busybox /bin/sh

# 连接到默认的 bridge 网络，下面的命令运行失败
$ docker run -it --rm busybox --ip 192.168.0.100 --rm busybox /bin/sh
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517198404575.png/wm)

### 3.3 host 和 none

`host` 网络，容器可以直接访问主机上的网络。

例如，我们启动一个容器，指定网络为 `host`：

```
$ docker run -it --network host --rm busybox /bin/sh
```

如下所示，该容器可以直接访问主机上的网络：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517214736450.png/wm)

`none` 网络，容器中不提供其它网络接口。

```
$ docker run -it --nerwork none --rm busybox /bin/sh
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517214875513.png/wm)

## 4. 总结

本节实验主要讲解了关于 docker 的存储和网络问题。