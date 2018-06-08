---
show: step
version: 1.0
enable_checker: true
---
# Redis 数据类型

## 1. 实验介绍

### 1.1 实验内容

在本节内容中，我们将介绍 Redis 的数据类型，及其一些相关的命令使用。而对于单独的键 `key` 的操作，将在学习数据类型的时候了解到相关的内容，不会单独提出。

Redis 不仅是一个 `key-value` 的存储系统，也是一款数据结构服务器（data structure server）。Redis 的键值可以包括字符串（`strings`）类型，同时它还包括哈希（`hashes`）、列表（`lists`）、集合（`sets`）和有序集合（`sorted sets`）等数据类型。

### 1.2 实验知识点

+ Redis 数据类型

+ string

+ list

+ hash

+ sets

+ sorted sets

## 2. string

字符串是一种最基本的 Redis 值类型。Redis 字符串是二进制安全的，这意味着一个 Redis 字符串能包含任意类型的数据，例如： 一张 JPEG 格式的图片或者一个序列化的 Ruby 对象。一个字符串类型的值最多能存储 512M 字节的内容。

字符串是 Redis 中最基本的数据类型。对于字符串而言，设置及获取一个 `key` 的值的语法格式为：

```bash
SET key value
GET key
```

对于上述语法中的 `SET` 和 `GET` 来说，也是不区分大小写的，在后面的内容中，将不再说明。

首先，我们打开 redis-cli ，`SET` 的语法格式如下：

```bash
SET key value [EX seconds] [PX milliseconds] [NX|XX]
```

对于 `SET` 来说，该命令就是将键 `key` 设定为指定的字符串值 `value`，而如果 `key` 已经保存了一个值，该操作会直接覆盖原来的值，并且忽略原始的数据类型。如下所示：

```bash
127.0.0.1:6379> SET shiyanlou001 001
OK
127.0.0.1:6379> GET shiyanlou001
"001"
```

对应 `SET` 的是 `GET`，使用 `GET key`，获得 `key` 的值。

我们可以使用下列语法查看键的存储类型：

```bash
TYPE key
```

如下所示，显示为 `string`:

```bash
127.0.0.1:6379> TYPE shiyanlou001
string
```

在上面的 `SET` 命令中，还有三个可选项，代表的意思如下：

+ `EX` 设置键的过期时间，单位为秒

+ `PX` 设置键的过期时间，单位为毫秒

+ `NX` 在不存在该键的时候，才会设置值

+ `XX` 只有该键存在的时候，才会设置值

过期时间是指 `key` 的有效时间或者说是生存时间，从设定开始，超过该时间后，`key` 就会自动删除。对于上述的可选项来说，在 `redis-cli` 中都有对应的命令，例如 `SETNX`,`SETEX` 等，但是 `SET` 也可以实现同样的功能，这里我们不再描述。

如下示例，我们设置一个有过期时间的键：

```bash
127.0.0.1:6379> SET test_key test_value EX 30
OK
127.0.0.1:6379> GET test_key
"test_value"
```

即在 `30` 秒后，`test_key` 将会失效，对于查看键的过期时间，我们可以使用如下语法：

```bash
TTL key
```

如下所示，查看 `test_key` 的过期时间（由于操作时间的不同，看到的值可能不同，这里我们显示的是还有 12 秒过期）：

```bash
127.0.0.1:6379> TTL test_key
(integer) 12
```

关于 `TTL` ，如果是未设置过期时间，返回 `-1`，代表没有过期时间，`-2` 则代表该键已过期或者不存在。

```bash
# 等过了 30 秒以后再执行就会返回已过期
127.0.0.1:6379> TTL test_key
(integer) -2

# shiyanlou001 未设置就返回 -1
127.0.0.1:6379> TTL shiyanlou001
(integer) -1
```

对于字符串而言，还有一些其它的常用命令，简单列举如下：

```bash
# 使用 `MSET` 设置多个键的值
MSET key value [key value ...]

# 使用 `MGET` 获取多个键的值，对于不存在的键会返回特殊值 nil
MGET key [key ...]

# 使用 `STRLEN` 获取指定键的长度
STRLEN key

# 使用 `DECR` 将整数减 1
DECR key

# 使用 `INCR` 将整数加 1
INCR key

# 使用 `DECRBY` 将整数减去指定的值
DECRBY key decrement

# 使用 `INCRBY` 将整数加上指定的值
INCRBY key increment

# 使用 `APPEND` 追加一个值
APPEND key value
```

