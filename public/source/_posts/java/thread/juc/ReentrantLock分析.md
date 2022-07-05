---
title: ReentrantLock 分析 (AQS)
categories: java
tags: 
  - ReentrantLock
  - java
abbrlink: a914049
date: 2022-02-01 12:00:00
---



ReentrantLock分析

首先 ReentrantLock也是内部维护了一个同步器，并且分为公平和非公平锁。

![](https://blog.lichenghao.cn/upload/2022/07/ReentrantLock.png)



## ReentrantLock 非公平锁，加解锁流程

通过构造方法可以看出，默认就是非公平锁

```java
//Creates an instance of ReentrantLock. This is equivalent to using ReentrantLock(false).
public ReentrantLock() {
        sync = new NonfairSync();
    }
```

### 1. Thread-0线程获得锁

当第一个线程没有竞争的时候调用 lock 方法获取锁

```java
public void lock() {
        sync.lock();
    }
```

也是调用了同步器的 lock方法,如下所示：

- 将状态从 0 变成 1
- 同时将锁的 owner 设置为当前线程
- 当前这种情况会成功，因为我们假设当前只有一个线程

```java
static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
    }
```

如下图所示，线程获取到了锁，状态为 1，锁持有者为当前线程 thread-0。

![](https://blog.lichenghao.cn/upload/2022/07/15.05.36.png)

### 2. Thread-1线程竞争锁

如果又来了一个新的线程 thread-1,出现了竞争，这个时候thread-1线程通过 CAS 把状态从 0 变成 1 肯定是失败的。

![](https://blog.lichenghao.cn/upload/2022/07/15.14.39.png)

那么就会进入上面的 lock 方法中的 acquire 方法如下所示：该方法还会尝试去获取锁，因为我们当前 thread-0拿着锁，所以tryAcquire(arg)方法始终会返回 false。那么就会调用acquireQueued方法创建一个队列的节点，并加入等待队列。

```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

### 3. Thread-0竞争失败，进入队列前再尝试

那么 thread-1尝试获取锁失败后，进入队列的过程如下所示：

- Thread-1 CAS 尝试把 state从 0 变成 1，然后失败了
- 进入tryAcquire() 方法，这时候 thread-0持有者锁，所以也失败了
- addWaiter(Node.EXCLUSIVE)方法，创建队列节点，并加入到队列中
  - 图中黄色三角表示该 Node的等待状态，为 0 表示默认的正常状态
  - Node 的创建是惰性的
  - 其中第一个节点称为 Dummy 或者哨兵，用来占位，并不关联线程。首次创建节点的时候会先创建该节点。

![](https://blog.lichenghao.cn/upload/2022/07/15.18.09.png)

线程 thread-1节点加入队列后，继续会进入acquireQueued方法逻辑，去尝试再次获取锁

```java
    /**
     * Acquires in exclusive uninterruptible mode for thread already in
     * queue. Used by condition wait methods as well as acquire.
     *
     * @param node the node
     * @param arg the acquire argument
     * @return {@code true} if interrupted while waiting
     
     该方法在会在一个死循环中不断的尝试去获得锁，如果还是失败了的话，就被 park 阻塞
     */
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
               // 如果自己前面是头节点，我在第二位的话，就再去尝试获取锁，因为 thread-1持有锁，所以这块又失败了
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
              	// 如果获取锁失败了，是不是要被阻塞，
                // 第一次方法会返回 false,同时会把它前面的节点的等待状态设置为-1
              	// 这样第二次在过来的时候返回 ture,就会执行parkAndCheckInterrupt 将线程阻塞
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

// 该方法把自己前面的节点的等待状态变成 -1；状态为-1 的节点有责任唤醒它后面的等待节点
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }

// park 阻塞线程
private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```

1. 在一个死循环中不断的进行尝试获取锁

2. 如果自己前面是头节点，自己在第二位的时候，就去尝试获取锁，当然这肯定是失败的，应为thread-0持有这锁
3. 上一步失败后，会进入shouldParkAfterFailedAcquire方法，会把自己前一个节点的等待状态设置为-1 （节点状态为-1 的的话，那么这个节点有责任去唤醒它后面等待的节点）

![](https://blog.lichenghao.cn/upload/2022/07/15.49.10.png)

### 4. Thread-0尝试多次失败，进入阻塞

4. 这一个循序结束后，会进入下一个循环，还是去tryAcquire还是失败了，应为这个时候 state=1,thread-0持有这锁
5. 再次进入shouldParkAfterFailedAcquire这个时候，方法就会返回 true，因为前驱节点的等待状态为-1
6. 然后就会进入parkAndCheckInterrupt()方法，线程thread-1进入阻塞状态

![](https://blog.lichenghao.cn/upload/2022/07/16.01.40.png)

### 5. 多个线程都竞争失败了

如果有多个线程都像 thread-1一样，都竞争锁失败了，就会进入队列中等待，如下所示：

![](https://blog.lichenghao.cn/upload/2022/07/16.15.10.png)



### 6. Thread-0 释放锁

Thread-0释放锁，就会进入tryRelease流程

```java
public void unlock() {
        sync.release(1);
    }
```

```java
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

如果释放锁成功，那么

- 设置ExclusiveOwnerThread为 null

- state 设置为 0

  （不成功的时候，是锁可重入的时候）

![](https://blog.lichenghao.cn/upload/2022/07/16.23.27.png)

### 7. 唤醒等待队列中的线程(没竞争)

并且当前队列不为空，head节点的等待状态为-1，那么就就需要进入unparkSuccessor唤醒等待线程的过程。找到离head 最近的一个节点，unpark 让其恢复运行。那么 thread-1就会回到锁竞争的流程。

上面讲过，线程在正式进入队列的时候还会尝试几次，然后才会被阻塞,如果 unpark 了。那么该线程就会再次进入循环，这个时候，它的头节点是 head,而且tryAcquire(arg)也返回 true,然后就会把当前节点设置为头节点，原来的头节点去掉。

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

所以当没有竞争的时候，thread-1成功获取了锁，都做了哪些：

1. ExclusiveOwnerThread 设置为 thread-1
2. State 设置为 1
3. head指向 thread-1所在的节点，原来的头节点去掉

![](https://blog.lichenghao.cn/upload/2022/07/16.40.26.png)

### 8. 唤醒等待队列中的线程(有竞争)

如果 thread-1在获取锁的时候，又来了 thread-4来竞争，并且 thread-4抢先占有锁，那么 thread-1又会再次进入acquireQueued中的tryAcquire流程，最终再次进入 park 阻塞。



## 锁可重入的原理

ReentrantLock # Sync

加锁，可重入

```java
final boolean nonfairTryAcquire(int acquires) {
  					
            final Thread current = Thread.currentThread();
            int c = getState();
  					// 如果状态为 0，则把状态设置为 1，并且把锁持有者设置为当前线程
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
  					// 如果 state 不是 0，并且锁持有者为当前线程
  					// 那么说明发生了锁重入，state自增，表示调用了几次锁
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

解锁

```java
protected final boolean tryRelease(int releases) {
  					// 状态自减
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
  					// 如果为 0，表示释放锁成功
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
  					// 有重入的锁，并没有完全释放锁
            setState(c);
            return free;
        }
```

## 打断原理

### 不可打断模式

默认情况下是不可以被打断的。

获取锁的方法：ReentrantLock#lock() ——>NonfairSync#lock——>AbstractQueuedSynchronizer#acquire() 获取锁失败——>AbstractQueuedSynchronizer#acquireQueued(addWaiter(Node.EXCLUSIVE), arg))



- 当线程获取锁失败后，就会进入acquireQueued方法，要进入队列等待了。当前一个节点的等待状态为-1 的时候，线程就会调用parkAndCheckInterrupt进入阻塞状态

- 当其他线程调用了当前线程的interrupted打断了线程的阻塞，这块代码的

  if (shouldParkAfterFailedAcquire(p, node) &&parkAndCheckInterrupt()) 逻辑就会为真，那么阻塞打断了，for(;;)就会继续走循环体再次尝试获取锁。如果依然没能获取锁，还是会再次进入 park 阻塞

- parkAndCheckInterrupt方法，在返回打断状态的同时，还会清除打断标记

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                  // 如果线程被interrupted了，那么就会继续走循环了
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

private final boolean parkAndCheckInterrupt() {
  			// 线程阻塞,如果其他线程调用当前线程的interrupted方法，那么park就会失效
        LockSupport.park(this);
  			// 返回打断状态，同时清除打断标记
        return Thread.interrupted();
    }
```

Thread的方法

```java
public static boolean interrupted() {
        return currentThread().isInterrupted(true);
    }
```

所以在不可打断模下，即便打断了线程，线程还是会在等待队列中继续尝试获取锁，直到获取成功后才能知道原来我被打断了。

### 可打断模式



获取锁的方法：ReentrantLock#lockInterruptibly() ——>Sync#acquireInterruptibly



```java
...AbstractQueuedSynchronizer

public final void acquireInterruptibly(int arg)
            throws InterruptedException {
        // 如果线程被打断，就直接抛出异常
        if (Thread.interrupted())
            throw new InterruptedException();
  			// 如果没有获得锁，那么进入 doAcquireInterruptibly
        if (!tryAcquire(arg))
            doAcquireInterruptibly(arg);
    }

private void doAcquireInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                  	// 和不可打断不同的是，如果线程被唤醒，就直接抛出异常了
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```



## ReentrantLock 公平/非公平代码

### 非公平锁获取锁的原理代码

```java
final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
              // 如果没人占用锁，直接去抢占，也不管等待队列中是否有等待的线程
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```



### 公平锁获取锁的实现原理代码

```java
protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
              // 如果没人占用锁，看看等待的队列中是否有等待的线程，没有的话再竞争锁
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```



## 条件变量实现原理

每一个条件变量，对应这一个等待队列，实现类是`ConditionObject`

### await 流程

```java
public final void await() throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            Node node = addConditionWaiter();
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            while (!isOnSyncQueue(node)) {
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }
```

- Thread-0持有锁并调用了 await 后进入等待
- 执行addConditionWaiter()方法，该方法会在等待队列上创建节点并关联上等待的线程，该节点的状态为-2（Node.CONDITION）表示是条件等待的线程

![](https://blog.lichenghao.cn/upload/2022/07/11.01.14.png)

- 然后fullyRelease(node) 这个方法会释放同步器上的锁

  ![](https://blog.lichenghao.cn/upload/2022/07/11.09.14.png)

- 释放锁后， unpark AQS 中等待队列中的下一个节点，如果下一个节点竞争锁成功的话，如下所示：

  ![](https://blog.lichenghao.cn/upload/2022/07/11.11.23.png)

  

### signal 流程

假设 thread-1来唤醒正在条件变量上等待的 thead-0

- 首先会调用signal()，这里会检查下是否为锁的持有者
- 然后等待队列的首个元素不为空，就doSignal(Node first)去唤醒

```java
public final void signal() {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node first = firstWaiter;
            if (first != null)
                doSignal(first);
        }

private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                first.nextWaiter = null;
            } while (!transferForSignal(first) &&
                     (first = firstWaiter) != null);
        }
```

![](https://blog.lichenghao.cn/upload/2022/07/12.50.56.png)

- 唤醒的操作就是把线程再次放到锁的等待队列中

  ```java
  final boolean transferForSignal(Node node) {
          /*
           * If cannot change waitStatus, the node has been cancelled.
           */
          if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
              return false;
  
          /*
           * Splice onto queue and try to set waitStatus of predecessor to
           * indicate that thread is (probably) waiting. If cancelled or
           * attempt to set waitStatus fails, wake up to resync (in which
           * case the waitStatus can be transiently and harmlessly wrong).
           */
          Node p = enq(node);
          int ws = p.waitStatus;
          if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
              LockSupport.unpark(node.thread);
          return true;
      }
  ```

  如果成功的话,就会到锁的队列中等待了。失败的情况例如：被打断主动放弃锁。

  ![](https://blog.lichenghao.cn/upload/2022/07/12.56.34.png)

  















