---
layout: post
title:  "MySQL之Json类型"
date:   2021-07-13 09:11:54
categories: 数据库
tags: 数据库 MySQL
mathjax: true
---

* content
{:toc}


MySQL 5.7 之后提供了Json类型，是MySQL 结合结构化存储和非结构化存储设计出来的一个类型。

在某些场景下，Json 类型简直是福音。





## 1 Json 类型简介

MySQL 5.7 之后提供了Json类型，是MySQL 结合结构化存储和非结构化存储设计出来的一个类型。

在某些场景下，Json 类型简直是福音。

场景1: 用户画像，描述用户的标签等类似场景，比如互联网医院类系统的患者健康档案，有很多信息不是必填项，如：身高、体重、三围等等信息，可以使用 Json 存储。

场景2: 游戏类场景；

场景3: 存储图片等附属信息，比如图片的分辨率，图片标题等。

## 2 让我们看看Json怎么用的

创建表，并插入数据

```sql
CREATE TABLE UserLogin (
    userId BIGINT NOT NULL,
    loginInfo JSON,
    PRIMARY KEY(userId)
);
INSERT INTO `UserLogin`(`userId`, `loginInfo`) VALUES (1, '{\"QQ\": \"82946772\", \"wxchat\": \"破产码农\", \"cellphone\": \"13918888888\"}');
INSERT INTO `UserLogin`(`userId`, `loginInfo`) VALUES (2, '{\"cellphone\": \"15026888888\"}');
INSERT INTO `UserLogin`(`userId`, `loginInfo`) VALUES (3, '{\"QQ\": \"82946772\", \"wxchat\": \"破产码农\", \"cellphone\": \"13918888889\"}');
```

### 2.1 JSON_EXTRACT 函数，获取Json字段中特定属性的值

```sql
SELECT JSON_UNQUOTE(JSON_EXTRACT(loginInfo, "$.cellphone")) from UserLogin;
```
获取cellphone属性的值。可以使用->写法或者->>写法。

```sql
-- 带引号
SELECT loginInfo->"$.cellphone" from UserLogin;
-- 不带引号
SELECT loginInfo->>"$.cellphone" from UserLogin;
```

函数说明：

`JSON_EXTRACT`(或->)从JSON文档返回数据。

`JSON_UNQUOTE`取消引用JSON值，并以utf8mb4字符串的形式返回结果。

### 2.2 JSON_CONTAINS 查询Json中满足cellphone等于13918888888的记录

```sql
SELECT * from UserLogin where JSON_CONTAINS(loginInfo, '"13918888888"', '$.cellphone')
```

说明：使用 `JSON_CONTAINS` 搜索指定键的值是否与指定值匹配。

### 2.3 给Json中的字段添加索引

```sql
-- 增加虚拟列-cellphone，值通过loginInfo计算而来
alter table UserLogin add COLUMN cellphone varchar(50) as (loginInfo->>"$.cellphone");
-- 给cellphone 这一列增加唯一索引
alter table UserLogin add unique index idex_cellphone(cellphone);
```
可以看到的确使用了该索引进行查询

![](https://files.mdnice.com/user/13344/ad43d84f-e842-4f25-be98-51453f23b992.png)

### 2.4 JSON_CONTAINS_PATH 判断Json中是否有对应字段

所有记录中有多少记录包含wxchat字段

```sql
SELECT count(*), JSON_CONTAINS_PATH(loginInfo, 'one', '$.wxchat') cp FROM UserLogin GROUP BY cp
```
返回

![](https://files.mdnice.com/user/13344/50d21763-b0b4-48e2-a118-eee2ac30f7e1.png)

说明，包含wxchat字段有两条记录，不包含的有一条记录。

### 2.5 JSON_PRETTY 让Json长的更好看

```SQL
SELECT JSON_PRETTY(loginInfo) from UserLogin 
```

可以返回格式化的json数据

```json
{
  "QQ": "82946772",
  "wxchat": "破产码农",
  "cellphone": "13918888888"
}
```

### 2.6 JSON_STORAGE_SIZE返回二进制表示的字节数

返回loginInfo字段中存储的二进制表示的字节数。

```sql
SELECT max(JSON_STORAGE_SIZE(loginInfo)) FROM UserLogin;
SELECT avg(JSON_STORAGE_SIZE(loginInfo)) FROM UserLogin;
SELECT min(JSON_STORAGE_SIZE(loginInfo)) FROM UserLogin;
```

### 2.7 其他函数

- JSON_OBJECT 计算键值对列表并返回包含这些键值对的JSON对象，使用JSON_OBJECT。
- JSON_OBJECTAGG 接受两个列名或表达式，并返回一个包含JSON_OBJECTAGG键值对的JSON对象。
- JSON_ARRAY 计算一个值列表，并使用JSON_ARRAY返回包含这些值的JSON数组。
- JSON_ARRAYAGG 将结果集聚合为单个JSON数组，其元素由带有JSON_ARRAYAGG的行组成。
- JSON_TABLE 从JSON文档中提取数据，并将其作为具有JSON_TABLE指定列的关系表返回。

## 总结

> JSON 类型是 MySQL 5.7 版本新增的数据类型，用好 JSON 数据类型可以有效解决很多业务中实际问题。最后，我总结下今天的重点内容： 使用 JSON 数据类型，推荐用 MySQL 8.0.17 以上的版本，性能更好，同时也支持 Multi-Valued Indexes；

- JSON 数据类型的好处是无须预先定义列，数据本身就具有很好的描述性；

- 不要将有明显关系型的数据用 JSON 存储，如用户余额、用户姓名、用户身份证等，这些都是每个用户必须包含的数据；

- JSON 数据类型推荐使用在不经常更新的静态数据存储。

参考：

1、30 mins with MySQL JSON functions：https://dasini.net/blog/2018/07/23/30-mins-with-mysql-json-functions/

2、拉钩教育《姜承尧的MySQL实战宝典》-04 | 非结构存储：用好 JSON 这张牌：https://kaiwu.lagou.com/course/courseInfo.htm?courseId=869#/detail/pc?id=7320





