# PHP与Mysql 配置

## 一、实验简介

通过上一节的实验，我们学会了如何去配置 Apache 环境。使用一些 apache 意想不到的功能，这节将带领大家简单的了解一下 PHP 与 Mysql 的配置参数。

### 1.1 相关知识点

- PHP 常用配置
- MySQL 一般优化配置

##二、 PHP常用配置

PHP 的配置文件在 `/etc/php5/` 中，并且使用 apt-get 装的话，针对的模块不同，配置文件也不同，我们可以看见在该目录下有这样几个文件夹:

- apache2：专门针对在 apache 调用 php5 时使用的配置文件
- cli：专门针对在 shell 调用 php5 时使用的配置文件
- fpm: 专门针对在 FastCGI process manager对 php 使用时的配置文件
- mods-available: 专门针对在其他程序模块中的配置

并且我们还可以在每个文件夹下面看见 php.ini 文件与 conf.d 文件夹。由此我们可以得知 php 的配置文件氛围两部分：

- php.ini 这个主要的配置文件；
- conf.d 这个文件夹下面的配置文件是额外的配置文件，配置某个扩展的配置文件。

进入该文件中，其实配置文件的语法很简单：

- 空白字符(” “,”/t”,…)和用分号(”;”)开始的行被简单地忽略；
- 章节标题（例如 : [php]）也被简单地忽略，即使将来它们可能有意义；
- 设置格式： directive（指令大小写敏感） = value（值）。

```
###常用的配置
    engine = On //允许 php 引擎在 apache 中运行
    short_open_tag = Off  #tags识别
    asp_tags = Off #asp tags识别
    precision = 14  #浮点类型数显示时的有效位数
    output_buffering = 4096   #输出缓存允许你在输出正文内容之后发送 header
    implicit_flush = Off   #告诉输出层在每个输出块之后自动刷新自身数据
    disable_functions =    #关闭特定函数
    display_errors = Off   #显示错误信息
    log_errors = On        #在日志文件里记录错误
    request_order = "GP"   #GET POST
    register_argc_argv = On #是否声明 argv和argc 变量
    post_max_size = 8M     #最大POST大小
    file_uploads = On      #是否允许HTTP方式文件上载 
	upload_tmp_dir =       #用于HTTP上载的文件的临时目录
	upload_max_filesize = 2M   #上载文件的最大大小
	max_file_uploads = 20      #每次最大上传文件个数
	allow_url_fopen = On   #是否允许把URLs当作http:.. 或把文件当作ftp:...
	default_socket_timeout = 60  #默认socker超时时间

	#MySQL相关
    mysql.allow_persistent = On #允许持久连接
    mysql.cache_size = 2000  #缓存大小
    mysql.max_persistent = -1  #持久连接的最大数目 -1不限制
    mysql.max_links = -1  #最大链接 
    mysql.connect_timeout = 60  #连接超时时间

	#session相关
	session.save_handler = files #用于保存/取回数据的控制方式
	session.use_cookies = 1 #是否使用cookies
	session.name = PHPSESSID  #设置cookies名
	session.use_only_cookies = 1  #强制SESSION ID必须以COOKIE传递
	session.auto_start = 0  #在请求启动时初始化session 
	session.cookie_lifetime = 0  #cookies保持时间
	
	#session垃圾回收参数
	session.gc_probability = 1 #按百分比整理
	session.gc_divisor = 1000  #每次请求时有1/1000的机会启动"垃圾回收"进程
	
	#Session ID传递参数
	session.use_trans_sid = 0 #使用URL的方式传递session id
	session.hash_function = 0 #生成SID MD5算法
	session.hash_bits_per_character = 5  #指定在SID字符串中的每个字符内保存多少bit
```

PHP配置相关链接：

 - PHP中文手册  http://php.net/manual/zh/
 - 完整php.ini 参数详解   http://php.net/manual/zh/ini.php

##三、 MySQL常用配置

mysql 的配置文件在这个文件中 `/etc/mysql/my.cnf`。

在配置 MySQL 之前我们先要了解到 MyISAM 与 InnoDB 两个常用的表存储引擎，

> **MyISAM**是MySQL关系数据库管理系统的默认储存引擎。这种MySQL表存储结构从旧的ISAM（Indexed Sequential Access Method，MySQL5.0已经不支持ISAM了）代码扩展出许多有用的功能。在新版本的MySQL中，InnoDB引擎由于其对事务，参照完整性，以及更高的并发性等优点开始广泛的取代MyISAM。

> **InnoDB**是MySQL存储引擎。MySQL 5.5和以后使用它的默认。它提供标准的符合 ACID 事务特性，随着国外重点支持（声明引用完整性）。它包括作为标准在大多数双星由MySQL AB分布，唯一的例外是一些OEM版本。

关于两者的比较，可以参看这篇[博文](http://www.ha97.com/4197.html)。

在使用这两个引擎的时候有这样一个小建议，仅供参考：

如果仅使用MyISAM存储引擎,设置 key_buffer_size为可用内存的20%,(再加上设置 innodb_buffer_pool_size = 0 )；

如果仅使用InnoDB存储引擎,设置 innodb_buffer_pool_size为可用内存的 70%, (设置 key_buffer_size = 10M,很小但不是0)。

MyISAM引擎的缓存分为两部分：

- 索引块(Index blocks,每个1 KB,BTree结构、存放于 .MYI 文件) 缓存到 “key buffer” 中.；
- 数据块缓存(Data block caching, 存放于 .MYD 文件中)交给操作系统负责, 所以确保留下了适量的空闲内存(给操作系统)。

InnoDB将所有缓存都放在 “buffer pool” 中, 缓存池的大小通过 innodb_buffer_pool_size 控制。包含被打开表(open tables)中的 16KB一块的数据/索引块,此外还有一些附加开销。

MySQL 5.5(以及带插件的 5.1版本)允许您指定 块大小(block size)为 8 KB或4 KB. MySQL 5.5可以有多个缓冲池,因为每个缓存池有一个互斥锁, 所以设置多个池可以缓解一些互斥锁瓶颈。

一般情况下主机不是太差，默认配置即可，但是也要根据站点的情况作相应的处理，具体的一些调优参数大家可以参看这篇[博文](http://www.ha97.com/4110.html)。

##四、 LAMP常用调优方法

LAMP环境下的调优，以下所有参数均来自于网络，请根据实际环境进行调整。

基于稳定的perfork工作模式，调整apache主配置文件以下参数。

```
    StartServers       50   #预创建50个进程
    MinSpareServers   15    #保持最小空闲进程
    MaxSpareServers   30    #保持最大空闲进程
    MaxClients       225    #最大进程数
    MaxRequestsPerChild  4000   #用来控制每个进程在处理了多少次请求之后自动销毁
```

请使用ps aux 查看进程占用内存数，以此调整实际环境中的参数，防止此消耗所有的资源。

php.ini常用优化配置参数：

```
    max_execution_time 30   #一个脚本可使用多少 CPU 秒
    max_input_time 60       #一个脚本等待输入数据的时间有多长（秒）
    memory_limit  32M       #在被取消之前，一个脚本可使用多少内存（字节）
    output_buffering  4096  #数据发送给客户机之前，有多少数据（字节）需要缓存
```

### 参考文献

[1]mysql调优：<http://mysql.rjweb.org/doc.php/memory>
[2]mysql调优博文：<http://www.ha97.com/4110.html>