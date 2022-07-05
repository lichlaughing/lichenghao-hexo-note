---
title: 锁 synchronized 起步
categories: java
tags: 
  - synchronized
  - Thread
abbrlink: bbe7b043
date: 2022-02-01 12:00:00
---



[我用阿里云盘分享了「Java编程语言（第三版）.pdf」](https://www.aliyundrive.com/s/qQYTqrs8o83)

------

使用 synchronized可以使临界区的代码是线程安全的，也是最能简单上手的。

## 同步且原子

演示线程同步，原子操作

**不加锁**

```java
/**
 * @author Chenghao.li
 * 演示 synchronized 同步，原子操作
 */
public class Test01 {

    public static void main(String[] args) {
        // 非同步
        SumWithOutLock sumWithOutLock = new SumWithOutLock();
        for (int i = 0; i < 2; i++) {
            new Thread(sumWithOutLock).start();
        }
    }

    static class SumWithOutLock implements Runnable {
        int sum = 0;

        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                sum++;
            }
            System.out.println(sum);
        }
    }
}
```

```java
// 结果不唯一
11185
10000
```

**加锁**

```java
/**
 * @author Chenghao.li
 * 演示 synchronized 同步，原子操作
 */
public class Test01 {

    public static void main(String[] args) {
        // 同步
        SumWithLock sumWithLock = new SumWithLock();
        for (int i = 0; i < 2; i++) {
            new Thread(sumWithLock).start();
        }
    }
    static class SumWithLock implements Runnable {
        int sum = 0;

        @Override
        public void run() {
            synchronized (this) {
                for (int i = 0; i < 10000; i++) {
                    sum++;
                }
                System.out.println(sum);
            }
        }
    }
}
```

```java
// 结果
10000
20000
```

## 修饰代码块、成员方法、静态方法

**同步代码块**

```java
/**
 * @author Chenghao.li
 * 演示 synchronized 修改代码块，修饰实例方法，修饰静态方法
 */
public class Test02 {

    public static void main(String[] args) {

        // 修饰代码块
        SumWithLock sumWithLock = new SumWithLock();
        for (int i = 0; i < 2; i++) {
            new Thread(sumWithLock).start();
        }  
    }

    /**
     * 同步代码块
     */
    static class SumWithLock implements Runnable {
        int sum = 0;

        @Override
        public void run() {
            synchronized (this) {
                for (int i = 0; i < 10000; i++) {
                    sum++;
                }
                System.out.println(sum);
            }
        }
    }
}
```

**同步静态方法**

```java
/**
 * @author Chenghao.li
 * 演示 synchronized 修改代码块，修饰实例方法，修饰静态方法
 */
public class Test02 {

    public static void main(String[] args) {
        // 修饰静态方法
        for (int i = 0; i < 2; i++) {
            new Thread(() -> Test02.sum2()).start();
        }
    }

    /**
     * 同步静态方法
     */
    public synchronized static void sum2() {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + "-" + i);
        }
    }
}
```

**同步实例方法**

```java
/**
 * @author Chenghao.li
 * 演示 synchronized 修改代码块，修饰实例方法，修饰静态方法
 */
public class Test02 {

    public static void main(String[] args) {
        // 修饰实例方法
        Test02 t02 = new Test02();
        for (int i = 0; i < 2; i++) {
            // 必须是相同的锁
            new Thread(() -> t02.sum()).start();
            // 这个是不同的锁对象，所以不是同步的
            new Thread(() -> new Test02().sum()).start();
        }
    }

    /**
     * 同步实例方法
     */
    public synchronized void sum() {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + "-" + i);
        }
    }
}
```

## 锁的类型

**对象锁**

```java
/**
 * @author Chenghao.li
 * 演示 对象锁、类锁、常量锁
 */
public class Test03 {

    public static void main(String[] args) {
        // 对象锁
        Test03 t1 = new Test03();
        for (int i = 0; i < 2; i++) {
            new Thread(() -> t1.print()).start();
        }
    }

    public void print() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "-" + i);
            }
        }
    }
}
```

```java
Thread-0-0
Thread-0-1
Thread-0-2
Thread-0-3
Thread-0-4
Thread-0-5
Thread-0-6
Thread-0-7
Thread-0-8
Thread-0-9
Thread-1-0
Thread-1-1
Thread-1-2
Thread-1-3
Thread-1-4
Thread-1-5
Thread-1-6
Thread-1-7
Thread-1-8
Thread-1-9

Process finished with exit code 0
```

**常量锁**

```java
/**
 * @author Chenghao.li
 * 演示 对象锁、类锁、常量锁
 */
public class Test03 {

    private static final String LOCK = "LOCK";

    public static void main(String[] args) {
        // 常量锁
        for (int i = 0; i < 2; i++) {
            new Thread(() -> new Test03().print3()).start();
        }
    }

    public void print3() {
        synchronized (LOCK) {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "-" + i);
            }
        }
    }
}
```

```java
Thread-0-0
Thread-0-1
Thread-0-2
Thread-0-3
Thread-0-4
Thread-0-5
Thread-0-6
Thread-0-7
Thread-0-8
Thread-0-9
Thread-1-0
Thread-1-1
Thread-1-2
Thread-1-3
Thread-1-4
Thread-1-5
Thread-1-6
Thread-1-7
Thread-1-8
Thread-1-9

Process finished with exit code 0
```

**类锁**

```java
/**
 * @author Chenghao.li
 * 演示 对象锁、类锁、常量锁
 */
public class Test03 {

    public static void main(String[] args) {

        // 类锁
        for (int i = 0; i < 2; i++) {
            new Thread(() -> new Test03().print2()).start();
        }
    }

    public void print2() {
        synchronized (Test03.class) {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + "-" + i);
            }
        }
    }
}
```

```java
Thread-0-0
Thread-0-1
Thread-0-2
Thread-0-3
Thread-0-4
Thread-0-5
Thread-0-6
Thread-0-7
Thread-0-8
Thread-0-9
Thread-1-0
Thread-1-1
Thread-1-2
Thread-1-3
Thread-1-4
Thread-1-5
Thread-1-6
Thread-1-7
Thread-1-8
Thread-1-9

Process finished with exit code 0
```

## 死锁

  获取锁的顺序不一致，导致死锁

```java
/**
 * @author chenghao.li
 * 演示死锁
 */
public class Test04 {
  public static void main(String[] args) {
      DeadLock deadLock = new DeadLock();
      new Thread(() -> deadLock.methodA()).start();
      new Thread(() -> deadLock.methodB()).start();
  }

  static class DeadLock {
      private static final String LOCK_A = "LOCK_A";
      private static final String LOCK_B = "LOCK_B";

      public void methodA() {
          synchronized (LOCK_A) {
              System.out.println(Thread.currentThread().getName() + "- 拿到了 A 锁,继续拿 B 锁");
              try {
                  Thread.sleep(1000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              synchronized (LOCK_B) {
                  System.out.println(Thread.currentThread().getName() + "- 拿到了 B 锁！");
              }
          }
      }

      public void methodB() {
          synchronized (LOCK_B) {
              System.out.println(Thread.currentThread().getName() + "- 拿到了 B 锁,继续拿 A 锁");
              synchronized (LOCK_A) {
                  System.out.println(Thread.currentThread().getName() + "- 拿到了 A 锁！");
              }
          }
      }
  }
}
```

```java
Thread-0- 拿到了 A 锁,继续拿 B 锁
Thread-1- 拿到了 B 锁,继续拿 A 锁
```

## VOLATILE

voliatile 可以看成是轻量级的 synchronized，它在多线程的环境下保证了共享变量的可见性。

定义：Java 编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致性的更新，线程应该确保通过排他锁单独的获得这个变量。

作用：只能修饰变量，保证变量在多线程之间是可见的，但是不能保证变量的计算是原子操作

实现：volatile修饰的共享变量在进行读写的操作的时候，在多核处理器下回发生两件事情：

- 将当前处理器缓存行的数据写回到系统内存中
- 这个写回内存的的操作会使其他 CPU 中缓存了该内存地址的数据无效

说白了就是，我这改了，其他的 CPU 就的重新读取这个变量。

**不使用 volatile**

```java
/**
 * @author chenghao.li
 * 演示 volatile 使变量在线程之间可见，但不是原子操作
 */
public class Test05 {
    public static void main(String[] args) throws InterruptedException {
        // 子线程
        Sub sub = new Sub();
        new Thread(() -> sub.sayHello()).start();
        // 修改状态，并不能让子线程可见
        Thread.sleep(2000);
        sub.setFlag(false);
    }

    static class Sub {
        boolean flag = true;

        public void setFlag(boolean flag) {
            this.flag = flag;
        }

        public void sayHello() {
            System.out.println("start ...");
            while (flag) {

            }
            System.out.println("end ...");
        }
    }
}
```

**使用 volatile**

```java
/**
 * @author chenghao.li
 * 演示 volatile 使变量在线程之间可见，但不是原子操作
 */
public class Test06 {
    public static void main(String[] args) throws InterruptedException {
        Sub sub = new Sub();
        new Thread(() -> sub.sayHello()).start();
        // 使用volatile,修改状态，线程可见
        Thread.sleep(2000);
        sub.setFlag(false);
    }

    static class Sub {
        volatile private boolean flag = true;

        public void setFlag(boolean flag) {
            this.flag = flag;
        }

        public void sayHello() {
            System.out.println("start ...");
            while (flag) {

            }
            System.out.println("end ...");
        }
    }
}
```

**非原子**

```java
/**
 * @author chenghao.li
 * 演示 volatile 修饰变量，不是原子操作
 */
public class Test07 {
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Sub().start();
        }
    }

    static class Sub extends Thread {
        private volatile static int num = 0;

        public void sum() {
            for (int i = 0; i < 10000; i++) {
                num++;
            }
        }

        @Override
        public void run() {
            sum();
            System.out.println(num);
        }
    }
}
```

## CAS

Compare And Swap  是由硬件实现的原子操作。将 Read-Modify-Write 这类的操作转换成原子操作。

?> 原理：我们读取了主内存中变量的值，进行一顿操作后，在变量的值更新到主内存时，再次读取主内存中该变量的值，如果现在的变量的值和"期望的值"一样就更新。这个"期望的值"就是我们起始读的变量的值。

```java
/**
 * @author chenghao.li
 * 演示 CAS 原子操作
 */
public class Test08 {
    public static void main(String[] args) {
        CASCounter casCounter = new CASCounter();
        for (int i = 0; i < 1000; i++) {
            new Thread(() -> {
                long l = casCounter.incrementAndGet();
                System.out.println(l);
            }).start();
        }
    }
}

/**
 * 原子计数器
 */
class CASCounter {
    private volatile long value = 0;

    public long getValue() {
        return value;
    }

    public boolean compareAndSwap(long expected, long newValue) {
        synchronized (this) {
            if (this.value == expected) {
                this.value = newValue;
                return true;
            } else {
                return false;
            }
        }
    }

    public long incrementAndGet() {
        long oldValue;
        long newValue;
        do {
            oldValue = value;
            newValue = oldValue + 1;
        } while (!compareAndSwap(oldValue, newValue));
        return newValue;
    }
}
```

### ABA 问题

共享变量的当前值和当前线程提供的期望值相等，就认为这个变量没有被其他线程修改过。这个情况不一定总是成立，假设有共享变量count = 0;

Thread-0 将 count 修改为 10;

Thread-1 将count 修改为 20;

Thread-2 将 count 修改为 0;

这个时候当前线程看到的 count 的值仍然是 0，但是其实这个变量已经被修改了多次！这个就是 CAS中的 ABA问题。

如果要规避这个问题，可以为变量添加个版本号即可。（可以使用原子类中的AtomicStampedReference）

## 原子变量类

原子变量类基于 CAS实现，通过原子变量类可以保障变量操作的原子性和可见性。

原子变量类有12 个

| 分组    | 原子变量类                                                                       |
| ----- | --------------------------------------------------------------------------- |
| 基础型   | AtomicInteger,AtomicLong,AtomicBoolean                                      |
| 数组型   | AtomicIntegerArray,AtomicLongArray,AtomicReferenceArray                     |
| 字段更新型 | AtomicIntegerFieldUpdater,AtomicLongFiledUpdater,AtomicReferenceFildUpdater |
| 引用型   | AtomicReference,AtomicStampedReference,AtomicMarkableReference              |

### AtomicLong

演示 AtomicLong 实现计数器

**计数器**

```java
/**
 * @author chenghao.li
 * 单例模式,使用AtomicLongCounter模拟计数器。记录网站的请求总数，成功数，失败数
 */
public class AtomicLongCounter {
    private volatile static AtomicLongCounter atomicLongCounter;

    private static final AtomicLong REQUEST_COUNT = new AtomicLong(0);
    private static final AtomicLong SUCCESS_REQUEST_COUNT = new AtomicLong(0);
    private static final AtomicLong FAIL_REQUEST_COUNT = new AtomicLong(0);

    private AtomicLongCounter() {

    }

    public static AtomicLongCounter getInstance() {
        if (atomicLongCounter == null) {
            synchronized (AtomicLongCounter.class) {
                if (atomicLongCounter == null) {
                    atomicLongCounter = new AtomicLongCounter();
                }
            }
        }
        return atomicLongCounter;
    }

    /**
     * 新的请求
     */
    public void requestReceive() {
        REQUEST_COUNT.incrementAndGet();
    }

    /**
     * 成功的请求
     */
    public void successRequestReceive() {
        SUCCESS_REQUEST_COUNT.incrementAndGet();
    }

    /**
     * 失败的请求
     */
    public void failRequestReceive() {
        FAIL_REQUEST_COUNT.incrementAndGet();
    }

    /**
     * 插件请求总数
     *
     * @return
     */
    public long getRequestCount() {
        return REQUEST_COUNT.get();
    }

    /**
     * 插件成功的请求总数
     *
     * @return
     */
    public long getSuccessRequestCount() {
        return SUCCESS_REQUEST_COUNT.get();
    }

    /**
     * 查看失败的请求总数
     *
     * @return
     */
    public long getFailRequestCount() {
        return FAIL_REQUEST_COUNT.get();
    }
}
```

**模拟网页请求**

```java
/**
 * @author chenghao.li
 * 模拟网页请求
 */
public class WebRequestTest {
    public static void main(String[] args) throws InterruptedException {
        AtomicLongCounter counter = AtomicLongCounter.getInstance();
        for (int i = 0; i < 1000; i++) {
            new Thread(() -> {
                counter.requestReceive();
                int r = new Random().nextInt(1000);
                if (r % 2 == 0) {
                    counter.successRequestReceive();
                } else {
                    counter.failRequestReceive();
                }
            }).start();
        }
        TimeUnit.SECONDS.sleep(1);
        long requestCount = counter.getRequestCount();
        long successRequestCount = counter.getSuccessRequestCount();
        long failRequestCount = counter.getFailRequestCount();
        System.out.println("请求总数：" + requestCount);
        System.out.println("请求成功总数：" + successRequestCount);
        System.out.println("请求失败总数：" + failRequestCount);
    }
}
```

### AtomicIntegerArray

**演示AtomicIntegerArray的基础操作**

```java
/**
 * @author chenghao.li
 * 演示 AtomicIntegerArray 的一些基础操作
 */
public class AtomicIntegerArrayTest {
    public static void main(String[] args) {
        // 1.声明数组，长度为 10
        AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(10);
        System.out.println("1." + atomicIntegerArray);
        // 2.返回指定位置的元素
        int i = atomicIntegerArray.get(2);
        System.out.println("2." + i);
        // 设置指定位置元素的值 atomicIntegerArray.set(0,10);
        // 3.设置指定位置的元素，并且返回旧值
        int andSet = atomicIntegerArray.getAndSet(3, 10);
        System.out.println("3." + andSet);
        // 4.把数组元素的值，加上某个值。返回这个元素的新值
        int i1 = atomicIntegerArray.addAndGet(4, 10);
        System.out.println("4." + i1);
        // 5.获取某个元素的值，然后再给这个元素加上某个值
        int andAdd = atomicIntegerArray.getAndAdd(4, 10);
        System.out.println("5." + andAdd);
        // 6.CAS 操作，数组下标为6的位置的元素的值为10就替换成20
        boolean b = atomicIntegerArray.compareAndSet(6, 10, 20);
        System.out.println("6." + b);
        // 自增/自减
        int i2 = atomicIntegerArray.incrementAndGet(7);
        int i3 = atomicIntegerArray.decrementAndGet(8);
        System.out.println("7." + i2 + "," + i3);
        System.out.println(atomicIntegerArray);
    }
}
```

**多线程下使用AtomicIntegerArray**

```java
/**
 * @author chenghao.li
 * 演示多线程下使用原子数组,给数组中的每个元素自增 1000
 */
public class AtomicIntegerArrayThreadTest {
    private static AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(10);

    public static void main(String[] args) {
        Thread[] threads = new Thread[10];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Sub();
        }
        for (Thread thread : threads) {
            thread.start();
        }
        for (Thread thread : threads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(atomicIntegerArray);
    }

    static class Sub extends Thread {

        @Override
        public void run() {
            for (int i = 0; i < 1000; i++) {
                for (int j = 0; j < 10; j++) {
                    // 给每个元素自增 1000
                    atomicIntegerArray.incrementAndGet(j % 10);
                }
            }
        }
    }
}
```

### AtomicIntegerFieldUpdater

对原子整型字段进行更新，满足如下要求才可以使用：

- 字段必须使用volatile修饰，使其线程之间可见

- 只能是实例变量，不能是静态变量，也不能用 private 、final修饰

**多线程修改AGE 属性**

```java
/**
 * @author chenghao.li
 * 演示多线程下对某个属性进行修改
 */
public class AtomicIntegerFieldUpdaterTest {

    private static AtomicIntegerFieldUpdater atomicIntegerFieldUpdater =
            AtomicIntegerFieldUpdater.newUpdater(User.class, "age");

    public static void main(String[] args) throws InterruptedException {
        User user = new User("1", 0);
        for (int i = 0; i < 10; i++) {
            new Sub(user).start();
        }
        TimeUnit.SECONDS.sleep(1);
        System.out.println(user);
    }

    static class Sub extends Thread {
        private User user;

        public Sub(User user) {
            this.user = user;
        }

        @Override
        public void run() {
            for (int i = 0; i < 1000; i++) {
                atomicIntegerFieldUpdater.incrementAndGet(user);
            }
        }
    }
}
```

**User对象**

```java
/**
 * @author chenghao.li
 */
public class User {
    private String name;
    volatile int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

### AtomicReference

原子操作一个对象

```java
/**
 * @author chenghao.li
 */
public class ReferenceTest {

    private static AtomicReference atomicReference = new AtomicReference<>("ABC");

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                try {
                    TimeUnit.SECONDS.sleep(new Random().nextInt(5));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                if (atomicReference.compareAndSet("ABC", "DEF")) {
                    System.out.println(Thread.currentThread().getName() + "修改为 DEF");
                }
            }).start();
        }
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                try {
                    TimeUnit.SECONDS.sleep(new Random().nextInt(5));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                if (atomicReference.compareAndSet("DEF", "ABC")) {
                    System.out.println(Thread.currentThread().getName() + "修改为 ABC");
                }
            }).start();
        }
    }
}
```

### AtomicStampedReference

AtomicStampedReference 解决 CAS中的 ABA 问题

```java
/**
 * @author chenghao.li
 * 演示AtomicStampedReference解决 CAS ABA 问题
 */
