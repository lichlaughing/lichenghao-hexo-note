---
title: IoTDB-安装及基本概念
categories: IoTDB
tags:
  - IoTDB
keywords: IoTDB
abbrlink: 9524427d
date: 2022-05-20 12:30:00
---
IoTDB

Apache IoTDB（物联网数据库）是一体化收集、存储、管理与分析物联网时序数据的软件系统。 Apache IoTDB 采用轻量式架构，具有高性能和丰富的功能，并与Apache Hadoop、Spark和Flink等进行了深度集成，可以满足工业物联网领域的海量数据存储、高速数据读取和复杂数据分析需求。

其他时序数据库如：InfluxDB、OpenTSDB、Cassandra。

官网：https://iotdb.apache.org/

特点：

- 目录结构的时间序列组织管理方式
- 面向时间序列的丰富查询语义
- ......

架构图：IoTDB套件由若干个组件构成，共同形成“数据收集-数据写入-数据存储-数据查询-数据可视化-数据分析”等一系列功能。

![](https://blog.lichenghao.cn/upload/2022/07/48ED18F2DAA34CBC81748EA095CB4F73.png)

## 安装

下载地址：

最新版：https://iotdb.apache.org/Download/

历史版本：https://archive.apache.org/dist/iotdb/

解压后放到opt下

```shell
[root@localhost ~]# unzip apache-iotdb-0.12.5-all-bin.zip 
[root@localhost ~]# mv apache-iotdb-0.12.5-all-bin apache-iotdb
```

启动服务

```shell
[root@localhost apache-iotdb]# nohup sbin/start-server.sh >/dev/null 2>&1 &
```

> 通常写个启动脚本比较香！start.sh
>
> ```shell
>  nohup sbin/start-server.sh > iotdb.log 2>&1 &
> ```
>
> 有启动就有停止! stop.sh
>
> ```shell
> ID=`ps -ef | grep IoTDB  | grep java | awk '{print $2}'`
> for id in $ID
> do
>         kill -9 $id
>         echo "kill $id"
> done
> 
> ```
>
> 脚本放在安装目录下即可！（授权后使用！）
>
> ```shell
> [root@lch apache-iotdb-0.12.5-server-bin]# ll
> ......
> -rwxr-xr-x. 1 root root    47 3月  24 11:19 start.sh
> -rwxr-xr-x. 1 root root   137 3月  24 11:22 stop.sh
> ```



使用 cli 客户端连接

```shell
[root@localhost apache-iotdb]# sbin/start-cli.sh -h 127.0.0.1 -p 6667 -u root -pw root
---------------------
Starting IoTDB Cli
---------------------
 _____       _________  ______   ______    
|_   _|     |  _   _  ||_   _ `.|_   _ \   
  | |   .--.|_/ | | \_|  | | `. \ | |_) |  
  | | / .'`\ \  | |      | |  | | |  __'.  
 _| |_| \__. | _| |_    _| |_.' /_| |__) | 
|_____|'.__.' |_____|  |______.'|_______/  version 0.13.0
                                           

IoTDB> login successfully
IoTDB> 
```

## 基础操作

定义存储组

```sql
IoTDB> SET STORAGE GROUP TO root.ln
Msg: The statement is executed successfully.
```

查询存储组

```sql
IoTDB> SHOW STORAGE GROUP
+-------------+
|storage group|
+-------------+
|      root.ln|
+-------------+
```

创建时间序列

```sql
IoTDB> CREATE TIMESERIES root.ln.wf01.wt01.status WITH DATATYPE=BOOLEAN, ENCODING=PLAIN
Msg: The statement is executed successfully.

IoTDB> CREATE TIMESERIES root.ln.wf01.wt01.temperature WITH DATATYPE=FLOAT, ENCODING=RLE
Msg: The statement is executed successfully.
```

查看时间序列列表

```sql
IoTDB> SHOW TIMESERIES
+-----------------------------+-----+-------------+--------+--------+-----------+----+----------+
|                   timeseries|alias|storage group|dataType|encoding|compression|tags|attributes|
+-----------------------------+-----+-------------+--------+--------+-----------+----+----------+
|root.ln.wf01.wt01.temperature| null|      root.ln|   FLOAT|     RLE|     SNAPPY|null|      null|
|     root.ln.wf01.wt01.status| null|      root.ln| BOOLEAN|   PLAIN|     SNAPPY|null|      null|
+-----------------------------+-----+-------------+--------+--------+-----------+----+----------+
Total line number = 2

```

查看具体的时间序列

```sql
IoTDB> SHOW TIMESERIES root.ln.wf01.wt01.status
+------------------------+-----+-------------+--------+--------+-----------+----+----------+
|              timeseries|alias|storage group|dataType|encoding|compression|tags|attributes|
+------------------------+-----+-------------+--------+--------+-----------+----+----------+
|root.ln.wf01.wt01.status| null|      root.ln| BOOLEAN|   PLAIN|     SNAPPY|null|      null|
+------------------------+-----+-------------+--------+--------+-----------+----+----------+
Total line number = 1

```

序列插入数据，NSERT 语句向 root.ln.wf01.wt01.status 时间序列中插入数据，在插入数据时需要首先指定时间戳和路径后缀名称：

```sql
IoTDB> INSERT INTO root.ln.wf01.wt01(timestamp,status) values(100,true);
Msg: The statement is executed successfully.
```

查询序列的数据

```sql
IoTDB> SELECT status FROM root.ln.wf01.wt01
+-----------------------------+------------------------+
|                         Time|root.ln.wf01.wt01.status|
+-----------------------------+------------------------+
|1970-01-01T08:00:00.100+08:00|                    true|
+-----------------------------+------------------------+
```

查询序列的多个数据

```sql
IoTDB> SELECT * FROM root.ln.wf01.wt01
+-----------------------------+-----------------------------+------------------------+
|                         Time|root.ln.wf01.wt01.temperature|root.ln.wf01.wt01.status|
+-----------------------------+-----------------------------+------------------------+
|1970-01-01T08:00:00.100+08:00|                         null|                    true|
+-----------------------------+-----------------------------+------------------------+

```

退出 cli

```sql
IoTDB> quit
or
IoTDB> exit
```

停止 IoTDB服务

```sh
[root@localhost apache-iotdb]# sbin/stop-server.sh
```



## 基本概念

### 属性层级组织数据

官网给出了一张以`属性层级组织数据`的概念图：

![](https://blog.lichenghao.cn/upload/2022/07/96919567055A4F08AB0D7B6DF26906E0.jpg)

要求按照数据的属性进行分层，然后在需要的层级上建立存储组，数据都存放在存储组的下边。（这就和 Mysql 的索引的数据结构Btree很像了，把数据都存放在叶子节点上。）比如我们的存储组为Root层。那么下面每一层都按照层级去存储数据。当我们查询的时候如果指定的路径，比如 root.ln.wf01.wt01.status 这样查询的话就很快了。







