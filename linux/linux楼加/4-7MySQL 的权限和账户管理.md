---
show: step
version: 1.0
enable_checker: true
---
# MySQL 的权限和账户管理

## 1. 实验介绍

### 1.1 实验内容

本节实验我们将带领大家学习 MySQL 的访问控制及用户的管理。

### 1.2 实验知识点

+ MySQL 的权限

+ 账户管理

+ 授权管理

## 2. 访问权限

MySQL 有自己的权限管理系统，而服务器将访问控制相关的权限信息存储在数据库中。在服务器启动时，将会读取这些表的内容，并且基于表中的内容进行访问控制。下面我们将简单介绍这些表的内容。

### 2.1 概述

关于账户权限的信息被存储在系统数据库 `mysql` 的 `user`, `db`, `tables_priv`, `columns_priv` 以及 `procs_priv` 表中。

包含的相关信息如下：

+ `user` 用户账户，全局权限和其它非特权列

+ `db` 数据库级别的特权

+ `tables_priv` 表级特权

+ `columns_priv` 列级特权

+ `procs_priv` 存储过程和函数权限

### 2.2 user

对于 MySQL 用户来说，账户相关的信息被存储在 `mysql` 数据中的数据表 `user` 中:

```bash
# 我们可以查看 `USER` 表的列信息
mysql> desc mysql.user;
+------------------------+-----------------------------------+------+-----+---------+-------+
| Field                  | Type                              | Null | Key | Default | Extra |
+------------------------+-----------------------------------+------+-----+---------+-------+
| Host                   | char(60)                          | NO   | PRI |         |       |
| User                   | char(16)                          | NO   | PRI |         |       |
| Password               | char(41)                          | NO   |     |         |       |
| Select_priv            | enum('N','Y')                     | NO   |     | N       |       |
| Insert_priv            | enum('N','Y')                     | NO   |     | N       |       |
| Update_priv            | enum('N','Y')                     | NO   |     | N       |       |
...

```

从表中可以知道 `user` 表是由 `Host` 和 `User` 项一起作为主键的，即在 MySQL 中，主机名和用户名一起作为**标识**来识别用户。

在第一节内容中，讲述了如何连接到 MySQL 服务器，而在对服务器进行访问时，会有一个**连接验证**的阶段。这个的验证阶段就需要主机名，用户名以及密码等内容进行验证。

> 这里需要特别强调，主机名和用户名一起作为标识。例如有一台服务器，这时我们有两个客户端分别为 A 和 B。两个客户端都使用用户名为 `shiyanlou` 的用户登陆，但是由于它们是不同的客户端主机，即使两者使用的用户名相同，它们在连接到服务器端时会被当作两个不同的用户

表中的 `Select_priv`，`Insert_priv`，`Update_priv`等代表该用户是否拥有全局相应操作的**特权（privilege）**，缩写为 `priv`，即查询，插入，更新的特权。

然后我们查看用户表 `mysql.user` 的部分信息，如下所示：

```bash
mysql> select host,user,password,select_priv from mysql.user;
+-----------+------------------+-------------------------------------------+-------------+
| host      | user             | password                                  | select_priv |
+-----------+------------------+-------------------------------------------+-------------+
| localhost | root             |                                           | Y           |
| ubuntu    | root             |                                           | Y           |
| 127.0.0.1 | root             |                                           | Y           |
| ::1       | root             |                                           | Y           |
| localhost | debian-sys-maint | *C6B4BF1D2B688BB0C6DFBD20FD5457C4D583D3A1 | Y           |
+-----------+------------------+-------------------------------------------+-------------+
```

可以看到此时系统中的用户都为 `root` 用户，拥有全局的查询特权。

### 2.3 db

而除了全局特权之外，我们还可以指定数据库级别的特权，如下所示：