public class AtomicStampedReferenceTest {
    private static AtomicStampedReference<String> atomicStampedReference =
            new AtomicStampedReference<>("ABC", 0);

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            atomicStampedReference.compareAndSet("ABC", "DEF", atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "-" + atomicStampedReference.getReference());
            atomicStampedReference.compareAndSet("DEF", "ABC", atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
        }).start();

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean b = atomicStampedReference.compareAndSet("ABC", "NEW ABC", stamp, stamp + 1);
            System.out.println(b);
        }).start();

        TimeUnit.SECONDS.sleep(2);
        System.out.println(atomicStampedReference.getReference());
    }
}
```

## 线程通信

### WAIT

使用条件：在同步代码中，由锁对象调用，调用后当前线程会释放锁对象

```java
/**
 * @author chenghao.li
 * 演示wait线程等待
 */
public class WaitTest01 {
    private static final Object LOCK = new Object();

    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "- 开始等待.....");
            synchronized (LOCK) {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        thread.start();
    }
}
```

### INTERRUPT

被终止的线程会抛出异常：`java.lang.InterruptedException`, 释放锁对象

```java
/**
 * @author chenghao.li
 * 演示interrupt终止线程
 */
public class WaitTest02 {
    private static final Object LOCK = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "- 开始等待.....");
            synchronized (LOCK) {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    System.out.println(Thread.currentThread().getName() + "- 线程被终止!");
                }
            }
        });
        thread.start();
        // 一秒后终止线程
        TimeUnit.SECONDS.sleep(1);
        thread.interrupt();
        boolean interrupted = thread.isInterrupted();
        System.out.println(interrupted);
    }
}
```

### NOTIFY

使用条件，在同步代码中，由锁对象调用，唤醒等待的线程。如果有多个等待的线程，随机唤醒其中的一个。

!> 如果在非同步代码中执行会报错：`Exception in thread "main" java.lang.IllegalMonitorStateException`

**nofity**

```java
/**
 * @author chenghao.li
 * 演示notify唤醒wait的线程
 */
