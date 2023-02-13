---
tags: [Mysql]
title: MySQL 导入导出
created: '2022-09-08T18:18:10.497Z'
modified: '2022-09-08T18:18:13.327Z'
---

# MySQL 导入导出

## 导出

```
# --set-gtid-purged=OFF 推荐加入 不配置 gtid
# --no-create-info 只有数据
# --no-data 只有结构
mysqldump --set-gtid-purged=OFF -hhostname -uusername -ppassword database tablename --where="id < 1000" > tablename.sql
```

## 导入

```
# 导入数据
mysql -h hostname -uusername -ppassword database < tablename.sql

# 修改一些环境变量避免报错
mysql -h hostname -uusername -ppassword database --init-command="SET SESSION FOREIGN_KEY_CHECKS=0;SET SQL_MODE='ALLOW_INVALID_DATES';" < filename.sql
```

怎么解决执行mysqldump出现SET @@SESSION.SQL_LOG_BIN等SQL的问题

开启了“gtid-mode=ON”参数。

如果一个数据库开启了GTID，使用mysqldump备份或者转储的时候，即使不是MySQL全库(所有库)备份，也会备份整个数据库所有的GTID号。

MySQL数据库在主从数据库进行导出备份和恢复的时候，需要注意是否启用数据库用GTID模式。

如果开启，则在mysqldump数据时，应该在mysqldump命令加上参数“–set-gtid-purged=OFF”。
