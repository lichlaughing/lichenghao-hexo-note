---
title: AbstractQueuedSynchronizer (AQS)
categories: java
tags: 
  - AQS
  - java
abbrlink: b5bca709
date: 2022-02-01 12:00:00
---



AQS 全称：`AbstractQueuedSynchronizer`，是一个抽象类位于`java.util.concurrent.locks`，它是阻塞式锁和一些同步器工具的框架。

其实现方式有如下几个特点：

- 通过`state`属性来表示资源的状态（独占模式和共享模式），子类需要自己去维护这个状态。
- CAS机制来设置state 状态
- 内部有 FIFO的等待队列
- 支持单个或者多个条件变量来实现等待唤醒机制

子类实现该抽象类，需要自己实现一些方法来完成基本的操作。

- tryAcquire 
- tryRelease 
- tryAcquireShared 
- tryReleaseShared 
- ......

示例个代码，演示简单的不可重入锁

```java
/**
 * 简单自定义不可重入锁
 */
@Slf4j
public class MyLockTest {

    public static void main(String[] args) {
        MyLock myLock = new MyLock();
        new Thread(() -> {
            try {
                myLock.lock();
                log.info("{}-locking", Thread.currentThread().getName());
                TimeUnit.SECONDS.sleep(1);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                myLock.unlock();
                log.info("{}-unlocking", Thread.currentThread().getName());
            }
        }, "thread-1").start();

        new Thread(() -> {
            try {
                myLock.lock();
                log.info("{}-locking", Thread.currentThread().getName());
            } finally {
                myLock.unlock();
                log.info("{}-locking", Thread.currentThread().getName());
            }
        }, "thread-2").start();
    }


    static class MyLock implements Lock {

        /**
         * 同步器类，独占锁
         */
        class Mysync extends AbstractQueuedLongSynchronizer {
            /**
             * 获取锁,状态为1表示持有锁
             *
             * @param arg 这个参数可用来重入锁
             * @return
             */
            @Override
            protected boolean tryAcquire(long arg) {
                if (compareAndSetState(0, 1)) {
                    // 加锁成功后，设置当前线程持有
                    setExclusiveOwnerThread(Thread.currentThread());
                    return true;
                }
                return false;
            }

            /**
             * 释放锁
             *
             * @param arg
             * @return
             */
            @Override
            protected boolean tryRelease(long arg) {
                setExclusiveOwnerThread(null);
                // 状态设置为0，表示释放锁了
                // volatile变量有写屏障，前面的操作对其他线程可见
                setState(0);
                return true;
            }

            /**
             * 是否拿着独占锁
             *
             * @return
             */
            @Override
            protected boolean isHeldExclusively() {
                // 1 就是拿着锁呢
                return getState() == 1;
            }

            /**
             * 条件变量对象
             *
             * @return
             */
            public Condition newCondition() {
                return new ConditionObject();
            }
        }

        private Mysync sync = new Mysync();

        /**
         * 加锁，不成功进入等到队列等待
         */
        @Override
        public void lock() {
            sync.acquire(1);
        }

        /**
         * 加锁，可以被打断
         *
         * @throws InterruptedException
         */
        @Override
        public void lockInterruptibly() throws InterruptedException {
            sync.acquireInterruptibly(1);
        }

        /**
         * 尝试加锁一次
         *
         * @return
         */
        @Override
        public boolean tryLock() {
            return sync.tryAcquire(1);
        }

        /**
         * 尝试加锁一次，带超时时间
         *
         * @param time
         * @param unit
         * @return
         * @throws InterruptedException
         */
        @Override
        public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
            return sync.tryAcquireNanos(1, unit.toNanos(time));
        }

        /**
         * 解锁
         */
        @Override
        public void unlock() {
            // 不但解锁，还能唤醒等待的线程
            sync.release(0);
        }

        /**
         * 条件变量
         *
         * @return
         */
        @Override
        public Condition newCondition() {
            return sync.newCondition();
        }
    }
}
```