public class NotifyTest01 {
    private static final Object LOCK = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "- 开始等待.....");
            synchronized (LOCK) {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "- 等待结束!");
        });
        thread.start();
        // 1秒后唤醒
        TimeUnit.SECONDS.sleep(1);
        synchronized (LOCK) {
            LOCK.notify();
        }
    }
}
```

**nofityAll**

唤醒所有等待的线程

```java
/**
 * @author chenghao.li
 * 演示notifyAll唤醒wait的线程
 */
public class NotifyTest02 {
    private static final Object LOCK = new Object();

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "- 开始等待.....");
                synchronized (LOCK) {
                    try {
                        LOCK.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + "- 等待结束!");
            }).start();
        }

        // 2秒后唤醒所有等待的线程
        TimeUnit.SECONDS.sleep(2);
        synchronized (LOCK) {
            LOCK.notifyAll();
        }
    }
}
```

**!反例**

```java
/**
 * @author chenghao.li
 * 演示notify唤醒wait的线程
 */
public class NotifyTest01 {
    private static final Object LOCK = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "- 开始等待.....");
            synchronized (LOCK) {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "- 等待结束!");
        });
        thread.start();
        // 1秒后唤醒
        TimeUnit.SECONDS.sleep(1);
        LOCK.notify();
    }
}
// 报错：
Exception in thread "main" java.lang.IllegalMonitorStateException
```

!> 除非你经过了深刻的深思熟虑，否则请你不要考虑，直接使用 `notifyAll()`

### 一些问题

?> notify 通知后，要等待当前的同步代码执行完毕后，才会去通知等待的线程，通知了等待的线程后，也不一定会立刻执行，需要等待CPU的调度。

```java
/**
 * @author chenghao.li
 * 演示notifyAll唤醒，wait的线程不会立刻释放锁对象
 */
