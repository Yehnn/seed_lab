---
show: step
version: 1.0
enable_checker: true
---
# Redis 常用配置应用

## 1. 实验介绍

#### 1.1 实验内容

本节实验主要是对 Redis 的系统管理方法和高级应用做进一步的讲解。

#### 1.2 实验知识点

+ 查询信息（INFO）
+ 配置
+ 数据库
+ 过期时间
+ 主从复制
+ 事务
+ 持久化

## 2. 查询信息

查询 Redis 有关的信息我们可以通过 `INFO` 命令来操作。

```
INFO [section] ：查询 Redis 相关信息。
```

例如，我们直接使用 `INFO` 命令（详细信息太多，这里使用省略号省略）：

```bash
127.0.0.1:6379> INFO
# Server
...

# Clients
...

# Memory
...

# Persistence
...

# Stats
...

# Replication
...

# CPU
...

# Cluster
...

# Keyspace
...
```

除此之外，我们可以使用命令选项查询相关的部分信息，而不是所有信息，`[section]` 的值如下所示：

+ server: Redis 服务器的常规信息

+ clients: 客户端的连接选项

+ memory: 内存占用相关信息

+ persistence: RDB and AOF 相关信息

+ stats: 常规统计

+ replication: Master/slave请求信息

+ cpu: CPU 占用信息统计

+ cluster: Redis 集群信息

+ keyspace: 数据库信息统计

如下所示，返回 `cpu` 的有关信息：

```bash
127.0.0.1:6379> INFO cpu
# CPU
used_cpu_sys:46.09
used_cpu_user:42.45
used_cpu_sys_children:0.02
used_cpu_user_children:0.01
127.0.0.1:6379>
```

## 3. 配置

对于 `Redis` 的配置而言，有两种方式：

- 使用系统环境中的配置文件是 `/etc/redis/redis.conf`，我们可以通过其来设定一些配置项
- 通过命令行参数的方式设定配置。

1.首先我们来查看默认的配置文件

如下所示，为 `redis.conf` 配置文件的部分内容（内容较多，仅列举部分内容）：

```txt
# 排除注释部分
shiyanlou:~/ $ sudo cat /etc/redis/redis.conf | grep "^[^#]"

bind 127.0.0.1
protected-mode yes
port 6379

logfile /var/log/redis/redis-server.log

databases 16

rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /var/lib/redis
```

2.我们可以通过服务启动命令的参数来设置相同的配置项

通过命令行参数的方式设定配置，例如配置端口用 `--port`：

```bash
redis-server --port 6379
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid4580timestamp1516258747518.png/wm)

更多的参数使用我们可以通过 `--help` 查看：

```
$ redis-server --help
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2767331515751428635-wm)

对于上述的两种方式而言，使用配置文件需要在启动 `Redis` 服务之前设定，使用命令行参数需要在启动服务时指定。

另外，我们还可以在服务运行的过程中，在不重启服务器的情况下更改服务器配置，需要使用到特定的 `CONFIG SET` 命令，但是这种方式并不能修改所有的配置项，针对一些特殊的配置项是无效的。

对应的可以通过 `CONFIG GET` 命令来获取部分配置信息。

两个命令的使用方式如下：

```bash
CONFIG SET parameter value
CONFIG GET parameter
```

如下所示，我们通过通配符 `*` 去获取该命令能够查看的配置信息（内容较多，仅列出一小部分配置项）：

```bash
127.0.0.1:6379> CONFIG GET *
  1) "dbfilename"
  2) "dump.rdb"
  3) "requirepass"
  4) ""

 58) "0"
 59) "slowlog-max-len"
 60) "128"
 61) "port"
 62) "6379"
 63) "cluster-announce-port"
 64) "0"

155) "dir"
156) "/var/lib/redis"
157) "save"
158) "900 1 300 10 60 10000"

163) "slaveof"
164) ""
165) "notify-keyspace-events"
166) ""
167) "bind"
168) "127.0.0.1"
127.0.0.1:6379>
```