```bash
mysql> desc mysql.db;
+-----------------------+---------------+------+-----+---------+-------+
| Field                 | Type          | Null | Key | Default | Extra |
+-----------------------+---------------+------+-----+---------+-------+
| Host                  | char(60)      | NO   | PRI |         |       |
| Db                    | char(64)      | NO   | PRI |         |       |
| User                  | char(16)      | NO   | PRI |         |       |
| Select_priv           | enum('N','Y') | NO   |     | N       |       |
| Insert_priv           | enum('N','Y') | NO   |     | N       |       |
| Update_priv           | enum('N','Y') | NO   |     | N       |       |
| Delete_priv           | enum('N','Y') | NO   |     | N       |       |
| Create_priv           | enum('N','Y') | NO   |     | N       |       |
...
```

对于 `mysql.db` 表来说，`Host`, `Db`, `User` 三者作为表的主键，描述的是用户与数据库的权限。

即用户是否具有对该数据库进行查找，插入，删除等操作的特权。

### 2.4 表和列

由于各个表所描述的权限所处的层级不一样，因此一些具体的操作权限也会有区别。因为在上面的内容中，我们并未列出表中的全部字段，所以看不到这一区别。

除了在全局和数据库级别上进行访问权限的限制之外，MySQL 中还提供表级和列级的权限控制，如下所示，分别为 `tables_priv` 和 `colums_priv` 的列：

```bash
mysql> desc mysql.tables_priv;

mysql> desc mysql.columns_priv;
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1515653445220.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1515653480245.png-wm)

## 3. 账户管理

在上面的内容中，我们描述了访问权限的一些知识，在本节内容中，我们将介绍关于用户的一些知识，并结合上面的权限，带领大家学习用户授权的相关知识。

### 3.1 root 用户

在实验楼的在线实验环境中，我们并未对 `root` 用户设置相应的密码，例如刚刚我们查询的 `mysql.user` 表的信息：

```
mysql> select Host,User,Password from mysql.user;
+-----------+------------------+-------------------------------------------+
| Host      | User             | Password                                  |
+-----------+------------------+-------------------------------------------+
| localhost | root             |                                           |
| ubuntu    | root             |                                           |
| 127.0.0.1 | root             |                                           |
| ::1       | root             |                                           |
| localhost | debian-sys-maint | *C6B4BF1D2B688BB0C6DFBD20FD5457C4D583D3A1 |
+-----------+------------------+-------------------------------------------+
```

如上图所示，前四个用户为 MySQL 创建的 `root` 用户，并且 `localhost`,`ubuntu`,`127.0.0.1`和 IPV6 格式的 `::1` 都是指向的同一个地址。它们的密码一栏都为空，因为实验环境中并未设置 `root` 用户的密码。

这里之所以会创建四个用户，是由于实验环境中的 MySQL 版本为 `5.5` 。而在 `5.7` 中只会创建一个 `root@localhost` 的用户。

而最后一个 `debian-sys-maint` 则为 ubuntu 系统创建的用户，为了方便管理 MySQL 服务在 Ubuntu 中的运行。

除此之外，还可能存在匿名账户，在实验环境中并不存在。

对于没有设置密码的用户，我们使用其连接 MySQL 服务时，不需要使用 `-p` 参数，如下所示：

```bash
shiyanlou:~/ $ mysql -u root                                         [15:21:10]
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 71
Server version: 5.5.50-0ubuntu0.14.04.1 (Ubuntu)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

### 3.2 创建账户

创建账户，一般有两种方式：

+ 使用 MySQL 提供的 `CREATE USER` 和 `GRANT` 等语句。（推荐使用的方式）

+ 通过修改 `mysql.user` 表创建账户。

#### CREATE USER

对于创建用户的详细语法较为复杂，这里没有全部列举。

```
CREATE USER [IF NOT EXISTS]
    user IDENTIFIED BY 'password';
```

`user` 为账户名称，合法的语法为 `'user_name'@'host_name'`，例如 `root@localhost`

如下所示，我们创建一个名为 `syl001@localhost` 的账户，密码为 `shiyanlou`:

```
mysql> CREATE USER syl001@localhost IDENTIFIED BY "shiyanlou";
Query OK, 0 rows affected (0.26 sec)
```

查询 `mysql.user` 表，查看 `syl001` 用户：

