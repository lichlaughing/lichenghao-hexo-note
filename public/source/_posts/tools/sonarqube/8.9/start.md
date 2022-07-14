---
title: Sonarqube8.9 部署&汉化
categories: 
  - tools
  - sonarqube
tags:
  - sonarqube
keywords: sonarqube
abbrlink: a18d24b3
date: 2022-05-26 13:50:00
cover: https://blog.lichenghao.cn/upload/2022/07/6A0298B6-sonarqube.png
---
SonarQube是一个用于代码质量管理的开源平台，用于管理源代码的质量。同时  SonarQube 还对大量的持续集成工具提供了接口支持，可以很方便地在持续集成中使用  SonarQube。SonarQube 支持插件扩展，拥有丰富的插件。

官网：https://www.sonarqube.org/

演示版本：8.9

## 安装 sonarqube-8.9.9

下载地址：https://www.sonarqube.org/downloads/  

注意📢：

环境需要 JDK11、 DB 和 ES 的支持，但是 ES 在sonarqube 5.2 版本上是支持使用外部的 ES，从6.7版本开始，已经不再支持使用外部的ES，只使用内部嵌入式的ES。而且 Mysql 数据库在 8.x 版本也不支持了。所以根据需要选择版本。

首先安装数据库，这里使用 Postgresql ——>[安装个 PostgreSQL](/tool/linux/centos-%E8%BD%AF%E4%BB%B6&%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85?id=postgresql)



解压 sonarqube 安装包，开始修改配置文件。

我的 java 环境变量是 jdk8，但是它需要 jdk11，手动指定 JDK 路径。修改配置文件  wrapper.conf 

```properties
# Path to JVM executable. By default it must be available in PATH.
# Can be an absolute path, for example:
#wrapper.java.command=/path/to/my/jdk/bin/java
wrapper.java.command=/opt/module/jdk-11.0.12/bin/java
#wrapper.java.command=java

```

修改数据库连接信息，用 PostgreSQL ，需要提前新建个数据库 sonarqube。配置文件  sonar.properties

```properties
#----- PostgreSQL 9.3 or greateri
sonar.jdbc.username=postgres
sonar.jdbc.password=123456
# By default the schema named "public" is used. It can be overridden with the parameter "currentSchema".
# sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube?currentSchema=my_schema
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube

```

修改 web 地址和端口。配置文件  sonar.properties

```properties
sonar.web.host=0.0.0.0
sonar.web.port=9000
```

修改内嵌 es 的相关配置，可以把占用的内存调整小点，或者其他 es 启动需要的配置。

启动

因为 es 服务需要不能用 root 启动，所以就用 es 用户来启动 sonarqube 服务。

新建用户，把 sonarqube 文件路径授权，然后启动。

```shell
[root@lch module]# useradd -g es es   
[root@lch module]# chown -R es:es sonarqube-8.9.9.56886/
```

切换用户到 es ，进去启动目录 sonarqube-8.9.9.56886/bin/linux-x86-64

```shell
[es@lch linux-x86-64]$ ./sonar.sh start

INFO  app[][o.s.a.SchedulerImpl] Process[es] is up
INFO  app[][o.s.a.SchedulerImpl] Process[ce] is up
INFO  app[][o.s.a.SchedulerImpl] Process[web] is up
INFO  app[][o.s.a.SchedulerImpl] SonarQube is up

```

依次打印如上，则表示启动成功，访问 web 页面 ：http://192.168.1.240:9000/  默认用户名：admin/admin

## 汉化

登录后 ，在插件市场搜索 `chinese pack` 可以在线安装，但是很慢，直接下载然后放到服务器上，然后重启服务即可。

汉化包地址：https://github.com/xuhuisheng/sonar-l10n-zh  注意版本的兼容性！

下载汉化包插件 sonar-l10n-zh-plugin-8.9.jar

插件存放位置：/sonarqube-8.9.9.56886/extensions/plugins

然后重启服务即可，最终效果：

![](https://blog.lichenghao.cn/upload/2022/07/28135456.png)