对于上述内容中的配置项，相比 `redis.conf` 配置文件中的配置项会少很多。有兴趣的同学可以自行去了解相关内容。

在配置项中，我们可以看到绑定地址和端口等信息，并且部分的配置项可以通过 `CONFIG SET` 修改，例如，我们通过 `CONFIG SET` 修改密码为 `shiyanlou`

```bash
# 获取密码信息
127.0.0.1:6379> CONFIG GET requirepass
1) "requirepass"
2) ""

# 设置密码
127.0.0.1:6379> CONFIG SET requirepass shiyanlou
OK

# 修改后需要进行认证，所以这里报错
127.0.0.1:6379> CONFIG GET requirepass
(error) NOAUTH Authentication required.
```

如上所示，初始密码为空，而密码已经被修改为 `shiyanlou`，此时，认证的密码已经被修改，而当前客户端访问服务器使用的密码依然是 `""`，所以，如果我们不断开本地链接想要继续使用当前客户端时，需要使用 `AUTH` 命令来为当前当前客户端进行密码验证，使用方式为 `AUTH password`，如下：

```bash
# 使用密码验证
127.0.0.1:6379> AUTH shiyanlou
OK

# 验证成功后命令执行成功
127.0.0.1:6379> CONFIG GET requirepass
1) "requirepass"
2) "shiyanlou"
```

验证（AUTH）成功后，返回一个 `OK` 的状态码


Redis 配置操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/16-1.mp4
@`

## 4. 数据库

在 Redis 中，数据库是由一个整数来标识，从 0 开始，而不是像 `MySQL` 和 `MongoDB` 一样使用数据库名称。

在上面的配置项中，我们可以看到有一个 `databases` 的配置项，如下所示：

```bash
127.0.0.1:6379[2]> CONFIG GET databases
1) "databases"
2) "16"
```

该配置项代表 Redis 中有多少个数据库，默认值为 `16`。所以这 16 个数据库的标识为 `0~15`。

在我们使用 `redis-cli` 客户端且并未指定某一个数据库时，默认使用的是整数标识为 `0` 的数据库。而在客户端中，如果我们需要切换数据库，可以使用 `SELECT` 语句：

```bash
SELECT index
```

`index` 代表的是数据库的整数标识索引，如下所示，我们切换数据库：

```bash
shiyanlou:~/ $ redis-cli
# 切换为 1
127.0.0.1:6379> SELECT 1
(error) NOAUTH Authentication required.

# 由于刚刚修改了密码，这里我们需要进行验证
127.0.0.1:6379> AUTH shiyanlou
OK

# 再次切换
127.0.0.1:6379> SELECT 1
OK

# 切换成功后会出现 [1] ，指示当前使用的是哪一个数据库
127.0.0.1:6379[1]>

# 再次切换
127.0.0.1:6379[1]> SELECT 15
OK

# 超出范围时会报错
127.0.0.1:6379[15]> SELECT 16
(error) ERR DB index is out of range
127.0.0.1:6379[15]>
```

除了使用上述方式切换数据库之外，我们还可以在使用客户端 `redis-cli` 连接服务器时就使用 `-n` 参数指定某一个数据库，默认值为 `0`:

```bash
shiyanlou:~/ $ redis-cli
127.0.0.1:6379> QUIT
shiyanlou:~/ $ redis-cli -n 1
127.0.0.1:6379[1]> QUIT
shiyanlou:~/ $ redis-cli -n 2
127.0.0.1:6379[2]> QUIT
shiyanlou:~/ $
```

这里需要注意的是，在使用客户端的 `-n` 参数进行连接时，如果使用了超出 `0~15` 范围的值，那么此时所使用的数据库的标识索引为 `0`。即使用 `16` 其实使用的是索引为 `0` 的数据库。使用值 `-1` 也是使用索引为 `1` 的数据库。如下所示：

```bash
shiyanlou:~/ $ redis-cli -n 16
127.0.0.1:6379[16]> AUTH shiyanlou
OK
127.0.0.1:6379[16]> KEYS *
(empty list or set)
127.0.0.1:6379[16]> SET test1 1
OK
127.0.0.1:6379[16]> KEYS *
1) "test1"
127.0.0.1:6379[16]> SELECT 0
OK
127.0.0.1:6379> KEYS *
1) "test1"
127.0.0.1:6379> GET test1
"1"
```

除了 `SELECT` 之外，还有两个与数据库有关的命令常被使用到。分别为返回当前数据库 `key` 的数量的命令 `DBSIZE` ，以及清空当前数据库的命令 `FLUSHDB`。

Redis 中数据库操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/16-2.mp4
@`

