---
layout: post
title:  "ORACLE一次大量数据删除导致问题的处理"
date:   2017-05-21 00:00:00
categories: oracle
tags: oracle
mathjax: true
---

由于项目运行时间较久，导致ORACLE表中记录数太多，对这个表（以下称table1）的操作也比较多，包括查询更新，最后导致对table1的操作消耗cpu猛增，最后网站不能访问。




由于知道是这个问题，所以就打算将历史数据备份，仅留最近一个月的数据在table1中。

1、方案1:根据某个与时间相关的字段，一年一年的删除
delete from table1 where data_id like '_2015'; data_id 是包含时间字段的产品id。

问题：由于table1的表数据量大，仅仅查询其中一条记录都耗时几分钟，所以一年一年删除的速度特别慢，让人以为出错的感觉。删除2015年数据耗时一个小时都没有删除掉。适用于对这个表的操作还不算太慢的情况下。



2、方案2: 
步骤如下：

采用链接中的步骤

    1） 备份所有数据到另外一张表baktable1
    create table baktable1 as select * from table1;
    2) 备份最近一个月的数据到一张临时表temptable1中
    create table temptable1 as select * from table1 where input_date > to_date("2017-05-01","yyyy-MM-dd");
    3）删除表table1中的数据
    truncate table table1 ;
    4) 删除表table1
    drop table table1；
    5） 创建table1，把最近一个月的数据倒回来
    create table1 as select * from temptable1 ;
    6) 删除临时表


我采用了方案2，因为方案1执行感觉不知道什么时候能结束。然而系统第二天一直出错，有文件上传可以解析，但是不能显示在网站上面。经过了十个小时的艰苦奋斗，终于找到了原因。因为
create table temptable1 as select * from table1 where input_date > to_date("2017-05-01","yyyy-MM-dd");
这条语句，除了能把表结构复制过来、数据复制过来之外，别的不会复制过来。比如说索引、主键、默认值，在这里主要是默认值丢失。



可以将方案2 优化如下：


    1） 备份所有数据到另外一张表baktable1
    
    create table baktable1 as select * from table1;
    
    2) 备份最近一个月的数据到一张临时表temptable1中
    
    create table temptable1 as select * from table1 where input_date > to_date("2017-05-01","yyyy-MM-dd");
    
    3）删除表table1中的数据
    
    truncate table table1 ;
    
    4) 把最近一个月的数据插入到table1  这里和方案2 不一样
    
    insert  into table1 (select * from temptable1);
    
    5) 删除临时表
    
    drop table temptable1;



如果已经采用了方案2
1、插入主键、索引

2、设置默认值

    alter table table1 modify tag default 'S';
    alter table table1 modify input_date default sysdate;
    查看表的默认值设置：
    select
    t.nullable as 是否为空,
    t.data_default as 默认值
    from USER_TAB_COLS t where TABLE_NAME ='NMC_DATA';



参考：http://www.cnblogs.com/songling/archive/2013/08/24/3279588.html
