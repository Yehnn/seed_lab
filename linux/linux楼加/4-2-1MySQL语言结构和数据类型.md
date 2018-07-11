---
show: step
version: 1.0
enable_checker: true
---
# MySQL 语言结构和数据类型

## 1. 实验介绍

#### 1.1 实验内容

在本节内容中，我们将介绍一些 MySQL 的语言结构和 **部分** 常用的数据类型，为后面课程的学习做一个铺垫。

#### 1.2 实验知识点

+ 数据类型
+ 语言结构

#### 1.3 推荐阅读

- [FLOAT 与 DOUBLE](http://yongxiong.leanote.com/post/mysql_float_double_decimal)

## 2. 数据类型

在这里需要说明的是，我们所讲的数据类型针对于实验环境中 `MySQL` 版本所采用的数据类型的部分内容。

首先，我们需要连接到服务器，并且选择我们上一节课程中创建的名为 `shiyanlou001` 的数据库。

### 2.1 字符串

#### CHAR，VARCHAR，TEXT 和 BLOB

`CHAR` 和 `VARCHAR` 类型类似。长度都代表着可以存储的最大字符数，例如 `CHAR(30)` 代表可以容纳 30 个字符。

不同的是 `CHAR` 的长度为 `0~255` 之间，并且 `CHAR` 类型的字符串的长度是固定的，当保存数据长度不够设置时会自动填充空格，而超出长度的数据则会丢失。

`VARCHAR` 则属于变长字符串，意思是根据需要保存数据的大小进行存储，小于设定的长度时并不会自动填充，超出也会丢失，它也有对应的长度范围，为 `0~65535`。

> 但是一个 `VARCHAR` 字符串的有效最大值会受到行大小，以及使用的字符集的限制

`TEXT` 和 `BLOB` 类似，但是 `TEXT` 不需要指定长度。而 `BLOB` 用来保存二进制的数据，例如可以用来保存图片，音乐等。

这里我们创建一个表，指定三个字段，数据类型分别为 `CHAR` ,`VARCHAR` 和 `TEXT` （关于创建表，插入，查询的操作在后面的内容中，我们会详细讲到）

```bash
# 这里我们创建了表 test1 ，有三个列，分别对应不同的格式，并且char 和 VARCHAR 指定长度为 5
mysql> CREATE TABLE test1(field1 CHAR(5), field2 VARCHAR(5), field3 TEXT);
Query OK, 0 rows affected (0.49 sec)

# 这时我们分别插入两条数据
mysql> INSERT INTO test1 VALUES("12345","12345","1234567890");
Query OK, 1 row affected, 2 warnings (1.52 sec)

mysql> INSERT INTO test1 VALUES("123","123","123");
Query OK, 1 row affected (0.52 sec)

打印表 test1 的数据
mysql> SELECT * FROM test1;
+--------+--------+------------+
| field1 | field2 | field3     |
+--------+--------+------------+
| 12345  | 12345  | 1234567890 |
| 123    | 123    | 123        |
+--------+--------+------------+
2 rows in set (0.00 sec)
```

```checker
- name: check sql
  script: |
    #!/bin/bash
	mysql -u root shiyanlou001 -e "show tables"|grep "test1"
  error: 没有在 shiyanlou001 创建 test1 表
- name: check sql
  script: |
    #!/bin/bash
	mysql -u root shiyanlou001 -e "select * from test1"|grep "12345"
  error: 没有在 test1 表插入数据
```

如上所示，我们设置长度为 `5`时，`CHAR` 和 `VARCHAR` 数据类型的多余内容将会被抛弃。

#### ENUM 和 SET

`ENUM` 我们一般称为枚举，`SET` 则被称为集合。两者都是从一个预先定义好的值列表中取值。除了列表中允许的取值之外，还包括 `NULL` 和空字符串 `''`。

下面我们简单描述两者的区别

+ 取值范围来说，`ENUM` 比 `SET` 大的多，`ENUM` 列理论上最多可以有 `65535` 个截然不同的值，但是实际上一般不到 `3000` 个，而 `SET` 只有 `64` 个。

+ 在一条记录中，`ENUM` 只能从中取一个值，但是 `SET` 可以有多个。因此，即使 `ENUM` 和 `SET` 的值都属于字符串，但是 `SET` 的成员不能包含逗号 (`,`) 字符。因为 `SET` 多个成员使用逗号 (`,`) 分隔。

如下示例，我们创建一张表

```bash
# 创建一个表 test2 包含有两列，分别为 ENUM 和 SET
mysql> CREATE TABLE test2(field1 ENUM("a", "b", "c"), field2 SET("001", "002", "003"));
Query OK, 0 rows affected (0.03 sec)

# 分别向表中插入数据
mysql> INSERT INTO test2 VALUES("a","001");
Query OK, 1 row affected (0.03 sec)

mysql> INSERT INTO test2 VALUES("b","002,003");
Query OK, 1 row affected (0.03 sec)

mysql> INSERT INTO test2 VALUES("shiyanlou","002,004");
Query OK, 1 row affected, 1 warning (0.02 sec)

# 查询表 test2
mysql> SELECT * FROM test2;
+--------+---------+
| field1 | field2  |
+--------+---------+
| a      | 001     |
| b      | 002,003 |
|        | 002     |
+--------+---------+
3 rows in set (0.00 sec)

mysql>
```
```checker
- name: check sql
  script: |
    #!/bin/bash
	mysql -u root shiyanlou001 -e "show tables"|grep "test2"
  error: 没有在 shiyanlou001 创建 test2 表
- name: check sql
  script: |
    #!/bin/bash
	mysql -u root shiyanlou001 -e "select * from test2"|grep "002"
  error: 没有在 test2 表插入数据
```

如上所示，插入不包括在变量中的值列表的值时，该值被设定为空字符串 `''`。并且 `set` 的多个成员之间使用逗号（`,`）分隔。

### 2.2 数字

#### 整型

整型即整数，可以分为有符号和无符号两类，无符号代表为非负的整数。

类型|存储（字节）|最小值（符号/无符号）|最大值（符号/无符号）
-----|-----|-----|---
TINYINT|1|-128/0|127/255
SMALLINT|2|-32768/0|32767/65535
MEDIUMINT|3|-8388608/0|8388607/16777215
INT/INTEGER|4|-214748363648/0|2147483647/4294967295
...|...|...|...

#### DECIMAL

DECIMAL 作为一种数字类型可以精确地划分整数和小数部分的位数。

```bash
DECIMAL(a,b)
a 指定整数部分和小数部分一共可以存储的十进制数字的最大位数，最大值为 65，但是会受限于操作系统等因素。
b 指定小数点后可以存储的十进制数字的最大位数。小数位数必须是从 0 到 a 之间的值。默认小数位数是 0。
如：123.45，则 a=5，b=2。
```

例如 `DECIMAL(3,2)`，取值范围为 `-9.99 ~ 9.99`。

#### FLOAT 和 DOUBLE

`FLOAT` 和 `DOUBLE` 分别使用 `4` 和 `8` 个字节进行存储。要知道的是，他们代表的是数字的近似值。

在计算机中浮点型数据在存储的时候，必须转化成二进制，而转换成二进制之后浮点数的存储主要分为：

- 符号位
- 指数位
- 尾数

`FLOAT` 使用 4 个字节存储，所以其有 32 位的空间，而 `DOUBLE` 使用 8 个字节存储，所以其有 64 位的空间，其存储空间的分配如下：

类型|符号位|指数位|尾数
--|--|--|--
float|1|8|23
double|1|11|52

具体的存储内容与使用大家可以参看推荐阅读中的 [FLOAT 与 DOUBLE](http://yongxiong.leanote.com/post/mysql_float_double_decimal)

简单来说三种浮点数类型的选用：

- FLOAT：用于较小的精度，容忍精度的丢失
- DOUBLE：主要用于工程项目，对进度要求更高的控制
- DECIMAL：主要用于科学的计算，与对精度更高要求的金融数据库

#### BOOL

`BOOL` 或者同义词 `BOOLEAN` 。在存储时表现为 `0` 或者 `1`。

`0` 值被视为假（`False`）。非 `0` 值视为真（`True`）。

### 2.3 日期和时间

#### 日期

`DATE` 支持的范围为 `'1000-01-01'到 '9999-12-31'`。`MySQL` 以 `YYYY-MM-DD` 格式显示 `DATE` 值，例如 `2018-01-01` 代表 `2018` 年 `1` 月 `1` 日。但允许使用字符串或数字类型数据为 `DATE` 列赋值（在下面的语言结构会演示相应的示例）。

#### 时间

`TIME` 时间。范围是 `-838:59:59 到 838:59:59`。`MySQL` 以 `HH:MM:SS` 格式或者 `HHH:MM:SS` 格式显示 `TIME` 值，但允许使用字符串或数字为 `TIME` 列分配值。

> `TIME` 的小时范围较大，因为其不仅可以表示 `24` 小时制的时间，还可以表示两个事件发生的时间间隔

#### DATETIME

日期和时间的组合。支持的范围是 `1000-01-01 00:00:00 到 9999-12-31 23:59:59`


## 3. 语言结构

在语言结构部分我们将简单示例一些数据在 `MySQL` 中的表现形式，以及使用方法。

> 语言结构跟数据类型的区别，类似于我们自己使用的 `SQL` 语句，与 `MySQL` 进行解析优化后的 `SQL` 语句

### 3.1 文字

文字部分我们将描述上述的数据结构在 `MySQL` 中的表现形式。

#### 字符串

`MySQL` 中字符串指使用单引号和双引号的字符序列。

在字符串中，部分序列具有特殊含义。

序列|值
--|--
\n|换行符
\r|回车符
\t|tab 字符
...|...

例如使用 `\t`：

```bash
mysql> SELECT "shiyan\tlou";
+------------+
| shiyan    lou |
+------------+
| shiyan    lou |
+------------+
1 row in set (0.00 sec)

mysql>
```

#### 数字

数值与我们平时使用的数字没有什么区别。`MySQL` 中可以使用科学计数法，如下示例：

```bash
mysql> SELECT true, false, TRUE, FALSE, 100, -100, 1.2e+3, 2.67;
+------+-------+------+-------+-----+------+--------+------+
| TRUE | FALSE | TRUE | FALSE | 100 | -100 | 1.2e+3 | 2.67 |
+------+-------+------+-------+-----+------+--------+------+
|    1 |     0 |    1 |     0 | 100 | -100 |   1200 | 2.67 |
+------+-------+------+-------+-----+------+--------+------+
1 row in set (0.00 sec)
```

#### 日期

时间和日期可以使用多种格式，MySQL 会将其进行解释，统一为一种格式，即日期为 `YYYY-MM-DD`，时间为 `HH:MM:SS`，如下示例。

```bash
mysql> SELECT DATE("11111111");
+------------------+
| DATE("11111111") |
+------------------+
| 1111-11-11       |
+------------------+
1 row in set (0.00 sec)

mysql> SELECT DATE("1111-11-11");
+--------------------+
| DATE("1111-11-11") |
+--------------------+
| 1111-11-11         |
+--------------------+
1 row in set (0.00 sec)

mysql> SELECT DATE("1111:11:11");
+--------------------+
| DATE("1111:11:11") |
+--------------------+
| 1111-11-11         |
+--------------------+
1 row in set (0.00 sec)

mysql>
```

#### 时间

如下示例

```bash
# HH:MM:SS
mysql> SELECT TIME("111111");
+----------------+
| TIME("111111") |
+----------------+
| 11:11:11       |
+----------------+
1 row in set (0.00 sec)

# HH:MM:SS
mysql> SELECT TIME("1111");
+--------------+
| TIME("1111") |
+--------------+
| 00:11:11     |
+--------------+
1 row in set (0.00 sec)

# HH:MM:SS
mysql> SELECT TIME("11");
+------------+
| TIME("11") |
+------------+
| 00:00:11   |
+------------+
1 row in set (0.00 sec)

# `D HH:MM:SS` 这里的 `D` 代表天数，将会被转换成 `HH:MM:SS`
mysql> SELECT TIME("1 11:");
+---------------+
| TIME("1 11:") |
+---------------+
| 35:00:00      |
+---------------+
1 row in set, 1 warning (0.00 sec)

mysql> SELECT TIME("1 11:11");
+-----------------+
| TIME("1 11:11") |
+-----------------+
| 35:11:00        |
+-----------------+
1 row in set (0.00 sec)

mysql>
```

#### 时间和日期

mysql 中时间和日期组合的格式为 `YYYY-MM-DD HH:MM:SS` 。

### 3.2 识别符

数据库名、表名、列名，及数据表，列的别名等被称作识别符。
反引号(`)一般用于与识别符与 MySQL 关键字冲突时使用。
我们查看创建语句时，会自动给数据库名和表名等添加反引号。

例如我们创建一个名为 `database` 的数据表，就可以使用反引号：

```bash
mysql> CREATE TABLE `database`(name CHAR);
Query OK, 0 rows affected (0.37 sec)
```

### 3.3 注释语句

MySQL 服务器支持 3 种注释风格：

以 `#` 字符开始到行尾。

以 `--` 符号序列开始到行尾。请注意 `--` (双破折号)注释风格要求第 2 个破折号后面至少跟一个空格符(例如空格、tab、换行符等等)。该语法与标准 `SQL`注释语法稍有不同。

以 `/*` 序列开始到 `*/` 序列结束。结束序列不一定在同一行中，因此该语句允许注释跨越多行。

下面的例子显示了3种风格的注释：

```bash
mysql> SELECT 1+1;     # shiyanlou
mysql> SELECT 1+1;     -- shiyanlou
mysql> SELECT 1 /* shiyanlou */ + 1;
mysql> SELECT 1+
/*
shiyan
lou
*/
1;
```


## 4. 总结

本节的内容，我们介绍了一些常用的数据类型，并没有对所有的内容都进行介绍。希望大家能够熟练使用一些常用的内容。
