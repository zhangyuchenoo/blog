---
title: mysql-14.4.3
date: 2017-11-07 10:06:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.4.3 自适应哈希索引

&emsp;&emsp;在分配给缓冲池充足的内存和一定的负载下，自适应哈希索引使得InnoDB更像一个内存数据库,且不会牺牲任何事务特性与可靠性。在数据库启动时，通过innodb_adaptive_hash_index选项开启，通过--skip-innodb_adaptive_hash_index选项关闭。

&emsp;&emsp;基于观察到的搜索模式，MySQL使用索引的前缀来构建哈希索引，索引前缀的长度可以是任意长度，且可能只有部分B树索引中的键值。哈希索引是为那些经常被访问的索引的页在需要时创建的。

&emsp;&emsp;如果一张表的大部分数据都能在内存中索引，哈希索引能够通过开启直接查询加速查询，使索引变成一种指针。InnoDB有监控索引搜索的机制，一旦InnoDB检测到建立哈希索引能够有利于查询，则其会自动创建哈希索引。

&emsp;&emsp;在一定负载下，建立哈希索引所带来的查询加速能够大大抵消创建哈希索引所带来的额外开销。有时候控制自适应索引的读写锁在大的负载下会成为争议的焦点。带‘%’的查询也有可能不会从哈希索引获益，对于不需要哈希索引的场景，关闭自适应索引能减少不必要的性能消耗。由于无法提前预估这一特性是否适合于特定系统，可以在真实的工作中通过开启或关闭这一特性进行测试。自MySQL5.6之后的架构变更是的更多的工作能够在不开启自适应索引的情景下工作，虽然这一特性任然默认开启。

&emsp;&emsp;在MySQL 5.7中自适应索引作了分区，每个索引分配到特定的分区，每一个分区被独立的latch保护。分区通过innodb_adaptive_hash_index_parts选项配置。在早期版本中，真个索引被同一个latch保护的状况使得在高负载场景下成为一个争议点。innodb_adaptive_hash_index_parts默认值为8，最大值是512.

&emsp;&emsp;哈希索引总是基于表中B树索引已有的索引。InnoDB能够根据B树中键值任意长度的前缀构建哈希索引，根据观察到的B树中的搜索模式。哈希索引可以只包含那些经常被访问的索引。


&emsp;&emsp;你通过SHOW ENGINE INNODB STATUS命令输出中SEMAPHORES区监控自适应索引的使用。如果你看到许多线程在等待btr0sea.c中创建的RW-latch，那很可能你应该关闭自适应索引。

&emsp;&emsp;要了解更多哈希索引的性能，参见[8.3.8B树索引与哈希索引的比较]()
For more information about the performance characteristics of hash indexes, see Section 8.3.8, “Comparison of B-Tree and Hash Indexes”.