## 5. 过期时间

在前面数据类型一节中，我们有提到设置数据类型为字符串的 `key` 过期时间的方式。
这里介绍另外一种方法，来设置 `key` 键的过期时间，使用 `EXPIRE`，使用方法如下：

```bash
EXPIRE key seconds
```

对应的有 `PERSIST` ，用于移除 `key` 的过期时间，并使其永久生效，但是必须在 `key` 未过期之前使用。

如下示例：

```bash
# 查询所有的 key
127.0.0.1:6379> KEYS *
(empty list or set)

# 设置一个键 test1,并设置过期时间为 60 s
127.0.0.1:6379> SET test1 1
OK
127.0.0.1:6379> EXPIRE test1 60
(integer) 1

# 设置过期时间之后 60s 之内使用下列命令查看所有的 key
127.0.0.1:6379> KEYS *
1) "test1"

# 查看 test1 还有多久过期，因操作速度不一。所以显示结果会不一样
127.0.0.1:6379> TTL test1
(integer) 20

# 等待一段时间后，查看键，此时已过期，所以显示为空
127.0.0.1:6379> KEYS *
(empty list or set)

127.0.0.1:6379> TTL test1
(integer) -2
```

Redis 中过期时间操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week7/16-3.mp4
@`

## 6. 主从复制

Redis 的主从复制配置和使用都比较简单，通过主从复制可以允许多个 `slave server` 拥有和 `master server` 相同的数据库副本。

其中 `master server` 和 `slave server` 分别代表主服务和从服务。

Redis 的主从复制的配置非常简单，如下示例：

首先，我们查看系统中的 Redis 服务，此时只有一个运行在 6379 端口上的服务。

```
shiyanlou:~/ $ ps -ef | grep redis-server
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2767331515942522290-wm)

这里，我们以上面运行在 `6379` 端口上的 redis 服务为 `master server`，再启动一个 `slave server`。

此时，我们启动一个新的 `redis-server`，使用 `6380` 端口。如下所示

```
$ redis-server --port 6380
```

接着新开一个终端，使用 `redis-cli` 连接到 `6380` 端口的 Redis 服务中

```
shiyanlou:~/ $ redis-cli -p 6380
127.0.0.1:6380>
```

然后只需使用 `SLAVEOF host port` 命令，就可以将当前服务转变为指定服务的从属服务（slave server）。

```
127.0.0.1:6380> SLAVEOF 127.0.0.1 6379
OK
```

对于 6380 的服务来讲，因为此时运行于 `6379` 的服务需要使用密码进行验证（在上一节的内容中设置了密码），所以还需要在 `6380` 上使用如下命令设置 `master server` 的密码：

```
127.0.0.1:6380> CONFIG SET masterauth shiyanlou
OK
```

此时，我们可以使用 `redis-cli` 来分别连接 `6379` 和 `6380` ，来验证两个服务之间的数据是否同步。

除了使用 `SLAVEOF` 命令之外，我们还可以在运行服务时，使用命令行参数 `--slaveof 127.0.0.1 6379` 的方式。或者指定配置文件，在配置文件中，添加如下所示的的配置项：

