---
show: step
version: 1.0
enable_checker: true
---
# MySQL 备份与恢复

## 1. 实验介绍

#### 1.1 实验内容

在本节内容中，我们将学习 MySQL 备份与恢复的一些相关知识

#### 1.2 实验知识点

+ 导出表的数据

+ mysqldump

+ 二进制日志备份

## 2. 概述

#### 2.1 方法介绍

在第一周 Linux 相关内容的学习中，我们有提到过备份的一些知识。

在大多数时候，对于一个Linux 系统来讲，并不是所有的内容都是需要备份的。有些时候我们只需要对于一些关键数据进行备份。而对于需要备份的数据可能会根据实际的情况有所不同，例如，当前系统主要提供的是数据库存储等服务，那么重要的就是你的数据库文件，以及一些重要的配置信息。

而对于这里的数据库备份来说，MySQL 有自己的一些备份的方法。这里我们简要介绍 MySQL 的几种备份方法：

+ 直接复制数据文件

+ 使用导出表数据到一个文件中的方式

+ 使用客户端工具，mysqldump

+ 使用二进制日志进行备份

#### 2.2 创建示例表和数据库

在开始学习备份相关的操作时，我们需要创建相应的示例表和数据库。这里直接使用前面的选课数据库用作演示示例。相应的创建步骤可以参考本周第五个实验，数据的搜索部分的相关知识。

## 3. 数据目录

我们知道 MySQL 的配置文件是存放在 `/etc/mysql/` 目录下，而它的数据保存路径则为 `/var/lib/mysql/`。该路径在 `/etc/mysql/my.cnf` 配置文件中有相应的定义：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516009517802.png/wm)

因此可以直接复制整个 `/var/lib/mysql` 目录来备份数据库。

首先，我们复制该文件夹到当前目录下：

```bash
$ sudo cp -rf /var/lib/mysql/ ~/mysql
```

这时我们手动删除 `shiyanlou001` 数据库，使用如下语句：

```
$ mysql -u root -p -e "DROP DATABASE IF EXISTS shiyanlou001;"
```

删除后，再次查看 `shiyanlou001` 数据库就已经不存在了。我们如果需要恢复 `shiyanlou001` 数据库，可以直接将当前目录下的 `mysql` 文件夹来替换复制到 `/var/lib/mysql` 目录，使用如下命令：

```bash
$ sudo rm -r /var/lib/mysql
$ sudo cp -rf ~/mysql /var/lib/mysql
```

命令执行成功后，该数据中的数据为我们在执行删除 `shiyanlou001` 数据库之前的内容，但是此时依旧不能正确使用该数据库，因为我们复制时使用的是 `sudo`，即该目录所属的用户和用户组都为 `root`，需要将其修改为 `mysql`，使用如下命令：

```
$ sudo chown -R mysql:mysql /var/lib/mysql
$ sudo service mysql restart
```

```checker
- name: check priv
  script: |
    #!/bin/bash
	stat -c %U /var/lib/mysql|grep mysql
	stat -c %G /var/lib/mysql|grep mysql
  error: /var/lib/mysql 的所属者和所属组不是 mysql
```

该命令执行成功后，就可以正确的使用该数据库了。例如我们通过如下语句，查看 `shiyanlou001` 数据库中的表：

```bash
$ mysql -u root -p -e "USE shiyanlou001;SHOW TABLES;"
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516012076389.png/wm)

## 4. 导出表数据

### 4.1 导出

在数据的搜索一节中我们介绍了很多 `SELECT` 的使用方式。除了之前学习过的使用方法，我们还可以使用 `SELECT... INTO` 的形式将查询的结果写入到文件中。通过这样的方式来达到备份的效果。

> 需要注意的是，写入的文件在写入时不能已经存在

详细的语法格式如下：

```bash
SELECT ... INTO OUTFILE 'file_name' [CHARACTER SET charset_name] export_options

或者使用

SELECT ... INTO DUMPFILE 'file_name'
```

1. 第一条语句可以将选定的行，即查询到的行写入文件。并且可以通过 `export_options`，即导出配置，指定输出格式。

2. 第二条语句，导出的文件中没有任何的格式。

例如，我们导出 `student` 数据表到 `student.txt` 文件中，就可以使用如下语句：

```bash
mysql> USE shiyanlopu001;

