---
title: JVM-GC相关参数
categories: jvm
tags:
  - java
  - jvm
  - GC
keywords: 'java,jvm'
cover: https://blog.lichenghao.cn/upload/2022/07/2c5696d8-jvm.png
abbrlink: 2c5696d8
date: 2022-05-15 13:54:00
---
## 相关参数

| 含义                | 参数                                                         |
| ------------------- | ------------------------------------------------------------ |
| 堆初始大小          | -Xms                                                         |
| 堆最大大小          | -Xmx 或者 -XX:MaxHeapSize=size                               |
| 新生代大小          | -Xmn 或者（-XX:NewSize=size  + -XX:MaxNewSize=size）         |
| 幸存区比例（动态）  | -XX:InitialSurvivorRatio=ratio 和 -XX:+UseAdaptiveSizePolicy |
| 幸存区比例          | -XX:SurvivorRatio=ratio  (默认：`伊甸园:FROM:TO  8:1:1`)     |
| 晋升阈值            | -XX:MaxTenuringThreshold=threshold                           |
| 打印晋升详情        | -XX:+PrintTenuingDistribution                                |
| 打印GC详情          | -XX:+PrintGCDetails -verbose:gc                              |
| Full GC前先 MinorGC | -XX:+ScavengeBeforeFullGC                                    |



## 演示 GC 过程

给定起始 JVM 参数，便于演示

```java
/**
 * 演示GC
 * -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc
 */
public class App {

    private static final int SIZE_512KB = 512 * 1024;
    private static final int SIZE_1MB = 1024 * 1024;
    private static final int SIZE_6MB = 6 * 1024 * 1024;
    private static final int SIZE_7MB = 7 * 1024 * 1024;
    private static final int SIZE_8MB = 8 * 1024 * 1024;

    public static void main(String[] args) {

    }
}
```

### 直接运行，看GC 信息

```java
Heap
 // 新生代 发现 eden:from:to 8:1:1
 def new generation   total 9216K, used 1685K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  eden space 8192K,  20% used [0x00000007bec00000, 0x00000007beda5440, 0x00000007bf400000)
  from space 1024K,   0% used [0x00000007bf400000, 0x00000007bf400000, 0x00000007bf500000)
  to   space 1024K,   0% used [0x00000007bf500000, 0x00000007bf500000, 0x00000007bf600000)
 // 老年代                              
 tenured generation   total 10240K, used 0K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
   the space 10240K,   0% used [0x00000007bf600000, 0x00000007bf600000, 0x00000007bf600200, 0x00000007c0000000)
 // 元空间
 Metaspace       used 2964K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 324K, capacity 388K, committed 512K, reserved 1048576K
```

其中，

def new generation   total 9216K, used 1685K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)

- 新生代总大小为 total 9216K，因为 to 区域始终要保持 1024 大小的空间。所以新生代的大小就去掉了 to 这块的大小。used 1685K 表示使用了 1685k。再后面的就是内存的地址了。

eden space 8192K,  20% used [0x00000007bec00000, 0x00000007beda5440, 0x00000007bf400000)

- 表示伊甸园的大小，和使用的容量，因为程序本省也需要创建一些对象，所以使用了 20%

from space 1024K,   0% used [0x00000007bf400000, 0x00000007bf400000, 0x00000007bf500000)

to   space 1024K,   0% used [0x00000007bf500000, 0x00000007bf500000, 0x00000007bf600000)

- 幸存区 FROM 和 TO 大小，使用的容量，以及内存地址

tenured generation   total 10240K, used 0K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)

- 老年代的大小，使用的容量，以及内存的地址



### 加 7MB 的数据

首先不允许任何额外的代码主程序在新生代都占用 了1685K，那么在加入 7MB 的数据一定会发生 GC

```java
/**
 * 演示GC
 * -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc
 */
public class App {

    private static final int SIZE_512KB = 512 * 1024;
    private static final int SIZE_1MB = 1024 * 1024;
    private static final int SIZE_6MB = 6 * 1024 * 1024;
    private static final int SIZE_7MB = 7 * 1024 * 1024;
    private static final int SIZE_8MB = 8 * 1024 * 1024;

    public static void main(String[] args) {
        List<byte[]> list = new ArrayList<>();
        list.add(new byte[SIZE_7MB]);
    }
}
```

打印 GC 信息

```java
[GC (Allocation Failure) [DefNew: 1520K->345K(9216K), 0.0023649 secs] 1520K->345K(19456K), 0.0023891 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 7923K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  eden space 8192K,  92% used [0x00000007bec00000, 0x00000007bf366850, 0x00000007bf400000)
  from space 1024K,  33% used [0x00000007bf500000, 0x00000007bf5566a0, 0x00000007bf600000)
  to   space 1024K,   0% used [0x00000007bf400000, 0x00000007bf400000, 0x00000007bf500000)
 tenured generation   total 10240K, used 0K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
   the space 10240K,   0% used [0x00000007bf600000, 0x00000007bf600000, 0x00000007bf600200, 0x00000007c0000000)
 Metaspace       used 3017K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 330K, capacity 388K, committed 512K, reserved 1048576K
```

