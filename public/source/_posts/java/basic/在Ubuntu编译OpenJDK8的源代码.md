---
title: Ubuntu编译OpenJDK8的源代码
categories: java
tags:
  - java
  - jdk8
abbrlink: fb309fc7
---



阅读 JDK 源码最好的方式，就是编译 JDK 源码！然后在开发工具中用源码的方式运行，网上有很多教程，但是只有自己亲身操作过，才会知道具体是怎么样的——>纸上得来终觉浅，绝知此事要躬行 👍

---

> 环境准备
>
> ubuntu 16.04
>
> ```bash
> lich@lich-virtual-machine:~/workspace$ uname -a
> Linux lich-virtual-machine 5.4.0-42-generic #46-Ubuntu SMP Fri Jul 10 00:24:02 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
> ```
>
> ```bash
> lich@lich-virtual-machine:~/workspace/openjdk8$ cat /proc/version
> Linux version 4.15.0-45-generic (buildd@lcy01-amd64-027) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.10)) #48~16.04.1-Ubuntu SMP Tue Jan 29 18:03:48 UTC 2019
> ```

最开始我是用 ubuntu 20.4 版本，但是编译过程中出现了许多奇怪的错误，所以选用的低版本的 ubuntu

因为主要的目的是编译源码阅读源码，不是研究版本的兼容性问题。高版本的 openJDK 应该是可以的，没有测试，用 openjdk8 比较多，所以还是想编辑 openjdk8。所以最终还是选用了低版本的 ubuntu。

---

# 下载 openJDK8 源码

两种方式：==① 利用 Mercurial(hg)== ==② 手动下载==

## 利用 Mercurial(hg)

openJDK 使用的代码管理工具为 Mercurial（hg），下载并安装 Mercurial 后就可以通过 hg clone 命令获取 openJDK8 的源码了，相关的命令如下：

```bash
hg clone http://hg.openjdk.java.net/jdk8u/jdk8u openjdk8u
cd openjdk
./get_source.sh
```

> 没有 hg 命令需要先安装：如果失败的话可能是网络的问题，在执行一次
>
> ```bash
> sudo apt-get install mercurial
> ```

实际上，执行了 hg clone http://hg.openjdk.java.net/jdk8u/jdk8u openjdk8u 命令以后，执行到如下所示就卡死了，不动了～ 所以还是使用下面手动下载的方式

```bash
lich@lich-virtual-machine:~/workspace$ hg clone http://hg.openjdk.java.net/jdk8u/jdk8u openjdk8u
正在请求全部修改
正在增加修改集
正在增加清单
正在增加文件改变
files [================>                                                             ]  32/142 2m50s
```

## 手动下载

下载地址：http://jdk.java.net/ 选择需要的版本，下载即可

RI Source Code

