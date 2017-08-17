---
title: mysql-14.5.3
date: 2017-08-15 14:23:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---

<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

14.5.3 InnoDB中不同SQL设置的锁


&emsp;&emsp;在锁定读中，UPDATE或DELETE通常在SQL语句处理中所扫描到的所有记录上加锁，而不会排除where条件中不满足的行。InnoDB不会记录精确的WHERE条件满足的行，而是所扫描的索引范围。因此通常都用会阻止在记录前区间插入的next-key锁，当然区间锁也可以被显式禁用，从而不适用next-key锁，见[14.5.1InnoDB锁](/tech/translate/mysql/mysql-14-5-1/)。当然事务隔离级别也会影响锁，见[14.5.2.1事务隔离级别](/tech/translate/mysql/mysql-14-5-2-1/)。

&emsp;&emsp;当然如果使用到二级索引，且是唯一的，InnoDB也会在相应的聚簇索引上加记录锁。

&emsp;&emsp;共享与排他锁的差别见[14.5.1InnoDB锁](/tech/translate/mysql/mysql-14-5-1/)。

&emsp;&emsp;如果在语句中没有用到合适的索引，MySQL会扫描整张表，从而导致整张表被锁住，因此创建合适的索引以使语句扫描更少的行是有必要的。

&emsp;&emsp;对于SELECT ... FOR UPDATE或SELECT ... LOCK IN SHARE MODE，所扫描到的行都会加锁，并且会在返回给MySQL后不满足条件的行锁会得到释放。但是在一些情况下，不满足条件的行并不会立即得到释放，因为原记录行与返回结果中的行的关系在执行过程中没有维护。例如在UNION语句中，原表中扫描到的行(被锁住)会在评估行是否满足条件前被插入到一张临时表中，此时临时表中的行与原表中的行的关系并没有维护，原表中不满足条件行的锁就不会释放知道整个语句执行结束。

&emsp;&emsp;InnoDB设置锁的情况如下：

&emsp;&emsp;SELECT ... FROM是一致读，只会读取数据库快照而不会加锁除非隔离级别被设置为SERIALIZABLE。在SERIALIZABLE级别下，查询会在扫描到的记录上加next-key锁，但是如果是对有唯一索引的表查询唯一的记录，则只会使用记录锁。

&emsp;&emsp;SELECT ... FROM ... LOCK IN SHARE MODE会在扫描到(encounter)的记录上加共享的next-key锁，然而对于有唯一索引表查询唯一记录的情况，只会使用记录锁。

&emsp;&emsp;SELECT ... FROM ... FOR UPDATE会在扫描到(encounter)的记录上加排他的next-key锁，然而对于有唯一索引表查询唯一记录的情况，只会使用记录锁。

&emsp;&emsp;对于扫描到的索引记录，SELECT ... FROM ... FOR UPDATE会阻塞SELECT ... FROM ... LOCK IN SHARE或者某些隔离级别下的读取。一致读会忽略记录上的锁。

&emsp;&emsp;UPDATE ... WHERE ... sets会在扫描到(encounter)的记录上加共享的next-key锁，然而对于有唯一索引表查询唯一记录的情况，只会使用记录锁。

&emsp;&emsp;当UPDATE修改聚簇索引指向的记录的时候，相应的二级索引也会隐式加锁。UPDATE操作也会执行唯一性检查前在影响到的二级索引上加共享锁。

&emsp;&emsp;DELETE FROM ... WHERE ...会在扫描到(encounter)的记录上加共享的next-key锁，然而对于有唯一索引表查询唯一记录的情况，只会使用记录锁。


&emsp;&emsp;会在要插入的行上设置排它锁，这里的锁是记录锁而不是next-key锁从而不会阻止其他会话在要插入记录的前边区间插入记录。

&emsp;&emsp;在插入行之前，会设置插入意向的区间锁，这种锁表明了将要插入记录的意图，通过这种方式在同一个区间插入不同记录的不同事务不必相互等待。例如在表中有两条有索引记录4和7，有两个事务分别想插入5和6，他们在获取要插入行的记录排它锁前都会获取获取插入意向锁来锁住4和7之间的区间，但是他们并不会相互阻塞，因为两条记录并不冲突。

&emsp;&emsp;为了防止键值重复(duplicate-key)错误，键上会设置共享锁。这种对共享锁的使用会导致死锁的发生：当多个会话插入同一条记录且其中一个会话已经获取了排它锁，然后删除该行的情况。实例如下：

    CREATE TABLE t1 (i INT, PRIMARY KEY (i)) ENGINE = InnoDB;

