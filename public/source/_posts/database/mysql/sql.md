---
title: Mysql 各种查询语句
categories: mysql
tags:
  - mysql
  - DDL
  - DML
  - DQL
  - DCL
keywords: 'msyql,数据库,DDL,DML,DQL,DCL'
abbrlink: cc5a40bb
date: 2022-02-02 09:32:00
---


SQL  分类：

| 分类 | 全称                       | 说明                                                 |
| ---- | -------------------------- | ---------------------------------------------------- |
| DDL  | Data Definition Language   | 数据定义语言，用来定义数据库对象（数据库，表，字段） |
| DML  | Data Manipulation Language | 数据操作语言，用来对数据库表中的数据进行增删改       |
| DQL  | Data Query Language        | 数据查询语言，用来查询数据库中表的记录               |
| DCL  | Data Control Language      | 数据控制语言，用来创建数据库用户，控制数据库访问权限 |



## DDL

### 🔹查询所有的数据库

```sql
show databases;
```

```sql
Database          |
------------------+
information_schema|
mysql             |
performance_schema|
sys               |
```

### 🔹使用数据库

```sql
user databaseName;
```

### 🔹删除数据库

```sql
drop database[if exists] databaseName;
```

### 🔹创建数据库

```sql
create database[if not exists] 数据库名称 [default charset utf8 字符集] [collate utf8_general_ci 排序规则] 
```

```sql
create database mydb default charset utf8 collate utf8_general_ci;
```

### 🔹查看所有表

```sql
show tables;
```

### 🔹查看表结构

```sql
desc tableName;
```

### 🔹查看表的建表语句

```sql
show create table tableName;
```

```sql
show create table mytb ;

Table|Create Table                                                                                 |
-----+---------------------------------------------------------------------------------------------+
mytb |CREATE TABLE `mytb` (¶  `name` varchar(100) DEFAULT NULL¶) ENGINE=InnoDB DEFAULT CHARSET=utf8|
```

### 🔹创建表

```sql
create table tableName(
	字段 字段类型 [comment 注释内容],
	字段 字段类型 [comment 注释内容],
	....
) [comment 表注释]
```



```sql
DROP TABLE IF EXISTS `base_region`;
CREATE TABLE `base_region` (
  `id` bigint(11) NOT NULL auto_increment comment '主键',
  `regionCode` varchar(12)  NOT null comment '编码',
  `province` varchar(50) DEFAULT null comment '省份',
  `city` varchar(50)  DEFAULT null comment '城市',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `index_regionCode` (`regionCode`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 comment '区域表';
```

### 🎉数据类型

整型 （无符号在数据类型后加 unsigned 关键字）。

| 类型名称      | 取值范围                                                    | 无符号                 | 大小    |
| :------------ | :---------------------------------------------------------- | ---------------------- | :------ |
| TINYINT       | -2^7（-128）〜2^7-1（127）                                  | 0~2^8-1(255)           | 1个字节 |
| SMALLINT      | -2^15（-32768）〜2^15-1（32767）                            | 0~2^16-1（65535）      | 2个宇节 |
| MEDIUMINT     | -2^23（-8388608）〜2^23-1（8388607）                        | 0~2^24-1（16777215）   | 3个字节 |
| INT (INTEGHR) | -2^31（-2147483648）〜2^31-1(2147483647)                    | 0~2^32-1（4294967295） | 4个字节 |
| BIGINT        | -2^63 (-9223372036854775808) ~ 2^63-1 (9223372036854775807) | 0~2^64-1               | 8个字节 |

浮点型	

| 类型名称            | 取值范围                                           | 无符号                                                  | 大小       |
| :------------------ | -------------------------------------------------- | ------------------------------------------------------- | :--------- |
| FLOAT               | -3.402823466E+38～-1.175494351E-38                 | 0 和 -1.175494351E-38～-3.402823466E+38                 | 4 个字节   |
| DOUBLE              | -1.7976931348623157E+308～-2.2250738585072014E-308 | 0 和 -2.2250738585072014E-308～-1.7976931348623157E+308 | 8 个字节   |
| DECIMAL (M, D)，DEC | 取决 M D DECIMAL 如果不指定精度，默认为（10，0）   | 取决 M D                                                | M+2 个字节 |

日期和时间

| 类型名称  | 日期格式                | 日期范围                                                 | 存储需求 |
| :-------- | :---------------------- | :------------------------------------------------------- | :------- |
| YEAR      | YYYY                    | 1901 ~ 2155                                              | 1 个字节 |
| TIME      | HH : MM : SS            | -838 : 59 : 59 ~ 838 : 59 : 59                           | 3 个字节 |
| DATE      | YYYY-MM-DD              | 1000-01-01 ~ 9999-12-3                                   | 3 个字节 |
| DATETIME  | YYYY-MM-DD HH : MM : SS | 1000-01-01 00 : 00 : 00 ~ 9999-12-31 23 : 59 : 59        | 8 个字节 |
| TIMESTAMP | YYYY-MM-DD HH : MM : SS | 1980-01-01 00 : 00: 01 UTC ~ 2040-01-19 03 : 14 : 07 UTC | 4 个字节 |