public class NotifyTest03 {
    private static final Object LOCK = new Object();

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "- 开始等待.....");
                synchronized (LOCK) {
                    try {
                        LOCK.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + "- 等待结束!");
            }).start();
        }

        // 2秒后唤醒所有等待的线程
        TimeUnit.SECONDS.sleep(2);
        synchronized (LOCK) {
            LOCK.notifyAll();
            for (int i = 0; i < 10; i++) {
                System.out.println("还在处理一些东西" + i);
            }
        }
    }
}
```

?>  notify 通知过早

<!-- panels:start -->

<!-- div:title-panel -->

 线程还么有进行等待，就通知了，那么会线程一直等待下去。

<!-- div:left-panel -->

```java
/**
* @author chenghao.li
* 演示wait-notify组合中通知过早的问题
*/
public class WaitTest03 {
   private static final Object LOCK = new Object();

   public static void main(String[] args) throws InterruptedException {
       Thread t1 = new Thread(() -> {
           System.out.println(Thread.currentThread().getName() + "- 开始等待.....");
           synchronized (LOCK) {
               try {
                   LOCK.wait();
               } catch (InterruptedException e) {
                   e.printStackTrace();
                   System.out.println(Thread.currentThread().getName() + "- 线程被终止!");
               }
           }
       });

       Thread t2 = new Thread(() -> {
           synchronized (LOCK) {
               LOCK.notify();
               System.out.println(Thread.currentThread().getName() + "- notify通知了.....");
           }
       });

       t2.start();
       t1.start();
   }
}
```

解决通知过早的问题

```java
/**
 * @author chenghao.li
 * 演示wait-notify组合中解决通知过早的问题
 */