```
mysql> SELECT Host,User,Password FROM mysql.user;
+-----------+------------------+-------------------------------------------+
| Host      | User             | Password                                  |
+-----------+------------------+-------------------------------------------+
| localhost | root             |                                           |
| ubuntu    | root             |                                           |
| 127.0.0.1 | root             |                                           |
| ::1       | root             |                                           |
| localhost | debian-sys-maint | *C6B4BF1D2B688BB0C6DFBD20FD5457C4D583D3A1 |
| localhost | syl001           | *FFB8814DBE01A23A0780B2DFF96C426CA67A9752 |
+-----------+------------------+-------------------------------------------+
6 rows in set (0.00 sec)
```

#### 直接修改 mysql.user 表

如下所示，我们直接使用 `mysql.user` 表进行操作，插入一条数据：

```bash
mysql> INSERT into mysql.user(Host, User, Password) values ("localhost", "syl002", "shiyanlou");

mysql> SELECT Host,User,Password FROM mysql.user;
+-----------+------------------+-------------------------------------------+
| Host      | User             | Password                                  |
+-----------+------------------+-------------------------------------------+
| localhost | root             |                                           |
| ubuntu    | root             |                                           |
| 127.0.0.1 | root             |                                           |
| ::1       | root             |                                           |
| localhost | debian-sys-maint | *C6B4BF1D2B688BB0C6DFBD20FD5457C4D583D3A1 |
| localhost | syl001           | *FFB8814DBE01A23A0780B2DFF96C426CA67A9752 |
| localhost | syl002           | shiyanlou                                 |
+-----------+------------------+-------------------------------------------+
7 rows in set (0.00 sec)
```

如上所示，可以看到 `syl002` 用户添加成功，但是 `password` 字段显示的是由我们指定的密码 `shiyanlou` ，不过这样插入的用户数据并不合法，因为密码字段并没有经过 `MySQL` 相应的处理，所以使用 `shiyanlou` 这个密码和用户名并不能进行合法登录。

### 3.3 删除用户

删除用户的语法为：

```
DROP USER [IF EXISTS] user [, user] ...
```

例如，删除刚刚创建的 `syl002@localhost` 账户：

```
mysql> DROP USER "syl002"@"localhost";
Query OK, 0 rows affected (0.00 sec)
```

此时，再次查看 `mysql.user` 表，可以发现 `syl002@localhost` 已经被删除。

### 3.4 修改密码

修改用户的密码，由于版本的差异，在 5.7.6 之后，需要使用如下语法：

```bash
ALTER USER user IDENTIFIED BY 'new_password';
```

在 5.7.6 之前，即实验环境中，使用：

```
SET PASSWORD FOR user = PASSWORD('new_password');
```

如下所示，我们为 `root` 用户设置密码为 `123456`:

```
mysql> SET PASSWORD FOR "root"@"localhost" = PASSWORD("123456");
Query OK, 0 rows affected (0.00 sec)
```

这时，我们再次查询 `mysql.user` 表的数据:

```
mysql> SELECT Host,User,Password FROM mysql.user;
+-----------+------------------+-------------------------------------------+
| Host      | User             | Password                                  |
+-----------+------------------+-------------------------------------------+
| localhost | root             | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
| ubuntu    | root             |                                           |
| 127.0.0.1 | root             |                                           |
| ::1       | root             |                                           |
| localhost | debian-sys-maint | *C6B4BF1D2B688BB0C6DFBD20FD5457C4D583D3A1 |
| localhost | syl001           | *FFB8814DBE01A23A0780B2DFF96C426CA67A9752 |
+-----------+------------------+-------------------------------------------+
```

需要说明的是，如果在安装 MySQL 的时候分配了 root 密码，那么前四个由 MySQL 创建的用户的密码就会是一样的。

## 4. 授权

在 MySQL 中，我们经常会接触到的有关权限的语句： `GRANT` 和 `REVOKE`，它们分别为用户授权和撤销授权。

而在 `GRANT` 语句中指定的账户不存在时，`GRANT` 会隐式的创建它。

授权的语法大致如下：