如下简单示例：

```bash
127.0.0.1:6379> MSET shiyanlou002 002 shiyanlou003 003 shiyanlou004 4
OK

127.0.0.1:6379> MGET shiyanlou001 shiyanlou002 shiyanlou003 shiyanlou004
1) "001"
2) "002"
3) "003"
4) "4"

127.0.0.1:6379> STRLEN shiyanlou001
(integer) 3
127.0.0.1:6379> STRLEN shiyanlou004
(integer) 1
127.0.0.1:6379> DECR shiyanlou004
(integer) 3
127.0.0.1:6379> GET shiyanlou004
"3"

127.0.0.1:6379> INCR shiyanlou004
(integer) 4
127.0.0.1:6379> GET shiyanlou004
"4"

127.0.0.1:6379> DECRBY shiyanlou004 2
(integer) 2
127.0.0.1:6379> GET shiyanlou004
"2"

127.0.0.1:6379> INCRBY shiyanlou004 10
(integer) 12
127.0.0.1:6379> GET shiyanlou004
"12"

127.0.0.1:6379> APPEND shiyanlou001 append_test
(integer) 14
127.0.0.1:6379> GET shiyanlou001
"001append_test"
```

## 3. list

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边），`LPUSH` 命令插入一个新元素到列表头部，而`RPUSH` 命令插入一个新元素到列表的尾部。如果 `key` 不存在，将会在执行插入操作前创建一个空列表；如果 `key` 已存在，且对应的值不是一个列表，会返回一个错误，这一点与字符串不同。相似的，如果一个列表操作清空一个列表那么对应的 key 将被从 key 空间删除。这是非常方便的语义，因为如果使用一个不存在的 key 作为参数，所有的列表命令都会像在对一个空表操作一样。

首先，我们先尝试查看并删除上面的内容中创建的键。查找所有匹配给定的模式的键使用如下语句：

```bash
KEYS pattern
```

`pattern` 代表的是给定模式（正则表达式）。相关的内容在前面几周的学习中，已经讲述了很多次，这里只会涉及到一些通用的内容。

例如，查看所有的键：

```bash
127.0.0.1:6379> keys *
1) "shiyanlou001"
2) "shiyanlou004"
3) "shiyanlou003"
4) "shiyanlou002"
```

删除指定的一个或多个键使用如下语句：

```bash
DEL key [key...]
```

除了删除之外，我们也可以将一个键重命名，使用如下语句：

```bash
RENAME key newkey
```

如下示例，删除刚刚所有创建的键：

```bash
127.0.0.1:6379> DEL shiyanlou001 shiyanlou002 shiyanlou003 shiyanlou004
(integer) 4
127.0.0.1:6379> keys *
(empty list or set)
```

对于一个列表而言，我们可以从列表的左边或者右边，即头部或者尾部插入元素。分别使用 `LPUSH` 和 `RPUSH` 命令：

```bash
LPUSH key value [value ...]
RPUSH key value [value ...]
```

从列表中获取元素时使用 `LRANGE`。start 和 stop 为指定范围，列表中的第一个元素下标为 0。也可以为负数，例如 -2 为倒数第二个元素。

```bash
LRANGE key start stop
```

在获取元素之前，我们还需要获取列表的长度，使用 `LLEN`

```bash
LLEN key
```

如下的简单示例，分别使用上面介绍的关于列表的操作：

```bash
127.0.0.1:6379> KEYS *
(empty list or set)
127.0.0.1:6379> LPUSH shiyanlou001 a b c d
(integer) 4
127.0.0.1:6379> LLEN shiyanlou001
(integer) 4
127.0.0.1:6379> LRANGE shiyanlou001 0 3
1) "d"
2) "c"
3) "b"
4) "a"
127.0.0.1:6379> RPUSH shiyanlou001 e f g h
(integer) 8
127.0.0.1:6379> LLEN shiyanlou001
(integer) 8
127.0.0.1:6379> LRANGE shiyanlou001 0 7
1) "d"
2) "c"
3) "b"
4) "a"
5) "e"
6) "f"
7) "g"
8) "h"
127.0.0.1:6379> LRANGE shiyanlou001 0 -4
1) "d"
2) "c"
3) "b"
4) "a"
5) "e"
127.0.0.1:6379>
```

对于上述的插入操作，如果列表不存在，会创建一个列表。

而只针对插入操作，在列表不存在时，什么操作也不执行的命令，如下：

