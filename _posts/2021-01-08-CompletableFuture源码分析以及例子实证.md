---
layout: post
title:  "CompletableFuture源码分析以及例子实证"
date:   2021-01-08 10:47:54
categories: 源码
tags: JAVA 多线程
mathjax: true
---

* content
{:toc}

CompletableFuture 可以应用在异步编程场景中。

比如经典的泡茶等场景




## CompletableFuture

## 介绍

默认情况下 CompletableFuture 会使用公共的 ForkJoinPool 线程池，这个线程池默认创建的线程数是 CPU 的核数
（也可以通过 JVM option:-Djava.util.concurrent.ForkJoinPool.common.parallelism 来设置 ForkJoinPool 线程池的线程数）。

但是也==不一定就使用ForkJoinPool==，要看（cpu的核数-1）是否大于1，如果大于1，使用过 ForkJoinPool，否则，创建普通线程执行。

cpu核数 = Runtime.getRuntime().availableProcessors(); // 4

源码如下：

```java
    // 是否使用 useCommonPool，如果（cpu的核数-1）大于1，使用ForkJoinPool，否则，不使用线程池。
    private static final boolean useCommonPool =
            (ForkJoinPool.getCommonPoolParallelism() > 1);

    /**
     * Default executor -- ForkJoinPool.commonPool() unless it cannot
     * support parallelism.
     */
    // 使用线程池还是创建普通线程
    private static final Executor asyncPool = useCommonPool ?
        ForkJoinPool.commonPool() : new ThreadPerTaskExecutor();
    
    /** Fallback if ForkJoinPool.commonPool() cannot support parallelism */
    static final class ThreadPerTaskExecutor implements Executor {
        public void execute(Runnable r) { new Thread(r).start(); }
    }
```

## 使用场景

CompletableFuture 可以应用在异步编程场景中。

比如经典的泡茶：

任务1：洗水壶、烧开水

任务2：洗茶壶、洗茶杯、拿茶叶

任务3：泡茶

其中任务1和任务2可以并行执行；--使用 `supplyAsync` 方法提交异步任务

任务3必须等待任务1和任务2完成之后执行。--使用`thenCombine`完成等待。

```java
//任务1：洗水壶->烧开水
CompletableFuture<Void> f1 = 
  CompletableFuture.runAsync(()->{
  System.out.println("T1:洗水壶...");
  sleep(1, TimeUnit.SECONDS);

  System.out.println("T1:烧开水...");
  sleep(15, TimeUnit.SECONDS);
});
//任务2：洗茶壶->洗茶杯->拿茶叶
CompletableFuture<String> f2 = 
  CompletableFuture.supplyAsync(()->{
  System.out.println("T2:洗茶壶...");
  sleep(1, TimeUnit.SECONDS);

  System.out.println("T2:洗茶杯...");
  sleep(2, TimeUnit.SECONDS);

  System.out.println("T2:拿茶叶...");
  sleep(1, TimeUnit.SECONDS);
  return "龙井";
});
//任务3：任务1和任务2完成后执行：泡茶
CompletableFuture<String> f3 = 
  f1.thenCombine(f2, (__, tf)->{
    System.out.println("T1:拿到茶叶:" + tf);
    System.out.println("T1:泡茶...");
    return "上茶:" + tf;
  });
//等待任务3执行结果
System.out.println(f3.join());

void sleep(int t, TimeUnit u) {
  try {
    u.sleep(t);
  }catch(InterruptedException e){}
}
// 一次执行结果：
T1:洗水壶...
T2:洗茶壶...
T1:烧开水...
T2:洗茶杯...
T2:拿茶叶...
T1:拿到茶叶:龙井
T1:泡茶...
上茶:龙井
```

## 实证及总结

CompletableFuture 提交的任务会按照顺序执行，如果最后提交的任务执行时间比较长，效果不好。尽量把

执行时间长的任务先提交。或者配置实际线程数，设置合理的顺序。



TestCompletableFuture.java 代码

```java
public static void main(String[] args) {
        System.out.println("启动线程数：" + (((Runtime.getRuntime().availableProcessors() - 1) > 1)
                ? (Runtime.getRuntime().availableProcessors() - 1) : "实际任务数"));
        System.out.println("start： " + new Date());
        CompletableFuture<Integer> task4 = CompletableFuture.supplyAsync(() -> {
            System.out.println("任务4, 线程名字" + Thread.currentThread().getName());
            try {
                sleep(4000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 3;
        });
        CompletableFuture<List<Integer>> task1 = CompletableFuture.supplyAsync(() -> {
            System.out.println("任务1, 线程名字" + Thread.currentThread().getName());
            try {
                sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return new ArrayList<>();
        });
        CompletableFuture<Integer> task2 = CompletableFuture.supplyAsync(() -> {
            System.out.println("任务2, 线程名字" + Thread.currentThread().getName());
            try {
                sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 2;
        });
        CompletableFuture<Integer> task3 = CompletableFuture.supplyAsync(() -> {
            System.out.println("任务3, 线程名字" + Thread.currentThread().getName());
            try {
                sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return 3;
        });

        CompletableFuture.allOf(task1, task2, task3, task4);
        System.out.println("end: " + new Date());
        try {
            task1.get();
            System.out.println("task1 end: " + new Date());
            task2.get();
            System.out.println("task2 end: " + new Date());
            task3.get();
            System.out.println("task3 end: " + new Date());
            task4.get();
            System.out.println("task4 end: " + new Date());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
```

TestCompletableFuture.java 输出如下


```
start： Fri Jan 08 09:54:09 CST 2021
任务4, 线程名字ForkJoinPool.commonPool-worker-1
任务1, 线程名字ForkJoinPool.commonPool-worker-2
任务2, 线程名字ForkJoinPool.commonPool-worker-3
end: Fri Jan 08 09:54:09 CST 2021
任务3, 线程名字ForkJoinPool.commonPool-worker-2
task1 end: Fri Jan 08 09:54:10 CST 2021
task2 end: Fri Jan 08 09:54:11 CST 2021
task3 end: Fri Jan 08 09:54:13 CST 2021
task4 end: Fri Jan 08 09:54:13 CST 2021
```

问题：能看到它创建了 3 个线程。但是什么呢？

```
cpu核数 = Runtime.getRuntime().availableProcessors(); // 4
```

所以使用了 ForkJoinPool 并且线程池中有三个线程。

提交的任务按照提交顺序执行，如果如果把task4，放在最后，整理执行时间会更长。


极客时间：https://time.geekbang.org/column/article/94604

代码：https://github.com/zhongsb/Java-learning.git