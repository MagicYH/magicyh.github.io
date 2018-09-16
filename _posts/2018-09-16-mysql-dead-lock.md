---
layout: post
title: Mysql死锁案例分析
categories:
  - Mysql
tags:
  - Mysql
---

本文仅记录某次在工作中实际遇到的死锁的例子，记录当时分析问题的过程，并由此引出对Innodb存储引擎加锁的原理的一些理解

#### 表结构
```
CREATE TABLE `dead_lock` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `id1` int(10) unsigned NOT NULL,
  `id2` int(10) unsigned NOT NULL,
  `name` varchar(16) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx1` (`id1`),
  UNIQUE KEY `idx2` (`id2`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 事务隔离级别
**RR (Repeatable Read)**

#### 死锁语句
```
delete from dead_lock where id2 = 1

insert into dead_lock set id1 = 1, id2 = 1, name = '1' on duplicate key update name = '01'
```

#### 死锁日志
```
LATEST DETECTED DEADLOCK
------------------------
2018-09-16 12:03:35 0x7f1e840ac700
*** (1) TRANSACTION:
TRANSACTION 1987, ACTIVE 4 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1136, 3 row lock(s), undo log entries 1
MySQL thread id 5, OS thread handle 139769041311488, query id 480 localhost root update
insert into dead_lock set id1 = 1, id2 = 1, name = '1' on duplicate key update name = '01'
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 30 page no 5 n bits 72 index idx2 of table `test`.`dead_lock` trx id 1987 lock_mode X waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 4; hex 00000001; asc     ;;
 1: len 4; hex 00000001; asc     ;;

*** (2) TRANSACTION:
TRANSACTION 1986, ACTIVE 17 sec updating or deleting
mysql tables in use 1, locked 1
4 lock struct(s), heap size 1136, 3 row lock(s), undo log entries 1
MySQL thread id 4, OS thread handle 139769041045248, query id 478 localhost root updating
delete from dead_lock where id2 = 1
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 30 page no 5 n bits 72 index idx2 of table `test`.`dead_lock` trx id 1986 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 4; hex 00000001; asc     ;;
 1: len 4; hex 00000001; asc     ;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 30 page no 4 n bits 72 index idx1 of table `test`.`dead_lock` trx id 1986 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 4; hex 00000001; asc     ;;
 1: len 4; hex 00000001; asc     ;;

*** WE ROLL BACK TRANSACTION (2)
```

#### 测试实验
##### 实验1
```
# session-1:
begin;
delete from dead_lock where id2 = 1

# session-2
begin;
insert into dead_lock set id1 = 1, id2 = 1, name = '1' on duplicate key update name = '01'
```
查看`INNODB_LOCKS`表可以看到，第二条sql正在等待给idx1加锁

##### 实验2
```
# session-1:
begin;
delete from dead_lock where id1 = 1

# session-2
begin;
insert into dead_lock set id1 = 1, id2 = 1, name = '1' on duplicate key update name = '01'
```
查看`INNODB_LOCKS`表可以看到，第二条sql正在等待给idx1加锁

##### 实验3
```
# session-1:
begin;
delete from dead_lock where id2 = 1

# session-2
begin;
insert into dead_lock set id2 = 1, id1 = 1, name = '1' on duplicate key update name = '01'
```
查看`INNODB_LOCKS`表可以看到，第二条sql正在等待给idx1加锁

##### 实验4
```
# session-1:
begin;
insert into dead_lock set id1 = 1, id2 = 1, name = '1' on duplicate key update name = '01'

# session-2
begin;
delete from dead_lock where id2 = 1
```
查看`INNODB_LOCKS`表可以看到，第二条sql正在等待给idx2加锁

##### 实验5
```
# session-1:
begin;
insert into dead_lock set id1 = 1, id2 = 1, name = '1' on duplicate key update name = '01'

# session-2
begin;
delete from dead_lock where id1 = 1
```
查看`INNODB_LOCKS`表可以看到，第二条sql正在等待给idx1加锁

##### 实验6
```
# session-1:
select * from dead_lock where where id = 1 for update

# session-2:
begin;
delete from dead_lock where id2 = 1
```
查看`INNODB_LOCKS`表可以看到，第二条sql正在等待给`PRIMARY`加锁

##### 实验7
```
# session-1:
select * from dead_lock where where id = 1 for update

# session-2:
begin;
insert into dead_lock set id1 = 1, id2 = 1, name = '1' on duplicate key update name = '01'
```
查看`INNODB_LOCKS`表可以看到，第二条sql正在等待给`PRIMARY`加锁

##### 实验8
```
# session-1:
select * from dead_lock where where id = 1 for update

# session-2:
begin;
delete from dead_lock where id2 = 1

# session-3
begin;
insert into dead_lock set id1 = 1, id2 = 1, name = '1' on duplicate key update name = '01'

# session-1
rollback
```
死锁，在session-1回滚前，session-2和session-3都在等待`PRIMARY`

##### 实验9
```
# session-1:
begin;
delete from dead_lock where id2 = 1

# session-2
begin;
insert into dead_lock set id1 = 3, id2 = 1, name = '1' on duplicate key update name = '03', id1 = 3

# session-1
rollback
```
查看`INNODB_LOCKS`表可以看到，第二条sql正在等待给idx2加锁


#### 死锁分析
死锁的原理就不详细说了，这里的两条语句之所以会产生死锁是因为加锁的顺序不一致导致的。因此首先我们来看看这两条sql的加锁顺序是怎么样的

通过实验1、2、3、7可以看出insert语句的加锁顺序是先给idx1加锁，并且其加锁的顺序与sql的写法无关，而是与索引的声明顺序有关，此处可以通过交换表的索引声明顺序予以验证

而通过实验4、5、6，可以知道delete语句的加锁顺序则是与其where条件有关，where条件使用了哪个索引进行查询，则会优先给相应的索引加锁，并且其加锁的顺序是先加检索使用到的索引，然后再给主键加锁（猜测：删除操作最终会给所有的索引加锁，并且这个加锁的顺序可以再研究研究，个人猜测很可能是按照索引声明的顺序）

最后根据实验8，复现我们的死锁场景，这里大概说明一下，session-1中的加锁操作是为了阻塞后面的两条更新语句，待后面两条语句都执行并且阻塞以后，回滚session-1，制造一种并发访问的场景，此时发生死锁

*delete from dead_lock where id2 = 1*
1. idx2加锁
2. 主键 id 加锁
3. idx1加锁

*insert into dead_lock set id1 = 1, id2 = 1, name = '1' on duplicate key update name = '01'*
1. idx1加锁
2. 主键 id 加锁
3. idx2加锁

> 困惑：这里的`insert on duplicate key update`为什么不是先给idx1，idx2加锁，然后给主键加锁，而是idx1加锁完后给主键加锁，最后给idx2加锁<br>
> <br>
> 解答：可能是因为两个唯一所以导致了误解，mysql中的`on duplicated key`因该是满足一个就开始走更新流程了，因此，两个唯一索引中只要满足一个就会启动更新的流程，因此这里实际上是因为idx1已经满足了更新的条件，因此就开始给主键加锁走更新步骤了，此时idx2的唯一性还根本没有得到判断。通过实验9就可以得到验证，inset 语句只满足了idx2是重复的，因此进行的是更新操作，而首先加锁的索引是最先通过条件判断相等的id2。因此，这个实验也从侧面说明了，一张表有多个唯一键的时候一定要特别注意其sql的更新逻辑

看到这个加锁顺序以后，我们因该就很好理解为什么会产生死锁了，假设两条sql并发执行，第一条sql获取了id2和主键的锁，此时要给id1加锁，此时第二条sql已经获取了id1的锁，要给id2和主键加锁，此时就产生了死锁。

明白了这个原理只有，要修改就比较简单了，把删除语句修改为：

`delete from dead_lock where id1 = 1`

保证两条sql的加锁顺序是一致的即可


#### 总结
1. 有更新操作时，主键一定会加锁（也可以理解为，因为Innodb时聚簇索引，行数据与主键数据存放在一起，因此一旦有数据更新，则主键、也就是那一行记录是一定要加锁的）
2. 其他索引则在需要更新时才加锁


> 参考文献：<br>
> [http://hedengcheng.com/?p=771](http://hedengcheng.com/?p=771)<br>
> [http://hedengcheng.com/?p=844](http://hedengcheng.com/?p=844)