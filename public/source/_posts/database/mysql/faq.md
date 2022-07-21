---
title: Mysql 问题记录
categories: mysql
tags:
  - mysql
  - faq
keywords: 'msyql,数据库'
abbrlink: '116e3217'
date: 2022-02-02 09:39:00
---


## JDBC  tinyint 自动变成boolean的问题

默认情况下，使用jdbc查询的时候，如果你的字段是tinyint类型，那么查询可能会自动转换成boolean.

解决：在配置连接参数上，追加：`&tinyInt1isBit=false`

```properties
dburl = jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true\
&rewriteBatchedStatements=true&connectTimeout=12000&socketTimeout=12000&failOverReadOnly=false&tinyInt1isBit=false
```



## 使用 Datetime 时区的问题

如果客户端和服务端的时区不一致，就会导致你代码中想插入的时间和数据库中实际插入的时间相差几个小时，通常会是 8 小时，因为我们是东八区。进行如下的调整，在查询和插入时间都没有问题。

1. 首先调整服务器时区,设置为东八区。

```shell
[root@lch ~]# timedatectl set-timezone Asia/Shanghai
```

2. 调整 mysql 时区 my.cnf 配置文件指定为东八区。（需要重启服务）

```properties
[mysqld]
default-time-zone="+08:00"
```

查询验证下设置的时区

```sql
mysql> show VARIABLES like '%time_zone%'; 
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | +08:00 |
+------------------+--------+
```

3. JDBC url 指定时区

```properties
url: jdbc:mysql://127.0.0.1:3306/test?useruseUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
```

4. 代码中使用 LocalDateTime 做字段的映射即可

```java
@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
private LocalDateTime createTime;
```

