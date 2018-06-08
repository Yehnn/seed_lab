---
show: step
version: 1.0
enable_checker: true
---
# MongoDB 基本操作

## 1. 实验介绍

### 1.1 实验内容

在本节内容中，我们将介绍 `MongoDB` 的一些基本操作，比如数据库的创建，查看等，集合的创建，查看和删除等，文档的增删改查。

### 1.2 实验知识点

+ 数据库的操作
+ 集合的操作
+ 文档的增删改查

## 2. 数据库

### 2.1 查看

我们可以使用 `db` 查看当前使用的数据库：

```bash
> db
test
```

此操作返回默认数据库 `test` 。

查看 `MongoDB` 中所有可用的数据库，我们可以使用 `show dbs` 命令,该命令是 `show databases` 的缩写，会显示已有的数据库名和使用的大小

```bash
> show dbs
admin         0.000GB
local         0.000GB
```

> 左边一列是数据库名，右边一列是数据库所占空间大小。
>
> - **admin**： 从权限的角度来看，这是 "root" 数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。
> - **local:** 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合。

### 2.2 选择

在 `MongoDB` 中，数据库由文档构成的集合组成。

选择一个数据库去使用，需要如下语法：

```bash
use <db>
```

`<db>` 指的是数据库的名字。

你可以切换到一个不存在的数据库，比如说我们切换到名为 `shiyanlou001` 这个数据库。

```
> use shiyanlou001
switched to db shiyanlou001
> db
shiyanlou001
```

### 2.3 创建

在我们刚刚使用 `use` 切换数据库的时候，还并未创建该数据库，而在我们往该数据库中插入第一条数据时（比如创建集合或者插入文档），系统会自动创建该数据库。如下所示我们插入第一条文档。

```
> db.users.insertOne({name: "Aiden", age: 30, email: "luojin@simplecloud.cn", addr: ["CD", "SH"]})
```

输出结果如下：

```
{
    "acknowledged" : true,
    "insertedId" : ObjectId("5a162f95c63ea7db33dba26d")
}
```

> `acknowledged` ：执行命令返回的状态，为 `true` 说明执行成功。
>
> `insertedId` ：在 MongoDB 中，存储于集合中的每一个文档都需要一个唯一的 `_id` 字段作为 `primary_key`。如果一个插入文档操作遗漏了``_id``  字段，`MongoDB` 会自动为``_id`` 字段生成一个 `ObjectId` 。

现在你使用 `show dbs` 去查看有哪些数据库，会发现多了一个名叫 `shiyanlou001` 的数据库。

## 3. 集合

### 3.1 创建

`MongoDB` 可以创建普通集合。比如我们来创建一个名为 `shiyanlou_col1` 的普通集合：

```
> db.createCollection("shiyanlou_col1")
{ "ok" : 1 }
```

> 提示信息 `{ "ok" : 1 }` 显示我们创建成功

MongoDB 除了可以创建普通的集合外，还可以创建一种特殊的集合，即固定集合，可以指定集合的大小，当集合容量已满的时候，我们再向集合添加文档，会覆盖之前原有的内容，以保持固定大小。

创建的语法格式如下：

```bash
db.createCollection(<name>, { capped: <boolean>,
                              autoIndexId: <boolean>,
                              size: <number>,
                              max: <number>,
                              ... } )
```

`<name>` 代表集合的名字，`{...}` 花括号中为可配置项，这里我们列举了一些常见的配置项，配置项的含义如下：

+ `capped`

    可以配置 `true` 或 `false`，默认值为 `false` 。指定为 `true` 的时候，即为固定集合，此时需要指定 `size`

+ `autoIndexId`

    可以配置 `true` 或 `false`，默认值为 `true`。每一个文档都有一个 _id 字段，该字段是主键，用于唯一的确定一条记录，如果往 MongoDB 中插入数据时没有指定 `_id` 字段，那么会自动产生一个 `_id` 字段。这里配置项指定为 `false` 时，不会自动生成该字段。

+ `size`

    对于固定集合时指定可以集合的大小，单位为 `byte`

+ `max`

    指定固定集合运行的文档数目

例如，我们创建 一个固定集合 `shiyanlou_col2` ，指定固定大小为 1024 byte :

```bash
> db.createCollection("shiyanlou_col2", {capped:true, size:1024})
{ "ok" : 1 }
```

### 3.2 查看

我们可以通过 `show collections` 来查看数据库中的集合

```bash
> show collections
shiyanlou_col1
shiyanlou_col2
users
```

在上面的结果中，多了一个  `users` 。`users` 是在 2.3 中执行的插入语句创建的，**对于不存在的集合，在插入数据时，会创建对应的集合**。

