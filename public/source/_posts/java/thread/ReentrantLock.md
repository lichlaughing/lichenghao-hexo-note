---
title: ReentranLock
categories: java
tags: 
  - Thread
  - ReentrantLock
abbrlink: 99c70d13
date: 2022-02-01 12:00:00
---



ReentrantLock 是 Lock 接口的一个实现类，和synchronized一样，都是可重入锁。与synchronized不同的是，它分为公平锁和非公平锁，下面的演示代码默认都是是非公平锁。

> 锁的可重入性：锁的可重入性就是**一个线程在已经持有锁的情况下，能否再次获取这把锁**

## Lock 锁示例

```java
/**
 * @author chenghao.li
 */
public class LockTest01 {
    static Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        LockTest01 lockTest01 = new LockTest01();
        for (int i = 0; i < 100; i++) {
            new Thread(() -> lockTest01.print()).start();
        }
    }

    public void print() {
        lock.lock();
        try {
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "--" + i);
            }
        } finally {
            lock.unlock();
        }
    }
}
```

**锁的可重入性示例**

```java
/**
 * @author chenghao.li
 * 演示锁的可重入性
 */
public class LockTest02 {
    static Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        LockTest02 lockTest01 = new LockTest02();
        for (int i = 0; i < 100; i++) {
            new Thread(() -> lockTest01.print()).start();
        }
    }

    public void print() {
        lock.lock();
        lock.lock();
        try {
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "--" + i);
            }
        } finally {
            lock.unlock();
            lock.unlock();
        }
    }
}
```





## 加锁解锁

lock()/unlock()，建议lock.lock()紧跟try代码块，且lock.unlock()要放到finally第一行

## lockInterruptibly()

方法的作用：线程获取锁可以被打断，并且抛出异常 `java.lang.InterruptedException`

ReentrantLock默认是不能被打断的，如下代码所示：线程被打断，但是依然能获得锁。

```java
/**
 * @author chenghao.li
 * 演示中断线程，依然会获得的锁的情况
 */
public class LockTest03 {
    static Lock lock = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {
        LockTest03 lockTest03 = new LockTest03();
        Runnable r1 = () -> lockTest03.print1();
        Runnable r2 = () -> lockTest03.print2();
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r1);
        Thread t3 = new Thread(r2);
        Thread t4 = new Thread(r2);
        t1.start();
        t2.start();
        t3.start();
        TimeUnit.SECONDS.sleep(1);
        // 中断线程，也会获得锁
        t4.start();
        t4.interrupt();
    }

    public void print1() {
        try {
            lock.lock();
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "--A");
            }
        } finally {
            lock.unlock();
        }
    }

    public void print2() {
        try {
            lock.lock();
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "--B");
            }
        } finally {
            lock.unlock();
        }
    }
}
```



```properties
# 执行结果
Thread-0--A
Thread-0--A
Thread-0--A
Thread-0--A
Thread-0--A
Thread-1--A
Thread-1--A
Thread-1--A
Thread-1--A
Thread-1--A
Thread-2--B
Thread-2--B
Thread-2--B
Thread-2--B
Thread-2--B
Thread-3--B
Thread-3--B
Thread-3--B
Thread-3--B
Thread-3--B
```

当使用`lockInterruptibly()` 的时候，如果线程被中断，那么线程将不会获得锁，并且抛出异常

 ```java
/**
 * @author chenghao.li
 * 演示中断线程，lockInterruptibly()方法抛出异常
 */
public class LockTest04 {
    static Lock lock = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {
        LockTest04 lockTest04 = new LockTest04();
        Runnable r1 = () -> lockTest04.print1();
        Runnable r2 = () -> lockTest04.print2();
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r1);
        Thread t3 = new Thread(r2);
        Thread t4 = new Thread(r2);
        t1.start();
        t2.start();
        t3.start();
        TimeUnit.SECONDS.sleep(1);
        // 中断线程，也会获得锁
        t4.start();
        t4.interrupt();
    }

    public void print1() {
        try {
            lock.lock();
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "--A");
            }
        } finally {
            lock.unlock();
        }
    }

    public void print2() {
        try {
            lock.lockInterruptibly();
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "--B");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
 ```



