---
title: mysql-14.4.7
date: 2017-11-13 11:24:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

##14.4.7双写缓冲

&emsp;&emsp;双写缓冲是InnoDB缓冲池中数据在写到合适的数据文件前存储数据的区域。
只有在刷新数据到双写缓冲后，InnoDB才会将数据页写到合适的文件中。如果在写出缓冲页的过程中发生了操作系统，存储系统或者mysql进程的故障，InnoDB可以在之后的崩溃恢复过程中从双写缓冲中找到数据页的备份。

&emsp;&emsp;尽管数据被写出两次，但是双写缓冲不需要多达两倍的I/O操作。写入双写缓冲的数据都是一次调用fsync()写出大量数据。

&emsp;&emsp;大多数情况下，双写缓冲是默认开启的。要关闭双写缓冲，设置innodb_doublewrite为0.

&emsp;&emsp;如果表空间文件(ibdata文件)存储在支持原子写的Fusion-io设备上,双写缓冲会自动关闭，并且所有数据文件都会使用Fusion-io原子写。由于双写缓冲设置是全局的，对于粉笔在非Fusion-io设备上的数据，双写缓冲也会被禁用。 这一特性只支持Fusion-io设备且在linux上只有Fusion-io NVMFS才能开启。要更好运用这一特性，推荐使用O_DIRECT的innodb_flush_method设置。