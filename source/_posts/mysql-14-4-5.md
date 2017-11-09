---
title: mysql-14.4.5
date: 2017-11-09 15:25:20
tags:
 - mysql
categories:
 - tech
 - translate
 - mysql
---


<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

## 14.4.5系统表空间

&emsp;&emsp;InnoDB系统表空间包括InnoDB数据字典(InnoDB相关对象的元数据)，也是双写(doublewrite)缓冲，修改缓冲(change buffer)与撤销日志(undo log)的存储区域。系统表空间还包括用户在系统表空间创建的表及索引。系统表空间是夺标共享的表空间。

&emsp;&emsp;系统表空间表现为一份或多分数据文件。默认情况下，一份叫做ibdata1的文件在MySQL数据目录中被创建。数据文件的大小与数量通过innodb_data_file_path选项在启动时设置。

&emsp;&emsp;要了解更多相关信息，见[14.6.1InnoDB启动配置](),[14.7.1变更InnoDB系统空间大小]()。