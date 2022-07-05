---
title: Spring Cloud Alibaba Nacos
categories: springcloud-alibaba
tags:
  - springcloud-alibaba
  - nacos
abbrlink: 5e095d1c
date: 2022-02-02 12:00:00
---
Nacos : Naming and Configuration Service 一个易于构建云原生应用的动态服务发现，配置管理和服务管理平台。

Nacos = Eureka + Config + Bus



## 服务注册中心

开源地址：https://github.com/alibaba/Nacos

官网：https://nacos.io/zh-cn/

### 安装 Nacos 服务端

下载地址：https://github.com/alibaba/nacos/releases   或者 (https://wwb.lanzoul.com/b01vbsfyb 密码:furm)

解压后放到 /opt 下，然后单机部署启动。启动文件在 bin目录下。

```sh
[root@localhost bin]# ./startup.sh  -m standalone
```

然后访问（ip 替换成自己的） http://172.16.127.3:8848/nacos/  用户名/密码 ：nacos /nacos

![](https://blog.lichenghao.cn/upload/2022/07/30102414.png)



父工程POM增加

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-alibaba-dependencies</artifactId>
  <version>2.1.0.RELEASE</version>
  <type>pom</type>
  <scope>import</scope>
</dependency>
```



### 服务提供者

增加一个提供者，cloudalibaba-provider-payment9001

pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>lichcloud</artifactId>
        <groupId>cn.lichenghao.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloudalibaba-provider-payment9001</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- 自定义通用 -->
        <dependency>
            <groupId>cn.lichenghao.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
</project>
```

配置文件 application.yml

```yaml
server:
  port: 9001
spring:
  application:
    name: cloudalibaba-provider-server
  cloud:
    nacos:
      discovery:
        server-addr: 172.16.127.3:8848
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

启动类和一个小的请求

```java
/**
 * 服务提供者
 */
@SpringBootApplication
@EnableDiscoveryClient
public class Payment9001 {
    public static void main(String[] args) {
        SpringApplication.run(Payment9001.class, args);
    }

    @Value("${server.port}")
    private String port;

    @RestController
    public class payController {
        @GetMapping(value = "/payment/nacos/{id}")
        public String echo(@PathVariable String id) {
            return "Nacos Discovery payment " + port + "\t id:" + id;
        }
    }
}
```

启动服务后，看注册中心

![](https://blog.lichenghao.cn/upload/2022/07/30110529.png)

同上可以复制多个生产者，同时自带负载均衡功能。

### 服务消费者

pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>lichcloud</artifactId>
        <groupId>cn.lichenghao.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloudalibaba-consumer-order83</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- 自定义通用 -->
        <dependency>
            <groupId>cn.lichenghao.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
</project>
```

配置文件 application.yml

```yaml
server:
  port: 83
spring:
  application:
    name: cloudalibaba-consumer-order
  cloud:
    nacos:
      discovery:
        server-addr: 172.16.127.3:8848
# 微服务名称
service-url:
  nacos-user-service: http://cloudalibaba-provider-server
```

RestTemplate

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

主启动带个小测试请求

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Order83 {
    public static void main(String[] args) {
        SpringApplication.run(Order83.class, args);
    }

    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serverUrl;

    @RestController
    public class orderController {

        @GetMapping(value = "/consumer/payment/nacos/{id}")
        public String echo(@PathVariable String id) {
            return restTemplate.getForObject(serverUrl + "/payment/nacos/" + id, String.class);
        }
    }
}
```

发送请求：http://localhost:83/consumer/payment/nacos/1  

```json
Nacos Discovery payment 9001 id:1
Nacos Discovery payment 9002 id:1
Nacos Discovery payment 9001 id:1
Nacos Discovery payment 9002 id:1
```

可以看出有负载均衡的效果，因为 nacos 整合了 ribbon 和 restTemplate 这一套负载均衡。

### 服务注册中心对比

| 框架      | CAP   | 控制台 | 活跃度 |
| --------- | ----- | ------ | ------ |
| Eureka    | AP    | 支持   | 低     |
| Zookeeper | CP    | 不支持 | 中     |
| Consul    | CP    | 支持   | 高     |
| Nacos     | AP/CP | 支持   | 高     |



## 服务配置中心

### 基础配置

增加一个微服务测试配置，cloudalibaba-config-nacos-client3377

pom，增加 spring-cloud-starter-alibaba-nacos-config

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>lichcloud</artifactId>
        <groupId>cn.lichenghao.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloudalibaba-config-nacos-client3377</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- 自定义通用 -->
        <dependency>
            <groupId>cn.lichenghao.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
</project>
```



配置文件，bootstrap.yml application.yml

bootstrap.yml

```yaml
server:
  port: 3377
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 172.16.127.3:8848
      config:
        server-addr: 172.16.127.3:8848
        file-extension: yaml
```

application.yml

```yaml
spring:
  profiles:
    active: dev
```

测试请求

```java
@RestController
@RefreshScope
public class ClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

主启动

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosClient3377 {
    public static void main(String[] args) {
        SpringApplication.run(NacosClient3377.class, args);
    }
}
```



nacas作为配置中心，需要在上面增加配置文件。

![](https://blog.lichenghao.cn/upload/2022/07/30143641.png)

其中的 Data Id 有一定的命名规则： `${prefix}-${spring.profiles.active}.${file-extension}`

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

所以我们上面的 Data Id 就是：`nacos-config-client-dev.yaml` ，然后填写其他发布即可。

![](https://blog.lichenghao.cn/upload/2022/07/30143938.png)

启动微服务，请求：http://localhost:3377/config/info ，然后修改配置文件的内容，再次请求，发现微服务会自动刷新配置文件的内容。因为我们使用了`@RefreshScope` nacos 也集成了自动刷新的功能。

### 分组配置

我们通过 Config Bus Stream实现了分组配置的功能。同样 Nacos也同样具有分类配置。多环境多项目管理。



Namespace + GroupId + DataId 实现分组管理，默认情况为 public + Default_Group + Default

- Namespace 可用于区分部署环境
- GroupId DataId 逻辑上区分不同的目标对象

它们的关系：

![](https://blog.lichenghao.cn/upload/2022/07/156121785731495ab332c.jpg ':size=30%')



#### DataId  方案

可以通过 spring.profiles.active 来指定开发，测试等环境。那么在 Nacos 中可以配置多个配置文件。

![](https://blog.lichenghao.cn/upload/2022/07/30145908.png)

在 application.yml 指定哪个环境就就在哪个配置文件。

```yaml
spring:
  profiles:
    active: dev # test
```



#### Group Id 方案

新建两个不同的分组，DEV_GROUP ,TEST_GROUP

![](https://blog.lichenghao.cn/upload/2022/07/30162849.png)

调整配置文件

application.yml  修改为 info

```yaml
spring:
  profiles:
    active: info
```

bootstrap.yml ,增加分组即可

```yaml
server:
  port: 3377
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 172.16.127.3:8848
      config:
        server-addr: 172.16.127.3:8848
        file-extension: yaml
        group: TEST_GROUP
```



#### Namespace 方案

增加 namespace

![](https://blog.lichenghao.cn/upload/2022/07/30163706.png)

以 dev 命名空间为例，增加多个不同分组的配置。

![](https://blog.lichenghao.cn/upload/2022/07/30164438.png)

同样修改配置文件即可

application.yml

```yaml
spring:
  profiles:
    active: dev
```

bootstarp.yml，增加 namespace 属性。

```yaml
server:
  port: 3377
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 172.16.127.3:8848
      config:
        server-addr: 172.16.127.3:8848
        file-extension: yaml
        group: TEST_GROUP
        namespace: a45eba43-f03e-440b-af8e-507f0434ea16
```

启动微服务，访问测试。



## 集群&持久化配置

集群架构示意图：

![](https://blog.lichenghao.cn/upload/2022/07/30170809.png ':size=50%')



Nacos 支持三种集群模式：

- 单机模式 - 用于测试和单机试用。
- 集群模式 - 用于生产环境，确保高可用。
- 多集群模式 - 用于多数据中心场景。



### 切换 Mysql 数据源

新增数据库 nacos_dev 执行安装包下 conf/ nacos-mysql.sql的脚本。

修改 nacos 的配置文件，增加数据库配置

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://11.162.196.16:3306/nacos_devtest?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=nacos_devtest
db.password=youdontknow
```

重启服务即可

```sh
[root@localhost logs]# ./startup.sh -m standalone
```

### 修改集群配置文件

伪集群

复制样例配置文件为 cluster.conf

```sh
[root@localhost conf]# cp cluster.conf.example cluster.conf
```

内容修改为：

```properties
172.16.127.3:333
172.16.127.3:444
172.16.127.3:555
```

修改启动脚本startup.sh，按照端口启动。

从

```sh
 57 while getopts ":m:f:s:" opt
 58 do
 59     case $opt in
 60         m)
 61             MODE=$OPTARG;;
 62         f)
 63             FUNCTION_MODE=$OPTARG;;
 64         s)
 65             SERVER=$OPTARG;;
 66         ?)
 67         echo "Unknown parameter"
 68         exit 1;;
 69     esac
 70 done
 
 .....
132 # start
133 echo "$JAVA ${JAVA_OPT}" > ${BASE_DIR}/logs/start.out 2>&1 &
134 nohup $JAVA ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
135 echo "nacos is starting，you can check the ${BASE_DIR}/logs/start.out"


```

修改为： 增加了 port 信息 57 行，66 、67、134 行

```sh
 57 while getopts ":m:f:s:p:" opt
 58 do
 59     case $opt in
 60         m)
 61             MODE=$OPTARG;;
 62         f)
 63             FUNCTION_MODE=$OPTARG;;
 64         s)
 65             SERVER=$OPTARG;;
 66         p)  
 67             PORT=$OPTARG;;
 68         ?)
 69         echo "Unknown parameter"
 70         exit 1;;
 71     esac