public class WaitTest04 {
    private static final Object LOCK = new Object();
    private static boolean flag = true;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "- 开始等待.....");
            synchronized (LOCK) {
                while (flag) {
                    try {
                        LOCK.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                        System.out.println(Thread.currentThread().getName() + "- 线程被终止!");
                    }
                }
                System.out.println(Thread.currentThread().getName() + "- 等待结束！");
            }
        });

        Thread t2 = new Thread(() -> {
            synchronized (LOCK) {
                LOCK.notify();
                flag = false;
                System.out.println(Thread.currentThread().getName() + "- notify通知了.....");
            }
        });

        t2.start();
        t1.start();
    }
}
```

```properties
# 可能会出现的结果
Thread-1- notify通知了.....
Thread-0- 开始等待.....
```

```properties
# 解决通知过早可能出现的结果
Thread-1- notify通知了.....
Thread-0- 开始等待.....
Thread-0- 等待结束！
```

?> wait条件改变导致的问题

**wait条件改变导致的问题**

```java
/**
 * @author chenghao.li
 * 演示wait条件发生改变，引起的问题
 */
public class WaitTest05 {

    public static void main(String[] args) throws InterruptedException {
        Sub sub = new Sub();
        // 一百个线程取值，一百个线程设置值
        for (int i = 0; i < 100; i++) {
            new Thread(() -> sub.getStr()).start();
            new Thread(() -> sub.setStr()).start();
        }
    }

