---
title: mysql-14.4.4
date: 2017-11-09 15:21:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.4.3重做日志缓冲

&emsp;&emsp;重做日志缓冲是内存中缓存写入重做日志的区域。重做日志缓冲大小通过innodb_log_buffer_size配置。重做日志缓冲会定期刷新到磁盘的日志文件中。大的重做日志缓冲能够使得大事务在提交前不用写重做日志到硬盘上。因此，如果你有更新插入或删除大量记录的事务，将重做日志缓冲调整大一些能够节省磁盘I/O。

&emsp;&emsp;innodb_flush_log_at_trx_commit选项控制重做日志缓冲的内容如何被写入日志文件，innodb_flush_log_at_timeout选项控制重做日志刷新的频率。