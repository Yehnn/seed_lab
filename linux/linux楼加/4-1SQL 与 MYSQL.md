---
show: step
version: 1.0
enable_checker: true
---
# SQL 与 MYSQL

## 1. 实验介绍

#### 1.1 实验内容

在本节内容，我们将带领大家学习了解数据库的一些相关知识，以及简单介绍 SQL。

#### 1.2 实验知识点

+ 数据库简介
+ SQL 简介
+ 安装 MySQL

## 2. 概述

#### 2.1 数据库和 SQL

数据库(`Database`)是按照数据结构来组织、存储和管理数据的仓库。它的产生距今已有六十多年。随着信息技术和市场的发展，数据库变得无处不在。它在电子商务、银行系统等众多领域都被广泛使用，且成为其系统的重要组成部分。

数据库用于记录数据，同时可以表现出各种数据间的联系，也可以很方便地对所记录的数据进行增、删、改、查等操作。

结构化查询语言(Structured Query Language)简称 `SQL`，是上世纪 70 年代由 IBM 公司开发，用于对数据库进行操作的语言。更详细地说，`SQL` 是一种数据库查询和程序设计语言，用于存取数据以及查询、更新和管理关系数据库系统，同时也是数据库脚本文件的扩展名。

#### 2.2 MySQL

`MySQL` 是一个 DBMS（数据库管理系统），由瑞典 MySQLAB 公司开发，目前属于 `Oracle` 公司，`MySQL` 是最流行的关系型数据库管理系统之一（关系数据库，是建立在关系数据库模型基础上的数据库，借助于集合代数等概念和方法来处理数据库中的数据）。由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发者都选择 MySQL 作为网站数据库。

`MySQL` 使用 `SQL` 语言进行操作。但是对于 `SQL` 来讲，它并不是只能在 `MySQL` 中使用，所有的数据库管理系统几乎都支持它，而 `MySQL` 支持几乎全部的 `SQL` 标准语句，仅仅对其中一小部分内容进行扩展和修改。

数据库管理系统（DBMS）的功能主要包括以下几个方面：

+ 数据定义

+ 数据组织，存储和管理

+ 数据操作

+ 数据库的运行和事务管理

+ 数据库的建立和维护

+ 通信

## 3. MySQL 的安装与使用

下面我们将会学习MySQL 的安装与使用。

### 3.1 安装

在实验楼的实验环境中，已经安装了 MySQL 服务。下面的安装步骤仅做参考：

对于实验楼环境中使用的 `ubuntu`，我们可以使用 `apt` 安装 `MySQL`。该命令会安装 `MySQL` 服务器，客户端和一些其它的组件。命令如下：

```bash
# 在安装 mysql-server 的时候 mysql-client 与 mysql-comman 会推荐安装所以不需要单独安装
$ sudo apt-get install mysql-server
···
```

在安装过程中，会有两个对话框，分别为：

+ 为 `MySQL` 设置 `root` 用户的密码

+ 选择是否安装测试数据库（根据版本变化，有时并不会出现该对话框）

### 3.2 启动 MySQL

`MySQL` 服务器在安装后会自动启动，但是在实验环境中虽然已经安装了 `MySQL`，但是并未启动 MySQL（这是因为实验环境的问题，导致所有的自启动项都不会生效），我们可以实验以下命令检查 MySQL 服务器的状态：

```bash
$ sudo service mysql status
 * MySQL is stopped.
```

使用以下命令启动 MySQL 服务器：

```bash
$ sudo service mysql start
```

### 3.3 连接及退出服务器

在启动 Mysql 服务之后，我们就可以连接到 Mysql 服务中，在连接时，通常需要提供用户名与对应的密码。如果需要远程连接到其他机器的 Mysql 服务，还需要指定主机名。大致的格式如下：

```bash
$ mysql -h <host> -u <user> -p
Enter password:
```

**若是登录的用户设置了密码，需要通过 `-p` 参数来输入密码，若是没有设置密码即可不使用 `-p` 参数**

**在实验环境中，在安装时并未设置密码，若是使用了 `-p` 参数，在需要交互式下输入密码时，直接按 `<ENTER>` 键即可。**

```bash
$ mysql -u root -p
Enter password: # root 用户登录直接回车即可
```

在连接成功后，如果需要断开连接，可以输入 `exit` 来退出：

```bash
# 也可以通过 `\q` 退出
mysql> exit
```

如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid3903timestamp1510042643102.png/wm)


