---
title: mysql-14.5.2
date: 2017-08-10 15:24:03
tags:
  - mysql
categories:
  - tech
  - translate
  - mysql
---

<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

### 14.5.2InnoDB事务模型

  14.5.2.1[事务隔离级别](/tech/translate/mysql/mysql-14-5-2-1/)
  14.5.2.2[自动提交，提交与回滚](/tech/translate/mysql/mysql-14-5-2-2/)
  14.5.2.3[一致性无锁读](/tech/translate/mysql/mysql-14-5-2-3/)
  14.5.2.4[有锁读](/tech/translate/mysql/mysql-14-5-2-4/)

&emsp;&emsp;InnoDB事务模型的目标是尽力将多版本数据库的好的特性与两阶段(锁two-phase locking)结合起来。InnoDB执行行级锁并且在默认情况下使用非锁定一致性读来实现查询，像Oracle那样。在InnoDB中锁信息的存储是非常高效的(space-efficient)，因此锁提升并没有必要。一般情况下，服务器用户允许对InnoDB表中的每一行或者随机的一部分行加锁，且不会造成内存耗竭。
