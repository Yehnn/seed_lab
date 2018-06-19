---
show: step
version: 1.0
enable_checker: true
---
# MongoDB 导入导出

## 1. 实验介绍

#### 1.1 实验内容

在本节内容中，我们将学习 MongoDB 的一些用于备份的实用工具。

#### 1.2 实验知识点

+ 导入

+ 导出

+ 备份

+ 恢复

## 2. 导入导出

首先，我们将介绍两个用于 MongoDB 的使用工具：

+ `mongoexport` ：导出

+ `mongoimport` ：导入

### 2.1 导出

+ `mongoexport` 用于将 MongoDB 数据库中的数据导出为 `JSON` 或者 `CSV` 格式

我们以上一节基本操作中的 `users` 集合为例，首先，我们向其中插入几条数据：

```bash
> db.users.insertMany([{name: "Aiden", age: 30, email: "luojin@simplecloud.cn", addr: ["CD", "SH"]}, {name: "Tom", age: 20, email: "tom@simplecloud.cn", addr: ["BJ", "SH"]}, {name: "Jim", age: 20, email: "jim@simplecloud.cn", addr: ["BJ", "SH"]}, {name: "Cat", age: 10, addr: ["SZ", "CD"]}])
```

输出结果如下：

```bash
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("5a17e091bb4310a50c2010f3"),
        ObjectId("5a17e091bb4310a50c2010f4"),
        ObjectId("5a17e091bb4310a50c2010f5"),
        ObjectId("5a17e091bb4310a50c2010f6")
    ]
}
```

我们来查看集合中的文档：


```bash
> db.users.find()
```

输出结果如下：


```json
{ "_id" : ObjectId("5a17e091bb4310a50c2010f3"), "name" : "Aiden", "age" : 30, "email" : "luojin@simplecloud.cn", "addr" : [ "CD", "SH" ] }
{ "_id" : ObjectId("5a17e091bb4310a50c2010f4"), "name" : "Tom", "age" : 20, "email" : "tom@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a17e091bb4310a50c2010f5"), "name" : "Jim", "age" : 20, "email" : "jim@simplecloud.cn", "addr" : [ "BJ", "SH" ] }
{ "_id" : ObjectId("5a17e091bb4310a50c2010f6"), "name" : "Cat", "age" : 10, "addr" : [ "SZ", "CD" ] }
```

可以看到已经将数据插入进去了。

下面我们来使用 `mongoexport` 命令导出 `users` 集合中的数据，保存为 `users.json` ：


```bash
$ mongoexport --db shiyanlou001 --collection users --out users.json
```

> - `--db` 选项是指定数据库的名称，我们这里为 `shiyanlou001`
> - `--collection` 是指定集合的名称，我们这里为 `users`
> - `--out` 是指定导出的文件的文件名，我们这里为 `users.json` 

输出结果如下：

```
2017-11-24T17:27:17.989+0800    connected to: localhost
2017-11-24T17:27:17.989+0800    exported 4 records
```

> 左边的一列是导出的时间。
>
> 右边的一列中第一行是显示我们连接的 MongoDB 的服务器地址。第二行是导出的文档数为 4 条。

查看 `users.json` 中的数据，可以发现里面保存了 `users` 集合中的所有文档。

```checker
- name: check file
  script: |
    #!/bin/bash
	ls /home/shiyanlou/users.json
  error: /home/shiyanlou 目录下没有 users.json 文件
```

```bash
$ cat users.json
{"_id":{"$oid":"5a17e091bb4310a50c2010f3"},"name":"Aiden","age":30.0,"email":"luojin@simplecloud.cn","addr":["CD","SH"]}
{"_id":{"$oid":"5a17e091bb4310a50c2010f4"},"name":"Tom","age":20.0,"email":"tom@simplecloud.cn","addr":["BJ","SH"]}
{"_id":{"$oid":"5a17e091bb4310a50c2010f5"},"name":"Jim","age":20.0,"email":"jim@simplecloud.cn","addr":["BJ","SH"]}
{"_id":{"$oid":"5a17e091bb4310a50c2010f6"},"name":"Cat","age":10.0,"addr":["SZ","CD"]}
```