```properties
Thread-0--A
Thread-0--A
Thread-0--A
Thread-0--A
Thread-0--A
Thread-1--A
Thread-1--A
Thread-1--A
Thread-1--A
Thread-1--A
Thread-2--B
Thread-2--B
Thread-2--B
Thread-2--B
Thread-2--B
java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireInterruptibly(AbstractQueuedSynchronizer.java:1220)
	at java.util.concurrent.locks.ReentrantLock.lockInterruptibly(ReentrantLock.java:335)
	at cn.com.lichenghao.lock.LockTest04.print2(LockTest04.java:44)
	at cn.com.lichenghao.lock.LockTest04.lambda$main$1(LockTest04.java:17)
	at java.lang.Thread.run(Thread.java:748)
Exception in thread "Thread-3" java.lang.IllegalMonitorStateException
	at java.util.concurrent.locks.ReentrantLock$Sync.tryRelease(ReentrantLock.java:151)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.release(AbstractQueuedSynchronizer.java:1261)
	at java.util.concurrent.locks.ReentrantLock.unlock(ReentrantLock.java:457)
	at cn.com.lichenghao.lock.LockTest04.print2(LockTest04.java:51)
	at cn.com.lichenghao.lock.LockTest04.lambda$main$1(LockTest04.java:17)
	at java.lang.Thread.run(Thread.java:748)
```



## tryLock()/tryLock(long timeout, TimeUnit unit)



**tryLock()**

不会等待获取不到锁就返回false

```java
/**
 * @author chenghao.li
 * 演示tryLock(),不会等待获取不到锁就返回false
 */
public class LockTest05 {
    static Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        LockTest05 lockTest05 = new LockTest05();
        Thread t1 = new Thread(() -> lockTest05.print());
        Thread t2 = new Thread(() -> lockTest05.print());
        t2.start();
        t1.start();
    }

    public void print() {
        if (lock.tryLock()) {
            try {
                System.out.println(Thread.currentThread().getName() + "--获得锁");
                for (int i = 0; i < 5; i++) {
                    System.out.println(Thread.currentThread().getName() + "--" + i);
                }
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        } else {
            System.out.println(Thread.currentThread().getName() + "--没有获得锁");
        }
    }
}
```

**tryLock(long timeout, TimeUnit unit)**

等待后获取锁，如果等待了还获取不到就返回false;如果线程被中断了，就取消等待，并且抛出异常

```java
/**
 * @author chenghao.li
 * 演示tryLock(long time, TimeUnit t) 等待后获取锁，
 * 如果等待了还获取不到就返回false;如果线程被中断了，就取消等待，并且抛出异常
 */
public class LockTest06 {
    static Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        LockTest06 lockTest05 = new LockTest06();
        Thread t1 = new Thread(() -> lockTest05.print());
        Thread t2 = new Thread(() -> lockTest05.print());
        t1.start();
        t2.start();
        // 调用interrupt()会终止等待锁
        //t2.interrupt();
    }

    public void print() {
        try {
            boolean b = lock.tryLock(2, TimeUnit.SECONDS);
            try {
                if (b) {
                    System.out.println(Thread.currentThread().getName() + "--获得锁");
                    for (int i = 0; i < 5; i++) {
                        System.out.println(Thread.currentThread().getName() + "--" + i);
                    }
                    TimeUnit.SECONDS.sleep(1);
                } else {
                    System.out.println(Thread.currentThread().getName() + "--没有获得锁");
                }
            } finally {
                lock.unlock();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## 获取锁的总结

假如 Thread1 和 Thread2 使用同一个锁，Thread1 首先获取到锁 objLock，如果此时 Thread2 要去获取锁，可能有如下几种情况：
* objLock.lock()：此方式会始终处于等待中，即使调用Thread2.interrupt()也不能中断(因为非公平锁默认是不可打断)，除非Thread1调用objLock..unlock()释放锁。(所以一般把释放锁放在finally中，不用影响他人用锁)
* objLock.lockInterruptibly(): 此方式会等待，但当调用Thread2.interrupt()会被中断等待，并抛出InterruptedException异常，否则会与objLock.lock()一样始终处于等待中，直到Thread1释放锁
* objLock.tryLock(): 该处不会等待，获取不到锁并直接返回false
* objLock.tryLock(1, TimeUnit.SECONDS):该处会在1秒时间内处于等待中，但当调用Thread2.interrupt()会被中断等待，并抛出InterruptedException。1秒时间内如果Thread1释放锁，会获取到锁并返回true，否则1秒过后会获取不到锁并返回false

## newCondition()-条件变量

实现线程的等待通知，就好像 synchronized与 wait()/notify()一样实现的等待通知模式。Lock 锁的newCondition()方法返回Condition 对象，Condition也可以实现锁的等待/通知模式。与 notify()随机通知不一样，Condition可以选择性的通知。

其常用的两个方法为：await()：使当前线程等待，同时释放锁对象，其他线程调用signal()时，调用await()/signal()也需要线程持有相关的锁对象。

调用signal()时会从 Conditioin 对象的线程等待队列中唤醒一个线程，唤醒的线程尝试获得锁，成功获得锁之后继续执行。

### 一个简单的等待通知

```java
/**
 * @author chenghao.li
 * 演示await()/signal()
 */
