---
title: mysql-14.4.9
date: 2017-11-16 17:02:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.4.9 单表单文件表空间(File-Per-Table Tablespaces)

&emsp;&emsp;单表单文件表空间是指单表的表空间在自己的数据文件中，而不在系统表空间中。当innodb_file_per_table选项开启时，表会在改表对应的数据文件中创建单表表空间，否则InnoDB表会在系统表空间中创建。没个单表单文件表空间表纤维一个.dba数据文件,默认创建在数据库目录下。

&emsp;&emsp;单表单文件表空间支持DYNAMIC与COMPRESSED行格式，这两种格式支持如变长数据的跨页(off-page storage)存储于表压缩。要了解更多这些特性，以及单表单文件表空间的优势，见[14.7.4InnoDB单表单文件表空间]()。