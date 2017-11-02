---
title: mysql-14.4.1
date: 2017-11-02 14:35:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.4.1缓冲池

&emsp;&emsp;缓冲池是内存中InnoDB用于缓冲被访问过的数据和索引数据的区域。缓冲池能够是数据能够使得常用的数据在内存中直接处理，从而加快处理速度。在专门的数据库服务器上，多达88%的物理内存被分配给InnoDB缓冲池。

&emsp;&emsp;为了高效读取大量数据，缓冲池被划分成能够存储多行记录的页。同时为了缓存管理的高效，缓冲池通过将页组成组成链表；使用LRU算法的一种，很少使用或者失效的数据将被移出缓存。

&emsp;&emsp;要了解更多，见 [14.6.3.1InnoDB缓冲池](),[14.6.3InnoDB缓冲池配置]()。