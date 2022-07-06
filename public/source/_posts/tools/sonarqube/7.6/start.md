---
title: Sonarqube7.6 安装&汉化
categories: sonarqube
tags:
  - sonarqube
keywords: sonarqube
abbrlink: 4726e5d1
date: 2022-05-26 13:50:00
---

SonarQube是一个用于代码质量管理的开源平台，用于管理源代码的质量。同时  SonarQube 还对大量的持续集成工具提供了接口支持，可以很方便地在持续集成中使用  SonarQube。SonarQube 支持插件扩展，拥有丰富的插件。

官网：https://www.sonarqube.org/

演示版本：7.6

## 安装 sonarqube7.6 

JDK 1.8 

修改配置文件，数据库可以使用 mysql 。配置文件 sonar.properties。

```properties
#----- MySQL >=5.6 && <8.0
#sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
sonar.jdbc.username=root
sonar.jdbc.password=Root@123456
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar7?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
```

修改 web 地址和端口。配置文件 sonar.properties。

```properties
sonar.web.host=0.0.0.0
sonar.web.port=9000
```

修改内嵌 es 的相关配置，可以把占用的内存调整小点，或者其他 es 启动需要的配置。配置文件夹 /sonarqube-7.6/elasticsearch/config

然后新建个 sonarqube 用户，使用该用户启动服务。

```shell
[root@lch config]# useradd sonarqube
```

文件夹授权

```shell
[root@lch module]# chown -R sonarqube:sonarqube  sonarqube-7.6/
```

切换用户，启动服务

```shell
[sonarqube@lch linux-x86-64]$ ./sonar.sh start
Starting SonarQube...
Started SonarQube.
```

然后稍等片刻，查看日志 sonar.log 依次打印如下，表示成功

```shell
INFO  app[][o.s.a.SchedulerImpl] Process[es] is up
INFO  app[][o.s.a.SchedulerImpl] Process[web] is up
INFO  app[][o.s.a.SchedulerImpl] Process[ce] is up
INFO  app[][o.s.a.SchedulerImpl] SonarQube is up
```

访问 web  http://192.168.1.240:9000/  默认账号 admin/admin。

![](https://blog.lichenghao.cn/upload/2022/07/30105520.png)



## 汉化

下载汉化插件：https://github.com/xuhuisheng/sonar-l10n-zh 版本选择：sonar-l10n-zh-plugin-1.26.jar

插件放在 sonarqube-7.6/extensions/plugins 下，然后重启服务。

在重新登录查看

![](https://blog.lichenghao.cn/upload/2022/07/30110447.png)



