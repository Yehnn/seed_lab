---
show: step
version: 1.0
enable_checker: true
---
# SQL 高级特性

## 1. 实验介绍

### 1.1 实验内容

本节内容，我们将补充之前提到的约束的内容，并学习有关索引和触发器的知识。

### 1.2 实验知识点

+ 约束
+ 索引
+ 触发器

## 2. 约束

完整性约束（简称**约束**），英文名为（`CONSTRAINT`）。

对于约束的定义，在之前的内容中，我们是在列的定义中使用约束定义，这种方式称为列级完整性约束，对应的还有表级完整性约束，即在定义完所有的列后进行约束的定义，如之前选课数据库的 `sc` 表主键的定义使用的就是表级完整性约束。

表级完整性约束定义的语法如下：

```bash
[CONSTRAINT [symbol]] {PRIMARY KEY | UNIQUE | FOREIGN KEY}
```

`[CONSTRAINT [symbol]]` 是可选项，`symbol` 可以理解为约束的名字或者 `ID`，我们可以给定义的约束起一个名字为 `symbol`，方便后续的修改删除等操作都可以通过 `symbol` 来操作，如果该值被给与，它在当前数据库中都是唯一的。

> 上面提到的**主键**，**唯一**和**外键约束**等可以在列的定义中使用约束，但是经过 MySQL 的解析和优化处理后，仍然是表级完整性约束。（可参考之前表的解析与优化处理结果，及使用 `show create table` 语句）。而默认(`DEFAULT`)和非空约束(`NOT NULL`)，只能在列级进行定义。

在下面的内容中，我们将使用一些实际的示例，来带领大家理解和使用这些约束。

### 2.1 主键

对于多个字段共同构成主键来讲，使用列级完整性约束是行不通的，如下所示：

```bash
# 这里我们依然使用 `shiyanlou001` 数据库进行示例

# 如下所示，多个字段一起构成主键时，使用列级完整性约束定义失败
mysql> CREATE TABLE test1(field1 int PRIMARY KEY, field2 int PRIMARY KEY);
ERROR 1068 (42000): Multiple primary KEY defined

# 此时只能使用表级完整性约束，即上述的语法才能成功的定义
mysql> CREATE TABLE test1(field1 INT, field2 INT, CONSTRAINT primary_key_test1 PRIMARY KEY(field1,field2));
Query OK, 0 rows affected (0.41 sec)

# 查看表结构，可以看到列 field1 和 field2 一起作为表 test1 的主键
mysql> DESC test1;
+--------+---------+------+-----+---------+-------+
| Field  | Type    | Null | Key | Default | Extra |
+--------+---------+------+-----+---------+-------+
| field1 | int(11) | NO   | PRI | 0       |       |
| field2 | int(11) | NO   | PRI | 0       |       |
+--------+---------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

这时，我们可以向其中插入两条重复的数据：

```bash
# 第一次插入，未存在主键相同的数据，插入成功
mysql> INSERT INTO test1 VALUES(1,2);
Query OK, 1 row affected (0.31 se
c)

