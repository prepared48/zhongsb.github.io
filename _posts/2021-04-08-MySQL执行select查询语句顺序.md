---
layout: post
title:  "MySQL执行select查询语句顺序"
date:   2021-04-08 13:27:54
categories: 数据库 MySQL
tags: MySQL 数据库
mathjax: true
---

* content
{:toc}

SQL 操作顺序——MySQL 执行查询的顺序

知道SQL执行查询的顺序很重要，主要体现在：知道SQL执行查询顺序，可以简化写新的查询语句的过程，对于优化SQL也很有帮助。

如果你在找一个简化的顺序版本，这就是操作的逻辑顺序：

1. FROM，包括JOINs
2. WHERE
3. GROUP BY
4. HAVING
5. WINDOW functions
6. SELECT
7. DISTINCT
8. UNION
9. ORDER BY
10. LIMIT and OFFSET

但是事实没有这么简单直接。就像我们说的，这个SQL标准定义了执行不同query查询语句的顺序。但是，现在有些数据库，通过应用一些优化技巧，已经在挑战这个默认顺序，

这样的话，就有可能改变这个默认的执行顺序，虽然他们必须返回默认的执行顺序下执行一样的结果。


为什么他们要这么做呢？嗯，如果数据库必须在查看where语句和它们的索引之前，先获取到from语句提到的表的所有数据，这是愚蠢的。

这些表可能存储非常多的数据，这样你就能想象如果数据库的优化器坚持传统的SQL查询语句执行顺序会发生什么了。


## FROM 和 JOINs

将首先评估FROM语句中指定的表（包括JOIN指定的表），以确定与查询相关的整个工作集。数据库会根据JOINs ON 语句合并所有相关表的数据，

同时还会从自查询中获取数据，甚至有可能创建一些临时表来存储从子查询中返回的数据。






















https://www.eversql.com/sql-order-of-operations-sql-query-order-of-execution/