    static class Sub {
        static String str = "";

        public void setStr() {
            synchronized (this) {
                if (StringUtils.isNotBlank(str)) {
                    try {
                        this.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                str = "hello";
                System.out.println(Thread.currentThread().getName() + "- 设置值成功!");
                this.notifyAll();
            }
        }

        public void getStr() {
            synchronized (this) {
                if (StringUtils.isBlank(str)) {
                    try {
                        this.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + "- 取值成功!>" + str);
                str = "";
                this.notifyAll();
            }
        }
    }
}
```

某一的执行结果：发现取的值为空！

```properties
Thread-88- 取值成功!>hello
Thread-0- 取值成功!>
Thread-85- 设置值成功!
Thread-84- 取值成功!>hello
Thread-83- 设置值成功!
Thread-82- 取值成功!>hello
Thread-86- 取值成功!>
Thread-116- 取值成功!>
Thread-129- 设置值成功!
```

**避免wait条件改变导致的问题**

产生的问题在于，被唤醒的线程没有判断下字符串是否为空，就去获取了，导致问题的出现。修改如下

```java
/**
 * @author chenghao.li
 * 演示wait条件发生改变，引起的问题
 */
public class WaitTest05 {

    public static void main(String[] args) throws InterruptedException {
        Sub sub = new Sub();
        // 一百个线程取值，一百个线程设置值
        for (int i = 0; i < 100; i++) {
            new Thread(() -> sub.getStr()).start();
            new Thread(() -> sub.setStr()).start();
        }
    }

    static class Sub {
        static String str = "";

        public void setStr() {
            synchronized (this) {
                while (StringUtils.isNotBlank(str)) {
                    try {
                        this.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                str = "hello";
                System.out.println(Thread.currentThread().getName() + "- 设置值成功!");
                this.notifyAll();
            }
        }

        public void getStr() {
            synchronized (this) {
                while (StringUtils.isBlank(str)) {
                    try {
                        this.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + "- 取值成功!>" + str);
                str = "";
                this.notifyAll();
            }
        }
    }
}
```

!> 除非你经过了深刻的深思熟虑，否则不要犹豫,请使用`while`来做条件判断！

## ThreadLocal

多线程下为了保证共享资源访问的正确性的，可以通过 synchornized 这样的内部锁来保护共享资源，除此之外，我们还可以增加资源的数量来保证资源访问的正确性。比如多个线程给每个线程一个资源，大家不用争抢也就不存在线程安全的问题了！

ThreadLocal 字面意思就是线程的本地存储，它可以为每个线程存储自己的数据。

```java
/**
 * @author chenghao.li
 * 演ThreadLocal增加资源
 */
public class ThreadLocalTest01 {

    static ThreadLocal<SimpleDateFormat> simpleDateFormatThreadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(new Sub()).start();
        }
    }

    static class Sub implements Runnable {
        @Override
        public void run() {
            for (int i = 0; i < 60; i++) {
                try {
                    if (simpleDateFormatThreadLocal.get() == null) {
                        simpleDateFormatThreadLocal.set(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
                    }
                    Date parse = simpleDateFormatThreadLocal.get().parse("2020-01-01 10:10:" + i % 60);
                    System.out.println(parse);
                } catch (ParseException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

ThreadLocal是没有默认值的，可以通过重写`initialValue`为其初始化默认值

```java
/**
 * @author chenghao.li
 * 演示演示 ThreadLocal 初始化默认值
 */
public class ThreadLocalTest02 {
    public static void main(String[] args) {
        SubThreadLocal subThreadLocal = new SubThreadLocal();
        String s = subThreadLocal.get();
        System.out.println(s);
    }

    static class SubThreadLocal extends ThreadLocal<String> {
        @Override
        protected String initialValue() {
            return "Hello";
        }
    }
}
```



## SYNCHRONIZED 原理

### JAVA对象头

对象头由三部分组成：

- Mark Word (Mark Word在32位JVM中的长度是32bit，在64位JVM中长度是64bit。)

- 指向类的指针

- 数组长度（只有数组对象才有）

例如32位JVM下的对象头。

```properties

|--------------------------------------------------------------|
|                     Object Header (64 bits)                  |
|------------------------------------|-------------------------|
|        Mark Word (32 bits)         |    Klass Word (32 bits) |
|------------------------------------|-------------------------|
```

```properties

|---------------------------------------------------------------------------------|
|                                 Object Header (96 bits)                         |
|--------------------------------|-----------------------|------------------------|
|        Mark Word(32bits)       |    Klass Word(32bits) |  array length(32bits)  |
|--------------------------------|-----------------------|------------------------|
```

那么在加锁解锁的过程中，Mark Word 会根据锁的类型会有不同的值：

```properties
|-------------------------------------------------------|--------------------|
|                  Mark Word (32 bits)                  |       State        |
|-------------------------------------------------------|--------------------|
| identity_hashcode:25 | age:4 | biased_lock:1 | lock:2 |       Normal       |
|-------------------------------------------------------|--------------------|
|  thread:23 | epoch:2 | age:4 | biased_lock:1 | lock:2 |       Biased       |
|-------------------------------------------------------|--------------------|
|               ptr_to_lock_record:30          | lock:2 | Lightweight Locked |
|-------------------------------------------------------|--------------------|
|               ptr_to_heavyweight_monitor:30  | lock:2 | Heavyweight Locked |
|-------------------------------------------------------|--------------------|
|                                              | lock:2 |    Marked for GC   |
|-------------------------------------------------------|--------------------|
```

| 锁状态  | 25bit      |       | 4bit | 1bit  | 2bit |
| ---- |:---------- | ----- | ---- | ----- | ---- |
|      | 23bit      | 2bit  |      | 是否偏向锁 | 锁标志  |
| 无锁   | 对象HashCode |       | 分代年龄 | 0     | 01   |
| 偏向锁  | 线程ID       | Epoch | 分代年龄 | 1     | 01   |
| 轻量级锁 | 指向栈中锁记录的指针 |       |      |       | 00   |
| 重量级锁 | 指向重量级锁的指针  |       |      |       | 10   |
| GC   | 空          |       |      |       | 11   |



### MONITOR

每一个对象都可以关联一个Monitor对象，这个对象是操作系统提供的，使用sychronized给对象加锁后，该对象头的Mark Word就会被设置指向该Monitor对象的指针。如下所示：

![](https://blog.lichenghao.cn/upload/2022/07/211202.png)





