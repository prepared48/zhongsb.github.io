---
layout: post
title: 设计模式之单例模式深度解析
category: 技术
comments: true
tags: java
---

* content
{:toc}

单例模式是一种常用的软件设计模式。在它的核心结构中只包含一个被称为单例的特殊类。通过单例模式可以保证系统中一个类只有一个实例。




1、为什么用到单例模式？

答：有一些对象我们只需要一个，比如说：线程池、缓存、对话框、日志对象、处理便好设置、注册表。而如果使用全局变量来实现这个功能，则有一些缺点：必须在程序已开始就创建好对象，如果这个对象非常耗费资源，而程序在这次执行的过程中又没有用到它，这样就形成了浪费。
2、如何创建单例模式？

   1）经典的单例模式


    public class Singleton{  
        private static singleton uniqueInstance;  
      
        private Singleton(){};//构造器私有，只有当前类才能调用构造器  
      
        public static Singleton getInstance(){  
            if(uniqueInstance==null){  
                uniqueInstance = new Singleton();  
            }  
            return uniqueInstance;  
      
        }  
      
        //其他方法  
      
    }  


这个方式有个缺点，多线程访问的时候可能出现多个实例。

getInstance()方法两个线程同时进行判断，都为空；然后两个线程都会创建对象。  

![多线程问题][https://github.com/zhongsb/zhongsb.github.io/blob/master/images/singelton.jpg] 
 
2）处理多线程－－加锁

    public class Singleton{  
        private static singleton uniqueInstance;  
      
        private Singleton(){};//构造器私有，只有当前类才能调用构造器  
      
        public static sychronized Singleton getInstance(){  
            if(uniqueInstance==null){  
                uniqueInstance = new Singleton();  
            }  
            return uniqueInstance;  
      
        }  
      
        //其他方法  
      
    }  

这个方法缺点：会降低性能

同步一个方法可能造成程序执行效率降低100倍。

3）双重加锁


    public class Singleton{  
        private volatile static  Singleton uniqueInstance;  
      
        private Singleton(){}//私有构造器  
      
        public static Singleton getInstance(){  
            if(uniqueInstance==null){  
                synchronized(Singleton.class){  
                if(uniqueInstance==null){  
                      
                        uniqueInstance = new Singleton();  
                    }  
                }  
      
            }  
            return uniqueInstance;  
        }  
      
    }  


注意：双重加锁只适用于java1.4以上（不包括1.4）的版本。

4) 内部类


    //内部类  
    public class Singleton{  
        private Singleton(){}  
      
        private static class SingletonHolder{  
            private static Singleton uniqueInstance = new Singleton();  
        }  
      
        public static Singleton getInstance(){  
            return SingletonHolder.uniqueInstance;  
        }  
    }  


当Singleton被加载时，内部类并不会被实例化，故可以确保当Singleton类被载入jvm时，不会初始化单例类，而当getInstance()方法被调用的时候才会加载内部类，从而创建uniqueinstance。同时由于实例的建立是在类加载时完成的，天生对多线程友好。


优点：延迟加载，线程安全，省略synchrozined开销，代码少。

5）最牛－－枚举类型


    public enum Singleton{  
        instance;  
      
        Singleton(){};  
      
        public static void main(String []args){  
            Singleton sole = Singleton.instance;  
              
        }  
    }  


这个方法无偿提供了序列化机制，绝对防止多次实例化，即使在面对复杂的序列化或者反射攻击的时候。


参考：《Head first 设计模式》

[单例模式（饿汉，懒汉，双重锁，内部类实现）：面试必备](https://www.toutiao.com/i6360511195685847554/?tt_from=weixin&utm_campaign=client_share&app=news_article&utm_source=weixin&iid=7526562569&utm_medium=toutiao_android&wxshare_count=1)



[1]:http://img.blog.csdn.net/20170126101450505?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUHJlcGFyZWQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center
