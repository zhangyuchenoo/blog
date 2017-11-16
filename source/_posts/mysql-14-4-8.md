---
title: mysql-14.4.8
date: 2017-11-13 14:51:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.4.8撤销日志

&emsp;&emsp;撤销日志是一个事务相关的多条回滚日志记录。撤销日志记录包含了如何撤销对聚簇索引记录撤销最近变更的信息。如果其他的事务需要查看原来的数据(一致性读), 则从撤销日志记录中获取未修改的数据。撤销日志存储于撤销日志段中，撤销日志段位于回滚段中,回滚段则分布于系统表空间，临时表空间和撤销表空间中(undo tablespace)。要了解更多，见[14.7.7配置撤销表空间](),要了解更多关于多版本的信息，见[14.3InnoDB多版本]()。

&emsp;&emsp;InnoDB支持128个回滚段，其中32个被作为临时表事务的非重做(non-redo)回滚段。每个更新临时表的事务(除了只读事务)都被分配两个回滚段，一个可重做(redo-enabled)回滚段和一个不可重做(non-redo)回滚段。只读事务只会分配不可重做回滚段，因为只读事务只被允许修改临时表。

&emsp;&emsp;剩下96个可用的回滚段，每个都支持多达1023个并发数据修改的事务，总共接近96k的并发修改数据事务。96k的总数是在假设事务不修改临时表的情况，如果所有的 数据修改事务也需要修改临时表，则总的并发修改事务只有32k左右。要了解更多关于临时表事务相关的回滚段信息，见[14.4.12.1临时表撤销日志]()。

&emsp;&emsp;innodb_rollback_segments选项定义了InnoDB使用的回滚段数量。