### 3.3 删除

删除集合使用 `db.collection.drop()` ，这里的 `collection` 为集合的名字，如下示例：

```bash
> db.shiyanlou_col2.drop()
true
> show collections
shiyanlou_col1
users
> db.shiyanlou_col1.drop()
true
> show collections
users
```

## 4. 文档

### 4.1 插入

MongoDB 存储的文档记录是一个 `BSON` 对象，类似于 `JSON` 对象，由键值对组成。比如这样一条用户记录：

```txt
db.users.insertOne(
    {
        name: "Aiden",
        age: 30,
        email: "luojin@simplecloud.cn"
    }
)
```

在上述示例中，`users` 是一个集合，而 `{...}` 花括号中的内容为文档内容。关于 `BSON` 支持的数据类型，可以参考 `MongoDB` 的官方链接：[bson-types](https://docs.mongodb.com/manual/reference/bson-types)

对于文档的插入操作，一般有两种常用的插入方式：

```txt
db.collection.insertOne(<document>)
db.collection.insertMany([ <document 1> , <document 2>, ... ])
```

> `collection` 为集合名称。`<document>` 为要插入的文档内容。

对于上述的两种操作，我们可以很容易从字面上理解它们的区别，即插入一条和插入多条，如下的简单示例：

```bash
# 使用 `insertOne()` 操作
> db.users.insertOne({name: "Tom", age: 20, email: "tom@simplecloud.cn", addr: ["BJ", "SH"]})
{
    "acknowledged" : true,
    "insertedId" : ObjectId("5a163138c63ea7db33dba270")
}

# 使用 `insertMany()` 操作
> db.users.insertMany([{name: "Jim", age: 20, email: "jim@simplecloud.cn", addr: ["BJ", "SH"]}, {name: "Cat", age: 10, addr: ["SZ", "CD"]}])
{
    "acknowledged" : true,
    "insertedIds" : [
    ObjectId("5a163208c63ea7db33dba272"),
    ObjectId("5a163208c63ea7db33dba273")
    ]
}
```

### 4.2 查询

我们查看一个集合中的文档，可以使用如下语法：

```bash
db.collection.find()
# collection 为集合名称
```

例如，我们查看 `users` 的文档：

```bash
> db.users.find()
```

输出结果如下：

```json
{ "_id" : ObjectId("5a162f95c63ea7db33dba26d"), "name" : "Aiden", "age" : 30, "email" : "luojin@simplecloud.cn", "addr" : [ "CD", "SH" ] }
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "age" : 20, "email" : "tom@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba272"), "name" : "Jim", "age" : 20, "email" : "jim@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba273"), "name" : "Cat", "age" : 10, "addr" : [ "SZ", "CD" ] }
```

除此之外，我们还可以指定过滤条件：

```bash
# 查询姓名为 "Tom" 的文档
> db.users.find({"name":"Tom"})
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "age" : 20, "email" : "tom@simplecloud.cn", "addr" : [ "BJ", "SH" ] }

# 或者使用 `pretty` 使得格式更明了
> db.users.find({"name":"Tom"}).pretty()
{
    "_id" : ObjectId("5a163138c63ea7db33dba270"),
    "name" : "Tom",
    "age" : 20,
    "email" : "tom@simplecloud.cn",
    "addr" : [
        "BJ",
        "SH"
    ]
}
```

指定查询的过滤条件是通过在 `find()` 中使用相应的表达式：

```txt
db.users.find({ <field1>: <value1>, ... })
```

例如命令 `db.users.find({"name":"Tom"})` ，以我们更熟悉的 SQL 操作表示如下：

```mysql
SELECT * FROM users WHERE name="Tom"
```

这种方式仅仅对于使用等号(`=`) 而言，对于一些其它的操作符，在 MongoDB 中也有对应的实现，如下：

| MongoDB | SQL      |
| ------- | -------- |
| `$eq`   | `=`      |
| `$gt`   | `>`      |
| `$gte`  | `>=`     |
| `$lt`   | `<`      |
| `$lte`  | `<=`     |
| `$ne`   | `!=`     |
| `$in`   | `in`     |
| `$nin`  | `not in` |
| `$and`  | `and`    |
| `$or`   | `or`     |
| `$not`  | `not`    |
| ...     | ...      |

还有一个较为特殊的用法，为 `$exists`，判断是否存在。例如是否存在某个字段。

如上所示，我们列举了部分常见的运算符。

- **对于数值判断，大于，小于等使用操作：**

```bash
# 查看年龄 > 20 的文档
> db.users.find({"age": { $gt: 20}})
{ "_id" : ObjectId("5a162f95c63ea7db33dba26d"), "name" : "Aiden", "age" : 30, "email" : "luojin@simplecloud.cn", "addr" : [ "CD", "SH" ] }

# 查看年龄 <= 20 的文档
> db.users.find({"age": {$lte:20}})
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "age" : 20, "email" : "tom@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba272"), "name" : "Jim", "age" : 20, "email" : "jim@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba273"), "name" : "Cat", "age" : 10, "addr" : [ "SZ", "CD" ] }
```

- **使用 `$exists` 和 `$in` :**

```bash
# 查看不存在 email 字段的文档
> db.users.find({"email":{$exists: false}})
{ "_id" : ObjectId("5a571c8839c3d2e572244f63"), "name" : "Cat", "age" : 10, "addr" : [ "SZ", "CD" ] }

# 查看存在 email 字段的文档
> db.users.find({"email": {$exists: true}})
{ "_id" : ObjectId("5a162f95c63ea7db33dba26d"), "name" : "Aiden", "age" : 30, "email" : "luojin@simplecloud.cn", "addr" : [ "CD", "SH" ] }
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "age" : 20, "email" : "tom@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba272"), "name" : "Jim", "age" : 20, "email" : "jim@simplecloud.cn", "addr" : [ "BJ", "SH" ] }

# 查看年龄范围在 10 到 30 的文档
> db.users.find({"age": {$in: [10,30]}})
{ "_id" : ObjectId("5a162f95c63ea7db33dba26d"), "name" : "Aiden", "age" : 30, "email" : "luojin@simplecloud.cn", "addr" : [ "CD", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba273"), "name" : "Cat", "age" : 10, "addr" : [ "SZ", "CD" ] }
```

- **逻辑运算符 `$and`， `$or` ， `$not` 的使用：**

```bash
# and 一般为隐式的使用，如下，查找年龄为 20，并且姓名为 Tom 的文档
> db.users.find({"age": 20, "name": "Tom"})
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "age" : 20, "email" : "tom@simplecloud.cn", "addr" : [ "BJ", "SH" ] }

# 查看年龄 > 20 或 不存在 email 字段的文档
> db.users.find({$or: [{"age": {$gt: 20}}, {"email": {$exists: false}}]})
{ "_id" : ObjectId("5a162f95c63ea7db33dba26d"), "name" : "Aiden", "age" : 30, "email" : "luojin@simplecloud.cn", "addr" : [ "CD", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba273"), "name" : "Cat", "age" : 10, "addr" : [ "SZ", "CD" ] }

# not 和 or 的使用方式类似，如下，查找年龄不为 20 的文档
> db.users.find({"age": {$not: {$eq: 20}}})
{ "_id" : ObjectId("5a162f95c63ea7db33dba26d"), "name" : "Aiden", "age" : 30, "email" : "luojin@simplecloud.cn", "addr" : [ "CD", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba273"), "name" : "Cat", "age" : 10, "addr" : [ "SZ", "CD" ] }
```

另外，还有对字段为一个数组的查询操作：

+ `$all` 用来匹配数组中包含所有的指定元素。

```bash
# 查找 users 中 addr 包含 `BJ` 的文档
> db.users.find({"addr": {$all: ["BJ"]}})
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "age" : 20, "email" : "tom@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba272"), "name" : "Jim", "age" : 20, "email" : "jim@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
```

+ `$elemMatch` 查询数组中至少一个元素满足所有指定条件的文档。对于使用其使用方式，我们给出一个官方的示例：（以下只是示例，不用输入。）

```bash
# 如下所示的集合记录
{ _id: 1, results: [ 82, 85, 88 ] }
{ _id: 2, results: [ 75, 88, 89 ] }

# 使用 $elemMatch 进行查询。
db.collection.find(
    { results: { $elemMatch: { $gte: 80, $lt: 85 } } }
)

# 结果如下
{ "_id" : 1, "results" : [ 82, 85, 88 ] }
```

+ `$size` 匹配数组的大小，这里指的是数组长度。

```bash
# 查找 users 中 addr 数组长度为 2 的文档
> db.users.find({"addr": {$size: 2}})
{ "_id" : ObjectId("5a162f95c63ea7db33dba26d"), "name" : "Aiden", "age" : 30, "email" : "luojin@simplecloud.cn", "addr" : [ "CD", "SH" ] }
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "age" : 20, "email" : "tom@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba272"), "name" : "Jim", "age" : 20, "email" : "jim@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba273"), "name" : "Cat", "age" : 10, "addr" : [ "SZ", "CD" ] }
```

### 4.3 更新

对于文档的更新，与查询类似，使用如下语法：

```bash
db.collection.updateOne()     #更新一个文档
db.collection.updateMany()    #更新多个文档
db.collection.replaceOne()    #替换一个文档
```

对于更新操作，我们需要指定过滤条件，并且指定更新操作，如下：

```bash
db.collection.updateMany(
   <filter>,     #过滤条件
   <update>,     #更新操作
   ...
}
```

上面的省略号是部分配置操作，一般不常用，而对于具体的更新操作，即 `<update>`，有三种：

+ `$set` 修改字段的值，或者添加新的字段

```bash
# 修改姓名为 Tom 的 age 值为 18
> db.users.updateOne({"name": "Tom"}, {$set: {"age": 18}})
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }

> db.users.find({"name": "Tom"})
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "age" : 18, "email" : "tom@simplecloud.cn", "addr" : [ "BJ", "SH" ] }


# 修改所有的 age 字段值为 22，可以使用 {} 即不使用过滤条件
> db.users.updateMany({}, {$set: {"age": 22}})
{ "acknowledged" : true, "matchedCount" : 4, "modifiedCount" : 4 }

> db.users.find()
{ "_id" : ObjectId("5a162f95c63ea7db33dba26d"), "name" : "Aiden", "age" : 22, "email" : "luojin@simplecloud.cn", "addr" : [ "CD", "SH" ] }
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "age" : 22, "email" : "tom@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba272"), "name" : "Jim", "age" : 22, "email" : "jim@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba273"), "name" : "Cat", "age" : 22, "addr" : [ "SZ", "CD" ] }
```

+ `$unset` 删除字段

```bash
# 删除 Tom 的 email 字段，下述示例中使用的 "" 不影响操作
> db.users.updateOne({"name": "Tom"}, {$unset: {"email": ""}})
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }

> db.users.find({"name": "Tom"})
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "age" : 22, "addr" : [ "BJ", "SH" ] }
```

+ `$rename` 重命名字段

```bash
# 修改所有的 age 字段名为 ages
> db.users.updateMany({"age": {$exists: true}}, {$rename: {"age": "ages"}})
{ "acknowledged" : true, "matchedCount" : 4, "modifiedCount" : 4 }

> db.users.find()
{ "_id" : ObjectId("5a162f95c63ea7db33dba26d"), "name" : "Aiden", "email" : "luojin@simplecloud.cn", "addr" : [ "CD", "SH" ], "ages" : 22 }
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "addr" : [ "BJ", "SH" ], "ages" : 22 }
{ "_id" : ObjectId("5a163208c63ea7db33dba272"), "name" : "Jim", "email" : "jim@simplecloud.cn", "addr" : [ "BJ", "SH" ], "ages" : 22 }
{ "_id" : ObjectId("5a163208c63ea7db33dba273"), "name" : "Cat", "addr" : [ "SZ", "CD" ], "ages" : 22 }
```

除了可以更新文档，还可以使用替代文档，例如使用新数据替换姓名为 `Tom` 的文档。

```bash
> db.users.replaceOne({"name": "Tom"}, {"name": "Tom", "age": 70, "company": "shiyanlou", "addr": ["CD"]})
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }

> db.users.find({"name": "Tom"})
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "ages" : 70, "company" : "shiyanlou", "addr" : [ "CD" ] }
```

### 4.4 删除

MongoDB 提供了如下删除文档的方法：

```bash
db.collection.deleteOne() 
db.collection.deleteMany()
```

简单示例如下：

```bash
# 删除 users 集合中姓名为 Jim 的文档
> db.users.deleteOne({"name": "Jim"})
{ "acknowledged" : true, "deletedCount" : 1 }
> db.users.find()
{ "_id" : ObjectId("5a162f95c63ea7db33dba26d"), "name" : "Aiden", "email" : "luojin@simplecloud.cn", "addr" : [ "CD", "SH" ], "ages" : 22 }
{ "_id" : ObjectId("5a163138c63ea7db33dba270"), "name" : "Tom", "ages" : 70, "company" : "shiyanlou", "addr" : [ "CD" ] }
{ "_id" : ObjectId("5a163208c63ea7db33dba273"), "name" : "Cat", "addr" : [ "SZ", "CD" ], "ages" : 22 }

# 删除 users 集合中所有的记录，{} 不指定过滤条件，为集合中所有文档
> db.users.deleteMany({})
{ "acknowledged" : true, "deletedCount" : 3 }
> db.users.find()
>
```

MongoDB 数据库、集合、文档基本操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/12-1.mp4
@`

## 5. 总结

在本节内容中，我们主要介绍了如下的相关操作：

+ 数据库
    - 查看
    - 选择
    - 创建

+ 集合
    - 创建
    - 查看
    - 删除

+ 文档
    - 插入
    - 查询
    - 更新
    - 删除