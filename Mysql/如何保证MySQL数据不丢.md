# **如何保证MySQL数据不丢？**

binlog，又称为二进制日志，它会记录数据库执行更改的所有操作，但是不包括查询select等操作。一般用于恢复、复制等功能。它的格式有三种：**statement、mixed和row**。

- statement：每一条会修改数据的sql都会记录到binlog中，不建议使用。
- row：基于行的变更情况记录，会记录行更改前后的内容，**推荐使用**。
- mixed：混合statement和row两个模式，不建议使用。

**binlog 的写入机制是怎样的呢？**

> ★事务执行过程中，先把日志写到 binlog cache，事务提交的时候，再把binlog cache写到binlog文件中 。

系统为每个客户端线程分配一个**binlog cache**，其大小值控制参数是**binlog_cache_size**。如果**binlog cache的值**超过阀值，就会临时持久化到磁盘。当事务提交的时候，再将 binlog cache中完整的事务持久化到磁盘中，并且清空binlog cache。

 	![](http://120.77.237.175:9080/photos/eight/mysql/08.jpg)

binlog写文件分**write和fsync**两个过程：

- write：指把日志写到文件系统的page cache，并没有把数据持久化到磁盘，因此速度较快。
- fsync，实际的写盘操作，即把数据持久化到磁盘。

write和fsync的写入时机，是由变量sync_binlog控制的：

 	![](http://120.77.237.175:9080/photos/eight/mysql/09.jpg)

如果IO出现性能瓶颈，可以将**sync_binlog**设置成一个较大的值。比如设置为（100~1000）。但是，会存在数据丢失的风险，当主机异常重启时，会**丢失N个最近提交的事务binlog**。

### redo log日志

redo log，又称为**重做日志文件**，只记录事务对数据页做了哪些修改，它记录的是数据修改之后的值。redo 有三种状态

![](http://120.77.237.175:9080/photos/eight/mysql/10.jpg)

- 物理上是在MySQL进程内存中，存在redo log buffer中，
- 物理上在文件系统的page cache里，写到磁盘 (write)，但是还没有持久化（fsync)。
- 存在hard disk，已经持久化到磁盘。

日志写到**redo log buffer**是很快的；wirte到**page cache**也很快，但是持久化到磁盘的速度就慢多了。

为了控制redo log的写入策略，Innodb根据innodb_flush_log_at_trx_commit参数不同的取值采用不同的策略，它有三种不同的取值：

- 1. 设置为0时，表示每次事务提交时都只是把redo log留在redo log buffer 中 ;

     

  2. 设置为1时，表示每次事务提交时都将 redo log 直接持久化到磁盘；

     

  3. 设置为2时，表示每次事务提交时都只是把redo log 写到page cache。

> ★三种模式下，0的性能最好，但是不安全，MySQL进程一旦崩溃会导致丢失一秒的数据。1的安全性最高，但是对性能影响最大，2的话主要由操作系统自行控制刷磁盘的时间，如果仅仅是MySQL宕机，对数据不会产生影响，如果是主机异常宕机了，同样会丢失数据。