---
layout: post
title: Mysql技术内幕与InnoDB存储引擎读书笔记
categories:
  - Mysql
tags:
  - Mysql
  - InnoDB
---

## 一些可能有用的知识点

#### 可能会用到的命令
- 查看当前会话的隔离级别：`SELECT @@tx_isolation`
- 查看全局事物的隔离级别：`SELECT @@global.tx_isolation`

#### InnoDB 的 SERIALIABLE 隔离级别
在该隔离级别下，所有的 SELECT 语句会自动加上 LOCK IN SHARE MODE，因此，在这个隔离级别下，读占用了锁，对一致性非锁定读不再支持。该隔离级别一般用于 InnoDB 存储引擎的分布式事务

#### XA事物，分布式事物
分布式事务通过两段提交的方式来实现分布式事物，通过命令`SHOW VARIABLES LIKE 'innodb_support_xa'`来查看mysql是否启用了XA事务

#### 数据备份
类备份数据库，例如使用mysqldump的时候一定要加上 `--single-transaction` 参数以保证数据的一致性

热备份工具推荐：XtraBackup

#### 数据库测试工具
sysbench

## 一些尚不明白的问题

#### 尚未提交的事物也会写入redo日志，那么这些日志会被刷如磁盘么？

#### repeatable read隔离级别下的幻读

