---
layout: post
title: "线程池"
date: 2012-08-16 18:05
comments: true
sharing: true
footer: true
categories: [Java]
---


###为什么要使用线程池
+ 为大量而短小的任务创建线程和销毁线程开销过大
+ 线程池可以重复使用线程，避免额外的创建和销毁开销
+ 通过队列和意外处理避免资源不足
+ 灵活调整线程池内线程的数量

###线程池基本配置
+ corePoolSize: 池中的核心线程数
+ maximumPoolSize: 池中的最大线程数（当核心线程用尽，工作队列已满时，开始创建线程） 
+ keepAliveTime: 大于核心线程数时，干掉空闲线程的最长等待时间
+ unit: keepAliveTime的单位，例如秒，毫秒
+ workQueue: 工作队列，线程根据工作队列来创建新线程去完成任务
+ threadFactory: 用来定义线程中的属性、名称、守护程序状态、ThreadGroup 等等
+ handler: 由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序

####默认的handler
+ ThreadPoolExecutor.AbortPolicy: 直接抛出RejectedExecutionException
+ ThreadPoolExecutor.CallerRunsPolicy: 再次尝试
+ ThreadPoolExecutor.DiscardOldestPolicy: 放弃最旧的未处理任务
+ ThreadPoolExecutor.DiscardPolicy: 直接丢弃

###线程池基本逻辑
+ 当池小于corePoolSize就新建线程，并处理任务
+ 当池等于corePoolSize，把请求放入workQueue中，池里的空闲线程就去从workQueue中取任务并处理
+ 当workQueue放不下新入的任务时，新建线程处理任务，如果池撑到了maximumPoolSize就用handler来做拒绝处理
+ 当池中线程数大于corePoolSize的时候，多余的线程会等待keepAliveTime长的时间，如果无请求可处理就自行销毁

<!-- More -->

###线程池实例
+ 线程处理的任务，我这里为取得随机数

```java
package wanghuida.test;

public class MyThread implements Runnable {
    
    @Override
    public void run() {
        String name = Thread.currentThread().getName();
        int number = (int)(Math.random() * 1000);
        System.out.println("[" + name + "] random number is " + number);
    }
}
```

+ 线程工厂类，用来为线程赋予属性，我这里主要是定义线程名称 

```java
package wanghuida.test;

import java.util.concurrent.ThreadFactory;

public class MyThreadFactory implements ThreadFactory {

    private int n = 0;
    
    public MyThreadFactory() {
        this.n = 0;
    }

    @Override
    public Thread newThread(Runnable r) {
        n++;
        Thread t = new Thread(r);
        t.setName("thread-" + n);
        t.setDaemon(false);
        t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }

}
```

+ 构造线程池并处理任务

```java
package wanghuida.test;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class ThreadPoolTest {

    private ThreadPoolExecutor pool;
    
    public ThreadPoolTest(int core,int max,int queue) {
        this.pool = new ThreadPoolExecutor(
                core, //核心线程数
                max, //最大线程数
                5, TimeUnit.SECONDS, //超过核心线程数，等多少时间释放 
                new ArrayBlockingQueue<Runnable>(queue), //任务队列
                new MyThreadFactory(), //线程构造器
                new ThreadPoolExecutor.AbortPolicy() //超出池的策略  
        );
    }
    
    public void test(int times) {
        for(int i = 0; i < times; i++ ){
            MyThread t = new MyThread();
            this.pool.execute(t);
        }
        
        this.pool.shutdown();
        boolean loop = true;
        do {//等待所有任务完成     
            try {
                System.out.println("core pollsize is " + pool.getCorePoolSize() + ",poolsize is " + pool.getPoolSize());
                loop = !this.pool.awaitTermination(2, TimeUnit.SECONDS);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }  
        } while(loop);  
        
        
        
    }
    
    public void test() {
        this.test(10);
    }

}
```

+ main主函数

```java
package wanghuida.test;

public class Entry {

    public static void main(String[] args) {
        testThreadPool();
    }

    public static void testThreadPool() {
        ThreadPoolTest test1 = new ThreadPoolTest(5,10,3);
        test1.test();
        System.out.println("\n");
        
        ThreadPoolTest test2 = new ThreadPoolTest(2,4,3);
        test2.test();
        System.out.println("\n");
        
        ThreadPoolTest test3 = new ThreadPoolTest(1,1,1);
        test3.test();
    }
}
```

+ 输出结果和意图相符合

```java
[thread-1] random number is 52
[thread-2] random number is 6
[thread-3] random number is 477
[thread-4] random number is 446
[thread-5] random number is 845
[thread-5] random number is 175
[thread-5] random number is 920
[thread-5] random number is 125
[thread-6] random number is 580
[thread-6] random number is 936
core pollsize is 5,poolsize is 6


[thread-1] random number is 433
[thread-2] random number is 442
[thread-1] random number is 957
[thread-1] random number is 536
[thread-3] random number is 337
core pollsize is 2,poolsize is 4
[thread-2] random number is 118
[thread-2] random number is 819
[thread-4] random number is 681
[thread-3] random number is 480
[thread-1] random number is 469


[thread-1] random number is 287
[thread-1] random number is 710
Exception in thread "main" java.util.concurrent.RejectedExecutionException
    at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:1768)
    at java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:767)
    at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:658)
    at wanghuida.test.ThreadPoolTest.test(ThreadPoolTest.java:25)
    at wanghuida.test.ThreadPoolTest.test(ThreadPoolTest.java:44)
    at wanghuida.test.Entry.testThreadPool(Entry.java:56)
    at wanghuida.test.Entry.main(Entry.java:22)

```
