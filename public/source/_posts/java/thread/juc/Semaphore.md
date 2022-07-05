---
title: Semaphore (AQS)
categories: java
tags: 
  - Semaphore
  - java
abbrlink: 59bdd7d5
date: 2022-02-01 12:00:00
---



Semaphore 翻译成信号量，用来限制同时访问共享资源的线程数量。使用也能简单。

## 使用

```java
/**
 * Semaphore
 *
 * @author chenghao.li
 */
@Slf4j
public class SemaphoreTest {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire();
                    log.info(Thread.currentThread().getName() + " get ");
                    TimeUnit.SECONDS.sleep(1);

                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                    log.info(Thread.currentThread().getName() + " release ");
                }
            }).start();
        }
    }
}
```

## 原理

### 加锁流程

Semaphore 就好比一个停车场，规定了停车位的数量，线程就好比停车场里面的车辆，如果车位满了你就停不了，需要等有车让出停车位，你才可以停进去。

Semaphore内部也是通过Sync 实现。

```java
 abstract static class Sync extends AbstractQueuedSynchronizer {
   .....
 }
```

默认情况下也是非公平。通过构造方法就可看出。

```java
public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }
```

假如现在我们有五个线程来访问共享资源。通过构造方法一直网上追代码，就会发现参数值赋给了state。

![](https://blog.lichenghao.cn/upload/2022/07/17.07.40.png)

假如 thread-0,thread-1,thread-2 能获取到锁，那么看下获取锁的过程。

```java
 public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
```

↓ 调用`sync.acquireSharedInterruptibly(1);`

```java
public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
```

↓ 调用 `tryAcquireShared(arg)`,最终会调用的如下的的方法：

```java
final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
              	// 获取状态值
                int available = getState();
              	// thread-0过来的话，还剩下2个位置返回了
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
```

这样的话，thread-0,thread-1,thread-2 一次都能获取成功。

然后就是 thread-3来了，nonfairTryAcquireShared(int acquires) 的返回值就是-1 了。这个时候就会执行到`doAcquireSharedInterruptibly(arg)`方法了。

↓ 执行到这个方法，就是再尝试，失败的话就进入等待队列了。

```java
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

![](https://blog.lichenghao.cn/upload/2022/07/17.37.42.png)



### 解锁流程

如果，thread-2用完了，释放了锁。

```java
public void release() {
        sync.releaseShared(1);
    }
```

↓ releaseShared(1)

```java
public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
```

↓ tryReleaseShared(arg)

```java
protected final boolean tryReleaseShared(int releases) {
            for (;;) {
              // 拿到 0
                int current = getState();
              // 1
                int next = current + releases;
                if (next < current) // overflow
                    throw new Error("Maximum permit count exceeded");
                if (compareAndSetState(current, next))
                    return true;
            }
        }
```

↓ 成功后，状态就变化了

![](https://blog.lichenghao.cn/upload/2022/07/17.42.23.png)

↓ 那么就会走 doReleaseShared() 方法去唤醒等待中的线程了

```java
private void doReleaseShared() {
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```

![](https://blog.lichenghao.cn/upload/2022/07/17.55.30.png)











