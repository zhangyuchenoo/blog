---
title: mysql-14.2
date: 2017-10-31 10:11:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.2 InnoDB与ACID模型

&emsp;&emsp;ACID模型是针对商业和任务严格的环境下提高可靠性而提出的一套数据库设计原则。MySQL包含一些想InnoDB那样符合ACID模型的组件,使得数据在如软件崩溃或硬件损坏的异常条件下数据不会遭到破坏。当依赖于ACID兼容的特性时，你不需要重复发明轮子来做一致性检查或者崩溃恢复。在你有其它的的软件，非常可靠的硬件或者应用可以接受少量数据丢失的环境下，你可以通过设置牺牲一些ACID特性来换取更好的性能或吞吐量。

&emsp;&emsp;下边将要介绍MySQL的一些特性，尤其是InnoDB引擎是如何遵循ACID模型的。

    A: atomicity(原子性).

    C: consistency(一致性).

    I:: isolation(隔离性).

    D: durability(持久性).

#### Atomicity

&emsp;&emsp;原子性主要是指InnoDB的事务，相关的MySQL特性包括:
 
 - 自动提交设置
 - COMMIT语句
 - ROLLBACK语句
 - Operational data from the INFORMATION_SCHEMA tables.

#### 一致性

&emsp;&emsp;一致性主要涉及InnoDB内部的处理以保证数据不会损坏。相关的MySQL特性包括:

 - InnoDB doublewrite buffer

 - InnoDB crash recovery

#### 隔离性

&emsp;&emsp;隔离性主要涉及InnoDB事务，特别是事务的隔离级别。相关的MySQL特性包括:
 
 - 自动提交设置
 - SET ISOLATION LEVEL语句
 - The low-level details of InnoDB locking. During performance tuning, you see these details through INFORMATION_SCHEMA tables.

#### 持久性

&emsp;&emsp;持久性主要涉及与硬件交互配置相关的MySQL特性。由于依赖于CPU，网络与存储设备，持久性是最难提供统一指导的（而大多数的指导都是类似‘买硬件’）相关的MySQL特性包括:

 - InnoDB doublewrite buffe,通过innodb_doublewrite配置选项开启或关闭。
 - innodb_flush_log_at_trx_commit 配置。
 - sync_binlog配置。
 - innodb_file_per_table配置。
 - 存储设备中的写缓冲,如硬盘,SSD或RAID阵列。
 - 存储设备中带电池后边的缓存
 - 运行MySQL的操作系统，特别是对于fsync()系统调用的支持。
 - 针对运行MySQL应用改的服务器与存储设备的不间断电源供应(UPS)。
 - 你的备份策略，如备份频率和备份类型,备份保留期。
 - 对于分布式或托管的数据应用，数据中心的一些特性，如MySQL服务器硬件的地址，数据中心之间的网络连接等。