字符串

| 类型名称   | 说明                                         | 存储需求                                                   |
| :--------- | :------------------------------------------- | :--------------------------------------------------------- |
| CHAR(M)    | 固定长度非二进制字符串                       | M 字节，1<=M<=255                                          |
| VARCHAR(M) | 变长非二进制字符串                           | L+1字节，在此，L< = M和 1<=M<=255                          |
| TINYTEXT   | 非常小的非二进制字符串                       | L+1字节，在此，L<2^8 ,0-255 bytes                          |
| TEXT       | 小的非二进制字符串                           | L+2字节，在此，L<2^16 ,0-65535 bytes                       |
| MEDIUMTEXT | 中等大小的非二进制字符串                     | L+3字节，在此，L<2^24 ,0-1677215 bytes                     |
| LONGTEXT   | 大的非二进制字符串                           | L+4字节，在此，L<2^32,0-4294967295 bytes                   |
| ENUM       | 枚举类型，只能有一个枚举字符串值             | 1或2个字节，取决于枚举值的数目 (最大值为65535)             |
| SET        | 一个设置，字符串对象可以有零个或 多个SET成员 | 1、2、3、4或8个字节，取决于集合 成员的数量（最多64个成员） |



### 🔹增加表字段

```sql
alter table 表名 add 字段名 类型（长度）[comment 注释] [约束]
```



```sql
alter table mytb add address varchar(255) comment '地址' unique ;
```



### 🔹修改数据类型

​		

```sql
alter table 表名 modify 字段名 类型（长度）;
```



### 🔹修改字段名和字段类型

```sql
alter table 表名 change 旧字段名 新字段名 类型（长度）[comment 注释] [约束]
```



### 🔹删除字段

```sql
alter table 表名 drop 字段名 
```



### 🔹修改表名称

```sql
alter table 表名 rename to 新表名
```



### 🔹删除表

```sql
drop table[if exists] 表名
```



### 🔹删除表,并重新创建

```sql
truncate table 表名
```





## DML

对数据库中的表进行增删改，主要有三个关键字 insert update delete



### 插入数据

插入一条数据

```sql
insert into 表名(字段1,字段2,...) value（值 1,值 2,...）
```

给全部字段插入数据

```sql
insert into 表名 values(值 1，值 2，...)
```

插入多条数据

```sql
insert into 表名 values(值 1，值 2...)，（值 1，值 2...）
```

例如：

```sql
insert into mytb values ('1','北京'),('2','天津');
```



### 修改数据



```sql
update 表名 set 字段 1=值 1,字段 2=值 2,... [where 条件];
```



例如：

```sql
update mytb set address  = '上海' where name ='1';
```



### 删除数据

```sql
delete from 表名 [where 条件]
```



## DQL

查询语句，也是使用最多的语句，语法格式如下：

```sql
select 
	字段列表
from 
	表名列表
where
	查询条件
group by 
	分组字段列表
having 
	分组后条件列表
order by 
	排序字段列表
limit 
	分页参数
```



### 基本查询

查找多个字段

```sql
select 字段 1,字段 2,... from 表名
```

或者

```sql
select * from 表名
```

给查询的字段设备别名

```sql
select 字段 1[as 别名 1],字段 2[as 别名 2],... from 表名
```

去重

```sql
select distinct 字段列表 from 表名
```

### 条件查询

语法

```sql
select 字段列表 from 表名 where 条件列表
```

有哪些条件？

| 比较运算符号           | 说明                                     |
| ---------------------- | ---------------------------------------- |
| >                      | 大于                                     |
| >=                     | 大于等于                                 |
| <                      | 小于                                     |
| <=                     | 小于等于                                 |
| =                      | 等于                                     |
| <> 或!=                | 不等于                                   |
| between .... and ..... | 在某个范围之间，包含边界                 |
| in(....)               | 在 in 之后的列表中，多选一               |
| like                   | 模糊匹配（_匹配单个字符，%匹配多个字符） |
| is null                | 判断是不是 null                          |



| 逻辑运算符   | 说明     |
| ------------ | -------- |
| and 或者 &&  | 并且     |
| or 或者 \|\| | 或者     |
| not 或者 ！  | 非，不是 |



```sql
select province,district from base_region where province = '北京市'
```



### 聚合函数

常见聚合函数，作用于表中的某一列。

| 函数  | 说明     |
| ----- | -------- |
| count | 统计数量 |
| max   | 最大值   |
| min   | 最小值   |
| avg   | 平均值   |
| sum   | 求和     |

语法

```sql
select 函数（字段列表） from 表名
```