The source code of the RI binaries is available under the [GPLv2](<javascript:void(0)>) in a single [zip file](https://download.java.net/openjdk/jdk8u41/ri/openjdk-8u41-src-b04-14_jan_2020.zip) ([md5](https://download.java.net/openjdk/jdk8u41/ri/openjdk-8u41-src-b04-14_jan_2020.zip.md5)) 123 MB.

下载后得到文件：openjdk-8u41-src-b04-14_jan_2020.zip

解压文件,得到文件夹 openjdk ,重命名为 openjdk8

```bash
unzip openjdk-8u41-src-b04-14_jan_2020.zip
```

# 编译源代码

## 安装 BootJDK

要编译 JDK 你的有个基础的 JDK，所以先安装个 JDK,版本小于等于我们要编译的 JDK，作为引导 JDK

方式一：

```bash
$ sudo add-apt-repository ppa:openjdk-r/ppa
$ sudo apt update
$ sudo apt install openjdk-7-jdk
```

方式二：

可以自己下载 jdk 然后自己配置环境变量就行了。

同样在 http://jdk.java.net/ 下载 JDK7 注意这次不是下载源码，是编译好的 JDK7

openjdk-7u75-b13-linux-x64-18_dec_2014.tar.gz 解压后重命名为 openjdk7

给 JDK7 配置系统环境变量,切换为 root

```bash
vim /etc/profile
```

文件末尾追加如下：注意修改为自己 JDK 的路径

```bash
export JAVA_HOME=/home/lich/workspace/openjdk7
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

切换成自己的用户，配置生效

```bash
source /etc/profile
```

测试

```bash
lich@lich-virtual-machine:~/workspace$ java -version
java version "1.7.0_95"
OpenJDK Runtime Environment (IcedTea 2.6.4) (7u95-2.6.4-3)
OpenJDK 64-Bit Server VM (build 24.95-b01, mixed mode)
```

## 安装编译相关的依赖

看其他教程都已经把需要的依赖给列出来了，所以我们就直接安装就好了

```bash
sudo apt install \
        libx11-dev \
        libxext-dev \
        libxrender-dev \
        libxtst-dev \
        libxt-dev \
        libcups2-dev \
        libfreetype6-dev \
        libasound2-dev \
        libfontconfig1-dev
```

## 依赖检查

在源码根路径下执行如下命令，检查是否还缺少其他的依赖

```bash
bash ./configure
```

比如我执行完毕后，后面会有提示,按照提示安装相关依赖即可

```bash
......
Build performance summary:
* Cores to use:   4
* Memory limit:   7934 MB
* ccache status:  not installed (consider installing)

Build performance tip: ccache gives a tremendous speedup for C++ recompilations.
You do not have ccache installed. Try installing it.
You might be able to fix this by running 'sudo apt-get install ccache'.
```

安装了 ccache 后出现问题: ccache status: installed, but disabled (version older than 3.1.4) 。没事不影响

```bash
Build performance summary:
* Cores to use:   4
* Memory limit:   7934 MB
* ccache status:  installed, but disabled (version older than 3.1.4)

WARNING: The result of this configuration has overridden an older
configuration. You *should* run 'make clean' to make sure you get a
proper build. Failure to do so might result in strange build problems.
```

## 编译

编译前检查下

（1）设定语言选项，如果不是 C 那么就修改下

```bash
lich@lich-virtual-machine:~/workspace/openjdk8$ echo $LANG
zh_CN.UTF-8
修改命令：
export LANG=C
```

（2）执行 echo $PATH

```bash
看下输出，如果没有boot JDK，则执行（注意修改为自己的路径,如果用上面打命令安装后，路径大概和我打一样）
export PATH="/usr/lib/jvm/java-7-openjdk-amd64/bin:${PATH}"
```

（3）检查 JAVA_HOME ，可先执行 echo $JAVA_HOME，看下输出，如果有值则需要 unset JAVA_HOME

检查完毕后，编译使用的命令如下：

```bash
make all
```

相关报错：

1》编译内核问题

```bash
*** This OS is not supported: Linux lich-virtual-machine 4.15.0-45-generic #48~16.04.1-Ubuntu SMP Tue Jan 29 18:03:48 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
/home/lich/workspace/openjdk8/hotspot/make/linux/Makefile:238: recipe for target 'check_os_version' failed
make[5]: *** [check_os_version] Error 1
/home/lich/workspace/openjdk8/hotspot/make/linux/Makefile:259: recipe for target 'linux_amd64_compiler2/debug' failed
```

查看内核版本

```bash
lich@lich-virtual-machine:~/workspace/openjdk8$ uname -a
Linux lich-virtual-machine 4.15.0-45-generic #48~16.04.1-Ubuntu SMP Tue Jan 29 18:03:48 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

查看 ubuntu 版本

```bash
lich@lich-virtual-machine:~/workspace/openjdk8$ cat /proc/version
Linux version 4.15.0-45-generic (buildd@lcy01-amd64-027) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.10)) #48~16.04.1-Ubuntu SMP Tue Jan 29 18:03:48 UTC 2019
```

解决办法：

修改`....../openjdk8/hotspot/make/linux/Makefile`，找到`SUPPORTED_OS_VERSION`变量定义的地方，在后面追加`4%`，如下所示

```
SUPPORTED_OS_VERSION = 2.4% 2.5% 2.6% 3% 4%
```

解决完报错后，再次执行 make all ,听电脑风扇一顿爆转就行了，最终如下所示：

```bash
----- Build times -------
Start 2020-07-25 10:57:00
End   2020-07-25 11:04:10
00:00:16 corba
00:00:12 demos
00:01:25 docs
00:02:48 hotspot
00:00:14 images
00:00:10 jaxp
00:00:15 jaxws
00:01:40 jdk
00:00:00 langtools
00:00:10 nashorn
00:07:10 TOTAL
-------------------------
Finished building OpenJDK for target 'all'
```

验证下，进入编译好的路径下 执行熟悉的 `java -version`

`....../openjdk8/build/linux-x86_64-normal-server-release/jdk/bin`

```bash
lich@lich-virtual-machine:~/workspace/openjdk8/build/linux-x86_64-normal-server-release/jdk/bin$ ./java -version
openjdk version "1.8.0-internal"
OpenJDK Runtime Environment (build 1.8.0-internal-lich_2020_07_25_10_27-b00)
OpenJDK 64-Bit Server VM (build 25.40-b25, mixed mode)
```

验证编译代码：

创建一个 Test.java 源文件，内容如下：

```java
public class Test{   public static void main(String[] args){ System.out.println("Hello World!");}}
```

通过 Javac 编译器编译如上的源代码，得到 Test.class 文件，`注意自己JDK的路径`

```
/home/lich/workspace/openjdk8/build/linux-x86_64-normal-server-release/jdk/bin/javac Test.java
```

运行如上的 Class 文件，命令如下：

```bash
/home/lich/workspace/openjdk8/build/linux-x86_64-normal-server-release/jdk/bin/java Test
```

输出如下的信息：`表示成功了`

```bash
lich@lich-virtual-machine:~$ /home/lich/workspace/openjdk8/build/linux-x86_64-normal-server-release/jdk/bin/java Test

Hello World!
```

# IDEA 中使用 JDK 源码调试

## 安装 IntelliJ IDEA

首先在 ubuntu 系统下，安装个 idea

下载 https://www.jetbrains.com/idea/download/#section=linux idea

解压到 opt 目录

```bash
 sudo tar -zxvf ideaIU-2020.1.4.tar.gz -C /opt
```

然后进入 bin 下执行 idea.sh 就行了，但是每次这么用比较麻烦，所以我们需要把 idea 添加到启动器

先进入到 applications 文件夹下

```
cd /usr/share/applications
```

添加

```bash
sudo vim idea.desktop
```

内容如下所示，修改为自己的`idea路径`

```xml
[Desktop Entry]
Encoding=UTF-8
Version=1.0
Name=IntelliJ IDEA
GenericName=Java IDE
Comment=IntelliJ IDEA is a code-centric IDE focused on developer
Exec=/opt/idea-IU-201.8743.12/bin/idea.sh
Icon=/opt/idea-IU-201.8743.12/bin/idea.png
Terminal=false
Type=Application
Categories=Development;IDE
```

赋予执行权限

```bash
sudo chmod +x idea.desktop
```

然后去软件列表找到启动器即可

## 使用 JDK

然后在 SDK 中导入编译的 JDK，debug 配置中去掉 Before launch 中打 build 然后执行测试代码

```java
public class Main {

    public static void main(String[] args) {
        System.out.println("hello world");
    }
}

执行结果：显示用打我们编译好的JDK
/home/lich/ideaworkspace/openjdk8-normal-server-release/jdk/bin/java -agentlib:jdwp=transport=dt_socket,address=127.0.0.1:47693,suspend=y,server=n -javaagent:/opt/idea-IU-201.8743.12/plugins/java/lib/rt/debugger-agent.jar -Dfile.encoding=UTF-8 -classpath /home/lich/ideaworkspace/openjdk8-normal-server-release/jdk/classes:/home/lich/ideaworkspace/test/out/production/test:/opt/idea-IU-201.8743.12/lib/idea_rt.jar com.org.Main
Connected to the target VM, address: '127.0.0.1:47693', transport: 'socket'

hello world
```

## 修改 JDK 源码

平时我们在查看源码打过程中，虽然可以通过 DEBUG 打断点来查看，但是有打时候不如加个打印输出更方便，所以我们可以修改源码，打造自己的 JDK。

举个例子，比如我们要修改`System.out.println`增加个打印前缀

我们知道 println 在 java.io 包下，我们去修改下源码，进入我们下载打源码中，然后重新 make 就可以了。

我们在打印字符串打时候增加一个前缀：

```java
 /**
     * Prints a String and then terminate the line.  This method behaves as
     * though it invokes <code>{@link #print(String)}</code> and then
     * <code>{@link #println()}</code>.
     *
     * @param x  The <code>String</code> to be printed.
     */
    public void println(String x) {
        synchronized (this) {
            print("lich:");
            print(x);
            newLine();
        }
    }
```

然后从新 make 下即可，有报错会提示，因为是增量编译，所以很快，然后再执行测试代码

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("hello world");
    }
}

结果：
lich:hello world
```

至此，入门打源码编译就结束了，纸上得来终觉浅，绝知此事要躬行 👍
