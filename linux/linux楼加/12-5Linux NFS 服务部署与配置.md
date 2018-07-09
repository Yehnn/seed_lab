---
show: step
version: 1.0
enable_checker: true
---
# Linux NFS 服务部署与配置

## 1 实验介绍

#### 1.1 实验内容

本节实验主要带着大家一起来搭建 NFS 服务器。

#### 1.2 实验知识点

+ NFS 是什么

+ 安装 NFS

+ 配置服务端

+ 配置客户端

#### 1.3 推荐阅读

+ [RPC and NFS](http://www.comptechdoc.org/independent/networking/guide/netrpcnfs.html)

+ [Accessing Remote File Systems Reference](https://docs.oracle.com/cd/E19455-01/806-0916/6ja8539fv/index.html)

+ [Linux NFS-HOWTO](http://nfs.sourceforge.net/nfs-howto/index.html)

+ [NFS archlinux](https://wiki.archlinux.org/index.php/NFS)

+ [Ubuntu 版：SettingUpNFS](https://help.ubuntu.com/community/SettingUpNFSHowTo)

## 2 NFS 是什么

#### 2.1 概述

**NFS**（`Network File System` ，网络文件系统）是由 `Sun Microsystems` 于 1984 年开发的一种分布式文件系统，允许客户端上的用户以类似于共享本地的方式通过网络方式共享文件。`NFS` 支持的功能比较多，所以不同的功能需要调用不同的程序来启动，因此，对应功能的端口就没有固定。在这种情况下，`NFS` 的启动就需要通过 `RPC` 服务来支持。

NFS 网络拓扑图

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513733124455.png/wm)
*（图片来源百度）*

#### 2.2 RPC

**RPC**(`Remote Procedure Call` ，远程过程调用)，采用的是一种 `client/server` 的模式，在计算机中是非常常见的模式。简单来说就是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术。

在 `NFS` 中 `RPC` 最主要的功能就是提供 `NFS` 相应功能的端口号，并返回给客户端，让客户端连上正确的端口，从而实现远程调用。

为了支持 `NFS` 的启动，RPC 会启动几个守护进程（`deamons`），其中主要的是 `mountd` （用于处理远程文件系统挂载请求和提供访问控制）和 `nfsd` （处理客户端文件系统的请求）进程。还有几个选用的进程（`lockd、statd`）。

#### 2.3 NFS 版本

NFS 协议从诞生到现在为止，主要的有 3 个版本：`NFSv2（rfc 1094）`，`NFSv3（rfc 1813）`，最新的版本是 `NFSv4` ，其中 NFSv4 包括了两个次版本 `NFSv4.0` 和 `NFSv4.1`。

在发展过程中最大的变化就是推动者从 Sun 变成了 [NetApp](http://www.netapp.com/us/index.aspx)，如今 Sun 已经被 Oracle 收购了。

版本对比：

**NFSv2**

`NFSv2` 是第一个以 RFC 形式发布的版本，实现了基本的功能。最初只是通过用户数据报协议（`UDP`）运行，无状态连接的明显优势就是发生故障时，客户端只用重试请求，直到服务器响应，而不用知道服务器是否崩溃或者网络是否暂时关闭。而保持状态的连接需要检测服务器故障并在服务器恢复时重建服务器状态等。

**NFSv3**

`NFSv3` 修正了 NFSv2 的一些 bug。取消了 NFSv2 的一些限制，如：读写操作中传输数据的最大长度，文件名称长度，文件长度，文件句柄长度从固定的 `32` 字节变为可变的，从只支持同步写变为了可异步写操作，增加了 `ACCESS` 请求，客户端可以检查是否有访问权限，以及对于一些请求调整了参数和返回信息。

**NFSv4**

`NFSv4` 发生的比较大的变化：

- 从 NFSv2 和 NFSv3 的无状态协议变成了有状态的增加了安全性；
- 将操作进行了整合，只提供了两个操作；
- 文件系统的命令空间发生了变化，服务器端必须设置一个根文件系统(`fsid=0`)，其他文件系统挂载在根文件系统上导出；
- 修改了文件属性的表示方法；
- NFSv4.1 版本中支持并行存储。

详细的版本对比说明，大家可以参看一下这个[文章](http://blog.csdn.net/ycnian/article/details/8515517)

## 3 安装 NFS

安装 NFS 服务器所需要的软件包：`nfs-utils` 和 `rpcbind`。

```bash
sudo yum install nfs-utils rpcbind
```

启动服务

```bash
sudo service rpcbind start
sudo service nfs start
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513680903779.png/wm)

## 4 配置服务端

首先，使用 ifconfig 查看下实验环境的网络地址：

```bash
ifconfig
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513680925310.png/wm)

我们根据 `ifconfig` 的输出决定后续的实验中的 IP 地址，请替换客户端的地址为你自己的环境地址：

```bash
# 这个主机地址根据大家的情况进行自定义
服务器端：127.0.0.1
客户端：10.29.113.73
```

### 4.1 创建共享目录

```bash
sudo mkdir /usr/share/test
```

### 4.2 配置文件

主要的配置文件是：`/etc/exports`，用于设定共享文件、访问 IP、以及访问参数等。

配置格式如下：

```
共享目录  主机名 1（IP1） 主机名 2（IP2） （参数...）
eg ：/test 192.168.0.1（rw,sync,...）
```

注：`/test` 表示共享文件目录，允许 `192.168.0.1` 的主机进行访问，参数的含义表示可读写、数据同步写入。

常用参数说明

| option           | describe                                                     |
| ---------------- | ------------------------------------------------------------ |
| `ro`             | 默认值，允许客户端只读访问共享文件系统                       |
| `rw`             | 默认值，运行客户端读写共享文件系统的访问权限                 |
| `sync`           | 同步，文件同步写入硬盘和内存                                 |
| `async`          | 文件暂存于内存，而不直接写入硬盘中                           |
| `root_squash`    | 默认情况下，root 用户在客户端上的任何文件请求都被视为由服务器上的匿名用户创建。 |
| `no_root_squash` | 客户端上的 root 将与服务器上的 root 用户对系统上的文件具有相同级别的访问权限。 |
| `all_squash`     | 将所有的 uid 和 gids 映射到匿名用户。用于 NFS 导出公共目录   |
| `no_all_squash`  | 默认值，和 `all_squash` 相反                                 |
| `anonuid`        | 匿名用户的 uid 值                                            |
| `anongid`        | 匿名用户的 gid 值                                            |

（参数还可以通过 `man exports` 查看参数帮助文档）

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513680968598.png/wm)

```bash
sudo vim /etc/exports
# 添加配置
/usr/share/test 10.29.113.73(rw,sync,no_root_squash,no_all_squash)
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513681021092.png/wm)

保存退出后，使配置生效

```bash
sudo exportfs -r
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513681041540.png/wm)

```shell
# 导出 NFS 文件系统的表
exportfs [参数]
```

参数说明

| options | describe                                                     |
| ------- | ------------------------------------------------------------ |
| `-r`    | 重新导出所有目录                                             |
| `-a`    | 导出所有目录                                                 |
| `-i`    | 忽略 `/etc/exports` 文件和 `/etc/exports.d` 目录下的文件。只使用命令行上给出的默认选项和选项。 |
| `-u`    | 取消导出一个或多个目录                                       |
| `-o`    | 指定导出选项列表                                             |

可以通过 `man exportfs` 命令来查看详细的说明。

### 4.3 重启服务

完成配置之后需要重启一下服务

```bash
sudo service rpcbind restart
sudo service nfs restart
```

至此，服务器端的配置就已经完成，还可以通过下面命令进行验证 NFS 的运行情况。

```bash
# 查看 NFS 开放的端口
sudo netstat -luntp| egrep 'rpc'
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513681090372.png/wm)

从打印出的信息可以看到 `rpcbind` 启动的端口号是 `111`，分别在 `UDP` 和 `TCP` 上。其他的 `rpc` 服务进程启动的端口号是随机的。

```bash
# 查看 RPC 状态
sudo rpcinfo -p
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513681110762.png/wm)

参数说明

| options | describe                                                     |
| ------- | ------------------------------------------------------------ |
| `-p`    | 显示所有注册的 RPC 程序的列表。                              |
| `-t`    | 使用 TCP 对指定主机上的程序进行 RPC 调用，报告是否收到响应   |
| `-u`    | 使用 UDP 对指定的主机上的程序进行 RPC 调用，并报告是否收到响应。 |

NFS 服务端配置操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week4/8-1.mp4
@`

## 5 配置客户端

下面就是对客户端进行配置

### 5.1 创建挂载目录

创建需要挂载的目录

```bash
sudo mkdir /mnt/test
```

### 5.2 挂载目录

测试挂载情况

```bash
showmount -e 127.0.0.1
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513681138172.png/wm)

```bash
sudo mount -t nfs 10.29.113.73:/usr/share/test /mnt/test

# 查看挂载目录
mount
df -h
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513681165491.png/wm)

### 6 测试

在客户端挂载的目录下生成一个文件：

```bash
cd /mnt/test
sudo touch file
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513681199539.png/wm)

服务器端进行检查

```bash
cd /usr/share/test
ll
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid3904timestamp1513681224448.png/wm)

即挂载成功。

NFS 客户端配置操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week4/8-2.mp4
@`

## 7 总结

通过本节实验学习了如何安装 NFS 的相关软件特点以及搭建一个 NFS 服务。