---
show: step
version: 1.0
enable_checker: true
---
# Redis 简介与安装

## 1. 实验介绍

### 1.1 实验内容

本节实验开始将给大家讲解另一个数据库—— Redis 。此次内容主要是简单介绍什么是 Redis ，以及如何进行安装。

### 1.2 实验知识点

+ Redis 简介

+ 安装

### 1.3 推荐阅读

+ [Redis download](http://www.redis.cn/download.html)

+ [Download links](http://download.redis.io/)

## 2.  Redis 简介

### 2.1 Redis 是什么

`Redis(REmote DIctionary Server)` 是一个由 Salvatore Sanfilippo 编写的 `key-value` 存储系统。Redis 提供了一些丰富的数据结构，以及对这些数据结构的操作。由于 Redis 的数据都存储在内存中，因此访问速度非常快，所以 Redis 大量用于缓存系统中来存储热点数据，这样可以极大的提高网站的响应速度。

Redis 常被称作是一款数据结构服务器（data structure server）。Redis 的键值可以包括字符串（`strings`）类型，同时它还包括哈希（`hashes`）、列表（`lists`）、集合（`sets`）和有序集合（`sorted sets`）等数据类型。对于这些数据类型，可以执行原子操作，例如：对字符串进行附加操作（append）；递增哈希中的值；向列表中增加元素；计算集合的交集、并集与差集等。

### 2.2 Redis 的优点

+ 支持数据的持久化，通过配置可以将内存中的数据保存在磁盘中，Redis 重启以后再将数据加载到内存中；

+ 支持列表，哈希，有序集合等数据结构，极大的扩展了 Redis 的用途；

+ 原子操作，Redis 的所有操作都是原子性的，这使得基于 Redis 实现分布式锁非常简单；

+ 支持发布/订阅功能，数据过期功能；

## 2. 安装

### 2.1 编译安装

Redis  需要下载解压后进行编译。可以通过 Redis 的版本列表页选择符合实验环境的 Redis 版本:

>这里给大家介绍一下如何编译安装 `Redis 4.0.6` 最新版本，由于实验楼环境已经安装了可以跳过，所以你们可以在自己的本地尝试下。
```bash
$ wget http://download.redis.io/releases/redis-4.0.6.tar.gz
$ tar xzf redis-4.0.6.tar.gz
$ cd redis-4.0.6
$ make
```

二进制文件是编译完成后在 `src` 目录下，通过下面的命令启动 Redis 服务：

```
$ src/redis-server
```

你可以使用内置的客户端命令 `redis-cli` 进行使用：

```
$ src/redis-cli
```

以上就是如何进行编译安装 Redis 的具体方法。

### 2.2 实验楼环境下启动 Redis

因为在实验楼环境中已经安装了 Redis，所以每次启动实验后，只需要手动启动 Redis 即可。打开终端后，通过以下命令启动数据库：

```bash
$ sudo service redis-server start
```

注意：由于实验楼使用的是 Docker 容器的实验环境，所以不具备 ulimit 的权限，当启动 Redis 服务器的时候会报错，但不会影响服务器的正常启动，可以忽略这个报错信息。真实的服务器环境中不会有这个报错。

数据库启动成功以后，通过以下命令链接到数据库：

```bash
$ redis-cli
127.0.0.1:6379>
```

redis-cli 是 Redis 的客户端 Shell，执行该命令时可以指定连接的 Redis 服务地址等信息，未指定时将连接默认地址，Redis 服务默认监听在 `127.0.0.1:6379` 地址。后文出现的所有 Redis 操作命令都将基于 redis-cli 输入。

对于需要指定 `host` 等的操作，我们可以使用 `-h` 等参数，需要注意的是，redis 中没有用户的概念，只有一个密码认证的功能，所以，还可以使用如下命令：

```bash
shiyanlou:~/ $ redis-cli -h localhost -p 6379 -a ""
localhost:6379>
```

`-h` 指定 host，`-p` 指定端口， `-a` 指定密码。通过 `redis-cli` 客户端连接到服务器后，我们可以使用 `QUIT` 命令来断开连接。

```bash
localhost:6379> QUIT
shiyanlou:~/ $
```

Redis 登入及退出操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/14-1.mp4
@`

## 3. 总结

本节实验简单介绍了 Redis 和它的安装方法，在下一节将继续探讨 Redis 的其他内容。