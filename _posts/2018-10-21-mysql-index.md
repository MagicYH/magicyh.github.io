---
layout: post
title: Mysql技术：索引与锁
categories:
  - Mysql
tags:
  - Mysql
---

## Innodb索引
Innodb种的索引是使用B+树进行存储的，因此我们首先来了解一下B+树的数据结构

### B+树

一般索引的数据结构是B+树，B+树的数据存储结构如下图所示

![B+树](https://s1.ax1x.com/2018/10/21/iBcc0x.png)

如图中所示，B+树种所有的数据都是按顺序存储的，并且其中的每一块数据都是存放在Innodb的页数据中的（Innodb一页数据是16k大小），这样联讯存放的数据能提高机械硬盘读取数据的效率。由于每个节点能够存放大量的数据，因此Innodb索引生成的B+树的高度一般不会很高（不超过4层）。

为了保持树的平衡，B+树在插入、删除数据的时候可能会进行叶子节点数据的拆分与合并操作，同时如果非叶子节点的数据存储不下的话会分裂出新的非叶子节点，此时B+树的高度会增加。具体细节这里不做过多描述。

### InnoDB索引结构

#### 聚集索引
Innodb的每张表都是按照聚集索引的方式存储的，就是按照每张表的主键构造一颗B+树，同时叶子节点中存放整张表中的行记录数据（如果没有显示声明主键，则会自动生成隐式的主键）。构造一张测试表，并且插入4条数据。其中b字段的长度为7000，这样可以人为的让Innodb每一页的只能存放两条数据（一页数据能保存16k数据）。
```
CREATE TABLE t (
    a INT NOT NULL,
    b VARCHAR(8000),
    c INT NOT NULL,
    PRIMARY KEY (a),
    KEY idx_c (c)
) ENGINE=INNODB;

INSERT INTO t SELECT 1, REPEAT('a', 7000), -1;
INSERT INTO t SELECT 2, REPEAT('a', 7000), -2;
INSERT INTO t SELECT 3, REPEAT('a', 7000), -3;
INSERT INTO t SELECT 4, REPEAT('a', 7000), -4;
```

则数据的存储结构如下图所示，并且因为通过主键能够直接找到存储行数据的地址，因此通过主键进行数据查找的效率是最高的，因此一定要合理利用主键

![Innodb数据存储结构](https://s1.ax1x.com/2018/10/21/iBL6Rs.png)

#### 辅助索引
辅助索引也叫做非聚集索引，每一个辅助索引都是一颗单独的树，无论索引中包括表中多少个字段。辅助索引的构造与聚集索引类似，只是叶子节点中存储的数据是主键索引的值。可以把辅助索引看作是以辅助索引为主键，主键值为行数据的一张表。如果索引只包含一列则索引为该列，如果索引是复合索引，则会将复合索引使用到的列，按照引用的顺序组成一个值进行索引。例如复合索引有a、b、c三列，则在辅助索引中的存放形式可以表示为(a, b, c)。索引的排序则是先a，a相等的情况下找b排序，最后是c。

通过辅助索引查找数据时，则首先通过辅助索引找到相应的数据行的主键，再根据主键找到相应的数据行。也就是说，假设某张表的B+树(辅助索引和主键索引都是)高度是3，则通过主键索引查找一条数据需要查找3次数据页。而通过辅助索引查找一行数据，首先需要通过辅助索引查找3次数据页获得主键ID，然后再在主键索引上查找3次数据页最终得到行数据，即一共需要查找6次数据页。

辅助索引的存储结构如下图所示：
![Innodb辅助索引存储结构](https://s1.ax1x.com/2018/10/21/iBx1CF.png)

### InnoDB索引的一些特性

举例用的表结构：
```
CREATE TABLE buy_log (
    id INT NOT NULL AUTO_INCREMENT,
    userid INT NOT NULL,
    date DATE,
    ts INT NOT NULL,
    PRIMARY KEY (id),
    KEY idx_user_date (userid, date)
) ENGINE=INNODB;

INSERT INTO buy_log (userid, date, ts) VALUES (1, '2018-10-01', 1), (1, '2018-11-11', 2), (2, '2018-10-02', 3), (3, '2018-11-10', 4);
```

1. 因为辅助索引是排好序的，因此对于一些有排序操作的sql，如果索引覆盖了的话，可以不用进行额外的排序操作，如果执行sql：`EXPLAIN SELECT * FROM buy_log WHERE userid = 1 ORDER BY date ASC` 可以发现执行计划中不包含`Using filesort`，因为`date`字段在索引中已经排好序了，因此可以直接给出排序结果

2. 当`SELECT`中选择的字段完全囊括在辅助索引中时，则查询可以直接从辅助索引中给出结果，而不再需要去查询聚簇索引。因为辅助索引的数据量要小得多，每一页能够存储更多的数据，因此能够减少IO操作频次，提高查询效率。另外对于某些统计查询，例如上边提到的表执行`SELECT COUNT(*) FROM buy_log`，则查询优化器会选择非聚集索引而非聚集索引，通过执行计划查看的话，可以发现，虽然`possible_keys`为NULL，但是实际会使用`idx_user_date`。类似的，执行`SELECT COUNT(*) FROM buy_log WHERE date > '2018-10-01'`时正常情况下是不会使用索引的，但实际情况依旧会使用`idx_user_date`。

## Innodb锁

### Innodb锁类型

Innodb上的锁类型应该包括表级锁、页级锁、行级锁

1. 表锁和页锁：
    - 共享锁（S Lock）
    - 排他锁（X Lock）
    - 意向共享锁（IS Lock）
    - 意向拍他锁（IX Lock）

2. 行锁：
    - 共享锁（S Lock），允许对行数据进行读操作
    - 排他锁（X Lock），允许对行数据进行写操作





> 参考资料《Mysql技术内幕 Innodb存储引擎》