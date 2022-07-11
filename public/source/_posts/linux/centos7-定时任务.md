---
title: Centos 利用 crontab 执行定时任务
tags:
  - linux
  - centos
  - cron
categories: linux
keywords: linux centos 定时任务
abbrlink: f115c21b
date: 2022-04-01 13:19:27
---

## 安装定时任务组件

查看定时任务

```shell
[root@VM-16-3-centos ~]# crontab -l 
*/5 * * * * flock -xn /tmp/stargate.lock -c '/usr/local/qcloud/stargate/admin/start.sh > /dev/null 2>&1 &'
10 0 * * *  /www/server/cron/3ab48c27ec99cb9787749c362afae517 >> /www/server/cron/3ab48c27ec99cb9787749c362afae517.log 2>&1
```

如果没有这个命令的话，可以安装并设置开机自启即可

```shell
[root@VM-16-3-centos ~]# yum install crontabs
[root@VM-16-3-centos ~]# systemctl enable crond
[root@VM-16-3-centos ~]# systemctl start crond
```

## 创建定时任务

任务的基本格式：`*　　*　　*　　*　　*　　command` 分别代表：分　时　日　月　周　命令

- 第1列表示分钟1～59 (每分钟用*或者 */1表示)
- 第2列表示小时1～23（0表示0点）
- 第3列表示日期1～31
- 第4列表示月份1～12
- 第5列标识号星期0～6（0表示星期天）
- 第6列要运行的命令

其中还有一些特殊的表示方法：

```properties
*：表示任意时间都，实际上就是“每”的意思。可以代表00-23小时或者00-12每月或者00-59分
-：表示区间，是一个范围，00 17-19 * * * cmd，就是每天17,18,19点的整点执行命令
,：是分割时段，30 3,19,21 * * * cmd，就是每天凌晨3和晚上19,21点的半点时刻执行命令
/n：表示分割，可以看成除法，*/5 * * * * cmd，每隔五分钟执行一次
```



### 方式 1：写在文件/etc/crontab

可以将定时任务的命令写进配置文件 /etc/crontab 中

比如：

每分钟打印一次 hello world 到文件中。

```shell
# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
# 自定义任务 分别为：时间 用户 脚本
*/1 * * * * root /opt/push/push.py 
```

启动定时任务

```shell
[root@VM-16-3-centos ~]#  crontab /etc/crontab
```

查看运行日志

```shell
[root@VM-16-3-centos ~]#  tail -f /var/log/cron
```



### 方式 2：crontab -e

直接用crontab -e

### 方式 3：/var/spool/cron/root

写入/var/spool/cron/root(是用户名称)

