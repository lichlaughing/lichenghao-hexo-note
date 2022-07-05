---
title: CountDownLatch (AQS)
categories: java
tags: 
  - CountDownLatch
  - java
abbrlink: f75360b3
date: 2022-02-01 12:00:00
---



CountDownLatch 直接翻译过来就是倒计时锁，用来做线程的同步协作，可以让一个线程等待其他线程“倒计时”操作完毕后，再继续执行。

例如下面的代码，线程 1 和 2 执行完毕后，主线程才会继续执行。

```java
/**
 * CountDownLatch
 *
 * @author chenghao.li
 */
public class CountDownLatchTest {

    public static void main(String[] args) {
        CountDownLatch countDownLatch = new CountDownLatch(2);

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " 执行...");
            try {
                TimeUnit.SECONDS.sleep(1);
                countDownLatch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "thread-1").start();

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " 执行...");
            try {
                TimeUnit.SECONDS.sleep(1);
                countDownLatch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "thread-2").start();

        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 执行...");
    }
}
```

```properties
thread-1 执行...
thread-2 执行...
main 执行...
```

> 对于线程等待其他线程执行完毕后，在继续执行，通过 join 方法也能实现。只不过 CountDownLatch 更优雅。



上面的代码“倒计时”只是执行了一次，如果需要重复“倒计时”的话，可以把CountDownLatch放进循环中。

```java
public class CountDownLatchTest2 {

    public static void main(String[] args) {

        for (int i = 0; i < 2; i++) {
            CountDownLatch countDownLatch = new CountDownLatch(2);
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + " 执行...");
                try {
                    TimeUnit.SECONDS.sleep(1);
                    countDownLatch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "thread-1").start();

            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + " 执行...");
                try {
                    TimeUnit.SECONDS.sleep(1);
                    countDownLatch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "thread-2").start();

            try {
                countDownLatch.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " 执行...");
        }
    }
}
```

但是上面的代码有个明显的问题，就是需要多次的“倒计时”就需要不断的重复创建CountDownLatch对象，如果这个对象可以复用就好了。那么这个功能就需要另外的一个工具`CyclicBarrier`了。



