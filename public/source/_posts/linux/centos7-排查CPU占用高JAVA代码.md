---
title: centos7-排查CPU占用高JAVA代码
categories: linux
tags:
  - linux
abbrlink: 5aca8ef1
date: 2022-04-01 12:00:00
---
首先做一个测试的程序，让 CPU 和内存增高：其中线程名字为`high_thread` 的线程肯定为让 CPU 高起来。

```java
public class Test03 {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                System.out.println(1);
            }).start();
        }
        Thread thread = new Thread(() -> {
            while (true) {
                new Object();
            }
        });
        thread.setName("high_thread");
        thread.start();
    }
}
```

## jstack

首先通过`top` 命令查看占用 CPU 高的进程 `PID`

```sh
[root@VM-0-2-centos ~]# top
```

![](https://blog.lichenghao.cn/upload/2022/07/20210619172200.png)

通过命令查看该进程对应的线程 CPU 占用 ,找到 CPU 占用高的线程

```sh
[root@VM-0-2-centos ~]# top -Hp 17807
```

![](https://blog.lichenghao.cn/upload/2022/07/20210619172307.png)

将线程的`PID`转换成十六进制

```sh
[root@VM-0-2-centos ~]# printf '%x' 17827
45a3
```

通过` jstack` 命令定位的具体的类，主要是根据 `nid=0x45a3 这个值来查找的。可以看到有问题的线程名称。

```sh
[root@VM-0-2-centos ~]# jstack 17807 | grep -a 45a3
"high_thread" #18 prio=5 os_prio=0 tid=0x00007f7224108800 nid=0x45a3 runnable [0x00007f7214dfc000]
```

直接查看不方便查询，还可以将具体的日志转存到自定义文件上,然后在文件上查找 `nid=0x45a3` 的记录，可以看到具体的代码行数等！

参数是进程的 ID

```sh
[root@VM-0-2-centos ~]# jstack 17807 >> jstack01.log
```

查看文件后，会发现具体的代码行数是 at Test03.lambda$main$1(Test03.java:10)

```sh

"high_thread" #18 prio=5 os_prio=0 tid=0x00007f7224108800 nid=0x45a3 runnable [0x00007f7214dfc000]
   java.lang.Thread.State: RUNNABLE
        at Test03.lambda$main$1(Test03.java:10)
        at Test03$$Lambda$2/303563356.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

```

## Arthas

阿里开源的工具`Arthas` https://arthas.gitee.io/quick-start.html

工具文档：https://arthas.aliyun.com/zh-cn/

### 启动

使用`arthas-boot`（推荐）

下载`arthas-boot.jar`，然后用`java -jar`的方式启动：

```sh
curl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jar
```

启动后，会列出当前的 JAVA 进程，输入对应的序号，回车即可，就进入了

```sh
[root@VM-0-2-centos ~]# java -jar arthas-boot.jar
[INFO] arthas-boot version: 3.5.1
[INFO] Process 17807 already using port 3658
[INFO] Process 17807 already using port 8563
[INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
* [1]: 17807 Test03
1
[INFO] arthas home: /root/.arthas/lib/3.5.1/arthas
[INFO] The target process already listen port 3658, skip attach.
[INFO] arthas-client connect 127.0.0.1 3658
  ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.
 /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'
|  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.
|  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |
`--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'


wiki       https://arthas.aliyun.com/doc
tutorials  https://arthas.aliyun.com/doc/arthas-tutorials.html
version    3.5.1
main_class
pid        17807
time       2021-06-19 17:37:31

[arthas@17807]$
```

进入看板：能看到线程 ID 等等

```sh
[arthas@17807]$ dashboard
```

![](https://blog.lichenghao.cn/upload/2022/07/20210619174506.png)

`thread id`查看有问题的线程`high_thread`，其中 id 就是线程的 ID

```sh
[arthas@17807]$ thread 18
"high_thread" Id=18 RUNNABLE
    at Test03.lambda$main$1(Test03.java:10)
    at Test03$$Lambda$2/303563356.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:748)

[arthas@17807]$
```

### 反编译类信息

反编译类，查看具体的代码：发现第 10 行代码有问题的地方是： new Object();

```sh
[arthas@17807]$ jad Test03

ClassLoader:
+-sun.misc.Launcher$AppClassLoader@4e0e2f2a
  +-sun.misc.Launcher$ExtClassLoader@52a7d4f2

Location:
/root/

       /*
        * Decompiled with CFR.
        */
       public class Test03 {
           public static void main(String[] stringArray) {
/* 3*/         for (int i = 0; i < 10; ++i) {
                   new Thread(() -> System.out.println(1)).start();
               }
               Thread thread = new Thread(() -> {
                   while (true) {
                       new Object();
                   }
               });
/*13*/         thread.setName("high_thread");
/*14*/         thread.start();
           }
       }

Affect(row-cnt:1) cost in 475 ms.
[arthas@17807]$
```



### watch:查看方法参数以及返回值

```sh
[arthas@17807]$ watch com.xx.xx addSomeThing "{params,returnObj}" -x 2 -b 
```

这里的-x 2 表示显示对象的深度；-b 表示方法调入前。

结果：发现返回结果是null，说明可能异常出现。

```sh
Press Q or Ctrl+C to abort.
Affect(class count: 3 , method count: 3) cost in 461 ms, listenerId: 4
method=com.xx.xx $$EnhancerBySpringCGLIB$$8346a983.add location=AtEnter
ts=2022-03-16 13:16:23; [cost=15.373494ms] result=@ArrayList[
    @Object[][
        @ItemDO[ItemDO{id=null, baseId='c8991f8c0df943e2bc1a1dc2f4d6316a', opType=null}],
    ],
    null,
]
```

Watch 命令：

https://arthas.aliyun.com/doc/watch.html

### 退出工具

如果只是退出当前的连接，可以用`quit`或者`exit`命令。Attach 到目标进程上的 arthas 还会继续运行，端口会保持开放，下次连接时可以直接连接上。

如果想完全退出 arthas，可以执行`stop`命令。



## jmap

jamp 是 JVM 的命令，它可以生成 java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列。我经常用它来生成程序的 dump 文件，然后在本地通过其他工具来分析堆对象的统计信息。

`jmap -heap pid ` 查看堆信息



注: MacOS jdk8 下会报错如下，因为 MAC 不让用！

➜  example jmap -heap 3956
Attaching to process ID 3956, please wait...
ERROR: attach: task_for_pid(3956) failed: '(os/kern) failure' (5)
Error attaching to process: sun.jvm.hotspot.debugger.DebuggerException: Can't attach to the process. Could be caused by an incorrect pid or lack of privileges.



`jmap -histo:live pid ` 显示堆中对象的统计信息



`jmap -dump:live,format=b,file=heapdump.hprof pid`  生成堆转储快照dump文件







