---
title: mysql-14.1
date: 2017-10-30 17:02:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

##InnoDB简介

- 14.1.1 使用InnoDB表的好处
- 14.1.2 InnoDB表最佳实践
- 14.1.3 检查InnoDB是否可用
- 14.1.4 InnoDB测试
- 14.1.5 关闭InnoDB

&emsp;&emsp;InnDB是一种能平衡高可用性与高性能常规存储引擎。MySQL5.7将其作为默认的存储引擎，除非你配置了不同的默认引擎，否则在使用CREATE TABLE创建表而不指定引擎时将默认创建InnoDB表。

&emsp;&emsp;InnoDB的主要优点：

- 它的DML操作遵循ACID模型，提供有事务的提交回滚与崩溃恢复机制能够保障用户数据。见[14.2InnoDB与ACID模型]()了解更多。

- 行级别的锁与ORACLE式的一致性读提高了多用户的并发与性能。见[14.5InnoDB锁与事务模型]()了解更多。

- InnoDB根据主键将数据存储在硬盘上以提高查询。每张InnoDB表都会根据一个叫做聚簇索引的主键索引来组织数据已减少主键查询的I/O。见[14.8.2.1聚簇索引与二级索引]()了解更多。

- 为了保证数据的一致性，InnoDB支持外键约束。有了外键约束，insert,update与删除操作都会做检查以保证不会导致表见数据的不一致。见[14.8.1.6InnoDB与外键约束]()了解更多。

表14.1 InnoDB存储引擎特性


 Storage limits|64TB|Transactions|Yes|Locking granularity|Row
 ----|----|----|----|----|----
 MVCC|Yes|Geospatial data type support|Yes|Geospatial indexing support|Yes[a]
B-tree indexes|Yes|T-tree indexes|	No	|Hash indexes|	No[b]
Full-text search indexes	|Yes[c]	|Clustered indexes|	Yes	|Data caches|	Yes
Index caches|	Yes	|Compressed data|	Yes[d]|Encrypted data[e]|Yes
Cluster database support	|No |	Replication support[f]|	Yes	|Foreign key support|Yes
Backup / point-in-time recovery[g]	|Yes	|Query cache support|	Yes	|Update statistics for data dictionary|	Yes


[a]: http:// "InnoDB support for geospatial indexing is available in MySQL 5.7.5 and later."

[b]: http:// "InnoDB utilizes hash indexes internally for its Adaptive Hash Index feature."

[c]: http:// "InnoDB support for FULLTEXT indexes is available in MySQL 5.6.4 and later."

[d]: http:// "Compressed InnoDB tables require the InnoDB Barracuda file format."

[e]: http:// "Implemented in the server (via encryption functions). Data-at-rest tablespace encryption is available in MySQL 5.7 and later."

[f]: http:// "Implemented in the server, rather than in the storage engine."

[g]: http:// "Implemented in the server, rather than in the storage engine."

&emsp;&emsp;要与MySQL提供的其他引擎作比较，见15章[可选的存储引擎]()。

### InnoDB改进与新功能

&emsp;&emsp;要了解更多InnoDB的新功能，参见：


- InnoDB功能改进列表,1.4章 “MySQL 5.7的新功能”.

- 版本说明.

### 其他InnoDB相关的信息与资源

- InnoDB相关的术语与定义，参见MySQL词汇表。
- InnoDB引擎相关的论坛,见MySQL论坛InnoDB板块。
- InnoDB遵循与MySQL一样的GNU GPL2(1991年6月)。要了解关于MySQL授权的信息，见[http://www.mysql.com/company/legal/licensing/](http://www.mysql.com/company/legal/licensing/);