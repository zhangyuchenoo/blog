---
title: mysql-14.4.2
date: 2017-11-02 14:45:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.4.2 Change Buffer

&emsp;&emsp;变更缓存是用于缓存二级缓存页变更的特殊数据结构，当发生INSERT, UPDATE, or DELETE等DML操作时，相应的数据页不在缓冲池（buffer pool）中，对于二级缓存的变更将缓存起来，等到有其它的读取操作读入数据页时，这些操作将被合并到数据页中。

&emsp;&emsp;不像聚簇索引，二级索引通常都不是唯一的，并且插入二级索引是相对随机的。类似地，删除域更新操作也会影响索引树中并不相邻的二级索引页。变更缓存将变更时间延后，在受影响的页被其他操作读入缓冲池后，避免了大量从硬盘随机读取二级索引页的I/O请求。

&emsp;&emsp;在系统空闲或者正常关闭(slow shutdown)时运行的purge操作会定期将更新后的索引页写入硬盘。相比每次索引变更就立即写入硬盘，purge操作能够一次写入包含多个连续索引的多个分区。

&emsp;&emsp;在设计大量二级索引更新或者大量行变更的时候，修改缓冲合并可能会花费数小时时间。在合并过程中，硬盘I/O将增加，从而可能导致查询速度显著下降。修改缓冲的合并也可能在事务提交后继续进行，事实上合并还可能在服务停止或者重启后，见[14.21.2InnoDB强制恢复]()了解更多。

&emsp;&emsp;在内存中修改缓冲占用部分缓冲池，在硬盘上则是表空间的一部分，因此在数据库重启后索引变更仍然被缓存着。

&emsp;&emsp;修改缓冲中的数据通过innodb_change_buffering配置管理，要了解更多见[14.6.5配置修改缓冲]()，也可以参考[14.6.5.1设置最大修改缓冲]()。

### 修改缓冲监控

&emsp;&emsp;通过以下方式可以监控修改缓冲:

&emsp;&emsp;InnoDB标准监控输出包括修改缓冲的信息。要查看监控数据，执行SHOW ENGINE INNODB STATUS命令。

	mysql> SHOW ENGINE INNODB STATUS\G

&emsp;&emsp;修改缓冲信息在INSERT BUFFER AND ADAPTIVE HASH INDEX下边，如下：

	-------------------------------------
	INSERT BUFFER AND ADAPTIVE HASH INDEX
	-------------------------------------
	Ibuf: size 1, free list len 0, seg size 2, 0 merges
	merged operations:
	 insert 0, delete mark 0, delete 0
	discarded operations:
	 insert 0, delete mark 0, delete 0
	Hash table size 4425293, used cells 32, node heap has 1 buffer(s)
	13577.57 hash searches/s, 202.47 non-hash searches/s

&emsp;&emsp;要了解更多，见[14.17.3InnoDB标准监控输出与锁监控输出]()。

&emsp;&emsp;INFORMATION_SCHEMA.INNODB_METRICS表提供了标准监控输出中的大部分数据，和其他数据，要查看修改缓冲的统计数据，执行下边的查询语句:

	mysql> SELECT NAME, COMMENT FROM INFORMATION_SCHEMA.INNODB_METRICS WHERE NAME LIKE '%ibuf%'\G

&emsp;&emsp;要了解INNODB_METRICS表的使用信息，见[14.15.6InnoDB INFORMATION_SCHEMA Metrics表]()

&emsp;&emsp;INFORMATION_SCHEMA.INNODB_BUFFER_PAGE表提供了缓冲池中每页的元数据，包括修改缓冲索引与修改缓冲位图页。修改缓冲也通过PAGE_TYPE标识，修改缓冲索引页的类型是IBUF_INDEX，修改缓冲位图页的类型是IBUF_BITMAP。

	注意
 	查询INNODB_BUFFER_PAGE表将对数据库带来性能影响，为了不影响数据库性能，请在测试环境的测试实例上实践。


&emsp;&emsp;比如，你可以查询INNODB_BUFFER_PAGE来决定IBUF_INDEX与IBUF_BITMAP页在总的缓冲页中所占的比例。

	mysql> SELECT (SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE
	       WHERE PAGE_TYPE LIKE 'IBUF%') AS change_buffer_pages, 
	       (SELECT COUNT(*) FROM INFORMATION_SCHEMA.INNODB_BUFFER_PAGE) AS total_pages,
	       (SELECT ((change_buffer_pages/total_pages)*100)) 
	       AS change_buffer_page_percentage;
	+---------------------+-------------+-------------------------------+
	| change_buffer_pages | total_pages | change_buffer_page_percentage |
	+---------------------+-------------+-------------------------------+
	|                  25 |        8192 |                        0.3052 |
	+---------------------+-------------+-------------------------------+

&emsp;&emsp;要了解INNODB_BUFFER_PAGE表提供的其他信息，见[24.31.1INNODB_BUFFER_PAGE表]()；了解更多，要了解如何查询，见[14.15.5InnoDB INFORMATION_SCHEMA缓冲池相关的表]()。

emsp;&emsp;Performance Schema提供了修改缓冲互斥等待实现高级监控，要查看请执行以下语句：

	mysql> SELECT * FROM performance_schema.setup_instruments
	       WHERE NAME LIKE '%wait/synch/mutex/innodb/ibuf%';
	+-------------------------------------------------------+---------+-------+
	| NAME                                                  | ENABLED | TIMED |
	+-------------------------------------------------------+---------+-------+
	| wait/synch/mutex/innodb/ibuf_bitmap_mutex             | YES     | YES   |
	| wait/synch/mutex/innodb/ibuf_mutex                    | YES     | YES   |
	| wait/synch/mutex/innodb/ibuf_pessimistic_insert_mutex | YES     | YES   |
	+-------------------------------------------------------+---------+-------+

emsp;&emsp;要了解更多关于监控InnoDB互斥等待的信息，见[14.16.2使用Performance Schema监控InnoDB互斥等待]()
