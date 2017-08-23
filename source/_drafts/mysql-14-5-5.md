---
title: mysql-14-5-5
tags:
  - mysql
categories:
  - tech 
  - translate
  - mysql
---



## 14.5.5 InnoDB中的死锁

 - [14.5.5.1 InnoDB死锁案例](/tech/translate/mysql/mysql-14-5-1/)
 - [14.5.5.2 死锁检测与回滚](/tech/translate/mysql/mysql-14-5-2/)
 - [14.5.5.3 如何最小化与处理死锁](/tech/translate/mysql/mysql-14-5-3/)
 
&emsp;&emsp;死锁是指不同的事务持有对方所需的锁，从而事务无法继续处理的情形，这种情形下，每个事务都在等待对方的锁，且都不会释放自己所占有的锁。

&emsp;&emsp;死锁可能在不同的事务以不同的顺序锁定表中的行(如UPDATE or SELECT ... FOR UPDATE)时发生。也可能发生在这些语句导致的一个获取一个范围锁的时候，因为时间的缘故，每个事务只获取了其中的一部分锁。死锁的例子，见[14.5.5.1 InnoDB死锁案例](/tech/translate/mysql/mysql-14-5-1/)

&emsp;&emsp;要降低死锁的可能，使用事务而不是锁表语句；减小事务中插入或更新语句的规模，以免长时间开启事务；当不同的事务更新多张表或者一个范围的行时，在每个事务中以相同的顺序操作（例如SELECT ... FOR UPDATE)；

create indexes on the columns used in SELECT ... FOR UPDATE and UPDATE ... WHERE statements. The possibility of deadlocks is not affected by the isolation level, because the isolation level changes the behavior of read operations, while deadlocks occur because of write operations. For more information about avoiding and recovering from deadlock conditions, see Section 14.5.5.3, “How to Minimize and Handle Deadlocks”.

When deadlock detection is enabled (the default) and a deadlock does occur, InnoDB detects the condition and rolls back one of the transactions (the victim). If deadlock detection is disabled using the innodb_deadlock_detect configuration option, InnoDB relies on the innodb_lock_wait_timeout setting to roll back transactions in case of a deadlock. Thus, even if your application logic is correct, you must still handle the case where a transaction must be retried. To see the last deadlock in an InnoDB user transaction, use the SHOW ENGINE INNODB STATUS command. If frequent deadlocks highlight a problem with transaction structure or application error handling, run with the innodb_print_all_deadlocks setting enabled to print information about all deadlocks to the mysqld error log. For more information about how deadlocks are automatically detected and handled, see Section 14.5.5.2, “Deadlock Detection and Rollback”.