[GC (Allocation Failure) [<span style="color:red">DefNew: 1520K->345K(9216K), 0.0023649 secs</span>] <span style="color:green">1520K->345K(19456K), 0.0023891 secs</span>] [Times: user=0.00 sys=0.00, real=0.00 secs] 

- 红色区域：表示新生代发生 GC，回收前的大小—>回收后的大小(初始大小)，回收用的时间
- 绿色区域：表示堆的回收前的占用的大小—>回收后占用的容量大小(初始大小) ，回收用的时间。从堆的角度来看。
- 回收后，伊甸园，FROM ,TO 占用发生了变化



### 加512k的数据

```java
/**
 * 演示GC
 * -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc
 */
public class App {

    private static final int SIZE_512KB = 512 * 1024;
    private static final int SIZE_1MB = 1024 * 1024;
    private static final int SIZE_6MB = 6 * 1024 * 1024;
    private static final int SIZE_7MB = 7 * 1024 * 1024;
    private static final int SIZE_8MB = 8 * 1024 * 1024;

    public static void main(String[] args) {
        List<byte[]> list = new ArrayList<>();
        list.add(new byte[SIZE_7MB]);
        list.add(new byte[SIZE_512KB]);
    }
}
```

GC信息

```java
[GC (Allocation Failure) [DefNew: 1520K->346K(9216K), 0.0009080 secs] 1520K->346K(19456K), 0.0009242 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 8436K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  eden space 8192K,  98% used [0x00000007bec00000, 0x00000007bf3e6838, 0x00000007bf400000)
  from space 1024K,  33% used [0x00000007bf500000, 0x00000007bf556880, 0x00000007bf600000)
  to   space 1024K,   0% used [0x00000007bf400000, 0x00000007bf400000, 0x00000007bf500000)
 tenured generation   total 10240K, used 0K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
   the space 10240K,   0% used [0x00000007bf600000, 0x00000007bf600000, 0x00000007bf600200, 0x00000007c0000000)
 Metaspace       used 3039K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 336K, capacity 388K, committed 512K, reserved 1048576K
```

同样发生了一次 minor gc ，伊甸园的占用变成了 98%。如果再次放 512K，那么一次回收可定不行了。

### 加512k的数据

```java
/**
 * 演示GC
 * -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc
 */
public class App {

    private static final int SIZE_512KB = 512 * 1024;
    private static final int SIZE_1MB = 1024 * 1024;
    private static final int SIZE_6MB = 6 * 1024 * 1024;
    private static final int SIZE_7MB = 7 * 1024 * 1024;
    private static final int SIZE_8MB = 8 * 1024 * 1024;

    public static void main(String[] args) {
        List<byte[]> list = new ArrayList<>();
        list.add(new byte[SIZE_7MB]);
        list.add(new byte[SIZE_512KB]);
        list.add(new byte[SIZE_512KB]);
    }
}
```

GC信息

```java
[GC (Allocation Failure) [DefNew: 1520K->345K(9216K), 0.0011261 secs] 1520K->345K(19456K), 0.0011506 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 8353K->858K(9216K), 0.0060489 secs] 8353K->8026K(19456K), 0.0060695 secs] [Times: user=0.00 sys=0.01, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 1699K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  eden space 8192K,  10% used [0x00000007bec00000, 0x00000007becd25c8, 0x00000007bf400000)
  from space 1024K,  83% used [0x00000007bf400000, 0x00000007bf4d69a0, 0x00000007bf500000)
  to   space 1024K,   0% used [0x00000007bf500000, 0x00000007bf500000, 0x00000007bf600000)
 tenured generation   total 10240K, used 7168K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
   the space 10240K,  70% used [0x00000007bf600000, 0x00000007bfd00010, 0x00000007bfd00200, 0x00000007c0000000)
 Metaspace       used 3040K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 337K, capacity 388K, committed 512K, reserved 1048576K
```

可以看出，发生了两次 minor gc ，同时 from 区域占用到了 83%，同时老年代也已经占用了 70%。



### 大对象直接晋升

如果一个对象比较大，新生代容不下，但是老年代可以容下，那么这个大对象直接就会晋升到老年代。

```java
/**
 * 演示GC
 * -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc
 */
public class App {

    private static final int SIZE_512KB = 512 * 1024;
    private static final int SIZE_1MB = 1024 * 1024;
    private static final int SIZE_6MB = 6 * 1024 * 1024;
    private static final int SIZE_7MB = 7 * 1024 * 1024;
    private static final int SIZE_8MB = 8 * 1024 * 1024;

    public static void main(String[] args) {
        List<byte[]> list = new ArrayList<>();
        list.add(new byte[SIZE_8MB]);
    }
}
```

GC信息

