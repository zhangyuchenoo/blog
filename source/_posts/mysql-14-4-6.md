---
title: mysql-14.4.6
date: 2017-11-09 15:58:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.4.6 InnoDB数据字典

&emsp;&emsp;InnoDB数据字典由用于跟踪表，索引，表中列的元数据的的内部表组成。
元数据物理尚未与系统表空间中。由于历史的原因，数据字典元数据在某种程度上也与InnoDB表空间文件(.frm文件)有所重叠。