```
slaveof 127.0.0.1 6379
```

## 7. 事务

Redis 的事务处理比较简单。只能保证 client 发起的事务中的命令可以连续的执行，并且不会插入其他的 client 命令。当一个 client 在连接中发出 `MULTI` 命令时，这个连接就将进入到一个事务的上下文，同时该连接的后续命令不会执行，而是存放到一个队列中。当执行 `EXEC` 命令时，Redis 会顺序的执行队列中的所有命令，如果其中执行出现错误，执行正确的也不会回滚，这不同于关系型数据库的事务。

```
# 开始一个事务
127.0.0.1:6379> MULTI
OK

# 设置 name 的值为 shiyanlou001，此时处于事务中，所以保存命令到队列中，并不会立即执行，提示信息为 QUEUED，
127.0.0.1:6379> SET name shiyanlou001
QUEUED

# 设置 name 的值为 shiyanlou002
127.0.0.1:6379> SET name shiyanlou002
QUEUED

# 设置 name 的值为 shiyanlou003。 该语句的语法错误，但是并不会执行，所以这里不会提示错误
127.0.0.1:6379> SET name shiyanlou003 error
QUEUED

# 设置 name 的值为 shiyanlou004，此时处于事务中，所以保存命令到队列中，并不会立即执行，提示信息为 QUEUED，
127.0.0.1:6379> SET name shiyanlou004 
QUEUED

# 使用 EXEC 执行，输出会提示我们，第三条语法错误的命令执行失败，其它执行成功
127.0.0.1:6379> EXEC
1) OK
2) OK
3) (error) ERR syntax error
4) OK

# 此时获取 name 的值为 shiyanlou004，可以看到即使发生错误，也不会有回滚操作
127.0.0.1:6379> GET name
"shiyanlou004"
```

## 8. 持久化

Redis 是一个支持持久化的内存数据库，需要经常将内存中的数据同步到磁盘来保证持久化。

Redis 支持两种持久化方式：

1.`snapshotting`（快照），将数据存放到文件里，默认方式。

是将内存中的数据以快照的方式写入到二进制文件中，默认文件 `/var/lib/redis/dump.rdb`，可以通过配置来设置自动做快照持久化的方式。可配置 Redis 在 n 秒内，如果超过 m 个 key 被修改就自动保存快照。

例如配置文件中的如下配置项：

```
# 900 秒内如果超过 1 个 key 就被修改，就发起快照保存
save 900 1

# 300 秒内如果超过 10 个 key 被修改，就发起快照保存
save 300 10
```

2.`Append-only file`（缩写为 aof），将读写操作存放到文件中。

由于快照方式在一定间隔时间执行一次，所以如果 Redis 意外 down 掉的话，就会丢失最后一次快照后的所有修改。

aof 比快照方式有更好的持久化性，是因为使用 aof 时，Redis 会将每一个收到的写命令都通过 `write` 函数写入到文件中。当 Redis 启动时会通过重新执行文件中保存的写命令在内存中重新建立整个数据库的内容。

由于操作系统会在内核中缓存 write 的修改，所以可能不是立即写到磁盘上，这样 aof 方式的持久化也还是有可能会丢失一部分数据。所以可以通过配置文件告诉 Redis 我们想要通过 `fsync` 函数强制 os 写入到磁盘的时机。

配置文件中的可配置参数：

```
appendonly   yes     //启用 aof 持久化方式

# appendfsync  always //收到写命令就立即写入磁盘，最慢，但是保证了数据的完整持久化

appendfsync   everysec  //每秒中写入磁盘一次，在性能和持久化方面做了很好的折中

# appendfsync  no     //完全依赖 os，性能最好，持久化没有保证
```

在 `redis-cli` 的命令中，`SAVE` 命令是将数据写入磁盘中。

```
> help save
> save
```

## 9. 总结

通过本节实验对 Redis 的管理及高级运用做了补充。