```bash
LPUSHX key value
RPUSHX key value
```

如下所示:

```bash
127.0.0.1:6379> LPUSHX shiyanlou001 1
(integer) 9
127.0.0.1:6379> LPUSHX shiyanlou002 1
(integer) 0
127.0.0.1:6379> RPUSHX shiyanlou002 1
(integer) 0
127.0.0.1:6379> KEYS *
1) "shiyanlou001"
127.0.0.1:6379>
```

移除列表中的元素可以使用 `LPOP` 和 `RPOP`

```bash
LPOP key
RPOP key
```

如下所示：

```bash
127.0.0.1:6379> LLEN shiyanlou001
(integer) 9
127.0.0.1:6379> LRANGE shiyanlou001 0 8
1) "1"
2) "d"
3) "c"
4) "b"
5) "a"
6) "e"
7) "f"
8) "g"
9) "h"
127.0.0.1:6379> LPOP shiyanlou001
"1"
127.0.0.1:6379> LLEN shiyanlou001
(integer) 8
```

除了这些常用的操作之外，还有很多其它的命令，这里就不一一列举了。

Redis 字符串和列表类型操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/15-1.mp4
@`

## 4. hash

Redis Hashes 是字符串字段和字符串值之间的映射，因此他们是展现对象的完美数据类型。一个带有一些字段的 hash 仅仅需要一块很小的空间存储，因此你可以存储数以百万计的对象在一个小的 Redis 实例中。哈希主要用来表现对象，但也能够存储许多元素，因此可以将哈希用于许多其他的任务。一个 hash 最多可以包含 2^32-1 个 key-value 键值对，但这已经超过 40 亿了。

对于设置和获取 hash 字段和值的语法如下：

```bash
HSET key field value
HGET key field value
```

首先，我们删除 `shiyanlou001` 的列表，然后设置 hash 集 `shiyanlou001` 的字段，并获取值。

```bash
127.0.0.1:6379> DEL shiyanlou001
(integer) 1
127.0.0.1:6379> HSET shiyanlou001 username aiden
(integer) 1
127.0.0.1:6379> HGET shiyanlou001 username
"aiden"
```

使用 `HSET` 设置字段的值时，如果指定的 `key` 即哈希集不存在，则会创建一个新的哈希集并与 `key` 关联。

对于多个字段值的设置和获取可以使用如下语句：

```bash
HMSET key field value [field value]
HMGET key field [field ...]
```

简单示例：

```bash
127.0.0.1:6379> HMSET shiyanlou001 age 30 sex man company shiyanlou
OK
127.0.0.1:6379> HMGET shiyanlou001 username age sex company
1) "aiden"
2) "30"
3) "man"
4) "shiyanlou"
```

以及一些其它的命令：

```bash
# 获取所有字段
HKEYS key

# 获取所有的值
HVALS key

# 获取所有的字段和值
HGETALL key

# 获取所有字段的数量
HLEN key

# 删除一个或者多个字段
HDEL key field [field ...]

# 判断字段是否存在于 Hash 中
HEXISTS key field
```


Redis Hash 操作视频：
`@
http://labfile.oss.aliyuncs.com/courses/980/week7/15-2.mp4
@`

## 5. set

Redis 集合（Set）是一个无序的字符串集合。你可以以 `O(1)` 的时间复杂度 (无论集合中有多少元素时间复杂度都是常量)完成添加，删除，以及测试元素是否存在。Redis 集合拥有不允许包含相同成员的属性，多次添加相同的元素，最终在集合里也只会有一个元素。实际上这些意味着在添加元素的时候无须检测元素是否存在。

Redis 集合有许多非常有趣的事情，比如：支持一些服务端的命令从现有的集合出发去进行集合运算，因此就可以在非常短的时间内进行合并（`unions`）， 求交集（`intersections`），找出不同的元素（`differences of sets`）。

这里我们讲述集合的有关操作。首先是，添加一个或多个元素到集合中，以及获取集合中的所有元素。语法格式如下：

```bash
SADD key member [member ...]
SMEMBERS key
```

这里的 `member` 代表的是集合中的元素，也称成员，如下示例：

```bash
127.0.0.1:6379> SADD shiyanlou002 a b 1 2
(integer) 4
127.0.0.1:6379> SMEMBERS shiyanlou002
1) "2"
2) "1"
3) "b"
4) "a"
```

以及一些常见的其它操作，简单列举：

```bash

# 获取集合中元素的数量
SCARD key

