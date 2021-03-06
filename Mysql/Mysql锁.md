# 乐观锁

乐观并发控制多数用于数据争用不大、冲突较少的环境中，这种环境中，偶尔回滚事务的成本会低于读取数据时锁定数据的成本，因此可以获得比其他并发控制方法更高的吞吐量。

## 使用版本号实现乐观锁

使用版本号时，可以在数据初始化时指定一个版本号，每次对数据的更新操作都对版本号执行+1操作。并判断当前版本号是不是该数据的最新的版本号。

```sql
-- 1.查询出商品信息
select (status,status,version) from t_goods where id=#{id}
-- 2.根据商品信息生成订单
-- 3.修改商品status为2
update t_goods
set status=2,version=version+1
where id=#{id} and version=#{version};
```

## 优点与不足

乐观并发控制相信事务之间的数据竞争(data race)的概率是比较小的，因此尽可能直接做下去，直到提交的时候才去锁定，所以不会产生任何锁和死锁。但如果直接简单这么做，还是有可能会遇到不可预 期的结果，例如两个事务都读取了数据库的某一行，经过修改以后写回数据库，这时就遇到了问题。

# 悲观锁

与乐观锁相对应的就是悲观锁了。悲观锁就是在操作数据时，认为此操作会出现数据冲突，所以在进行每次操作时都要通过获取锁才能进行对相同数据的操作

悲观并发控制主要用于数据争用激烈的环境，以及发生并发冲突时使用锁保护数据的成本要低于回滚事务的成本的环境中。

## 使用MySQL InnoDB中使用悲观锁

要使用悲观锁，我们必须关闭`mysql`数据库的自动提交属性，**因为MySQL默认使用`autocommit`模式**，也就是说，当你执行一个更新操作后，MySQL会立刻将结果进行提交。

```
set autocommit=0;
```

> 关闭自动提交

```sql
#0.开始事务
begin;/begin work;/start transaction; (三者选一就可以)
#1.查询出商品信息
select status from t_goods where id=1 for update;
#2.根据商品信息生成订单
insert into t_orders (id,goods_id) values (null,1);
#3.修改商品status为2
update t_goods set status=2;
#4.提交事务
commit;/commit work;
```

上面的查询语句中，我们使用了`select…for update`的方式，这样就通过开启排他锁的方式实现了悲观锁。此时在`t_goods`表中，`id`为1的那条数据就被我们锁定了，其它的事务必须等本次事务提交之后才能执行。这样我们可以保证当前的数据不会被其它事务修改。

上面我们提到，使用`select…for update`会把数据给锁住，不过我们需要注意一些锁的级别，

> **MySQL InnoDB默认行级锁。行级锁都是基于索引的，如果一条SQL语句用不到索引是不会使用行级锁的，会使用表级锁把整张表锁住，这点需要注意。**

## 优点与不足

悲观并发控制实际上是”先取锁再访问”的保守策略，为数据处理的安全提供了保证。但是在效率方面，处理加锁的机制会让数据库产生额外的开销，还有增加产生死锁的机会；另外，在只读型事务处理中由于不会产生冲突，也没必要使用锁，这样做只能增加系统负载；还有会降低了并行性，一个事务如果锁定了某行数据，其他事务就必须等待该事务处理完才可以处理那行数

共享锁和排它锁是悲观锁的不同的实现，它俩都属于悲观锁的范畴

# 共享锁和排他锁 #

- **共享锁（S锁）**：共享 (S) 用于不更改或不更新数据的操作（只读操作），如 `SELECT `语句。

  如果事务T对数据A加上共享锁后，则其他事务只能对A再加共享锁，不能加排他锁。获准共享锁的事务只能读数据，不能修改数据。

- **排他锁（X锁）**：用于数据修改操作，例如 INSERT、UPDATE 或 DELETE。确保不会同时同一资源进行多重更新。

  如果事务T对数据A加上排他锁后，则其他事务不能再对A加任任何类型的封锁。获准排他锁的事务既能读数据，又能修改数据。

```sql
-- 情况一
-- 开启事务排他锁
begin;
select * from user where id = 10 for update;
#COMMIT;

id  	username		birthday   		sex		address
10	 	 张三			2014-07-10			1		北京市

-- 排他锁查询现处于阻塞
select * from user where id = 10 for update;
-- 共享锁查询现处于阻塞
select * from user where id=10 lock in share mode;
-- 普通查询可查询到结果
select * from user where id = 10

-- 情况二
-开启事务共享锁
begin;
select * from user where id = 10 lock in share mode;
#COMMIT;

10	张三	2014-07-10	1	北京市

##排他锁查询现处于阻塞
select * from user where id = 10 for update;
#共享锁查询可查询到结果
select * from user where id=10 lock in share mode;
#普通查询可查询到结果
select * from user where id = 10

#情况三（mysql InnoDb引擎中update,delete,insert语句自动加排他锁的问题）
#开启事务更新数据
begin;
UPDATE user set sex=2 where id = 10;
#COMMIT;

##排他锁查询现处于阻塞
select * from user where id = 10 for update;
#共享锁查询现处于阻塞
select * from user where id=10 lock in share mode;
#普通查询可查询到结果，但结果是事务修改前的旧值
select * from user where id = 10

#当事务成功commit后，排他查，共享查，普通查都可以查询到更新后的值
```