mysql> SELECT * FROM student INTO OUTFILE "student.txt" CHARACTER SET utf8;
```

上述命令会在 `/var/lib/mysql/shiyanlou001/` 目录下生成一个 `student.txt` 文件。保存的路径会受到环境变量 `secure_file_priv` 的影响，我们直接查看该变量的值为：


```bash
mysql> SHOW VARIABLES LIKE "secure_file%";
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| secure_file_priv |        |
+------------------+--------+
```

该变量的值为空，所以这里会保存到当前所使用的数据库的数据目录下。这里需要着重强调的是，该文件不应放在 `/var/lib/mysql/shiyanlou001/` 目录下，会影响我们对于 `shiyanlou001` 数据库的一些操作，例如，我们通过 `sql` 语句删除 `shiyanlou001` 数据库时就会因为无法删除该文件而提示错误。

> 而在 MySQL 5.5.53 之后的版本中，该变量的值变为 `/var/lib/mysql-files/`，导出文件路径请使用完整路径，比如“/var/lib/mysql-files/student.txt”。实验环境中的 MySQL 版本为 5.5.50。

所以这里我们需要删除掉刚刚生成的 `student.txt` 文件，并修改 MySQL 的配置文件以设置 `secure_file_priv` 的值：

1. 删除文件
```
$ sudo rm /var/lib/mysql/shiyanlou001/student.txt
```

```checker
- name: check file
  script: |
    #!/bin/bash
	! ls /var/lib/mysql/shiyanlou001/student.txt
  error: 没有删除 /var/lib/mysql/shiyanlou001/student.txt 文件
```

2. 修改 `/etc/mysql/my.cnf` 配置文件，并在其中加入一行内容：

```
[mysqld]
...

secure_file_priv = /var/lib/mysql-files
```

注意需要手动创建这个目录 /var/lib/mysql-files，并使用 chown 设置目录的所有者为 mysql 用户才可以启动 MySQL 服务器。

```
sudo mkdir /var/lib/mysql-files
sudo chown mysql.root -R /var/lib/mysql-files
```

```checker
- name: check file
  script: |
    #!/bin/bash
	ls /var/lib/mysql-files
  error: /var/lib 目录下没有 mysql-files 目录
- name: check priv
  script: |
    #!/bin/bash
	stat -c %U /var/lib/mysql-files|grep mysql
  error: /var/lib/mysql-files 所属者不是 mysql
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1516013080509.png/wm)

3. 重启 MySQL

```bash
$ sudo service mysql restart
```

```checker
- name: check service
  script: |
    #!/bin/bash
	ps -ef|grep -v grep|grep mysql
  error: 没有启动 mysql
```

如上所示，上述操作执行成功后，可以通过如下语句查看该变量设置的值：

```bash
mysql> SHOW VARIABLES LIKE "secure_file%";
+------------------+-----------------------+
| Variable_name    | Value                 |
+------------------+-----------------------+
| secure_file_priv | /var/lib/mysql-files/ |
+------------------+-----------------------+
```

该变量的值为 `/var/lib/mysql-files/`，所以我们在进行导出的时候，需要修改相应的路径：

```bash
mysql> SELECT * FROM student INTO OUTFILE "/var/lib/mysql-files/student.txt" CHARACTER SET utf8;
Query OK, 5 rows affected (0.01 sec)
```

导出成功后，查看 `/var/lib/mysql-files/student.txt` 可以看到表中的数据被导出，使用如下命令：

```bash
$ sudo cat /var/lib/mysql-files/student.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515983098688.png/wm)

该文件的内容按照一定的格式被导出，而这种格式我们也可以通过一些配置项来进行设置。即语法格式中的 `export_options`，它们的使用方式如下：

```
[FIELDS
    [TERMINATED BY 'string']
    [[OPTIONALLY] ENCLOSED BY 'char']
    [ESCAPED BY 'char']
]
[LINES
    [STARTING BY 'string']
    [TERMINATED BY 'string']
]
```

上述语法中描述了两个可选的子句，即 `FIELDS` 和 `LINES` 子句。

首先是 `FIELDS` 子句的三个可配置项，它们代表的意思如下：

+ `TERMINATED BY 'string']` 用来指定字段之间的分隔符，例如我们指定为逗号(`,`)，则导出的格式类似于 `csv` 格式。

+ `[[OPTIONALLY] ENCLOSED BY 'char']` 可以指定字符值使用什么字符包裹，例如使用双引号(`'"'`)，则字符值的数据都会使用双引号引起来。

+ `[ESCAPED BY 'char']` 用来指定转义字符，默认值为斜杠符号 `\`。

如下示例，我们使用上述的三个可选项：

```bash
# 设定以逗号为分隔符，字符值使用双引号引起来，并设定转义字符为 # ,保存到 student1.txt 文件中
mysql> SELECT * FROM student INTO OUTFILE "/var/lib/mysql-files/student1.txt" CHARACTER SET utf8 FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' ESCAPED BY '#';
Query OK, 5 rows affected (0.00 sec)
```

这时我们查看该文件的值，可以看到对应的修改：

```
$ sudo cat /var/lib/mysql-files/student1.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515983297285.png/wm)