&emsp;&emsp;现在假设有三个会话一次执行如下的操作:

Session 1:

    START TRANSACTION;
    INSERT INTO t1 VALUES(1);

Session 2:

    START TRANSACTION;
    INSERT INTO t1 VALUES(1);

Session 3:

    START TRANSACTION;
    INSERT INTO t1 VALUES(1);

Session 1:

    ROLLBACK;

&emsp;&emsp;session 1的第一次操作获取了行的排它锁，session 2和session 3的操作都会出现键重复(duplicate-key）错误，而且他们都会请求该行的共享锁。当session 1执行回滚操作时，会释放持有的排它锁时session 2和session3会获得该行的共享锁,这时session 2和session 3发生了死锁，因为他们都持有了共享锁，从而都不能获取排它锁。

&emsp;&emsp;类似的情况是表已经有了一条键值为1的记录然后3个会话依次执行如下操作：

Session 1:

    START TRANSACTION;
    DELETE FROM t1 WHERE i = 1;
Session 2:

    START TRANSACTION;
    INSERT INTO t1 VALUES(1);
Session 3:

    START TRANSACTION;
    INSERT INTO t1 VALUES(1);
Session 1:

    COMMIT;

&emsp;&emsp;session 1的操作获取了行的排它锁，为了防止键重复错误，session 2和session 3都会获取行的共享锁(The operations by sessions 2 and 3 both result in a duplicate-key error and they both request a shared lock for the row)。当session 1提交的时候，会释放行上的排它锁，等待分配共享锁的session 2和session 3都获得共享锁，而此时他们都因彼此获得的共享锁而无法获取排它锁。

&emsp;&emsp;INSERT ... ON DUPLICATE KEY UPDATE与简单的插入不同，当主键重复错误发生时会在要更新的行上设置排它锁而不是共享锁，对于主键会设置记录排它锁，对于唯一索引，会设置排它next-key锁。

&emsp;&emsp;在没有唯一索引冲突时REPLACE和INSERT一样，在出现冲突时会在要替换的行上设置排它next-key锁。

&emsp;&emsp;INSERT INTO T SELECT ... FROM S WHERE ...会在插入T的每一行上设置索引记录锁。如果事务隔离级别是READ COMMITTED或者innodb_locks_unsafe_for_binlog选项开启且隔离级别不是SERIALIZABLE，InnoDB会通过一致读来执行查询，否则InnoDB在S中的行上设置next-key锁。 InnoDB has to set locks in the latter case: In roll-forward recovery from a backup, every SQL statement must be executed in exactly the same way it was done originally.

&emsp;&emsp;CREATE TABLE ... SELECT ...在执行SELECT时设置next-key锁或者像INSERT ... SELECT一样执行一致读。
&emsp;&emsp;REPLACE INTO t SELECT ... FROM s WHERE ...或者UPDATE t ... WHERE col IN (SELECT ... FROM s ...)中的SELECT，InnoDB会在表S中设置next-key锁。

&emsp;&emsp;在初始化表中的自增列时，InnoDB会在列对应的索引最后设置排它锁。在访问自增计数器(auto-increment counter)时，InnoDB使用特定的自增表锁(AUTO-INC table lock mode),此时其他会话不能插入记录，这种锁在当前SQL语句执行完后就被释放，而不是直到整个事务结束，见[14.5.2InnoDB事务模型](/tech/translate/mysql/mysql-14-5-2/) 。

&emsp;&emsp;InnoDB在获取之前初始化的自增列的值时，不会设置任何锁。

&emsp;&emsp;在有外键约束的表中，任何INSERT,UPDATE或DELETE需要检查约束的语句都会在关联表相应的行上设置记录共享锁来检查约束。 InnoDB also sets these locks in the case where the constraint fails.

&emsp;&emsp;LOCK TABLES设置表锁，但是只有InnoDB之上的MySQL层会设置表锁。如果设置innodb_table_locks = 1(默认值)且autocommit = 0，InnoDB能够感知到表锁，InnoDB之上的MySQL层也能感知到行级锁。

&emsp;&emsp;否则，InnoDB的死锁检测机制不能检测出这种表锁相关的死锁，同时在这种情况下，InnoDB之上的MySQL层不知道行级锁的存在，当其他会话拥有一张表的行级锁的时候，仍有可能获取该表的表锁，但是这并不会影响事务的完整性，正如[14.5.5.2死锁检测域回滚](/tech/translate/mysql/mysql-14-4-5-2/)中讨论的那样，另见[14.8.1.7InnoDB表的局限](/tech/translate/mysql/mysql-14-8-1-7/)