public class ConditionTest01 {
    static Lock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();

    public static void main(String[] args) throws InterruptedException {
        Sub sub = new Sub();
        sub.start();
        TimeUnit.SECONDS.sleep(1);
        try {
            lock.lock();
            condition.signal();
        } finally {
            lock.unlock();
        }
    }

    static class Sub extends Thread {
        @Override
        public void run() {
            try {
                lock.lock();
                System.out.println(Thread.currentThread().getName() + "-等待...");
                condition.await();
                System.out.println(Thread.currentThread().getName() + "-等待结束");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }
}
```

### 多个Condition部分通知

```java
/**
 * @author chenghao.li
 * 演示多个Condition，部分通知
 */
public class ConditionTest02 {
    static Lock lock = new ReentrantLock();
    static Condition c1 = lock.newCondition();
    static Condition c2 = lock.newCondition();

    public static void main(String[] args) throws InterruptedException {
        ConditionTest02 conditionTest02 = new ConditionTest02();
        new Thread(() -> conditionTest02.m1()).start();
        new Thread(() -> conditionTest02.m2()).start();
        TimeUnit.SECONDS.sleep(1);
        conditionTest02.signalM1();
    }

    public void m1() {
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + "等待...");
            c1.await();
            System.out.println(Thread.currentThread().getName() + "等待结束");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void signalM1() {
        try {
            lock.lock();
            c1.signal();
        } finally {
            lock.unlock();
        }
    }

    public void m2() {
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + "等待...");
            c2.await();
            System.out.println(Thread.currentThread().getName() + "等待结束");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void signalM2() {
        try {
            lock.lock();
            c2.signal();
        } finally {
            lock.unlock();
        }
    }
}
```

### await()/signal()实现交替打印

```java
/**
 * @author chenghao.li
 * 演示Condition实现交替打印
 */
public class ConditionTest03 {
    static Lock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();
    static boolean flag = false;