除了 `FIELDS` 子句，还有 `LINES` 子句，其两个可选的配置项如下所示：

+ `[STARTING BY 'string']` 指定一行开始的标识字符串。

+ `[TERMINATED BY 'string']` 指定一行结束的标识字符串。

如下示例，分别指定开始字符串为 `'##'`，结束的字符串为 `'####'`：

```bash
mysql> SELECT * FROM student INTO OUTFILE "/var/lib/mysql-files/student2.txt" CHARACTER SET utf8 FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' ESCAPED BY '#' LINES STARTING BY '##' TERMINATED BY '####';
```

查看文件的内容如下图所示:

```bash
$ sudo cat /var/lib/mysql-files/student2.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515983377447.png/wm)

### 4.2 导入

导出数据之后，如果需要导入数据，则可以使用 `LOAD DATA ... INFILE` 语句，相关的语法格式如下：

```bash
LOAD DATA INFILE 'file_name'
    INTO TABLE tbl_name
    [CHARACTER SET charset_name]

    [{FIELDS | COLUMNS}
        [TERMINATED BY 'string']
        [[OPTIONALLY] ENCLOSED BY 'char']
        [ESCAPED BY 'char']
    ]
    [LINES
        [STARTING BY 'string']
        [TERMINATED BY 'string']
    ]
    [(col_name, col_name ...)]
```

上述语法中的可选项 `(col_name, col_name ...)` 用于选择将要导入的部分列，因为导入与导出的数据有时并不一致。

> 需要注意的是，在导出表的时候，我们只导出了其中的数据，但是对于表结构的定义我们并没有进行导出。所以在进行导入的时候，表结构需要自己定义，即该表需要提前创建。

例如，我们将该文件的数据导入到 `import_student` 表中，首先需要创建该表：

```
mysql> USE shiyanlou001;
Database changed

mysql> CREATE TABLE import_student(
    ->     s_id INT,
    ->     s_name VARCHAR(20) NOT NULL,
    ->     s_sex ENUM("man","woman") DEFAULT "man",
    ->     s_age INT NOT NULL,
    ->     PRIMARY KEY (s_id)
    -> ) DEFAULT CHARSET=utf8;
Query OK, 0 rows affected (0.09 sec)

mysql> 

```

创建成功后，我们执行导入语句，将 `student.txt` 文件中内容导入到 `import_student` 表中，并查看导入的数据：

```
mysql> LOAD DATA INFILE "/var/lib/mysql-files/student.txt" INTO TABLE import_student;
Query OK, 5 rows affected (0.03 sec)
Records: 5  Deleted: 0  Skipped: 0  Warnings: 0

mysql>
mysql> select * from import_student;
+------+---------------+-------+-------+
| s_id | s_name        | s_sex | s_age |
+------+---------------+-------+-------+
| 1001 | shiyanlou1001 | man   |    10 |
| 1002 | shiyanlou1002 | woman |    20 |
| 1003 | shiyanlou1003 | man   |    18 |
| 1004 | shiyanlou1004 | woman |    40 |
| 1005 | shiyanlou1005 | man   |    17 |
+------+---------------+-------+-------+
5 rows in set (0.00 sec)
```

```checker
- name: check sql
  script: |
    #!/bin/bash
	mysql -u root -p 123456 shiyanlou001 -e "select * from import_student"|grep "man"
  error: 导入数据不成功
```

如上所示，导入成功。

数据库导出和导入操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/10-1.mp4
@`

## 5. mysqldump

MySQL 提供了很多的客户端程序和工具用于各种辅助操作。这里我们将要介绍的 `mysqldump` 就属于 MySQL 提供的一个客户端应用程序。

该程序用于执行逻辑备份，产生一组能够被执行用于再现原始数据库对象定义和表数据的 SQL 语句。它与上述保存数据到文件中操作不同的是，它还能保存数据库及表的结构，即刚刚所述的原始数据库的对象定义。

`mysqldump` 作为一个客户端的应用程序，我们可以很方便的查看其使用方式和各个配置项的使用方式，使用如下命令：

```bash
$ mysqldump --help
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515983970737.png/wm)

下面我们列出官方所给出的三种常用于 `mysqldump` 的使用方法：

```bash
# 备份数据库或数据库中的一个或多个表
shell> mysqldump [options] db_name [tbl_name ...]

# 备份一个或多个数据库
shell> mysqldump [options] --databases db_name ...

# 备份所有的数据库
shell> mysqldump [options] --all-databases
```

例如，我们备份示例数据库 `shiyanlou001`，就可以使用如下语句：

```bash
$ mysqldump -u root --databases shiyanlou001 > dump.sql
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515984135872.png/wm)