例如

```sql
select max(age) from tb_emp ;
```



### 分组查询

语法，通常分组和聚合函数一起使用

```sql
select 字段列表 from 表名 [where 查询条件] group by 分组字段 [having 分组后过滤条件]
```



注：where  having

- where 是分组之前对查询进行的过滤，having 是分组之后对分组结果进行过滤
- where 不能对聚合函数进行判断，having 可以



例如：

```sql
-- 10 岁以上男女员工的数量
select gender ,count(*) from tb_emp where age>10 group by gender;

-- 10 岁以上男女员工的平均年龄
select gender ,avg(age) from tb_emp where age>10 group by gender;
```

结果：

```sql
gender|count(*)|
------+--------+
女     |       2|
男     |       4|

gender|avg(age)|
------+--------+
女     | 35.0000|
男     | 17.0000|
```

### 排序查询

语法  , 其中排序方式 desc 倒叙，asc 自然顺序

```sql
select 字段列表 from 表名 order by 字段 1 排序方式 1,字段 2 排序方式 2
```



例如：

```sql
select * from tb_emp order by age desc, gender desc;

id|emp_name|age|gender|dept_id|
--+--------+---+------+-------+
 4|李黑      | 40|女     |      2|
 2|张二      | 30|女     |      1|
 1|张一      | 20|男     |      1|
 5|王五      | 20|男     |      3|
 3|李白      | 15|男     |      2|
 6|王六      | 13|男     |       |
```



### 分页查询

关键字 limit 

```sql
select 字段列表 from 表名 limit 起始索引,查询记录数
```



注：

- 起始索引从 0 开始，起始索引 = （查询页码数-1）*每页显示记录数
- 分页查询不同的数据库有不同的实现方式
- 查询第一页的数据，起始索引可以省略，简写为 limit 查询记录数



例如：

```sql
--  第一页
select * from tb_emp  limit 0,5;
--  第二页
select * from tb_emp limit 5,5;
```



### SQL执行顺序

MySql SQL执行示意图如下所示：

![](https://blog.lichenghao.cn/upload/2022/07/214448.png)

执行过程如下：

1. 先通过连接器建立连接，需要提供用户名和密码
2. 连接成功后，先去缓存中查询，有则直接返回，否则去分析器
3. 分析区需要做语法分析，识别里面的关键字如 Select、delete 等，判断 SQL 语句语法是否正确
4. 分析结束后，优化器会对查询进行优化比如使用哪些索引之类
5. 优化完毕后，执行器拿着优化好的语句去执行了，还需要判断权限相关信息，如果有权限的话就调用存储引擎的提供的接口查询数据返回即可





手写SQL语句和MySql解析语句的顺序

通常我们的 SQL 语句会是如下的这样：

```sql
select * from t1,t2 where t1.id = t2.id group by xxx having xxx order by xxx limit 0,10
```

MySql 在解析语句执行的时候，是按照下面的顺序去解析：

```sql
from t1,t2
where xxx
group by xxx
having xxx
select
order by xxx
limit xxxx
```



## DCL

### 用户管理

查询用户

```sql
use mysql;
select * from user;
```

创建用户

```sql
create user '用户名'@'主机名' identified by '密码';
```

修改用户密码

```sql
alter user '用户名'@'主机名' identified with mysql_native_password by '新密码'
```

删除用户

```sql
drop user '用户名'@'主机名';
```



例如：

```sql
--  创建用户
create user 'lich'@'localhost' identified by 'Root@123'; 
--  修改密码
alter user 'lich'@'localhost' identified with mysql_native_password by 'Root@1234'; 
--  删除用户
drop user 'lich'@'localhost';
```



### 权限控制

创建了用户后，必须给权限才能正常的使用。mysql 常用中的权限如下所示：

| 权限                 | 说明                 |
| -------------------- | -------------------- |
| ALL , ALL PRIVILEGES | 所有权限             |
| SELECT               | 查询                 |
| INSERT               | 插入                 |
| UPDATE               | 更新                 |
| DELETE               | 删除                 |
| ALTER                | 修改表               |
| DROP                 | 删除数据库，表，视图 |
| CREATE               | 创建库，表           |



### 查询权限

```sql
show grants for '用户名'@'主机名'
```

如：

```sql
show grants for 'root'@'%';

Grants for root@%                                          |
-----------------------------------------------------------+
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION|
```



### 授权语句

```sql
-- 给某个库表的权限
grants 权限列表 on 数据库.表名 to '用户名'@'主机名';
-- 给所有库表的权限
grants 权限列表 on *.* to '用户名'@'主机名';
```

如:

```sql
grant all on *.* to 'lich'@'localhost';
```

### 撤销权限

```sql
revoke 权限列表 on 数据库.表名 from  '用户名'@'主机名';
```

如：

```sql
revoke all on *.* from 'lich'@'localhost';
```







