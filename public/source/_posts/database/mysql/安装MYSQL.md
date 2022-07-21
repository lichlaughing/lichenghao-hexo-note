---
title: 安装 Mysql
categories: mysql
tags:
  - mysql
keywords: 'msyql,数据库'
abbrlink: f895c2ce
date: 2022-02-02 09:30:00
---
## Centos

安装个环境：[centos7使用RPM包安装Mysql](https://note.lichenghao.cn/#/TOOL/LINUX/centos7-%E5%AE%89%E8%A3%85Mysql)

## Window 绿色版

**以下涉及到的路径都需要修改自己机器上的路径！**

1. 下载地址：https://downloads.mysql.com/archives/community/

2. 解压到指定目录下后，在根目录下新增`my.ini` 配置文件如下所示：

```properties
[mysqld]
# 设置默认端口号为 3306
port=3306
# 设置 MySQL 的安装目录
basedir=D:\\tools\\mysql-5.7.22\\bin
# 设置错误信息保存目录文件夹
lc-messages-dir=D:\\tools\\mysql-5.7.22\\share\\english
# 设置 MySQL 数据库的数据保存目录
datadir=D:\\tools\\mysql-5.7.22\\data
# 允许最大连接数量
max_connections=200
# 允许连接失败的次数
max_connect_errors=10
# 字符集默认为 UTF8
character-set-server=utf8
# 使用默认存储引擎
default-storage-engine=INNODB
# 默认使用mysql_native_password插件认证
default_authentication_plugin=mysql_native_password
[mysql]
# 设置 MySQL 客户端默认字符集
default-character-set=utf8
[client]
# 设置 MySQL 客户端连接服务端时默认使用端口
port=3306
default-character-set=utf8
[WinMySQLAdmin]
#设置将 MySQL 的服务添加到注册表
Server=D:\\tools\\mysql-5.7.22\\bin\\mysqld.exe
```

3. 新增一个系统环境变量，在path环境变量中增加 ：`D:\tools\mysql-5.7.22\bin`

4. 然后用`管理员`身份打开cmd窗口，执行如下，会得到一个临时的密码。

   <div style='color: red'>（注：如果不使用管理员身份，后面会有问题）</div>

```bash
C:\Users\chenghao.li> mysqld --initialize --console

......
2021-02-03T03:09:27.658273Z 1 [Note] A temporary password is generated for root@localhost: ).bVStm,C3&N
```

5. 继续执行命令安装服务

```bash
C:\Users\chenghao.li>mysqld --install
Service successfully installed.
```

6. 启动服务

```bash
C:\Users\chenghao.li> net start mysql
```

7. 登录账号修改密码，第一次登录会强制修改密码`ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.`

```bash
C:\Users\chenghao.li> mysql -uroot -p

mysql> ALTER USER root@localhost IDENTIFIED BY "123456";
```

8. 开启远程登录

```bash
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;
mysql>flush privileges;
```

## Mac

我很少用 Mac 装这些数据库，消息队列这类软件，用虚拟机跑！