> 需要注意的是，如果在上一节的账户管理操作部分修改了密码，则这里需要输入对应的 root 用户的密码

上述语句会产生一个 `dump.sql` 文件，该文件中包含能够重现 MySQL 数据库 `shiyanlou003` 的定义和数据的 `sql` 语句，因此，如果我们需要恢复该语句，只需要执行 `dump.sql` 文件中的 `sql` 语句即可。

为了演示这一操作，首先我们可以删除掉 `shiyanlou001` 数据库，然后通过 `dump.sql` 文件进行恢复：

```bash
# 删除 shiyanlou001 数据库
$ mysql -u root -p -e "DROP DATABASE IF EXISTS shiyanlou001;"

# 从 mysql.dump 文件中恢复 shiyanlou001 数据库
$ mysql -u root -p < dump.sql

# 恢复后，查看是否成功恢复
$ mysql -u root -p -e "USE shiyanlou001; SHOW TABLES;"
```

运行截图如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515984786026.png/wm)

mysqldump 使用操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/10-2.mp4
@`

## 6. 二进制日志

对于上述两种方式的操作，其实我们只是在某一时刻对数据库进行了备份，即相当于该时刻数据库的一个快照，我们并不可能通过上面的方式随时备份数据。而当数据丢失或者损坏时，我们只能恢复已经备份的文件。

对于这个问题，我们可以通过日志来解决。二进制日志包含我们对数据库进行更改的事件，例如创建或者修改表等操作。我们可以通过其进行恢复操作。

### 6.1 启用日志

首先我们需要启用二进制日志，编辑 `/etc/mysql/my.cnf` 文件，在其中加入一行内容：

```txt
[mysqld]
...

log-bin = mysql_bin_log
...
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515986889183.png/wm)

编辑完成后，指定日志文件名为 `mysql_bin_log`，此时我们还需要重启 `mysql`，使用如下命令：

```bash
$ sudo service mysql restart
```

重启后，会在 `/var/lib/mysql` 路径下生成相应的日志文件，如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515987169724.png/wm)

图片中的 `mysql_bin_log.000001` 文件，会自动生成数字后缀，每次创建新的日志文件时，这个数字都会增加。而 `mysql_bin_log.index` 文件则是自动创建的二进制日志的索引文件，会记录相应的二进制日志文件。

### 6.2 mysqlbinlog

对于生成的日志来说，会以二进制的格式写入数据。所以如果需要查看二进制格式的日志，我们需要借助 `mysqlbinlog` 程序，该程序用于处理 MySQL 的二进制日志文件。

此时，我们可以尝试进行一些修改操作，例如我们创建一个 `test_binlog` 数据库：

```bash
mysql> CREATE DATABASE test_log;
Query OK, 1 row affected (0.00 sec)
```

创建成功之后，我们需要切换到 `root` 用户，并切换目录到 `/var/lib/mysql` 目录下，如下所示：

```bash
$ sudo su root

# cd /var/lib/mysql
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515988774295.png/wm)

这时，我们可以使用 `mysqlbinlog` 查看日志文件：

```bash
mysqlbinlog mysql_bin_log.000001
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515988895435.png/wm)

由于输出的信息较多，我们可以使用 `grep` 命令匹配信息。使用如下命令：

```bash
mysqlbinlog mysql_bin_log.000001 | grep "test_log"
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515989044233.png/wm)

可以看到记录的操作，此时我们手动删除掉 `test_log` 数据库，再通过日志恢复该数据库，相应的操作步骤如下：

1. 手动删除 `test_log` 数据库：

```bash
mysql> DROP DATABASE test_binlog;
Query OK, 0 rows affected (0.02 sec)
```

2. 通过 `mysqlbinlog` 得到一个包含 `sql` 语句的文件：

```bash
# root 用户下执行
mysqlbinlog mysql_bin_log.000001 > new_sql.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1515989430436.png/wm)

3. 最后执行该文件中的 `sql` 语句即可：

```
mysql -u root -p < new_sql.txt
```

此时，刚刚删除的数据库就已经恢复成功了。

上述操作我们演示了一个非常简单的示例，但是 `mysqlbinlog` 还有很多其它常用的参数，例如下面两个参数：

+ `--start-datetime="2018-01-01 11:11:11"` 指定开始时间

+ `--stop-datetime="2018-01-01 11:11:11"`  指定结束时间

该选项可以用来截取指定时间段的日志，即通过这种方式，我们可以实现指定时间点的恢复等操作。

> 由于操作时间的不一致，这里我们就不演示用例，有兴趣的同学可以自行尝试

## 7. 总结

在本节实验内容中，我们介绍了一些用于备份和恢复的简单方法。