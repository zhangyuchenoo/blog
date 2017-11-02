---
title: mysql-14.4
date: 2017-11-02 14:34:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.4 InnoDB Architecture

 - 14.4.1[缓冲池]()
 - 14.4.2[变更缓冲]()
 - 14.4.3[自适应哈希索引]()
 - 14.4.4[重做日志缓冲]()
 - 14.4.5[系统表空间]()
 - 14.4.6[InnoDB数据字典]()
 - 14.4.7[双重写缓冲]()
 - 14.4.8[撤销日志Undo Log]()
 - 14.4.9[ File-Per-Table Tablespaces]()
 - 14.4.10[普通表空间General Tablespaces]()
 - 14.4.11[撤销表空间 Undo Tablespace]()
 - 14.4.12[临时表空间 Temporary Tablespace]()
 - 14.4.13[重做日志 Redo Log]()

&emsp;&emsp;本小节将对InnoDB存储引擎架构的主要组件做介绍。