```java
Heap
 def new generation   total 9216K, used 1857K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  eden space 8192K,  22% used [0x00000007bec00000, 0x00000007bedd0788, 0x00000007bf400000)
  from space 1024K,   0% used [0x00000007bf400000, 0x00000007bf400000, 0x00000007bf500000)
  to   space 1024K,   0% used [0x00000007bf500000, 0x00000007bf500000, 0x00000007bf600000)
 tenured generation   total 10240K, used 8192K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
   the space 10240K,  80% used [0x00000007bf600000, 0x00000007bfe00010, 0x00000007bfe00200, 0x00000007c0000000)
 Metaspace       used 3045K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 338K, capacity 388K, committed 512K, reserved 1048576K
```



### OOM

```java
/**
 * 演示GC
 * -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc
 */
public class App {

    private static final int SIZE_512KB = 512 * 1024;
    private static final int SIZE_1MB = 1024 * 1024;
    private static final int SIZE_6MB = 6 * 1024 * 1024;
    private static final int SIZE_7MB = 7 * 1024 * 1024;
    private static final int SIZE_8MB = 8 * 1024 * 1024;

    public static void main(String[] args) {
        List<byte[]> list = new ArrayList<>();
        list.add(new byte[SIZE_8MB]);
        list.add(new byte[SIZE_8MB]);
    }
}
```

GC 信息

```java
[GC (Allocation Failure) [DefNew: 1693K->357K(9216K), 0.0015136 secs][Tenured: 8192K->8548K(10240K), 0.0018068 secs] 9885K->8548K(19456K), [Metaspace: 3076K->3076K(1056768K)], 0.0033601 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [Tenured: 8548K->8530K(10240K), 0.0016809 secs] 8548K->8530K(19456K), [Metaspace: 3076K->3076K(1056768K)], 0.0017078 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 410K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  eden space 8192K,   5% used [0x00000007bec00000, 0x00000007bec66800, 0x00000007bf400000)
  from space 1024K,   0% used [0x00000007bf500000, 0x00000007bf500000, 0x00000007bf600000)
  to   space 1024K,   0% used [0x00000007bf400000, 0x00000007bf400000, 0x00000007bf500000)
 tenured generation   total 10240K, used 8530K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
   the space 10240K,  83% used [0x00000007bf600000, 0x00000007bfe54bc8, 0x00000007bfe54c00, 0x00000007c0000000)
 Metaspace       used 3139K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 345K, capacity 388K, committed 512K, reserved 1048576K
                                
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at org.example.App.main(App.java:21)
```

连续放两个大对象，新生代老年代都放不下了，垃圾回收做了两次的补救，一次 minor gc 一次 full gc 发现还是么有足够的空间，那么就 OOM 了。

!>如果发生 OOM 的 代码运行在一个线程中，那么它不会影响主线程的正常的运行。

```java
/**
 * 演示GC
 * -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc
 */
public class App {

    private static final int SIZE_512KB = 512 * 1024;
    private static final int SIZE_1MB = 1024 * 1024;
    private static final int SIZE_6MB = 6 * 1024 * 1024;
    private static final int SIZE_7MB = 7 * 1024 * 1024;
    private static final int SIZE_8MB = 8 * 1024 * 1024;

    public static void main(String[] args) {
        new Thread(() -> {
            List<byte[]> list = new ArrayList<>();
            list.add(new byte[SIZE_8MB]);
            list.add(new byte[SIZE_8MB]);
        }).start();

        System.out.println("main go");
    }
}
```

GC信息

```java
main go
[GC (Allocation Failure) [DefNew: 3858K->590K(9216K), 0.0019024 secs][Tenured: 8192K->8780K(10240K), 0.0064335 secs] 12050K->8780K(19456K), [Metaspace: 4041K->4041K(1056768K)], 0.0083881 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
[Full GC (Allocation Failure) [Tenured: 8780K->8725K(10240K), 0.0052135 secs] 8780K->8725K(19456K), [Metaspace: 4041K->4041K(1056768K)], 0.0052637 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 def new generation   total 9216K, used 327K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  eden space 8192K,   4% used [0x00000007bec00000, 0x00000007bec51ee0, 0x00000007bf400000)
  from space 1024K,   0% used [0x00000007bf500000, 0x00000007bf500000, 0x00000007bf600000)
  to   space 1024K,   0% used [0x00000007bf400000, 0x00000007bf400000, 0x00000007bf500000)
 tenured generation   total 10240K, used 8725K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
   the space 10240K,  85% used [0x00000007bf600000, 0x00000007bfe85518, 0x00000007bfe85600, 0x00000007c0000000)
 Metaspace       used 4066K, capacity 4672K, committed 4864K, reserved 1056768K
  class space    used 458K, capacity 496K, committed 512K, reserved 1048576K
Exception in thread "Thread-0" java.lang.OutOfMemoryError: Java heap space
	at org.example.App.lambda$main$0(App.java:23)
	at org.example.App$$Lambda$1/1078694789.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)
```

发现并没有影响到主线程的正常运行。



