---
title: springcloud 服务监控 skywalking
categories: springcloud
tags:
  - springcloud
abbrlink: 4643409e
date: 2022-02-02 12:00:00
---
开源地址：https://github.com/apache/skywalking

官网：https://skywalking.apache.org/

演示地址：http://demo.skywalking.apache.org/    用户名/密码：skywalking/skywalking



## 入门

### 功能列表

- 服务、服务实例、端点指标分析。
- 服务拓扑图分析
- 服务、服务实例和端点（Endpoint）SLA 分析
- 慢查询检测
- 告警，日志



### 架构设计

SkyWalking 逻辑上分为四部分: 探针、平台后端、 存储和用户界面。

![](https://blog.lichenghao.cn/upload/2022/07/E9318C5D9859BD3.png)

### APM  安装

下载地址：https://skywalking.apache.org/downloads/

解压后，默认什么都不用修改，直接启动就可以 。（默认使用 H2 内存数据库，监听 8080 端口）。

可以修改端口：(apache-skywalking-apm-bin/webapp/webapp.yml)

```yaml
server:
  port: 38080
```

启动服务

```bash
[root@lch bin]# ./startup.sh 
SkyWalking OAP started successfully!
SkyWalking Web Application started successfully!
```

>会启动如下两个服务：
>（1）SkyWalking OAP：追踪信息收集器，通过 gRPC/Http 收集客户端的采集信息 ，Http默认端口 12800，gRPC默认端口 11800。
>（2）Skywalking-Web：管理平台页面 默认端口 8080

访问 ：http://192.168.1.240:38080

9.x 版本

![](https://blog.lichenghao.cn/upload/2022/07/13111904.png)



> 有启动脚本，但是么有关闭脚本，自己写个关闭脚本 `stop.sh` 内容如下所示：
>
> 

```bash
#!/bin/bash

ID=`ps -ef | grep -E "OAPServer|skywalking" | grep -v grep |awk '{print $2}'`
for id in $ID
do
        kill -9 $id
        echo "kill $id"
done

```



### Springboot 集成

本地开发，apm 服务在远程。

下载探针 jar 包 https://archive.apache.org/dist/skywalking/

然后修改其中的配置文件 `gent.config` 

```properties
# The service name in UI
agent.service_name=${SW_AGENT_NAME:SECOND-PRODUCER-SERVICE}

# Backend service addresses.
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:192.168.1.240:11800}
```

启动命令（以agent的方式加载对程序无任何侵入性）

如果使用 IDEA测试的话，可以添加 VM 参数，指定 agent 包中的 jar 包位置即可。

```java
-javaagent:/Users/lichenghao/tools/skywalking-agent/skywalking-agent.jar
```

如果 jar 包启动的话

```bash
java -javaagent:/Users/lichenghao/tools/skywalking-agent/skywalking-agent.jar
```

然后查看服务

![](https://blog.lichenghao.cn/upload/2022/07/13171423.png)

## 切换数据源

### MYSQL

默认使用的 H2 内存模式数据库，修改为 MYSQL，修改配置文件如下：

```yaml
storage:
  selector: ${SW_STORAGE:mysql}
  ....
  
  mysql:
    properties:
   		# 自己手动创建数据库
      jdbcUrl: ${SW_JDBC_URL:"jdbc:mysql://localhost:3306/skywalking?useruseUnicode=true&characterEncoding=utf8&serverTimezone=UTC&rewriteBatchedStatements=true&useSSL=false"}
      dataSource.user: ${SW_DATA_SOURCE_USER:root}
      # 修改为自己的密码
      dataSource.password: ${SW_DATA_SOURCE_PASSWORD:123456}
      dataSource.cachePrepStmts: ${SW_DATA_SOURCE_CACHE_PREP_STMTS:true}
      dataSource.prepStmtCacheSize: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_SIZE:250}
      dataSource.prepStmtCacheSqlLimit: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_LIMIT:2048}
      dataSource.useServerPrepStmts: ${SW_DATA_SOURCE_USE_SERVER_PREP_STMTS:true}
    metadataQueryMaxSize: ${SW_STORAGE_MYSQL_QUERY_MAX_SIZE:5000}
    maxSizeOfBatchSql: ${SW_STORAGE_MAX_SIZE_OF_BATCH_SQL:2000}
    asyncBatchPersistentPoolSize: ${SW_STORAGE_ASYNC_BATCH_PERSISTENT_POOL_SIZE:4}
```

然后将数据库的连接驱动放到 `oap-libs` 文件夹中。 下载地址：[mysql-connector-j](https://mvnrepository.com/artifact/mysql/mysql-connector-java/5.1.27)

重启服务即可。

启动完毕后，可以查看数据库表的情况：

![](https://blog.lichenghao.cn/upload/2022/07/13172117.png)





## 日志跟踪

[Spring Boot第三弹，一文带你搞懂日志如何配置？](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU3MDAzNDg1MA%3D%3D%26mid%3D2247484716%26idx%3D1%26sn%3D04db24b54f73e7eaf72e393cc1f215a3%26chksm%3Dfcf4dae1cb8353f738b3b61d9a935e256f2f885d13dff5d6e8cff03b85a5153d4aee8fc8328e%26token%3D1889058377%26lang%3Dzh_CN%23rd)

日志框架使用 logback，也是 springboot 默认的框架。

增加如下依赖 ：

```xml
 <dependency>
   <groupId>org.apache.skywalking</groupId>
   <artifactId>apm-toolkit-logback-1.x</artifactId>
   <version>8.9.0</version>
</dependency>
```

调整 logback 配置文件 `logback-spring.xml`，来显示日志的跟踪 ID。

<details>
<summary>logback-spring.xml（点击展开）</summary>


```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <contextName>logback</contextName>
    <property name="log.path" value="logs"/>

    <!--输出到控制台-->
    <!--<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        &lt;!&ndash;此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息&ndash;&gt;
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
            <charset>utf-8</charset>
        </encoder>
    </appender>-->

    <!-- 添加 skywalking 的追踪id，和其他的特性-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%tid] %highlight(%5.5level) %magenta(${PID}) --- [%15.15thread]
                    %cyan(%logger{20}) %boldBlue(%5.5line) : %msg%n
                </pattern>
            </layout>
        </encoder>
    </appender>

    <!--输出到debug-->
    <appender name="debug" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>app_log/log/logback-debug-%d{yyyy-MM-dd}.log</fileNamePattern>
        </rollingPolicy>
        <append>true</append>
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %contextName [%tid] [%thread] %-5level %logger{36} - %msg%n
                </pattern>
            </layout>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印DEBUG日志 -->
            <level>DEBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--输出到info-->
    <appender name="info" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>app_log/log/logback-info-%d{yyyy-MM-dd}.log</fileNamePattern>
        </rollingPolicy>
        <append>true</append>
        <!-- <encoder>
             <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %contextName [%tid] [%thread] %-5level %logger{36} - %msg%n</pattern>
             <charset>utf-8</charset>
         </encoder>-->
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %contextName [%tid] [%thread] %-5level %logger{36} - %msg%n
                </pattern>
            </layout>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印INFO日志 -->
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--输出到error-->
    <appender name="error" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>app_log/log/logback-error-%d{yyyy-MM-dd}.log</fileNamePattern>
        </rollingPolicy>
        <append>true</append>
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %contextName [%tid] [%thread] %-5level %logger{36} - %msg%n
                </pattern>
            </layout>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印ERROR日志 -->
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--输出到warn-->
    <appender name="warn" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>app_log/log/logback-warn-%d{yyyy-MM-dd}.log</fileNamePattern>
        </rollingPolicy>
        <append>true</append>
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %contextName [%tid] [%thread] %-5level %logger{36} - %msg%n
                </pattern>
            </layout>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印WARN日志 -->
            <level>WARN</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- skywalking grpc 日志收集 8.4.0版本开始支持 -->
    <appender name="grpc-log" class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.log.GRPCLogClientAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.mdc.TraceIdMDCPatternLogbackLayout">
                <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%tid] [%thread] %-5level %logger{36} -%msg%n</Pattern>
            </layout>
        </encoder>
    </appender>

    <!--分别设置对应的日志输出节点 -->
    <root level="info">
        <appender-ref ref="console"/>
        <appender-ref ref="debug"/>
        <appender-ref ref="info"/>
        <appender-ref ref="error"/>
        <appender-ref ref="warn"/>
        <appender-ref ref="grpc-log"/>
    </root>
</configuration>
```

</details>

如果你的 agent 和 skywalking apm 不在一个服务器上，那么需要在 agent 的配置文件 `agent.conf` 追加如下配置：

```properties
# log report
# 服务地址
plugin.toolkit.log.grpc.reporter.server_host=192.168.1.240
# 端口，默认的
plugin.toolkit.log.grpc.reporter.server_port=11800
```

然后启动服务，默认打印的日志中的 tid 为 `[TID:N/A]`，当有请求的时候会打上唯一的 ID。

然后在控制台就可以看到上报的日志，如下图所示：

![](https://blog.lichenghao.cn/upload/2022/07/14105124.png)



然后我们，制作一个报错，看下能否监控到：

```java
/**
  * 演示请求
*/
@RestController
public class SecondProducerController {
  @GetMapping(value = "/second-producer/{id}")
  public String producer(@PathVariable String id) {
    int a = 1/0;
    LOGGER.info("处理请求：{}", id);
    return "second-producer-service : " + port + "\t id:" + id;
  }
}
```

然后在链路跟踪和日志中都可以查询到：

链路跟踪：

![](https://blog.lichenghao.cn/upload/2022/07/14110756.png)



日志监控：

![](https://blog.lichenghao.cn/upload/2022/07/14110659.png)



## 开启自监控

开启遥测功能 telemetry，修改配置文件 application.yaml。Prometheus 可做为遥测功能（telemetry）的实现者。

```yaml
telemetry:
  selector: ${SW_TELEMETRY:prometheus}
  none:
  prometheus:
    host: ${SW_TELEMETRY_PROMETHEUS_HOST:0.0.0.0}
    port: ${SW_TELEMETRY_PROMETHEUS_PORT:1234}
    sslEnabled: ${SW_TELEMETRY_PROMETHEUS_SSL_ENABLED:false}
    sslKeyPath: ${SW_TELEMETRY_PROMETHEUS_SSL_KEY_PATH:""}
    sslCertChainPath: ${SW_TELEMETRY_PROMETHEUS_SSL_CERT_CHAIN_PATH:""}

```

开启 Prometheus Fetcher

SkyWalking 支持将 Prometheus 遥测数据直接收集到 OAP 后台。用户可以通过 UI 或 GraphQL API 查看它们。

默认情况下，Prometheus Fetcher 没有启动的，修改 selector 为 default 即可开启：

```yaml
prometheus-fetcher:
  selector: ${SW_PROMETHEUS_FETCHER:default}
  default:
    enabledRules: ${SW_PROMETHEUS_FETCHER_ENABLED_RULES:"self"}
    maxConvertWorker: ${SW_PROMETHEUS_FETCHER_NUM_CONVERT_WORKER:-1}
```

具体的规则在配置文件 self.yaml，文件目录 `apache-skywalking-apm-9.1.0/config/fetcher-prom-rules/self.yaml`

都开启后，在后台自监控中就可以看到监控信息：

![](https://blog.lichenghao.cn/upload/2022/07/17135357.png)
