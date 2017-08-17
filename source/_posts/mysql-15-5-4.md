---
title: mysql-15.5.4
date: 2017-08-15 16:57:53
tags:
 - mysql
categories:
  - tech
  - translate
  - mysql
---

<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.5.4幻影行

&emsp;&emsp;所谓的幻读问题是指在同一个事物中，不同时间的同一个查询返回不同的结果。例如，一个SELECT语句被执行了两次，但是第二次查询返回的某一行在第一次查询中没有返回，这一行就叫做“幻影”行。

 &emsp;&emsp;假如child表的id列上有索引，你想要读取并锁定该表中id值大于100的行，稍候对其中的一些行进行更新：

>SELECT * FROM child WHERE id > 100 FOR UPDATE;

 &emsp;&emsp;该查询从id大于100的记录索引开始扫描，假如表中有id为90和102的记录，而如果只对扫描范围中的记录加锁而不对区间加锁(在本例中为90到102的区间)，另一个会话就可以在该区间插入一条id为101的记录。而如果你再同一个事务中再次执行之前的SELECT语句，就会发现id为101的新记录(幻影行)。如果我们将一些行的集合作为一个数据项，那么这一新的幻影行就会违反在一次事务中读取的事务不应改变的隔离原则。

&emsp;&emsp;为了防止幻读现象，InnoDB使用结合了行锁和区间锁的Next-Key锁。InnoDB在查询或扫描表索引时，会在相应的索引记录上设置共享或排它锁，也就是说在行级别上的锁实际上是索引记录锁(index-record lock)，另外在索引记录上的Next-key锁还会影响该索引记录前边的区间，如果一个会话拥有一条记录R的共享或者排他记录锁，其他的会话是不能在此时向R之前的区间插入记录的。

&emsp;&emsp;当InnoDB扫描索引的时候，也可能锁住最后一条记录索引之后的区间，就像在之前的例子中，为了防止其他会话插入id大于100的记录，InnoDB也会在id大于102的区间加锁。

&emsp;&emsp;你可以使用Next-key锁来实现唯一检查：假如使用共享模式读取数据并且没有和将要插入行重复的数据，那么你可以安全地插入该行，因为在读取该行的时候设置的Next-key锁可以阻止其他的会话插入重复的数据，也就是说，Next-key锁使你可以锁住表中表中不存在的记录。

&emsp;&emsp;正如[14.5.1InnoDB锁](/tech/translate/mysql/mysql-14-5-1/)中所讨论的，区间锁可以禁用，当然这会导致幻读问题因为其他的session可以插入记录到区间中。