# 重复插入，插入失败
mysql> INSERT INTO test1 VALUES(1,2);
ERROR 1062 (23000): Duplicate entry '1-2' for KEY 'PRIMARY'
```

如上所示，主键不可重复。

而对于主键的删除，我们可以使用 `ALTER` 语句，对应前面在数据表的基本操作一节中的修改表的定义：

```bash
ALTER TABLE tbl_name DROP PRIMARY KEY
```

一个表只能有一个主键，所以删除时可以直接使用删除语句，只需要指定表名即可。如下所示，删除表 `test1` 的主键：

```bash
mysql> ALTER TABLE test1 DROP PRIMARY KEY;
Query OK, 0 rows affected (0.32 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC test1;
+--------+---------+------+-----+---------+-------+
| Field  | Type    | Null | Key | Default | Extra |
+--------+---------+------+-----+---------+-------+
| field1 | int(11) | NO   |     | 0       |       |
| field2 | int(11) | NO   |     | 0       |       |
+--------+---------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

在删除掉主键之后，还可以再一次的定义主键，语法如下：

```bash
ALTER TABLE tbl_name ADD [CONSTRAINT [symbol]] PRIMARY KEY
```

例如，这一次，我们以列 `field1` 创建主键，如下示例：

```bash
mysql> ALTER TABLE test1 ADD PRIMARY KEY(field1);
Query OK, 0 rows affected (0.30 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC test1;
+--------+---------+------+-----+---------+-------+
| Field  | Type    | Null | Key | Default | Extra |
+--------+---------+------+-----+---------+-------+
| field1 | int(11) | NO   | PRI | 0       |       |
| field2 | int(11) | NO   |     | 0       |       |
+--------+---------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

### 2.2 唯一

唯一约束，也可以称为唯一键，可以看作主键约束的补充内容，因为主键只能有一个，而唯一约束可以有多个。

对于唯一约束来讲，不同的地方在于，值可以为 `NULL`，并且可以有多个 `NULL` 值，除非对该列设定非空约束（当然，值为 `NULL` 是不推荐的）。它也可以多个字段共同构成一个唯一的约束。

唯一约束和主键约束的用法基本相同，将 `PRIMARY` 修改为 `UNIQUE`。

如下示例，我们创建一张表，有两个字段，分别设置两个唯一约束，并设置相应的 `symbol` 为 `unique_field1` 和 `unique_field2` 标识这两个唯一键：

```bash
# 创建表并设置唯一约束
mysql> CREATE TABLE test2(field1 INT, field2 INT, CONSTRAINT unique_field1 UNIQUE(field1),CONSTRAINT unique_field2 UNIQUE(field2));
Query OK, 0 rows affected (0.32 sec)

# 查看列
mysql> DESC test2;
+--------+---------+------+-----+---------+-------+
| Field  | Type    | Null | Key | Default | Extra |
+--------+---------+------+-----+---------+-------+
| field1 | int(11) | YES  | UNI | NULL    |       |
| field2 | int(11) | YES  | UNI | NULL    |       |
+--------+---------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

如上所示，我们可以简单的插入几行数据用作测试：

```bash
mysql> INSERT INTO test2 VALUES(1,1);
Query OK, 1 row affected (0.00 sec)

# 插入重复值时出错
mysql> INSERT INTO test2 VALUES(1,1);
ERROR 1062 (23000): Duplicate entry '1' for KEY 'unique_field1'

# 插入 NULL
mysql> INSERT INTO test2 VALUES();
Query OK, 1 row affected (0.00 sec)

# 插入 NULL
mysql> INSERT INTO test2 VALUES();
Query OK, 1 row affected (0.06 sec)

# 查询
mysql> SELECT * FROM test2;
+--------+--------+
| field1 | field2 |
+--------+--------+
|      1 |      1 |
|   NULL |   NULL |
|   NULL |   NULL |
+--------+--------+
3 rows in set (0.00 sec)
```

而对于设定的 `unique_field1` 和 `unique_field2` 的值来讲，如果要查看它们，这里我们需要用到一个新的语句：

```bash
SHOW INDEX FROM tbl_name
```

这里查看的是索引，关于索引和约束的关系在接下来的内容中我们会描述到，如下，查看 `test2` 表的索引：

```bash
mysql> SHOW INDEX FROM test2;
+-------+------------+---------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name      | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+---------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| test2 |          0 | unique_field1 |            1 | field1      | A         |           3 |     NULL | NULL   | YES  | BTREE      |         |               |
| test2 |          0 | unique_field2 |            1 | field2      | A         |           3 |     NULL | NULL   | YES  | BTREE      |         |               |
+-------+------------+---------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.00 sec)
```

如上所示，我们在 `Key_name` 一栏中可以看到 `unique_field1` 和 `unique_field2`。对于主键来讲，一张表中只能有一个主键，所以如果是主键的话，这一栏的数据将永远是 `PRIMARY` 而不是设置的 `symbol`。

这里对应的唯一约束的删除方式：

```bash
ALTER TABLE tbl_name DROP {INDEX|KEY} symbol
```

如下，我们删除表 `test2` 的约束：

```bash
# 通过设置的唯一标识键的 `symbol`，即刚刚设定的 `unique_field1`，则可成功删除
mysql> ALTER TABLE test2 DROP KEY unique_field1;
Query OK, 0 rows affected (0.33 sec)
Records: 0  Duplicates: 0  Warnings: 0

# 如下所示， field1 的唯一键已经被删除
mysql> DESC test2;
+--------+---------+------+-----+---------+-------+
| Field  | Type    | Null | Key | Default | Extra |
+--------+---------+------+-----+---------+-------+
| field1 | int(11) | YES  |     | NULL    |       |
| field2 | int(11) | YES  | UNI | NULL    |       |
+--------+---------+------+-----+---------+-------+
```

### 2.3 外键

对于外键的定义语法如下：

```bash
[CONSTRAINT [symbol]] FOREIGN KEY(index_col_name,...) reference_definition
```

这里 `reference_definition` 的格式如下所示：

```bash
REFERENCES tbl_name (index_col_name,...)

    [ON DELETE reference_option]
    [ON UPDATE reference_option]
```

参照数据搜索一节中，我们定义选课表 `sc` 的语句，这里我们省去了一些非必要项：

```bash
CREATE TABLE sc(
    s_id INT,
    c_id INT,
    grade INT,
    PRIMARY KEY (s_id, c_id),   # s_id, c_id 共同构成主键
    FOREIGN KEY (s_id) REFERENCES student(s_id), # 外键
    FOREIGN KEY (c_id) REFERENCES course(c_id)  # 外键
) default charset=utf8;
```

则必须的项为：

```bash
FOREIGN KEY(index_col_name,...) REFERENCES tbl_name (index_col_name,...)
```

关于外键的修改和删除，可以参考主键和唯一约束的操作。这里我们详细介绍一些非必选项。即上面提到的 `reference_option`，语法如下：

```bash
reference_option:
    RESTRICT | CASCADE | SET NULL | NO ACTION | SET DEFAULT
```

对于当前 MySQL 的默认引擎 `InnoDB` 来说，允许外键约束引用非主键或者非唯一键，但是在被引用的表中，必须存在主键或者唯一键。

一个表可以有多个外键，每个外键必须 REFERENCES (参考) 另一个表的列或者一组列。这是在之前我们对于外键的定义，这里的另一个表我们称为父表，而在对父表进行删除或者更新操作时，子表中的外键列为了保证参照的完整性，会有一些可选的操作项，如下：

+ `RESTRICT`
    拒绝父表的删除或者是更新操作，为默认选项。

+ `CASCADE`
    删除或更新父表中的行，并自动删除或更新子表中的匹配行。

+ `SET NULL`
    从父表中删除或更新行，并将子表中的外键列设置为NULL。这里需要确保字表中的外键列没有被设置为 `NOT NULL`。

+ `NO ACTION`
    与 `RESTRICT` 相同

> 例如，选课数据库中的 `sc` 表，其中的学号 `s_id` 必须要在学生表 `student` 中存在对应的值。而我们在对 `student` 表中的 `s_id` 进行操作时，因为 `sc` 表的 `s_id` 必须要与 `student` 中 `s_id` 一一对应。为了不破坏这种对应关系，会有相应的操作项，即上述列出的操作项。

这里，我们以选课数据库的三张表为例，如下，我们从学生表中删除 `s_id` 为 `1001` 的学生：

```bash
mysql> SELECT * FROM student;
+------+---------------+-------+-------+
| s_id | s_name        | s_sex | s_age |
+------+---------------+-------+-------+
| 1001 | shiyanlou1001 | man   |    10 |
| 1002 | shiyanlou1002 | woman |    20 |
| 1003 | shiyanlou1003 | man   |    18 |
| 1004 | shiyanlou1005 | woman |    40 |
| 1005 | shiyanlou1005 | man   |    17 |
+------+---------------+-------+-------+
5 rows in set (0.23 sec)

mysql> DELETE FROM student WHERE s_id=1001;
ERROR 1451 (23000): Cannot DELETE or update a parent row: a FOREIGN KEY CONSTRAINT fails (`shiyanlou001`.`sc`, CONSTRAINT `sc_ibfk_1` FOREIGN KEY (`s_id`) REFERENCES `student` (`s_id`))
mysql>
```

如上所示，该操作被拒绝执行，即上述可选项中提到的默认项 `RESTRICT`，拒绝此次操作。

下面我们修改定义，首先删除该外键，并且重新定义删除时的默认操作为 `CASCADE` ，并再次执行删除操作，如下：

```bash
# 首先，我们可以使用 `SHOW CREATE TABLE sc` 查看外键约束 `s_id` 自动生成的 symbol 为 sc_ibfk_1，然后将其删除
mysql> ALTER TABLE sc DROP FOREIGN KEY sc_ibfk_1;
Query OK, 7 rows affected (0.24 sec)
Records: 7  Duplicates: 0  Warnings: 0

# 添加外键定义，设置为 `CASCADE`
mysql> ALTER TABLE sc ADD CONSTRAINT sc_ibfk_1 FOREIGN KEY(s_id) REFERENCES student(s_id) ON DELETE CASCADE;
Query OK, 7 rows affected (0.08 sec)
Records: 7  Duplicates: 0  Warnings: 0

# 执行删除操作
mysql> DELETE FROM student WHERE s_id=1001;
Query OK, 1 row affected (0.01 sec)

# 这时，我们再次查看选课表 sc ，就会发现 s_id=1001 的记录都被删除了
mysql> SELECT * FROM sc;
+------+------+-------+
| s_id | c_id | grade |
+------+------+-------+
| 1002 |    1 |   100 |
| 1002 |    2 |    80 |
| 1002 |    4 |    80 |
| 1003 |    3 |    75 |
+------+------+-------+
4 rows in set (0.00 sec)
```

**除此之外，还可以设置为 `SET NULL` ，有兴趣的同学可以自行尝试。**

SQL 约束操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/8-1.mp4
@`

## 3. 索引

在无索引的表中，数据记录是无序的。如果我们需要查找一个匹配查找条件的行，基本上需要遍历表中的每一行，并给出所有符合条件的行。

因此，我们需要建立索引。索引用于快速查找具有特定列值的行。如果没有索引，MySQL 必须从第一行开始，然后通读整个表来查找相关的行。如果表中有相关​​列的索引，MySQL 可以快速确定记录在数据文件中间的位置，而无需查看所有数据。这比按顺序读取每一行要快得多。

在之前的内容中，我们介绍了一条语句为 `SHOW INDEX FROM tbl_name`。该语句用于查看表中的索引。

如下所示：

```bash
mysql> SHOW INDEX FROM sc\G
*************************** 1. row ***************************
        Table: sc
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: s_id
    Collation: A
  Cardinality: 4
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
*************************** 2. row ***************************
        Table: sc
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 2
  Column_name: c_id
    Collation: A
  Cardinality: 4
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
*************************** 3. row ***************************
        Table: sc
   Non_unique: 1
     Key_name: c_id
 Seq_in_index: 1
  Column_name: c_id
    Collation: A
  Cardinality: 4
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
3 rows in set (0.00 sec)

```

这里我们简单介绍索引中各列所代表的含义：

+ `Table`

    表名

+ `Non_unique`

    不重复为 0 ，可以重复为 1

+ `Key_name`

    索引的名称

+ `Seq_in_index`

    从该列建立索引，列的序号，从 1 开始。例如两列一起创建主键索引时，上述出现的值为 1，2。

+ `Column_name`

    列名

+ `Collation`

    `A` 代表升序

+ `Sub_part`

    列以什么方式存储在索引中。在 MySQL 中，有值 ‘A’（升序）或 NULL（无分类）。

+ `Null`

    如果列含有 NULL，则含有 YES。如果没有，则该列含有 NO。

### 3.1 创建索引

创建索引的语法格式如下：

```bash
CREATE [UNIQUE|FULLTEXT] INDEX index_name
    [index_type]
    ON tbl_name (index_col_name,...)
    [index_option]
```

上述中 `[UNIQUE|FULLTEXT|SPATIAL]` 分别代表使用 `CREATE INDEX` 方法可以创建的索引类型，但 MySQL 中索引不仅是上面所列出的三种。

下面我们简单介绍索引的分类：

+ `INDEX`

    普通索引

+ `UNIQUE`

    索引列中的值都只能出现一次

+ `PRIMARY KEY`

    主键索引，跟 `UNIQUE` 不同的是，一张表中只能有一个主键索引

+ `FULLTEXT`

    全文索引，只能用于 `VARCHAR` 和 `TEXT` 数据类型。（但是在当前实验环境中的 MySQL 5.5 版本中的 `InnoDB` 引擎并不支持，在 5.6.4 版本中才添加的相应的支持）

在索引的分类中，我们看到了 `UNIQUE` 和 `PRIMARY KEY`。

这里需要与主键约束和唯一性约束进行区分的是，我们在定义主键的时候，会自动创建 `PRIMARY` 索引，同样，在定义唯一约束的时候，也会创建 `UNIQUE` 索引，反过来，定义 `UNIQUE` 索引的时候，也会建立唯一约束。

但是对于 `PRIMARY KEY` 索引而言，不能使用 `CREATE INDEX` 的方式创建，也不能使用 `ALTER INDEX` 的方式删除索引。只能通过上述所讲的约束的方式去创建它。

对于 `index_type` 而言，使用如下格式：

```bash
index_type:
    USING {BTREE | HASH}
```

`BTREE` 和 `HASH` 的实现方式在这里不做讨论，了解有这两种类型即可。默认的为 `BTREE`。

例如，我们创建一个表 `test_index`，有几个字段，并在它的基础上创建相应的索引，如下所示：

```bash
# 创建表 test_index，有两个字段
mysql> CREATE TABLE test_index(field1 INT, field2 INT);
Query OK, 0 rows affected (0.05 sec)

# 在字段 field1 上创建唯一索引，名字为 unique_field1
mysql> CREATE UNIQUE INDEX unique_field1 USING BTREE ON test_index(field1);
Query OK, 0 rows affected (0.07 sec)
Records: 0  Duplicates: 0  Warnings: 0

# 在字段 field2 上创建普通索引，省略索引类型的描述，使用默认值 BTREE
mysql> CREATE INDEX index_field2 ON test_index(field2);
Query OK, 0 rows affected (0.09 sec)
Records: 0  Duplicates: 0  Warnings: 0

# 创建成功后，就可以查看相应的索引值了
mysql> SHOW INDEX FROM test_index\G
*************************** 1. row ***************************
        Table: test_index
   Non_unique: 0
     Key_name: unique_field1
 Seq_in_index: 1
  Column_name: field1
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE
      Comment: 
Index_comment: 
*************************** 2. row ***************************
        Table: test_index
   Non_unique: 1
     Key_name: index_field2
 Seq_in_index: 1
  Column_name: field2
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE
      Comment: 
Index_comment: 
2 rows in set (0.00 sec)

```

### 3.2 删除索引

对于索引的删除，我们可以使用如下语法：

```
DROP INDEX index_name ON tbl_name
```

或者使用修改表的语法去删除索引：

```
ALTER TABLE tbl_name

    DROP {INDEX|KEY} index_name
    | DROP PRIMARY KEY
    | DROP FOREIGN KEY fk_symbol
```

例如删除刚刚在表 `test_index` 中创建的索引，分别使用上述的两种方式：

```bash
mysql> DROP INDEX unique_field1 ON test_index;
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> ALTER TABLE test_index DROP INDEX index_field2;
Query OK, 0 rows affected (0.05 sec)
Records: 0  Duplicates: 0  Warnings: 0

```

索引操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/8-2.mp4
@`

## 4. 触发器

触发器 (`TRIGGER`) 是一个与数据表关联的已命名的数据库对象，当数据表发生特定事件时会激活该对象，并执行触发语句。

### 4.1 概述

#### 触发事件

触发器会在发生特定事件时被自动激活，而对于触发事件(`trigger_event`)，在 MySQL 中为三种事件：

+ `INSERT`

+ `UPDATE`

+ `DELETE`

#### 触发主体

对于触发器，最主要的就是触发时将会执行的操作，即(`trigger_body`)。

#### 触发时间

对于三种事件，即插入，更新，删除操作。触发器可以在事件发生之前(`BEFORE`)或者之后(`AFTER`)执行，被称为触发时间 (`trigger_time`)。

#### 触发顺序

在最新的 MySQL 中，你可能会定义多个触发器，他们具有相似的触发事件和触发时间（即之前或之后）。而对于多个触发器的执行顺序，可以通过设定触发顺序(`trigger_order`)来指定。

分别有两种顺序执行方式，第一个为 `FOLLOWS`，即最新定义的触发器在之前定义的后面执行。另一种为 `PERCEDES`，即在之前执行。

### 4.1 创建触发器

创建触发器的 `SQL` 语句如下：

```
CREATE

    TRIGGER trigger_name
    trigger_time trigger_event
    ON tbl_name FOR EACH ROW
    [trigger_order]
    trigger_body

trigger_time: { BEFORE | AFTER }

trigger_event: { INSERT | UPDATE | DELETE }

trigger_order: { FOLLOWS | PRECEDES } other_trigger_name
```

对于 SQL 语句中出现的名词，这里我们补充说明 `trigger_body`。

在 `trigger_body` 中，我们可以通过 `OLD` 或者 `NEW` 来使用当前触发器对象所关联的数据表的列。

使用方式为 `OLD.col_name` 或者 `NEW.col_name`。这里的 `col_name` 为列名。需要注意的是，`OLD.col_name` 是在删除和更新操作之前就已经存在的字段。而 `NEW.col_name` 是一个新插入的或者被更新的内容。

下面我们尝试创建一张数据表，并且定义一个触发器：

```bash
mysql> CREATE TABLE test_trigger(field1 int, field2 int);
Query OK, 0 rows affected (0.06 sec)

# 创建一个触发器，在插入之前，将第一列的值 +1
mysql> CREATE TRIGGER before_insert BEFORE INSERT ON test_trigger FOR EACH ROW SET new.field1=new.field1+1;
Query OK, 0 rows affected (0.03 sec)

# 插入一行
mysql> INSERT INTO test_trigger VALUES(1,1);
Query OK, 1 row affected (0.02 sec)

# 如下所示，我们可以看到 field1 的值为 2 = 1+1
mysql> SELECT * FROM test_trigger;
+--------+--------+
| field1 | field2 |
+--------+--------+
|      2 |      1 |
+--------+--------+
1 row in set (0.00 sec)
```

### 4.2 查看

我们可以通过如下语句，查看当前数据库中定义的触发器：

```bash
SHOW TRIGGERS
    [{FROM | IN} db_name]
    [LIKE 'pattern' | WHERE expr]
```

`LIKE` 可以匹配表名，而不是触发器名。我们可以不指定数据库名，使用默认的数据库，如下示例：

```bash
mysql> SHOW TRIGGERS\G
*************************** 1. row ***************************
             Trigger: before_insert
               Event: INSERT
               Table: test_trigger
           Statement: SET new.field1=new.field1+1
              Timing: BEFORE
             Created: NULL
            sql_mode:
             Definer: root@localhost
character_set_client: utf8
collation_connection: utf8_general_ci
  Database Collation: utf8_general_ci
1 row in set (0.00 sec)

```

### 4.3 删除

在创建了触发器之后，如果需要删除触发器，可以使用如下语法：

```bash
DROP TRIGGER trigger_name
```

例如，我们删除上面定义的触发器，

```bash
mysql> DROP TRIGGER before_insert;
Query OK, 0 rows affected (0.00 sec)
```

## 5. 总结

本节实验我们学习了约束、索引和触发器的概念及它们的相关操作。