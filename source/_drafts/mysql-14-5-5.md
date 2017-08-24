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

&emsp;&emsp;要降低死锁的可能，使用事务而不是锁表语句；减小事务中插入或更新语句的规模，以免长时间开启事务；当不同的事务更新多张表或者一个范围的行时，在每个事务中以相同的顺序操作（例如SELECT ... FOR UPDATE)；SELECT ... FOR UPDATE and UPDATE ... WHERE 用到的列上添加索引。死锁发生的几率并不受隔离级别的影响，因为隔离级别只改变读操作的行为，而死锁与写操作有关。要了解更多关于如何避免与从死锁中恢复，见[14.5.5.3 如何最小化与处理死锁](/tech/translate/mysql/mysql-14-5-3/)。

&emsp;&emsp;当死锁检测开启（默认开启）且发生死锁的情况，InnoDB检测到死锁并回滚其中的一个事务。如果死锁检测通过innodb_deadlock_detect被禁用，则在死锁发生时InnoDB只有等到innodb_lock_wait_timeout设置的时间达到时回滚事务。因此，即使你的应用逻辑是对的，你也需要处理事务需要重试的情况。要查看InnoDB事务中最近出现的死锁，可以使用SHOW ENGINE INNODB STATUS命令。如果经常出现死锁，从而使得问题上升到事务结构或者应用处理，可通过开启innodb_print_all_deadlocks设置来输出索引死锁信息到mysqld错误日志中。要了解死锁是如何自动检测并处理的，见[14.5.5.2死锁检测与回滚](/tech/translate/mysql/mysql-14-5-5-2/)。
