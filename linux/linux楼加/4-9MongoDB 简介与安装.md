---
show: step
version: 1.0
enable_checker: true
---
# MongoDB 简介与安装

## 1. 实验介绍

#### 1.1 实验内容

`MongoDB` 是一个高性能，高伸缩，易部署，易使用，存储方便的 `NoSQL` 数据库，它非常适合实时地插入，更新与查询数据。使用于对数据库性能要求较高，灵活性要求更强，数据模型比较简单的场景。本实验将带领大家学习 `MongoDB`。

> `NoSQL` 是对不同于传统的关系数据库的数据库管理系统的统称。和传统的关系型数据库相比，它不使用 `SQL` 作为查询语言，并且数据存储不是以表格形式，而是以一种键值对的方式。

#### 1.2 实验知识点

+ `MongoDB` 的简介
+ `MongoDB` 的安装与启动
+ `MongoDB` 的概念理解

## 2. 简介

`MongoDB` 是非常流行的 `NoSQL` 数据库，支持自动化的水平扩展，同时也被称为文档数据库，因为数据按文档的形式进行存储（`BSON` 对象，类似于 `JSON`）。在 `MongoDB` 中数据存储的组织方式组要分为四级：

+ `database`，数据库实例，比如一个 `app` 使用一个数据库； 

+ `collection`，文档集合 ，一个数据库包含多个文档集合，类似于 `MySQL` 中的表；

+ `document`，文档，一个文档代表一项数据，类似于 `JSON` 对象，对应于 `MySQL` 表中的一条记录；

+ `field`，字段，一个文档包含多个字段；

`MongoDB` 存储的数据可以是无模式的，比如在一个集合中的所有文档不需要有一致的结构。也就是说往同一个表中插入不同的数据时，这些数据之间不必有同样的字段。这和 `MySQL` 彻底不同，在 `MySQL` 中创建表时就已经确定了数据项的字段，往其中插入数据时，必须是相同的结构。

## 3. 安装启动

在实验楼的实验环境中，已经安装有 `MongoDB`，如果需要在自己的机器上安装，可以参考下面的安装步骤。

### 3.1 安装

在 ubuntu 里直接使用 `apt` 安装
> 由于实验环境中已经安装了 mongo . 此步仅作为本地环境安装步骤，实验环境无需安装,
> 安装会因 版本问题 产生冲突

```bash

$ sudo apt-get install -y mongodb
```

### 3.2 启动服务

安装成功后，我们还需启动 `MongoDB` 服务：

```bash
$ sudo service mongod start
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep mongo
  error: 没有启动 mongodb
```

### 3.3 常用命令行参数

服务启动后，我们可以通过以下命令链接到数据库：

```bash
$ mongo
```

该命令会启动 `MongoDB` 的客户端 `Shell`，执行该命令时可以指定连接的 `MongoDB` 地址等信息，未指定时将连接默认地址，默认地址为 `127.0.0.1:27107`。

除此之外，`mongo` 还有一些常用的命令行参数：

+ `--version` 显示版本号

+ `--port <port>` 指定端口。默认情况为 `27107`

+ `--host <hostname>` 指定主机。默认为 `localhost`

+ `--username <username>` 或者使用 `-u <username>` 。指定一个用户名，用来验证需要进行验证的数据库，需要结合 `--password` 和 `--authticationDatabase` 选项

+ `--password <password>` 指定密码。

后文出现的所有 `MongoDB` 操作命令都将基于 `mongo shell` 输入。

例如，我们输入 `help` 可以查看一些帮助信息

```bash
> help
    db.help()                    help on db methods
    db.mycoll.help()             help on collection methods
    sh.help()                    sharding helpers
    rs.help()                    replica set helpers
    help admin                   administrative help
    help connect                 connecting to a db help
    help keys                    key shortcuts
    help misc                    misc things to know
    help mr                      mapreduce

    show dbs                     show database names
    show collections             show collections in current database
    show users                   show users in current database
    show profile                 show most recent system.profile entries with time >= 1ms
    show logs                    show the accessible logger names
    show log [name]              prints out the last segment of log in memory, 'global' is default
    use <db_name>                set current database
    db.foo.find()                list objects in collection foo
    db.foo.find( { a : 1 } )     list objects in foo where a == 1
    it                           result of the last line evaluated; use to further iterate
    DBQuery.shellBatchSize = x   set default number of items to display on shell
    exit                         quit the mongo shell
>
```

> 使用 `exit` 或 `ctrl + C` 可以退出 mongo shell 。

## 4. 概念理解

在 `MongoDB` 中，一个数据库包含多个集合，类似于 `MySQL` 中一个数据库包含多个表；一个集合包含多个文档，类似于 `MySQL` 中一个表包含多条数据。

### 4.1 数据库

一个 `MongoDB` 可以创建多个数据库。数据库名可以是任何字符，但是不能有空格、点号和 `$` 字符。

### 4.2 文档

文档是 `MongoDB` 的核心，类似于 `SQLite` 数据库（关系数据库）中的每一行数据。多个键及其关联的值放在一起就是文档。在 `MongoDB` 中使用一种类 `JSON` 的 `BSON` 存储数据，`BSON` 数据可以理解为在 `JSON` 的基础上添加了一些 `JSON` 中没有的数据类型。

例：

```bash
{"company":"Chenshi keji"}
```

### 4.3 集合

集合就是一组文档的组合，就相当于是**关系数据库中的表**，在 MongoDB 中可以存储的文档可以是不同的结构。

例:

```bash
{"company":"Chenshi keji"} {"people":"man","name":"peter"}
```

上面两个文档就可以存储在同一个集合中。

## 5. 总结

本节实验主要介绍了如下内容：

+ MongoDB 的安装与启动
+ MongoDB 的常用命令行参数
+ 数据库的概念
+ 文档的概念
+ 集合的概念