---
title: mysql-14.3
date: 2017-10-31 15:11:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.3 InnoDB Multi-Versioning

&emsp;&emsp;InnoDB是多版本存储引擎:它保存变更行的旧版本来支持如并发和回滚的事务特性。这些信息存放在表空间中叫做回滚段的数据结构中(after an analogous data structure in Oracle)。InnoDB使用回滚端中的信息来执行事务回滚中的撤销(undo)操作，在一致性读中也用回滚段中的信息来读取就版本的数据。

&emsp;&emsp;在InnoDB为每行记录添加三个字段。一个6字节的DB_TRX_ID标识上一次插入或更新该条记录的事务。删除操作也被当做一次更新操作，更新行中的某一特殊位来标识该行被删除。每一行还包含一个7字节回滚指针DB_ROLL_PTR字段，回滚指针指向回滚端中的一条回滚记录。如果某一行被更新，撤销日志（undo）包含重构该行记录更新前必要的信息。一个单调递增的6字节字段DB_ROW_ID包含新的一行插入时的id。如果InnoDB自动产生了聚簇索引，则索引包含行的ID，否则DB_ROW_ID不会出现在任何索引中。

&emsp;&emsp;回滚段中的撤销日志(undo log)被分为插入和删除两种。插入撤销日志只在事务回滚时用到，一旦事务提交就可以立即丢弃。更新撤销日志会在一致性读时用到，所以只能在没有需要为快照读事务提供之前版本的情况下删除。


&emsp;&emsp;定期提交事务，包括那些只有一致性读的事务，否则InnoDB不能丢弃撤销日志，从而导致回滚段过大，填满表空间。

&emsp;&emsp;撤销日志的物理大小会比相应插入或更新的行记录小。你可以根据这个信息计算回滚段需要的空间。

&emsp;&emsp;在InnoDB多版本约束下，在执行删除语句的时候，一行记录并不会立即从物理上删除，InnoDB只在丢弃删除对应的更新撤销日志时才会物理删除该行和其对应的索引，这一操作称作purge，而且速度和删除该记录的删除语句一样快。

&emsp;&emsp;如果以恒定速率小批量插入或删除表中的记录，purge线程可能会开始落后，从而使得表不断增大，消耗大量硬盘空间从而速度变慢。这种情况下，你可以调节插入记录的速率，并且通过innodb_max_purge_lag参数为purge线程分配更多资源。见[14.14, InnoDB启动选项和系统变量]()了解更多。

### 多版本与二级索引

&emsp;&emsp;InnoDB多版本并发控制将聚簇索引与二级索引区别对待。聚簇索引中的Records in a clustered index are updated in-place, and their hidden system columns point undo log entries from which earlier versions of records can be reconstructed. Unlike clustered index records, secondary index records do not contain hidden system columns nor are they updated in-place.

&emsp;&emsp;当二级索引的列被更新时，旧的二级索引被标记删除，新纪录被插入，并且标记删除的索引记录最终被purge。当二级索引被标记删除或二级索引页被新的事务更新的时候，InnoDB会通过聚簇索引查询记录。在聚簇索引中检查DB_TRX_ID，如果在查询事务开始后记录曾被修改过，能从撤销日志中得到正确的记录版本。

When a secondary index column is updated, old secondary index records are delete-marked, new records are inserted, and delete-marked records are eventually purged. When a secondary index record is delete-marked or the secondary index page is updated by a newer transaction, InnoDB looks up the database record in the clustered index. In the clustered index, the record's DB_TRX_ID is checked, and the correct version of the record is retrieved from the undo log if the record was modified after the reading transaction was initiated.

&emsp;&emsp;如果一个二级索引记录被标记删除或者二级索引页被新的事务更新，不会使用覆盖索引，InnoDB会从聚簇索引查找记录而不是从二级索引直接返回。

&emsp;&emsp;然而，如果索引条件下移(index condition pushdown)优化被启用，并且where条件中可以用到索引中的部分字段，MySQl服务器会将WHERE条件下放到引擎去通过索引判断。如果没有匹配到记录，则聚簇索引查找可以避免。如果找到对应的记录，甚至是标记删除的记录，InnoDB查找再通过聚簇索引查找相应记录。