> **从上面的查看结果中可以发现原来的 `ObjectId` 显示成了 `$oid` ，这是为什么？**
>
> `BSON` 是 `MongoDB` 支持的数据类型，而 `JSON` 支持的内容仅仅是 `BSON` 的一个子集而已。因此，对于一些只在 `BSON` 中的数据类型，只有 `MongoDB` 能够解析。
>
> `MongoDB` 对导出的 `JSON` 格式有一些扩展内容，详细的内容可以参考官方文档[MongoDB JSON Extended](http://www.mongoing.com/docs/reference/mongodb-extended-json.html)

下面我们介绍 `mongoexport` 中的一些常用的命令行参数：

+ `--db <db_name>` 指定数据库名。

+ `--collection <coll_name>` 指定集合名。

+ `--type <json|csv>` 指定文件类型为 `csv` 或者 `json`。默认为 `json` 。

+ `--out <file>` 指定导出的文件名。如果没有指定，将输出到 `stdout` 。

+ `--pretty` 对于 `json` 类型而言，将输出一个更直观清晰的格式。

+ `--limit <number>` 指定导出的文档数目。

+ `--sort <JSON>` 指定导出结果的排序。比如：`--sort '{"age":1}'` 表示对 `age` 字段的值升序排列，如果想降序排列，可以把冒号后面改为 `-1` 。

+ `--query <JSON>` 提供一个 `json` 文档作为匹配条件，限制返回的结果集。

### 2.2 导入

与 `mongoexport` 对应的即为 `mongoimport`。用于导入由 `mongoexport` 创建的 `json` 和 `csv` 的内容。

因此，对于 `mongoimport` 来说，常见的参数与 `mongoexport` 类似。我们这里是从文件导入到 MongDB 的数据库中，而 `mongoimport` 的 `--db` 和 `--collection` 指的是导入的地方，所以会用到两个 `mongoexport` 中不存在的参数：

+ `--file` 指定将要导入的文件的文件名。未指定时，从标准输入导入。

+ `--drop` 在导入时，指定的集合存在时，使用该选项，将删除原来的内容

例如，我们导入之前导入的 `users.json` 文件：

```bash
$ mongoimport --db shiyanlou001 --collection users --file users.json --drop
2017-11-27T10:22:35.207+0800    connected to: localhost
2017-11-27T10:22:35.207+0800    dropping: shiyanlou001.users
2017-11-27T10:22:35.380+0800    imported 4 documents
```

```checker
- name: check mongo
  script: |
    #!/bin/bash
	mongo --eval "db=db.getSiblingDB('shiyanlou001');db.getCollectionNames()"|grep users
  error: 没有导入 users 集合
```

MongoDB 数据导出导入操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/13-1.mp4
@`

## 3. 备份和恢复

在前面的内容中，我们说过，`JSON` 支持的内容仅仅是 `BSON` 的一个子集而已。虽然 `MongoDB` 有对应的扩展，但是对于数据的备份，使用导入导出工具进行备份也是不合理的。在 `MongoDB` 中，有专门的备份工具。在接下来的内容中，我们将介绍他们。

### 3.1 备份

`mongodump` 能够把导出的数据库中的内容保存为一个二进制文件。不同于导入导出，这里针对的是数据库。你也可以通过配置项选择集合进行备份。

`mongodump` 仅备份数据库中的文档，不包含索引（ MongoDB 中默认索引为 `_id` ）数据，并且本地数据库 `local`  不会被备份。

我们可以直接使用 `mongodump` 进行备份操作：


```bash
$ mongodump
```
输出结果如下：

```bash
2017-11-27T10:45:00.904+0800    writing admin.system.version to 
2017-11-27T10:45:00.905+0800    done dumping admin.system.version (1 document)
2017-11-27T10:45:00.905+0800    writing shiyanlou001.users to 
2017-11-27T10:45:00.909+0800    done dumping shiyanlou001.users (4 documents)
```
查看 `dump` 的目录结构：

```bash
$ tree dump
dump
|--- admin
|     |--- system.version.bson
|     |--- system.version.metadata.json
|
|--- shiyanlou001
      |--- users.bson
      |--- users.metadata.json

2 directories, 4 files
```

```checker
- name: check dir
  script: |
    #!/bin/bash
	ls /home/shiyanlou/dump/shiyanlou001
  error: 没有备份成功
```

如上所示，并未备份 `local` 数据库（即本地数据库）中的内容，默认的 `IP` 是 `localhost` 和 `PORT` 是 `27017`，默认输出为当前目录下的 `dump` 目录 。

下面简单列举一些常见的参数：

+ `--host <hostname><:port>, -h <hostname><:port>`

    默认值: localhost:27017

+ `--port <port>`

    默认值: 27017

+ `--username <username>, -u <username>`

    指定用户名，需要与 `--password and --authenticationDatabase` 配置项一起使用

+ `--db <database>, -d <database>`

    指定数据库进行备份。未指定时会备份除本地数据库的所有数据库。

+ `--collection <collection>, -c <collection>`

    指定集合进行备份，未指定针对指定数据库中的所有集合

+ `--out <path>, -o <path>`

    指定保存目录。默认值为 `dump`

+ `--archive <file>`

    将备份内容写入到一个存档文件中，该选项可以用来代替 `--out` 选项，但不能在 `--out` 选项中使用 `--archive` 选项

### 3.2 恢复

`mongorestore` 可以从 `mongodump` 创建的备份中读取内容。对于服务器中存在的内容，`mongorestore` 不会覆盖原有的内容，如果数据库中集合的文档与要恢复的的文档有相同的值，原有文档不会被覆盖。

+ `<path>` 该参数是`mongorestore` 的最后一个参数，指定要从恢复的备份内容的位置

+ `--archive <file>` 从归档文件中恢复

+ `--dir` 与`<path>` 功能相同，两者不能同时使用

如下示例，恢复备份中的内容：

```bash
$ mongorestore --dir dump 
```
输出结果如下：

````bash
2017-11-27T13:09:50.431+0800    preparing collections to restore from
2017-11-27T13:09:50.639+0800    reading metadata for shiyanlou001.users from dump/shiyanlou001/users.metadata.json
2017-11-27T13:09:50.746+0800    restoring shiyanlou001.users from dump/shiyanlou001/users.bson
2017-11-27T13:09:50.916+0800    error: multiple errors in bulk operation:

  - E11000 duplicate key error collection: shiyanlou001.users index: _id_ dup key: { : ObjectId('5a17e091bb4310a50c2010f4') }
  - E11000 duplicate key error collection: shiyanlou001.users index: _id_ dup key: { : ObjectId('5a17e091bb4310a50c2010f5') }
  - E11000 duplicate key error collection: shiyanlou001.users index: _id_ dup key: { : ObjectId('5a17e091bb4310a50c2010f6') }
  - E11000 duplicate key error collection: shiyanlou001.users index: _id_ dup key: { : ObjectId('5a17e091bb4310a50c2010f3') }

2017-11-27T13:09:50.917+0800    no indexes to restore
2017-11-27T13:09:50.917+0800    finished restoring shiyanlou001.users (4 documents)
2017-11-27T13:09:50.917+0800    done
````

`done` 表示恢复完成，可以从上述的日志中发现哪些数据没有被恢复。

MongoDB 备份与恢复操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/13-2.mp4
@`

## 4. 总结

本节实验主要学习了如下内容：

- 集合的导入导出
- 数据库的备份和恢复