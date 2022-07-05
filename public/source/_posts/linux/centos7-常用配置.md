---
title: centos7-常用配置
categories: linux
tags:
  - linux
abbrlink: 1bb7f58c
date: 2022-04-01 12:00:00
---
## 配置网卡名称

装一些公司的产品需要统一网卡名称，修改 centos7 网卡名称的步骤记录下

修改网卡配置文件的名称

```sh
cd /etc/sysconfig/network-scripts/
mv ifcfg-eno1 ifcfg-eth0
```

进入修改后的网卡匹配文件，修改设备名称

```properties
NAME="eth0"
DEVICE="eth0"
```

接下来禁用网卡命名规则,在文件/etc/default/grub 中，`GRUB_CMDLINE_LINUX`属性下添加加`net.ifnames=0 biosdevname=0`即可

{% note primary modern %}
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap biosdevname=0 net.ifnames=0 rhgb quiet"
{% endnote %}

```properties
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap biosdevname=0 net.ifnames=0 rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

执行命令生成更新 grub 配置参数

```sh
grub2-mkconfig  -o  /boot/grub2/grub.cfg
```

重启系统即可

```sh
reboot
```



## 配置 JDK 环境变量

例如 JDK存放的位置为：`/opt/jdk8`

修改环境变量：

```bash
[root@localhost jdk8]# vim /etc/profile
```

增加内容如下：

```bash
#jdk
export JAVA_HOME=/opt/jdk8
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

刷新环境变量

```bash
[root@localhost jdk8]# source /etc/profile
```

测试即可

```bash
[root@localhost jdk8]# java -version
openjdk version "1.8.0_322"
OpenJDK Runtime Environment (Temurin)(build 1.8.0_322-b06)
OpenJDK 64-Bit Server VM (Temurin)(build 25.322-b06, mixed mode)
```



## 查看&修改系统时区

查询时区

```bash
[root@lch ~]# timedatectl | grep "Time zone"
       Time zone: America/New_York (CST, +0800)
```

时区列表

```bash
[root@lch ~]# timedatectl list-timezones
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
.....
```

设置时区

```bash
[root@lch ~]# timedatectl set-timezone Asia/Shanghai
[root@lch ~]# timedatectl | grep "Time zone"
       Time zone: Asia/Shanghai (CST, +0800)
```



