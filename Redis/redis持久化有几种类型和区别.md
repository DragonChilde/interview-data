# Redis持久化有几种类型，他们的区别

## Redis 提供了 2 个不同形式的持久化方式

- RDB ( Redis DataBase)
- AOF (Append OF File)

### RDB

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。

> 备份是如何执行的

Redis会单独创建(fork)一个子进程来进行持款化，并将数据写入到一个临时文件中，待持久化过程都结束了,再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的,这就确保了极高的性能如果需要进行大规模数据的恢复，对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。

rdb 的保存文件

在 redis.conf 中配置文件名称 默认为 dump.rdb

```
# The filename where to dump the DB
dbfilename dump.rdb
```

rbd 文件的保存路径，也可以修改，默认为 Redis启动命令行所在目录下

```
# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir ./
```

> RDB 保存的策略

```
# Unless specified otherwise, by default Redis will save the DB:
#   * After 3600 seconds (an hour) if at least 1 key changed
#   * After 300 seconds (5 minutes) if at least 100 keys changed
#   * After 60 seconds if at least 10000 keys changed
#
# You can set these explicitly by uncommenting the three following lines.
#
# save 3600 1
# save 300 100
# save 60 10000
```

- 60 分钟 1 次添加 key 的操作
- 5 分钟 100 次添加 key 的操作
- 1 分钟 10000 次添加 key 的操作都会触发保存策略。

> rdb 的备份

先通过 config get dir 查询 rdb文件的目录

将 *.rdb 的文件拷贝到别的地方

> rdb 的恢复

1. 关闭 Redis
2. 先把备份文件拷贝到拷贝到工作目录下
3. 启动 Redis，备份数据会直接加载

> rdb 的优点

- 节省磁盘空间
- 恢复速度快

![](http://120.77.237.175:9080/photos/eight/redis/18.png)

> rdb 的缺点

虽然Redis在fork时使用了写时拷贝技术,但是如果数据庞大时还是比较消耗性能。

在备份周期在一定间隔时间做一次备份, 所以如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改

------

### AOF

> 什么是 AOF 呢

以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，Redis启动之初会读取该文件重新构建数据，换言之，Redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

> AOF 默认不开启，需要手动在配置文件中配置

```
# An AOF file may be found to be truncated at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# {server-mode}     Special mode, i.e. "[sentinel]" or "[cluster]".
# {port}            TCP port listening on, or 0.
# {listen-addr}     Bind address or '*' followed by TCP or TLS port listening on, or
#                   Unix socket if only that's available.
# {server-mode}     Special mode, i.e. "[sentinel]" or "[cluster]".
# {port}            TCP port listening on, or 0.
# {tls-port}        TLS port listening on, or 0.
# {unixsocket}      Unix domain socket listening on, or "".
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check https://redis.io/topics/persistence for more information.

appendonly no
```

可以在`redis.conf`中配置文件名称，默认为 `appendonly.aof`

```
# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"
```

> AOF 和 RDB 同时开启，redis 听谁的

系统默认取AOF的数据。

> AOF 文件故障备份

AOF的备份机制和性能虽然和RDB不同, 但是备份和恢复的操作同RDB一样，都是拷贝备份文件，需要恢复时再拷贝到Redis工作目录下，启动系统即加载。

> AOF 文件故障恢复

如遇到AOF文件损坏，可通过

```sh
redis-check-aof --fix appendonly.aof #进行恢复
```

> AOF 同步频率设置

- 始终同步，每次Redis的写入都会立刻记入日志。
- 每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。
- 把不主动进行同步，把同步时机交给操作系统。

```
# An AOF file may be found to be truncated at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# {server-mode}     Special mode, i.e. "[sentinel]" or "[cluster]".
# {port}            TCP port listening on, or 0.
# {listen-addr}     Bind address or '*' followed by TCP or TLS port listening on, or
#                   Unix socket if only that's available.
# {server-mode}     Special mode, i.e. "[sentinel]" or "[cluster]".
# {port}            TCP port listening on, or 0.
# {tls-port}        TLS port listening on, or 0.
# {unixsocket}      Unix domain socket listening on, or "".
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always
appendfsync everysec
# appendfsync no
```

> Rewrite

AOF采用文件追加方式，文件会越来越大为避免出现此种情况,新增了重写机制,当AOF文件的大小超过所设定的阈值时,Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集可以使用命令`bgrewriteaof`.

> Redis 如何实现重写？

AOF文件持续增长而过大时，会fork出一条新进程来将文件写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，一条记录有一条的Set语句。 写aof文件的操作，并没有读取旧的aof文件，并将整个内存中的数据库内容用命令的方式写了一个新的aof文件, 这点和快照有点类似。

> 何时重写

写虽然可以节约大量磁盘空间,减少恢复时间。但是每次重写还是有一定的负担的，因此设定Redis要满足一条件才会进行重写。

```
# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

同载入时或者上次重写完毕时, Redis会记录此时AOF大小,设为 `base size` ,如果Redis的AOF当前大小 大于base size + base_ size* 100% (默认)且当前大小 >=64mb (默认)的情况下， Redis会对AOF进行重写。

> AOF 的优点

- 备份机制更稳健，丢失数据概率更低。
- 可读的日志文本，通过操作AOF稳健，可以处理误操作。

![](http://120.77.237.175:9080/photos/eight/redis/19.png)

> AOF的缺点

- 比起RDB占用更多的磁盘空间。
- 恢复备份速度要慢。
- 每次读写都同步的话，有一定的性能压力。
- 存在个别Bug，造成不能恢复。