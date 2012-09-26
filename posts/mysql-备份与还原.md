---
title: MySQL备份与还原
date: '2012-09-26'
description:
categories: lectures
tags: ['MySQL']
---

1 备份不等于复制：

+ 备份需要确定数据状态
+ 备份要保证数据一致性

2 保证数据一致性的方法：

+ 粗暴式方法：
1.停止正在进行的数据库实例
2.锁住所有的写操作
flush tables with read lock
+ 较好点的：
3.如果支持事务的存储引擎，利用事务取得一致性
repeatable read 
mvcc
4.快照（需要操作系统或第三方支持）
mysqldump 只备份结构而不备份数据

3 备份原因

+容灾---->高可用性避免
+容错---->避免【安全管理、权限管理、限制update/delete】
+存档---->注意与备份的区别，在以下方面【目的、保留时间、是否保留源数据】
+制作新库/测试

4 备份内容---只针对MYSQL而言

+ 用户数据库
+ 系统数据库：mysql
+ 日志文件：binlog,redo_log
+ 配置文件:my.cnf
+ 其他文件:master.info,slave_info
+ 系统文件:crontab

5 备份方式

选择因素：
+ 是否在线：冷热备份
+ 完整性：完全/增量/差异备份
+ 实现不同：逻辑/物理备份
+ 粒度不同：
+ 位置不同：本地/远程【同城、异地】
+ 时间不同：白天/晚上 备份多长时间
+ 自动化程度：手动/自动化工具
 
- 物理备份：数据文件块
- 逻辑备份：sql语句或带分隔符的文本文件
- 两者最大区别在于备份速速
 
6 备份工具

+ mysqldump
+ select into outfile
+ cp\rsync\scp\nc
+ xtrabackup\ibbackuo
+ 快照备份【LVM、XFS】