MySQL 的安装与使用操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/1-1.mp4
@`

## 4. 配置

`MySQL` 程序可以从配置文件中读取启动选项，配置文件提供了一种方便的方式来指定常用选项，以便每次运行程序时不需要在命令行中输入。

而对于所有的可配置项，在 `MySQL` 服务器中以变量的形式存在。例如，我们一般查看 MySQL 服务器使用的命令选项和系统变量的值，可以通过以下命令来执行：

```bash
$ mysqld --verbose --help
...
```

这里的 `mysqld` 指代 `MySQL 服务器`，该命令会显示一些可用的配置项及描述信息，并会生成一个列表，显示对应的可配置项的默认值，运行结果所生成的列表如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515466155722.png/wm)

为了更好的看见两者的对应关系，使用如下命令：

```bash
$ mysqld --verbose --help | grep "character"
```

下图中的运行结果不仅会显示命令行选项的描述，还会显示对应的值：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515467434211.png/wm)

而对于部分命令行的参数来说，例如 `--character-set-server` 选项，在配置文件中有对应的可配置项，为 `character-set-server`。即将两个前置的短横线（`--`） 去掉，则得到对应的配置文件中可以编辑的配置项，如下表所示： 

| 命令行选项                 | 配置项                   |
| :------------------------- | :----------------------- |
| --auto-increment-increment | auto-increment-increment |
| --basedir                  | basedir                  |
| --character-set-server     | character-set-server     |
| ...                        | ...                      |

除此这种方式之外，我们还可以连接到 `mysql` 服务器，查看运行时使用的当前系统变量的值，使用如下命令:

```bash
mysql> SHOW VARIABLES;
```

**需要说明的是 `mysql` 中一个完整的语句是以 `;` 号或者 `\g` 结尾**。在连接服务端时，也可以看到对应的提示信息：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515464406247.png/wm)

因此上面的命令也可以修改为:

```
mysql> show variables\g
+--------------------------+--------+
| Variable_name            | Value  |
+--------------------------+--------+
| auto_increment_increment |    1   |
| auto_increment_offset    |    1   |
| autocommit               |   ON   |
....
```

上述只截取了部分输出结果，在这里我们可以查看常见的一些变量值，例如如下所示的当前系统使用的字符集：

```bash
# 显示数据库连接使用的字符集的情况，关于 like 和通配符的使用后面会涉及到，这里只是简单使用
mysql> SHOW VARIABLES LIKE "character%";
```

运行结果如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515465132936.png/wm)

对于临时修改变量，我们可以使用 `set` 命令，如下所示：

```bash
# 设置修改数据库的编码为 utf8
mysql> SET character_set_database=utf8;

# 修改后再次查看
mysql> SHOW variables LIKE 'character_set_database';
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515465414388.png/wm)

使用 `set` 修改的是临时的变量，若要永久生效，可以编辑 `/etc/mysql/my.cnf` 配置文件，我们列举出其中的部分内容，忽略掉注释，内容如下：

```txt
[client]
port = 3306
socket = /var/run/mysqld/mysqld.sock

[mysqld_safe]
socket = /var/run/mysqld/mysqld.sock
nice = 0

[mysqld]
user = mysql
port = 3306
basedir = /usr
datadir = /var/lib/mysql
tmpdir = /tmp
bind-address = 127.0.0.1

[mysqldump]
quick
quote-names
max_allowed_packet = 16M

[mysql]

[isamchk]
key_buffer = 16M


!includedir /etc/mysql/conf.d/
```

配置文件中的 `[mysqld]` ，即服务端的可配置项，可以使用命令 `mysqld --verbose --help` 查看。

而 `[mysql]`，即客户端，对应的命令为 `mysql --help`。

关于配置的详细描述在使用相关的 `--help` 参数时描述的已经足够清楚，例如使用如下命令：

```bash
$ mysql --help | grep character

  --character-sets-dir=name
                      Directory for character set files.
  --default-character-set=name
                      Set the default character set.
character-sets-dir                (No default value)
default-character-set             auto
```

上述内容中的命令行选项 `--default-character-set` 对应于配置文件中的 `default-character-set`，即只需要去除掉开头的两个短横线 `--` 即可，所以，我们可以增加相应的配置项：

```txt
[mysqld]
...
character-set-server=utf8
```

在修改配置文件后，我们需要重启 `mysql` 服务使其生效。使用如下命令：

```bash
$ sudo service mysql restart
```

这时，`mysql` 的默认的数据库的字符集为 `utf8`。

MySQL 配置操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/1-2.mp4
@`

## 5. 总结

在本节内容中，我们简单介绍了 MySQL 数据库的安装，服务的启动，以及怎么连接到数据库和如何修改配置。