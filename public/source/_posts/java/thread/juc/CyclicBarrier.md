---
title: CyclicBarrier (AQS)
categories: java
tags: 
  - CyclicBarrier
  - java
abbrlink: 3dd6eb15
date: 2022-02-01 12:00:00
---





CyclicBarrier，翻译过来就是循环屏障。用来进行线程协作，等待线程满足某个计数。通过构造方法设置计数的个数，然后每个线程通过 await 方法进行等待，当满足了计数之后再继续执行。它最大的好处就是可以重复使用计数。

使用方法和CountDownLatch差不多如下所示：

```java
/**
 * CyclicBarrier,可以复用
 *
 * @author chenghao.li
 */
public class CyclicBarrierTest2 {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        CyclicBarrier cyclicBarrier = new CyclicBarrier(2, () -> System.out.println("执行完毕"));

        for (int i = 0; i < 2; i++) {
            executorService.submit(() -> {
                try {
                    System.out.println(Thread.currentThread().getName() + " 开始执行");
                    TimeUnit.SECONDS.sleep(1);
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });

            executorService.submit(() -> {
                try {
                    System.out.println(Thread.currentThread().getName() + " 开始执行");
                    TimeUnit.SECONDS.sleep(2);
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });

        }
        executorService.shutdown();
    }
}
```

> 上面的示例中，我们设置了线程池的大小为 2，同时CyclicBarrier计数为 2，这是有意为之。如果这两个不一致，如果线程池的数量大于CyclicBarrier的计数，那么循环执行任务的时候，可能是当前循环的任务和下次循环的任务执行完毕了。不能保证是当先循环的任务都执行完毕了！