    public static void main(String[] args) {
        ConditionTest03 conditionTest03 = new ConditionTest03();
        new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                conditionTest03.printA();
            }
        }).start();
        new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                conditionTest03.printB();
            }
        }).start();
    }

    public void printA() {
        try {
            lock.lock();
            while (flag) {
                condition.await();
            }
            System.out.println(Thread.currentThread().getName() + "-A");
            flag = true;
            condition.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() {
        try {
            lock.lock();
            while (!flag) {
                condition.await();
            }
            System.out.println(Thread.currentThread().getName() + "-B");
            flag = false;
            condition.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

### await()/signalAll()实现交替打印

```java
/**
 * @author chenghao.li
 * 演示Condition/await(),signalAll()实现交替打印
 */
public class ConditionTest04 {
    static Lock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();
    static boolean flag = false;

    public static void main(String[] args) {
        ConditionTest04 conditionTest04 = new ConditionTest04();
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                for (int j = 0; j < 100; j++) {
                    conditionTest04.printA();
                }
            }).start();
            new Thread(() -> {
                for (int j = 0; j < 100; j++) {
                    conditionTest04.printB();
                }
            }).start();
        }
    }

    public void printA() {
        try {
            lock.lock();
            while (flag) {
                condition.await();
            }
            System.out.println(Thread.currentThread().getName() + "-A");
            flag = true;
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() {
        try {
            lock.lock();
            while (!flag) {
                condition.await();
            }
            System.out.println(Thread.currentThread().getName() + "-B");
            flag = false;
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

```
ReentrantLock
```

```
ReentrantLock
```

## ReentrantLock其他的方法

### getHoldCount()

返回锁调用lock 的次数

```java
/**
 * @author chenghao.li
 * 演示getHoldCount()返回锁调用lock的次数
 */
public class Test01 {
    static ReentrantLock LOCK = new ReentrantLock();

    public static void main(String[] args) {
        new Thread(() -> new Test01().m1()).start();
    }

    public void m1() {
        try {
            LOCK.lock();
            System.out.println(LOCK.getHoldCount());
            m2();
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "-" + i);
            }
        } finally {
            LOCK.unlock();
        }
    }

    public void m2() {
        try {
            LOCK.lock();
            System.out.println(LOCK.getHoldCount());
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "-" + i);
            }
        } finally {
            LOCK.unlock();
        }
    }
}
```



### getQueueLength()

返回等待锁的线程数

```java
/**
 * @author chenghao.li
 * 演示getQueueLength()返回等待获取锁的线程数
 */
public class Test02 {
    static ReentrantLock LOCK = new ReentrantLock();

    public static void main(String[] args) {
        Test02 test02 = new Test02();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> test02.m1()).start();
        }
    }

    public void m1() {
        try {
            LOCK.lock();
            TimeUnit.SECONDS.sleep(1);
            System.out.println("等待锁的线程数：" + LOCK.getQueueLength());
            for (int i = 0; i < 5; i++) {
                // System.out.println(Thread.currentThread().getName() + "-" + i);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            LOCK.unlock();
        }
    }
}
```



### hasQueuedThreads()

返回值true/false查询是否有线程在等待获取锁,重载的方法hasQueuedThreads(Thread thread)



**hasQueuedThreads()**

```java
/**
 * @author chenghao.li
 * 演示hasQueuedThreads()返回true/false,是否有线程在等待获取锁
 */
public class Test03 {
    static ReentrantLock LOCK = new ReentrantLock();

    public static void main(String[] args) {
        Test03 test03 = new Test03();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> test03.m1()).start();
        }
    }

    public void m1() {
        try {
            LOCK.lock();
            TimeUnit.SECONDS.sleep(1);
            System.out.println("是否有线程在等待锁：" + LOCK.hasQueuedThreads());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            LOCK.unlock();
        }
    }
}
```

**hasQueuedThreads(Thread thread)**

```java
/**
 * @author chenghao.li
 * 演示hasQueuedThreads(Thread thread)返回true/false,指定的线程是否在等待获取锁
 */
public class Test04 {
    static ReentrantLock LOCK = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {
        Test04 test04 = new Test04();
        Runnable runnable = () -> test04.m1();
        Thread t1 = new Thread(runnable);
        t1.setName("t1");
        Thread t2 = new Thread(runnable);
        t2.setName("t2");
        Thread t3 = new Thread(runnable);
        t3.setName("t3");

        t1.start();
        t2.start();
        t3.start();
        System.out.println(LOCK.hasQueuedThread(t1));
        System.out.println(LOCK.hasQueuedThread(t2));
        System.out.println(LOCK.hasQueuedThread(t3));
    }

    public void m1() {
        try {
            LOCK.lock();
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "-" + i);
            }
        } finally {
            LOCK.unlock();
        }
    }
}
```



### getWaitingThreads(Condition condition)

返回在 Condition条件上等待的线程数



 ```java
/**
 * @author chenghao.li
 * 演示getWaitQueueLength(Condition condition)获取等待条件的线程数
 */
public class Test05 {
    static ReentrantLock LOCK = new ReentrantLock();
    static Condition condition = LOCK.newCondition();

