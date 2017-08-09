---
title: mysql-14.5.1
tags:
---

###INNODB锁

本节将介绍INNODB所使用的锁.
- 共享锁(S)和排他锁(X)
- 意向锁(Itention Locks)
- 行锁(Record Locks)
- 区间锁(Gap Locks)
- (Next-Key Locks)
- 插入意向锁(Insert Intention Locks)
- 自增锁(AUTO-INC Locks)
- (Predicte Locks for Spatial Indexes) 

####共享锁(S)和排他锁(X)

InnoDB 实现了标准的两种行级别锁，共享锁和排它锁。

- 共享锁(S)允许拥有该锁的事务读取一行数据。
- 排它锁(X)允许拥有该锁的事务更新或删除一行数据。

如果事务T1拥有一行数据r的共享锁,另一个事务T2尝试获取该行r的锁时将做如下处理：
- 如果T2请求的是共享锁，则该事务将立即获得该锁，T1和T2分别获得该行r的一个共享锁。
- 如果T2请求的是排它锁,则该锁不能立即获取,T2将等待T1释放对行r的共享锁。
如果事务T1拥有行r的排它锁，则其他事务T2无论是尝试获取共享锁还是排它锁都需要等待T1释放其所拥有的行r的锁。

####意向锁(Intention Locks)

InnoDB支持多种粒度(行级锁和表级锁)的锁。意向锁的引入使得多种粒度的锁成为现实。意向锁是InnoDB中的表级锁，它声明了一个事务在稍候将要获取该表中一行的哪一种(共享或排他)锁。加入一个事务T请求获得一张表的意向锁，将有两种类型:

- 共享意向锁(IS):事务T打算获取该表中某一行的共享锁(S)。
- 排他意向锁(IX):事务T打算获取该表中某一行的排它锁(X)。
例如SELECT ... LOCK IN SHARE MODE设置IS锁， SELECT ... FOR UPDATE 获取IX锁.
意向锁的规则如下:

-在一个事务获取表中一行记录的S锁之前，必须先获取表的IS锁或者更强的锁。
-在一个事务获取表中一行记录的x锁之前，必须先获取表的IX锁。

这些规则可能用下边的一张锁兼容矩阵表概括：

||X|IX|S|IS|
|-------|-------|-------|-------|-------|
|X|不兼容|不兼容|不兼容|不兼容|
|IX|不兼容|兼容|不兼容|兼容|
|S|不兼容|不兼容|兼容|兼容|
|IS|不兼容|兼容|兼容|兼容|

事务能够获得锁，如果事务所请求的锁与当前存在的锁兼容。反之则不能，并等待知道存在的锁得到释放。如果所请求的锁与存在的锁冲突并且会导致死锁，将不能获取锁并报错。
因此，意向锁不会阻塞表级别外的请求(例如LOCK TABLES ... WRITE)。意向锁主要的目的是表明有人正在或将要对表中的一行加锁。
意向锁的事务数据在 SHOW ENGINE INNODB STATUS 和InnoDB 监视器( InnoDB monitor)的输出中看起来是类似的。
`TABLE LOCK table `test`.`t` trx id 10080 lock mode IX`
####行级锁
行级锁是在索引记录上的锁。例如， SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE 阻止其他的事务插入更新或删除t.c1的值为10的行。
行级锁总是在索引上加锁，在定义的表的时候没有添加索引的情况，InnoDB会为表创建隐藏的聚簇索引并在改索引上加行级锁。见[14.8.2.1聚簇索引和二级索引](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html)。

事务数据在 SHOW ENGINE INNODB STATUS 和InnoDB 监视器( InnoD    B monitor)的输出中看起来是类似的:

`RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t` 
trx id 10078 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc    O;;
 2: len 7; hex b60000019d0110; asc        ;;`

####区间锁

区间锁是在索引记录间的锁，或是第一个索引记录之前或最后一个索引记录之后的区间的锁。例如SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE会阻止其他事务其他事务插入t.c1列的值为15的记录不管是否已经存在相同列值的记录，因为该范围内的整个区间已被锁住。
区间锁可以只跨域单个索引，多个索引或者为空
区间锁是效率和并发度的一种交换，会在某些隔离级别中使用。
区间锁不会被用在使用唯一索引获取唯一行的语句中(当然这不包括搜索条件只包括多列唯一索引中一部分列的情况，这种情况下会使用区间锁)例如，如果id列有唯一索引，以下的语句将只会锁住id值为100的行的索引记录，并不会关心其他事务是否在之前的的区间中插入记录:
`SELECT * FROM child WHERE id = 100;`
如果id没有索引或者有非唯一索引，这个语句确实会锁住之前的区间。
值得注意的是在区间上不兼容的锁可以被不同的事务获得。例如事务A拥有一个区间共享锁(gap S-lock)的情况下另一个事务可以获取该区间的排它锁(gap X-lock)。之所以可以在同一个区间上获取不兼容的锁是因为如果一条记录从索引中移除了(purged)，不同事务在该记录上的区间锁需要被合并。
区间锁是“纯粹抑制性(purely inhibitive)”，意味着只能阻止事务对区间进行插入而不会阻止其他事务在同一区间上获取区间锁。因此区间排它锁(gap X-lock)和区间共享锁(gap S-lock)是一样的效果。
区间锁能够被显式(explicitly)禁用，当设置隔离级别为READ_COMMITTED或者启用innodb_locks_unsafe_for_binlog(现已禁用)系统参数。这种情况下，区间锁对于查询或者索引扫描都是禁用的，只有外键约束检查和键重复检查会使用区间锁。
设置隔离级别为READ_COMMITTED或者启用innodb_locks_unsafe_for_binlog系统参数还有其他的作用，MySQL在解析完where条件以后就会释放不匹配行记录的锁。对于UPDATE语句来说，InnoDB进行“一致读(semi-consistent)”返回最近提交的记录给MySQL，以使MySQL能够判断该行是否UPDATE的where条件。
####Next-Key Locks

Next-Key Lock 是行锁和索引记录之前的区间锁的结合。
当搜索或扫描表索引的时候InnoDB执行行级锁，在其遇到的索引记录上设置共享或排它锁，因此行级锁实际上是索引记录锁。索引记录上的Next-Key锁还会影响该索引记录前的区间，也就是说Next-Key锁是索引记录锁加上改索引记录前的区间的区间锁。如果一个会话拥有记录R的索引的共享或排它锁，其他会话不能立即插入新的索引记录到R之前的区间中。
加入索引记录包含10,11,13和20，可能的next-key锁包括如下的区间(圆括号代表不包括端点，方括号代表包括端点):
`(negative infinity, 10]
(10, 11]
(11, 13]
(13, 20]
(20, positive infinity)`
对于最后一个区间，next-key锁锁住索引中大于最大值的区间并且“上确界”虚记录('supremum'pseudo-record)是一个比索引中所有值都大的记录。上确界并不是一个真实的索引记录，因此实际上，next-key锁锁住的是索引中最大值之后的区间。
默认情况下，InnoDB的隔离级别是 REPEATABLE READ，在改隔离级别下，InnoDB对搜索和索引扫描使用next-key锁，以防止幻读的发生(见14.5.4幻读(https://dev.mysql.com/doc/refman/5.7/en/innodb-next-key-locking.html))。
对于next-key锁，事务数据在 SHOW ENGINE INNODB STATUS 和InnoDB 监视器( InnoDB monitor)的输出中看起来是类似的:
`RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t` 
trx id 10080 lock_mode X
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     O;;
 2: len 7; hex b60000019d0110; asc        ;;`

####插入意向锁(Insert Intention Locks)
插入意向锁是INSERT操作在插入一行前设置的一种区间锁。这个锁表明将要插入记录的意图，通过这种方式多个事务在插入同一区间的不同位置时不需要相互等待。假如索引中有4和7的值，有两个事务分别准备插入5和6且在获取插入行的排它锁之前各自都用插入意向锁锁住了4和7之间的区间，但是并不不会相互阻塞，因为两行记录并不冲突。
下面的例子演示了事务在获取插入记录的排它锁之前获取了插入意向锁，改例子设计A和B两个客户端。
客户端A创建了一张含有两条索引记录的表，然后开启一个事务并在id大于100的记录上设置排它锁，这个排它锁包含一个记录102之前的区间锁:
`mysql> CREATE TABLE child (id int(11) NOT NULL, PRIMARY KEY(id)) ENGINE=InnoDB;
mysql> INSERT INTO child (id) values (90),(102);

mysql> START TRANSACTION;
mysql> SELECT * FROM child WHERE id > 100 FOR UPDATE;
+-----+
| id  |
+-----+
| 102 |
+-----+`
客户端B开启一个事务来插入一条记录到该区间，这个在等待获取排他锁之前获取了插入意向锁。
`mysql> START TRANSACTION;
mysql> INSERT INTO child (id) VALUES (101);`

事务数据在 SHOW ENGINE INNODB STATUS 和InnoDB 监视器( InnoDB monitor)的输出中看起来是类似的:
`RECORD LOCKS space id 31 page no 3 n bits 72 index `PRIMARY` of table `test`.`child`
trx id 8731 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 80000066; asc    f;;
 1: len 6; hex 000000002215; asc      ;;
 2: len 7; hex 9000000172011c; asc     r  ;;...`

####自增锁(AUTO-INC Locks)
自增锁是事务在插入有自增列的时候的一种特别的表级锁。一般情况下，如果一个事务正在向表中插入多条记录，其他事务必须等待以插入记录到该表，以使第一个事务在插入的时候获取连续的主键值。
innodb_autoinc_lock_mode设置控制着自增锁的算法。它允许你在可预测的自增序列和最大的并发插入之间进行权衡。
想要了解更多可以参看[14.8.1.5 AUTO_INCREMENT Handling in InnoDB](https://dev.mysql.com/doc/refman/5.7/en/innodb-auto-increment-handling.html) 

####空间索引与测锁(Predicate Locks for Spatial Indexes)

InnoDB支持包含空间数据列的空间索引(见11.5.8 优化空间分析(https://dev.mysql.com/doc/refman/5.7/en/optimizing-spatial-analysis.html))
对于空间索引的锁处理，next-key锁并不能很好的支持REPEATABLE READ或者SERIALIZABLE隔离级别，因为在多维的数据中，并没有严格的顺序概念，因此也就不清楚next key是哪一个。
为了对空间索引的表提供隔离级别的支持，InnoDB使用预测锁(predicate locks)。空间索引包括最小外接矩形(MBR minimum bounding rectangle)值，InnoDB在查询时在MBR值上设置预测锁，其他事务不能插入或者修改匹配该查询条件的行记录。

