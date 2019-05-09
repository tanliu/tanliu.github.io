---
layout: post
title: 理解Mysql底层B+tree 索引机制
date: 2019-05-07
tags:
   - mysql
---
#索引谁实现
todo: 补图
是由存储引擎来实现的如：MyISAM、 InnoDB等

# 索引的定义 
> 正确地创建合适的索引是提升数据库查询性能的基础

索引是为了加速对表中数据行的检索而创建的一种分散存储的数据结构
如下图所示：
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/D7A79F8DBEA6004FAA3E5B56EF51168E.jpg)
通过id对应数据行的硬盘地址。索引提作用是：

1. 索引大大减少了服务器需要扫描的数据量。
2. 索引可以帮助服务器避免排序和临时表。
3. 索引可以将随机I/O变为顺序I/O。


# 为什么选择B+tree
## 如果选择二叉树
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/EDA9EAC79E0E00737642241F12DF2F87.jpg)
二叉树如果数据分布是连续的时候，就会出现树的高度很高的问题;
## 如果选择平衡二叉树
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/A28590B2495833997E4A82FC2CB6ED96.jpg)
完全平衡二叉树的高度差不会超级过1，所以会出现rebalance。 

## 如果选择红黑树
红黑树 = 平衡二叉树 + 标记。同样会存在rebalance的问题


## 数据结构提总结
- 树的深度太深
- 每一个磁盘块使用率太低
- 没有很好地利用操作磁盘io的数据交换特性
- 没有利用好磁盘IO的预读能力。

## 如果选择B-tree
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/FB96BAED1D788CA85FC6ACC5F97818C8.jpg)

在排序上，做得不够好


# Mysql中B+ tree的体现
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/976B0FDED83B7E498CE4A21E6A4C490A.jpg)
- B+节点关键字搜索采用闭合区间
- B+非叶节点不保存数据相关信息，只保存关键字和子节点的引用
- B+关键字对应的数据保存在叶子节点中数据是有序的。
由于以上原因:
1. 扫表能力强（一个磁盘块可以加载很多地址数据）
2. 磁盘读写能力更强
3. 排序能力更强（本来就是带排序的列表）

# 索引相关知识

## 列的离散性
离散性越高，选择性越好。

## 最左匹配原则
> 对索引中关键字进行计算 ，一定是从左向右匹配


## 联合索引

- 单列索引
  - name
- 联合索引
  -name 、sex
1. 经常用的列优先
2. 选择性高的列优先
3. 宽度小的列优先

## 覆盖索引
如果查询列可通过索引节点中的关键字直接返回，刚索引称之为覆盖索引
优点：可以沽少数据库IO,将随机IO变为顺序IO，可以提高查询性能


## 问题
- 索引的数据长度能少就少（空间，匹配都成问题）
- 索引不是越多越好，一定要建合适的
- 匹配列前缀不一定可以用到索引like 1111%（离散性差的时候会用不上）,后缀是一定不可以用到索引的。
- where条件中not in 和<>用不索引（还是离散性差的问题）
- 匹配范围值，也是可以用到order by也可用到索引
- 多用指定列查询，只返回自己想到的数据列，少用select *
- 联合索引中如果不是按照索引最左列开始查找，无法使用索引
- 联合索引中精确匹配最左前列，范围匹配另外一列可以用到索引
- 联合索引中如果查询中有某个列的范围查询，则其右边的所有列都无法用到索引


# 索引选择策略
## 独立的列
例如，下面这个查询无法使用actor_id列的索引：
`mysql> SELECT actor_id FROM sakila.actor WHERE actor_id + 1 = 5;`

## 前缀索引和索引选择性
- 前缀索引：有时候需要索引很长的字符列，这会让索引变得大且慢。可以截取一部分作为索引。这样可以节省空间也提交查询效率
- 索引选择性：不重复的索引值（也称为基数，cardinality）和数据表的记录总数（#T）的比值，范围从1/#T到1之间，索引的选择性越高则查询效率越高（TODO：查看命令）
注：MySQL无法使用前缀索引做ORDER BY和GROUP BY，也无法使用前缀索引做覆盖扫描。
## 多列索引
- 用到多列的时候考虑建立联合索引
- 考虑多列索引的顺序（最左匹配原则）

- 将选择性最高的列放到索引最前列。这个建议有用吗?在某些场景可能有帮助，但通常不如避免随机IO和排序那么重要

## 聚簇索引
> 如果没有定义主键，InnoDB会选择一个唯一的非空索引代替。

`UNIQUE KEY rental_date (rental_date,inventory_id,customer_id),`
### 以下是可以使用索引排序的
```
mysql> EXPLAIN SELECT rental_id, staff_id FROM sakila.rental
        -> WHERE rental_date = '2005-05-25'
        -> ORDER BY inventory_id, customer_id\G
    *************************** 1. row ***************************
             type: ref
    possible_keys: rental_date
              key: rental_date
             rows: 1
            Extra: Using where
```

### 以下是不可以使用提索引排序的
- 下面这个查询使用了两种不同的排序方向，但是索引列都是正序排序的：

    `... WHERE rental_date = '2005-05-25' ORDER BY inventory_id DESC, customer_id ASC;`
- 下面这个查询的ORDER BY子句中引用了一个不在索引中的列：

    `... WHERE rental_date='2005-05-25' ORDER BY inventory_id，staff_id;`
- 下面这个查询在索引列的第一列上是范围条件，所以MySQL无法使用索引的其余列：

   ` ... WHERE rental_date > '2005-05-25' ORDER BY inventory_id, customer_id;`
- 这个查询在y inventory_id列上有多个等于条件。对于排序来说，这也是一种范围查询：

    `... WHERE rental_date = '2005-05-25' AND inventory_id IN(1,2) ORDER BY customer_
    id;`
    
## 冗余和重复索引
- 如果创建了索引（A，B），再创建索引（A）就是冗余索

## 索引和锁
InnoDB只有在访问行的时候才会对其加锁，而索引能够减少InnoDB访问的行数，从而减少锁的数量

# 索引相关书：
强烈建议阅读由Tapio Lahdenmaki和Mike Leach编写的Relational Database Index Design and the Optimizers（Wiley出版社）一书，该书详细介绍了如何计算索引的成本和作用、如何评估查询速度、如何分析索引维护的代价和其带来的好处等。