    public static void main(String[] args) throws InterruptedException {
        Test05 test05 = new Test05();
        for (int i = 0; i < 5; i++) {
            new Thread(() -> test05.m1()).start();
        }
        TimeUnit.SECONDS.sleep(1);
        try {
            LOCK.lock();
            System.out.println("等待condition的线程数" + LOCK.getWaitQueueLength(condition));
        } finally {
            LOCK.unlock();
        }
        // 唤醒
        try {
            LOCK.lock();
            condition.signalAll();
            System.out.println("等待condition的线程数" + LOCK.getWaitQueueLength(condition));
        } finally {
            LOCK.unlock();
        }
    }

    public void m1() {
        try {
            LOCK.lock();
            System.out.println(Thread.currentThread().getName() + "-开始等待！");
            condition.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            LOCK.unlock();
        }
    }
}
 ```



 ```properties
Thread-0-开始等待！
Thread-1-开始等待！
Thread-2-开始等待！
Thread-3-开始等待！
Thread-4-开始等待！
等待condition的线程数5
等待condition的线程数0
 ```



### hasWaiters(Condition condition)

查询是否有线程在等待指定的 Condition 条件



```java
/**
 * @author chenghao.li
 * 演示hasWaiters(Condition condition)方法，判断是否有线程在等待condition的条件
 */
public class Test06 {
    static ReentrantLock LOCK = new ReentrantLock();
    static Condition condition = LOCK.newCondition();

    public static void main(String[] args) throws InterruptedException {
        Test06 test06 = new Test06();
        for (int i = 0; i < 5; i++) {
            new Thread(() -> test06.m1()).start();
        }
        TimeUnit.SECONDS.sleep(1);
        try {
            LOCK.lock();
            System.out.println("是否有线程在等待condition的条件：" + LOCK.hasWaiters(condition));
        } finally {
            LOCK.unlock();
        }
        // 唤醒
        try {
            LOCK.lock();
            condition.signalAll();
            System.out.println("是否有线程在等待condition的条件：" + LOCK.hasWaiters(condition));
        } finally {
            LOCK.unlock();
        }
    }

    public void m1() {
        try {
            LOCK.lock();
            System.out.println(Thread.currentThread().getName() + "-开始等待！");
            condition.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            LOCK.unlock();
        }
    }
}
```



```properties
Thread-0-开始等待！
Thread-3-开始等待！
Thread-2-开始等待！
Thread-1-开始等待！
Thread-4-开始等待！
是否有线程在等待condition的条件：true
是否有线程在等待condition的条件：false
```



### isFair()

判断是否是公平锁

### isHeldByCurrentThread()

判断锁是否被当前线程持有

```java
/**
 * @author chenghao.li
 * 演示 isFair() 和 isHeldByCurrentThread()
 */
public class Test07 {
    static ReentrantLock LOCK = new ReentrantLock(true);

    public static void main(String[] args) {
        Test07 test07 = new Test07();
        new Thread(() -> test07.m1()).start();
    }

    public void m1() {
        try {
            System.out.println(LOCK.isFair());
            System.out.println(LOCK.isHeldByCurrentThread());
            LOCK.lock();
            System.out.println(LOCK.isHeldByCurrentThread());
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "-" + i);
            }
        } finally {
            if (LOCK.isHeldByCurrentThread()) {
                LOCK.unlock();
            }
        }
    }
}
```

### isLocked()

查询当前锁是否被线程持有

```java
/**
 * @author chenghao.li
 * 演示 isLocked() 判断锁是否被线程持有
 */
public class Test08 {
    static ReentrantLock LOCK = new ReentrantLock(true);

    public static void main(String[] args) throws InterruptedException {
        Test08 test08 = new Test08();
        new Thread(() -> test08.m1()).start();
        TimeUnit.SECONDS.sleep(3);
        System.out.println(LOCK.isLocked());
    }

    public void m1() {
        try {
            System.out.println(LOCK.isLocked());
            LOCK.lock();
            TimeUnit.SECONDS.sleep(1);
            System.out.println(LOCK.isLocked());
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "-" + i);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            if (LOCK.isHeldByCurrentThread()) {
                LOCK.unlock();
            }
        }
    }
}
```

### 



