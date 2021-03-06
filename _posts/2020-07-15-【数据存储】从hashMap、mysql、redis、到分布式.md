---
layout: post
title:  "从hashMap/mysql/redis/到分布式"
date:   2020-07-15 08:14:54
categories: 大数据
tags: 大数据
mathjax: true
---

* content
{:toc}

从hashMap/mysql/redis/到分布式




## 1 HashMap 

问题：从一个大数组(10000)中，找到特定的X。

通常的解答：都是循环遍历一遍，查找X，需要**全量IO**。

**优化**

把大数据量，分为小数据量(4个数字)的组合。组成了2500个4个小数组。

分而治之，依赖索引 / 路由 / hash 。

X 计算hash值。 hashcode % 2500

HashMap 或者 HashTable 的原理。

![image](/images/data-1.png)

优化后的查找过程：计算X的hash值，查找索引（左侧），定位X的大致位置，根据索引查找小数组。一定比全量IO快。

问题：可能数据大部分都落在一个hash值内，效率会降低。

总结：

1. 只做分治，不一定能够提速。
2. 大数据查询，要使用分治。


## 2 mysql

1. 为什么存数据，要出来这么多的技术？

答：因为要快。

2. 为什么快？谁快？谁慢？

![image](/images/data-2.png)

总结：

1. 同样的数据，存在文件中，速度慢。但是，存在数据库中，会快很多，为什么？答：下图解决。
2. 数据库高并发场景，都命中索引，慢？因为什么：吞吐量限制。IO 一秒返回64M，但是总共数据要返回 512M 。数据库有 TPS 、PVS 的阈值限制，超过之后就会很慢。--》分库、分表解决等。
3. 数据库的硬伤：磁盘IO。

答：datapage 4KB. 做分治。4K对齐。

HDFS block 大小。应该设置符合机器性能，不应该设置太大。

![image](/images/data-3.png)

二级索引的作用：避免全量IO（全量遍历索引）的过程。

问题：

1. 如果要查找的内容，没有做过索引？——也就是索引没有命中。那么就会走全量IO.

怎么解决数据库的问题？

答：数据有特征，不是每个数据每天都要查询，热数据的概念。只要保证热数据访问够快，用户感官上整理快。--》引出：redis、memcache（缓存）。


## 3 redis

问题1：redis为什么不支持sql。

答：因为redis存储的是部分数据，所以通过sql查询，结果不完整。


问题2：redis 为什么快？

答：以下三点：

1. worker 单线程模型，所以快。问题：单线程为什么就快？
2. 使用内存。
3. 6.x IO 使用多线程。

redis value有类型，每种类型有本地方法。

秒杀场景，一定要用redis。语境：高并发。

![image](/images/data-4.png)

四层LVS 或者 7层Ngnix反向代理

串行化：每个server请求加锁。

## HBase

区别于redis，能存全量数据。

以上所有内容整理自：马士兵教育，周志垒老师公开课。