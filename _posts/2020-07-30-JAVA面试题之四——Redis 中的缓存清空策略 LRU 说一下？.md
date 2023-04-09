---
layout: post
title:  "JAVA面试题之四——Redis 中的缓存清空策略 LRU 说一下？"
date:   2020-07-30 06:47:54
categories: 面试
tags: Java Redis
mathjax: true
---

* content
{:toc}

Redis 的缓存清空策略了解嘛？LRU 是什么？能说一下原理嘛？



==LRU== 是一种缓存淘汰策略。常见的策略有三种：先进先出策略 FIFO（First In，First Out）、最少使用策略 LFU（Least Frequently Used）、==最近最少使用策略 LRU==（Least Recently Used）

## 方案一：使用链表实现 LRU

思路是这样的：维护一个有序单链表，越靠近链表尾部的结点是越早之前访问的。当有一个新的数据被访问时，我们从链表头开始顺序遍历链表。

- 1. 如果此数据之前已经被缓存在链表中了，我们遍历得到这个数据对应的结点，并将其从原来的位置删除，然后再插入到链表的头部。O(N)
- 2. 如果此数据没有在缓存链表中，又可以分为两种情况：如果此时缓存未满，则将此结点直接插入到链表的头部；如果此时缓存已满，则链表尾结点删除，将新的数据结点插入链表的头部。O(N)

当要缓存某个数据的时候，先在链表中查找这个数据。如果没有找到，则直接将数据放到链表的尾部；如果找到了，我们就把它移动到链表的尾部。因为查找数据需要遍历链表，所以单纯用链表实现的 LRU 缓存淘汰算法的时间复杂很高，是 O(n)。

## 方案二：使用数组实现 LRU

思路：数组首位保存最新访问数据、末位置优先被清理

- 1. 如果此数据之前已经被缓存在数组中了，找到对应的数据将其插入到首位，并从原位置删除，插入到首位需要移动剩余所有数据，时间复杂度O(N);
- 2. 如果此数据没有在数组中，分两种情况：如果此时缓存未满，则将节点直接插入到数组首位；如果此时缓存已经满了，则删除末尾节点，将新的数据插入到首位，时间复杂度O(N)



## 方案三：使用散列表优化第一种方法

散列表是什么？

散列表英文是 “Hash Table", 也叫”Hash 表“或者”哈希表“。

散列表用的是数组支持按照下标随机访问数据的特性，所以==散列表其实就是数组的一种扩展==，由数组演化而来。可以说，如果没有数组，就没有散列表。

**LRU 优化**

![image](/images/lru.png)

==散列表+双向链表==的形式。——》LinkedHashMap也是这种结构。查看 LinkedHashMap 源码就知道 JAVA 是怎么实现 LRU 了。

**查找数据**：散列表查找数据时间复杂度接近 O(1)，如果存在散列冲突，时间复杂度会上升。


**删除数据**：找到数据所在的节点，然后将其删除。删除时间复杂度为O(1)。

**添加数据**：1）如果数据在缓存中了，将其启动到双向链表的尾部；2) 如果不在缓存中，看缓存有没有满: a) 如果满了，将双向链表的头部节点删除，然后将数据插入到双向链表的尾部；b) 如果没有满，直接插入到链表尾部。


## Redis 中是怎么实现 LRU 的

Redis 中 LRU 有两种。1）所有key都参与 LRU 算法的策略；2）只包含设置了过期时间的 key 参与 LRU 。

- allkeys-lru: evict keys by trying to remove the less recently used (LRU) keys first, in order to make space for the new data added.
- volatile-lru: evict keys by trying to remove the less recently used (LRU) keys first, but only among keys that have an expire set, in order to make space for the new data added.

==Redis 中的 LRU 算法==并不是一个完整的 LRU 算法，==只是一种近似==。Redis 并不能选出那个最长空闲时间的 key 进行删除，他会在局部（选择的样本 keys）删除空闲时间最长的那个。

Redis 不实现完整的 LRU 算法，是因为完整的 LRU 算法太消耗内存了。近似实现在 Redis 中实际上是等效的。

Redis 3.0 做了==升级==，能实现一个更近似的 LRU 算法。

可以通过设置一个参数，进行调整 LRU 策略。参数如下：

    maxmemory-samples 5

下图比较真实 LRU 算法和 Redis 2.8和3.0版本中的 LRU.

![image](/images/lru2.png)

Redis 3.0 设置 `maxmemory-samples 10` 最接近真实 LRU.

参考：

1. Redis 官方文档：https://redis.io/topics/lru-cache；
2. 王争，极客时间《数据结构与算法之美》06讲 https://time.geekbang.org/column/article/41013