# 从集合中随机删除一个或多个元素
SPOP key [count]

# 从集合中删除指定的一个或者多个元素
SREM key member [member ...]

# 判断一个给定的值是该集合的成员，如果是就返回 1 ，否就返回 0 。
SISMEMBER key member
```

简单示例：

```bash
127.0.0.1:6379> SCARD shiyanlou002
(integer) 4
127.0.0.1:6379> SPOP shiyanlou002 2
1) "b"
2) "1"
127.0.0.1:6379> SMEMBERS shiyanlou002
1) "2"
2) "a"
127.0.0.1:6379> SREM shiyanlou002 2
(integer) 1
127.0.0.1:6379> SMEMBERS shiyanlou002
1) "a"
127.0.0.1:6379> SISMEMBER shiyanlou002 a
(integer) 1
127.0.0.1:6379> SISMEMBER shiyanlou002 b
(integer) 0
```

Redis Set 操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/15-3.mp4
@`

## 6. sorted set

Redis 有序集合与普通集合非常相似，是一个没有重复元素的字符串集合。不同之处是有序集合的每一个成员都关联了一个评分，这个评分被用来按照从最低分到最高分的方式排序集合中的成员。

集合的成员是唯一的，但是评分可以是重复。使用有序集合可以以非常快的速度（`O(log(N)`)）添加，删除和更新元素。因为元素是有序的，所以你也可以很快的根据评分（`score`）或者次序（`position`）来获取一个范围的元素。访问有序集合的中间元素也是非常快的，因此你能够使用有序集合作为一个没有重复成员的智能列表。

在有序集合中，你可以很快捷的访问一切你需要的东西：有序的元素，快速的存在性测试，快速访问集合的中间元素！简而言之使用有序集合你可以完成许多对性能有极端要求的任务，而那些任务使用其他类型的数据库真的是很难完成的。

经过上面的学习我们会发现，对于 Redis 中各种数据类型相关的操作命令，都有一定的规律。所以这里我们只是简单列举命令并示例。

给有序集合添加成员，指定：分数/成员（score/member)。如果成员已经存在，则会更新 score 并更改排序位置。 Redis 使用的 score 为`双精度 64 位的浮点数`。

```bash
ZADD key [NX|XX] [CH] [INCR] score member [score member ...]
```

在 REDIS 3.x 之后的版本中支持一些参数，分别如下所示：

+ NX: 不更新存在的元素，只添加新成员。

+ XX: 仅更新存在的元素

+ CH: 修改返回值为发生变化的成员总数，原始是返回新添加成员的总数 (CH 是 changed 的意思)。更改的元素是新添加的成员，已经存在的成员更新分数。所以在命令中指定的成员有相同的分数将不被计算在内。注：在通常情况下，ZADD 返回值只计算新添加成员的数量。

+ INCR: 当 ZADD 指定这个选项时，成员的操作就等同 ZINCRBY 命令，对成员的分数进行递增操作。

除此之外，一些其它的常用命令

```bash
# 获得成员数量
ZCARD key

# 根据 Index 获得指定的成员列表
ZRANGE key start stop

# 确定成员的索引
ZRANK key member

# 删除一个或者多个成员
ZREM key member [member ...]

# 获取一个成员的分数（score）
ZSCORE key member
```

使用示例如下：

```bash
# 添加有序集的分数/成员
127.0.0.1:6379> ZADD shiyanlou003 NX 1 a 2 b 3 c
(integer) 3
# 获取成员数量
127.0.0.1:6379> ZCARD shiyanlou003
(integer) 3
# 根据 Index 获得指定的成员列表
127.0.0.1:6379> ZRANGE shiyanlou003 0 2
1) "a"
2) "b"
3) "c"
# 确定成员 b 的索引
127.0.0.1:6379> ZRANK shiyanlou003 b
(integer) 1
# 获取成员的分数
127.0.0.1:6379> ZSCORE shiyanlou003 b
"2"
# 删除多个成员
127.0.0.1:6379> ZREM shiyanlou003 a b
(integer) 2
# 获取成员数量
127.0.0.1:6379> ZCARD shiyanlou003
(integer) 1
# 根据 Index 获得指定的成员列表
127.0.0.1:6379> ZRANGE shiyanlou003 0 1
1) "c"
```

## 7. 总结

通过本节实验的学习，认识到了 Redis 的相关数据类型以及各个数据类型的基本操作。

+ strings
+ list
+ hash
+ sets
+ sort sets