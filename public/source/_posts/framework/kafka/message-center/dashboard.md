---
title: kafka 可视化监控工具
categories: kafka
tags:
  - kafka
keywords: 'kafka,消息中心'
abbrlink: 4b3ba034
date: 2022-02-10 12:00:00
---
可视化监控工具

## kafka-eagle

官网：https://www.kafka-eagle.org/

Github：https://github.com/smartloli/kafka-eagle

下载：http://download.kafka-eagle.org/

官网下载很慢，给出蓝奏云：https://wwb.lanzoul.com/iCrBo060iuni 版本 kafka-eagle-bin-2.1.0

解压后修改配置文件 `system-config.properties` 修改如下两个地方就可以基本使用。

```properties
######################################
# multi zookeeper & kafka cluster list
# Settings prefixed with 'kafka.eagle.' will be deprecated, use 'efak.' instead
######################################
#efak.zk.cluster.alias=cluster1,cluster2
#cluster1.zk.list=tdn1:2181,tdn2:2181,tdn3:2181
#cluster2.zk.list=xdn10:2181,xdn11:2181,xdn12:2181
# zk的集群配置
efak.zk.cluster.alias=cluster1
cluster1.zk.list=192.168.1.240:2181,192.168.1.240:2182,192.168.1.240:2183

.....

######################################
# kafka mysql jdbc driver address
######################################
# 修改为自己的数据库
efak.driver=com.mysql.cj.jdbc.Driver
efak.url=jdbc:mysql://127.0.0.1:3306/ke?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
efak.username=root
efak.password=Root@123456
```

然后后bin 下的启动脚本授权

```bash
[root@lch bin]# chmod +x ke.sh 
```

启动服务即可，启动成功后，给出了登录地址，账号和密码。

```bash
[2022-06-06 22:50:45] INFO: Port Progress: [##################################################] | 100%
[2022-06-06 22:50:49] INFO: Config Progress: [##################################################] | 100%
[2022-06-06 22:50:52] INFO: Startup Progress: [##################################################] | 100%
[2022-06-07 10:50:38] INFO: Status Code[0]
[2022-06-07 10:50:38] INFO: [Job done!]
Welcome to
    ______    ______    ___     __ __
   / ____/   / ____/   /   |   / //_/
  / __/     / /_      / /| |  / ,<   
 / /___    / __/     / ___ | / /| |  
/_____/   /_/       /_/  |_|/_/ |_|  
( Eagle For Apache Kafka® )

Version 2.1.0 -- Copyright 2016-2022
*******************************************************************
* EFAK Service has started success.
* Welcome, Now you can visit 'http://192.168.1.240:8048'
* Account:admin ,Password:123456
*******************************************************************
* <Usage> ke.sh [start|status|stop|restart|stats] </Usage>
* <Usage> https://www.kafka-eagle.org/ </Usage>
*******************************************************************
```

![](https://blog.lichenghao.cn/upload/2022/07/07105223.png)