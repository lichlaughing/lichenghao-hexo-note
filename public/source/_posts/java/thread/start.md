---
title: JAVA-线程
categories: java
tags: 
  - Thread
abbrlink: c033f1e5
date: 2022-02-01 12:00:00
---



## 进程-线程

进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是[操作系统](https://baike.baidu.com/item/操作系统)结构的基础。当一个程序运行了，它可能有多个进程，一个进程去读写文件，一个进程去发送消息。还有一些程序就一个进程，比如我们常用的记事本。

线程，一个进程内又可以分成很多线程。线程交给 cpu 去执行就可以了。

举个例子，你打开电脑后，打开了网易云，又打开了微信，一边听歌，一边聊天，这是就是启动了两个进程。在微信聊天中，你可以跟很多朋友聊天，打开了好几个聊天窗口，这就是多线程了。

## 并发-并行

CPU将时间片分给不同的线程使用，因为切换的时间非常短，用户感觉就是并发执行的。其实 CPU 是轮流把时间片给每个线程执行，这种方式称为"并发"（通常在单个 CPU 下）。

如果 cpu 都是多核的了，随便拿一个都是八核十六线程的。每个核心都能分配时间片给线程执行。这种情况下，线程就是"并行"执行了。

线程 1 和线程 3 在并行执行。但是绝大多数情况下，线程数远大于核心数，所以多个核心还是在不同的线程中切换，也就是并发。

![](https://blog.lichenghao.cn/upload/2022/07/194727.png)



## 创建运行线程

### 方式一：Thread

```java
public class Test01 {
    public static void main(String[] args) {
        // 子线程
        new Thread("Thread-01") {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + "-run");
            }
        }.start();
        // 主线程
        System.out.println(Thread.currentThread().getName() + "-run");
    }
}
```



### 方式二：Runnable 配合 Thread

```java
public class Test01 {
    public static void main(String[] args) {
        // 主线程
        System.out.println(Thread.currentThread().getName() + "-run");
        // 子线程
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + "-run");
            }
        };
        Thread thread = new Thread(runnable, "Thread-01");
        thread.start();
    }
}
```



### 方式三：FutureTask任务配合 Thread

FutureTask接口继承了 Runnable 接口，所以也可以用 Thread。并且它是有返回结果的。

```java
public class Test01 {
    public static void main(String[] args) {
        FutureTask<Integer> futureTask = new FutureTask<>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                System.out.println(Thread.currentThread().getName() + "-run");
                return 1;
            }
        });
        new Thread(futureTask, "Thread-01").start();
        // 执行结果
        try {
            Integer res = futureTask.get();
            System.out.println(res);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}

```