......
132 # start
133 echo "$JAVA ${JAVA_OPT}" > ${BASE_DIR}/logs/start.out 2>&1 &
134 nohup $JAVA -Dserver.port=${PORT} ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
135 echo "nacos is starting，you can check the ${BASE_DIR}/logs/start.out"
```

这样就可以根据端口启动

```sh
./startup.sh -p 333
```



### 配置 Nginx

安装 ：[centos7-安装 Nginx](https://note.lichenghao.cn#/tool/linux/centos-%E5%AE%89%E8%A3%85Nginx)

修改配置文件 nginx.conf 如下所示：

```properties
 upstream cluster{
    server 127.0.0.1:333;
    server 127.0.0.1:444;
    server 127.0.0.1:555;
  }

 server {
        listen       1111;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
          #  root   html;
          #  index  index.html index.htm;
          proxy_pass http://cluster;
        }

```

然后按照指定的配置文件启动

```sh
[root@localhost sbin]# ./nginx -c /opt/nginx/conf/nginx.conf
```

### 启动三个 Nacos

```sh
[root@localhost bin]# ./startup.sh -p 333
/opt/jdk8/bin/java  -server -Xms512m -Xmx512m -Xmn256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m -XX:-OmitStackTraceInFastThrow -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/opt/nacos/logs/java_heapdump.hprof -XX:-UseLargePages -Djava.ext.dirs=/opt/jdk8/jre/lib/ext:/opt/jdk8/lib/ext:/opt/nacos/plugins/cmdb:/opt/nacos/plugins/mysql -Xloggc:/opt/nacos/logs/nacos_gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M -Dnacos.home=/opt/nacos -Dloader.path=/opt/nacos/plugins/health -jar /opt/nacos/target/nacos-server.jar  --spring.config.location=classpath:/,classpath:/config/,file:./,file:./config/,file:/opt/nacos/conf/ --logging.config=/opt/nacos/conf/nacos-logback.xml --server.max-http-header-size=524288
nacos is starting with cluster
nacos is starting，you can check the /opt/nacos/logs/start.out
```

同理启动 444,555，然后访问：http://172.16.127.3:1111/nacos 出现登录页面即可。

### 微服务测试

然后按照 Namespace 方案在 Nacos 上配置下。

修改微服务配置文件，修改地址

```yaml
server:
  port: 3377
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 172.16.127.3:1111
      config:
        server-addr: 172.16.127.3:1111
        file-extension: yaml
        group: DEV_GROUP
        namespace: 686eebdc-c6b0-4aaa-aaff-38345f2cbbef
```

访问测试：http://localhost:3377/config/info