```bash
GRANT priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON priv_level
    TO user [IDENTIFIED BY 'password'] [, user [IDENTIFIED BY 'password']] ...

priv_level: {
   *.*
  | db_name.*
  | db_name.tbl_name
  | tbl_name
  | db_name.routine_name
}
```

`priv_type` 代表的是特权权限，部分内容如下图所示， 

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515678655814.png/wm)

`ALL` 代表所有的权限，其它权限都对应着 mysql 数据库的 `user`,`db` 等表中的内容。而可选的 `[(column_list)]` 用于定义了列级权限时使用。

`priv_level` 代表的是特权级，具体含义如下所示：

+ `*.*` 代表全局，即所有的数据库以及数据库中所有的内容

+ `db_name.*` 代表数据库，即数据库所有的内容

+ `db_name.tbl_name` 即数据库中指定的表

+ `tbl_name` 未指定数据库，为默认数据库中的表

+ `db_name.routine_name` 存储过程的权限（未涉及）

对于创建的用户 `syl001` 来说，此时未授予其任何权限。

如下示例：

```bash
# 首先，我们创建一个 shiyanlou002 数据库，并在其中创建一张 test 表作为演示示例
mysql> create database shiyanlou002;
Query OK, 1 row affected (0.00 sec)
mysql> use shiyanlou002;
Database changed
mysql> create table test(name varchar(10), age int);
Query OK, 0 rows affected (0.35 sec)

```

这时，我们新打开一个终端，避免后面来回切换用户。

```bash
# 使用 `syl001` 用户进行登陆。只能查看 MySQL 系统的信息数据库
shiyanlou@:~$ mysql -u syl001 -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
...
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
+--------------------+
1 row in set (0.00 sec)
```

然后换到 `root` 用户的终端，授予其访问 test 表的 `name` 列的插入和更新的权限。

```bash
mysql> GRANT INSERT(name), UPDATE(name) ON shiyanlou002.test TO "syl001"@localhost;
Query OK, 0 rows affected (0.28 sec)
```

```bash
# 接着，再次使用 `syl001` 进行操作
mysql> insert into shiyanlou002.test(name) values("h");
Query OK, 1 row affected (0.28 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| shiyanlou002       |
+--------------------+

# 因为在上面的内容中，我们只给予了其插入和更新 `name` 列的权限，所以执行查询和删除操作都会被拒绝，如下所示：

mysql> select * from shiyanlou002.test;
ERROR 1142 (42000): SELECT command denied to user 'syl001'@'localhost' for table 'test'

mysql> delete from shiyanlou002.test where name="h";
ERROR 1142 (42000): DELETE command denied to user 'syl001'@'localhost' for table 'test'
```

需要注意的是，如果给予了用户全局的权限，尽管其不具备对低于全局的特权级别，如某个数据库的权限，该用户对于该数据库的操作也会被执行。

## 5. 撤销授权

对应 `GRANT`，撤销授权的语法为 `REVOKE`:

```
REVOKE
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON priv_level
    FROM user [, user] ...
```

如下示例，撤销 `syl001` 对于 `shiyanlou002.test` 表中 `name` 列的插入权限。需要切换到 root 用户登录的终端：

```bash
mysql> REVOKE INSERT(name) ON shiyanlou002.test FROM "syl001"@"localhost";
Query OK, 0 rows affected (0.00 sec)

```

到 `syl001` 用户的终端执行。

```bash
# 这时，再次插入时就会失败，只能更新 `name` 列，因为更新权限还未被撤销
mysql> insert into shiyanlou002.test(name) values("h");
ERROR 1142 (42000): INSERT command denied to user 'syl001'@'localhost' for table 'test'

mysql> update shiyanlou002.test set name="shiyanlou";
Query OK, 1 rows affected (0.30 sec)
Rows matched: 1  Changed: 1  Warnings:0
```

MySQL 权限和账号管理操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/9-1.mp4
@`

## 6. 总结

通过本节实验一起学习了 MySQL 中的访问控制权限，用户的权限管理操作，以及授权用户和撤销授权等。