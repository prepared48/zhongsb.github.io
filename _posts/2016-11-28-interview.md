---
layout: post
title:  "一次面试"
date:  2016-11-28 15:14:54
categories: 工作
tags: Interview
mathjax: true
---


可能会遇到的面试题：

### 1、autowire 的原理

### 2、void arry(int [] array){
//array 值在【1,100】之间，长度【1,1000】，求时间复杂度最小的算法。注：不是四种排序的中的算法
}
### 3、怎么查看Linux内存？几核，硬盘空间
答：uname -a
### 4、ArrayList 实现了哪几个接口？
### 5、JVM参数设置，进程参数设置：设置xmx等的值？
### 6、jsp和servlet的区别？ 2016年11月26日--华三通信：感觉靠谱
### 1、是否有深入研究某种知识？
### 2、是否会自己总结学到的东西？
### 3、写入文件过大的问题？
### 4、数据库三范式？
第一范式：1NF
第一范式是对关系模式的基本要求，不满足第一范式的数据库，就不叫关系数据库。即数据库的每一列都是不可分割的基本数据项，同一列中不能有多个值。 即实体中的某个属性属性不能有多个值或不能有重复的属性。 简而言之，第一范式就是无重复的列。  

第二范式：2NF  
满足第二范式，必须先满足第一范式。  
第二范式要求：数据库表中的每个实例或者行必须可以被唯一的标识

第三范式：
满足第三范式，必须先满足第二范式：  
第三范式要求一个数据库表不包含已经在其他表中已包含的非主关键字信息。

### 5、数据库锁分类：
注：并发


### 1、泰勒公式
### 2、二进制、8进制、十进制之间的等式计算
### 3、家用电脑计算时间


### 1、servlet生命周期
### 2、== 和 equals 区别
解释一下==号，他比较的是一个对象在内存中的地址值，
比如2个字符串对象  
`
String s1 = new String("str"); 
`  
`
String s2 = new String("str");  
`  
如果用==号比较，会返回false，因为创建了两个对象，他们在内存中地址的位置是不一样的。  
equals的情况比较复杂，它是java.lang.Object类中的一个方法。因为java中所有的类都默认继承于Object，  所以所有的类都有这个方法。
在Object类源码中是这样写的。  
`
    public boolean equals(Object obj) {
        return (this == obj);
    }  
`  
他同样使用==号进行内存地址的比较。但是许多java类中都重写了这个方法，比如String。  
 
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
    if (anObject instanceof String) {
    String anotherString = (String)anObject;
    int n = count;
    if (n == anotherString.count) {
    char v1[] = value;
    char v2[] = anotherString.value;
    int i = offset;
    int j = anotherString.offset;
    while (n-- != 0) {
    if (v1[i++] != v2[j++])
    return false;
    }
    return true;
    }
    }
    return false;
    }   
 
String里的方法，如果==号比较不相等，还会进行一下值的比较。
所以equals方法具体的作用要看当前的那个类是如何实现重写父类中该方法的。如果没有重写该方法，那么他和==号等价。
### 3、spring原理
### 4、Linux命令：对文件内容进行排序
http://www.cnblogs.com/zhuyp1015/archive/2012/07/11/2586985.html  

    cat /etc/passwd |awk '{FS=":"} $3 < 10 {print $1 "\t " $3}'  
    last -n 5 | awk '{print $1 "\t" $3}'  
    cat /etc/passwd | awk '{FS=":"} $3 < 10 {print $1 "\t " $3}' 第一行不显示  
    cat /etc/passwd | awk 'BEGIN {FS=":"} $3 < 10 {print $1 "\t " $3}'  
    cat pay.txt | awk 'NR==1{printf "%10s %10s %10s %10s %10s\n",$1,$2,$3,$4,"Total" }  
    NR>=2{total = $2 + $3 + $4  
    printf "%10s %10d %10d %10d %10.2f\n", $1, $2, $3, $4, total}'  

### 5、接口和抽象类的区别
### 6、单例模式
### 7、instanceof
### 8、hashCode怎么计算的？
## 总结：
1、用到什么东西，需要自己深入了解
### 面试题（自己查找）： 
1、Linux中awk命令？ 
