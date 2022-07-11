---
title: 笔记导航
category: 导航
tags:
  - chenghao.li
  - blog
cover: /assets/images/upup2.png
sticky: 999999
abbrlink: eb511f00
---



{% tabs top-tab %}



<!-- tab Java -->

​	{% tabs java-tab %}

​	<!-- tab 基础知识 -->

​	{% post_link java/basic/数据类型 %}

​	{% post_link java/basic/包装类 %}

​	{% post_link java/basic/关键字 %}

​	{% post_link java/basic/内部类 %}

​	{% post_link java/basic/类中代码块和属性的加载顺序问题 %}

​	{% post_link java/basic/反射 %}

​	{% post_link java/basic/泛型 %}

​	{% post_link java/basic/正则表达式 %}

​	{% post_link java/basic/位运算-原码-反码-补码 %}

​	{% post_link java/basic/项目中的很多if-else的优化方式 %}

​	{% post_link java/basic/在Ubuntu编译OpenJDK8的源代码 %}

​	{% post_link java/basic/消息协议&JMS %}

​	<!-- endtab -->



​	<!-- tab Guava -->
​	{% post_link java/guava/start %}

​	{% post_link java/guava/string %}

​	{% post_link java/guava/collections %}

​	{% post_link java/guava/caches %}

​	{% post_link java/guava/io %}

​	<!-- endtab -->



​	<!-- tab JVM -->

​	<!-- endtab -->

​	



​	{% endtabs %}

<!-- endtab -->



<!-- tab Database -->

​	{% tabs datebase-tab %}	

​		<!-- tab Mysql -->

​		{% post_link database/mysql/安装MYSQL %}

​		{% post_link database/mysql/sql %}

​		{% post_link database/mysql/函数 %}

​		{% post_link database/mysql/约束 %}

​		{% post_link database/mysql/多表查询 %}

​		{% post_link database/mysql/事务 %}

​		{% post_link database/mysql/索引查询 %}

​		{% post_link database/mysql/其他查询 %}

​		{% post_link database/mysql/问题记录 %}

​		{% post_link database/mysql/锁 %}

​		{% post_link database/mysql/存储引擎 %}

​		<!-- endtab -->

​		<!-- tab Elasticsearch -->

  - Elasticsearch 5.5

    {% post_link database/elasticsearch/5.5/README %}
    {% post_link database/elasticsearch/5.5/倒排索引 %}
    {% post_link database/elasticsearch/5.5/环境安装 %}
    {% post_link database/elasticsearch/5.5/一些概念 %}
    {% post_link database/elasticsearch/5.5/数据类型 %}
    {% post_link database/elasticsearch/5.5/操作索引 %}
    {% post_link database/elasticsearch/5.5/操作文档 %}
    {% post_link database/elasticsearch/5.5/文档检索查询 %}
    {% post_link database/elasticsearch/5.5/复合检索 %}
    {% post_link database/elasticsearch/5.5/聚合统计查询 %}
    {% post_link database/elasticsearch/5.5/数据备份 %}
    {% post_link database/elasticsearch/5.5/问题记录 %}
    {% post_link database/elasticsearch/5.5/SpringBoot整合ES %}
    {% post_link database/elasticsearch/5.5/测试数据 %}
    {% post_link database/elasticsearch/5.5/数据写入和检索过程 %}

  - Elasticsearch 7.6

    {% post_link database/elasticsearch/7.6/README  %}

    {% post_link database/elasticsearch/7.6/install  %}

​		<!-- endtab -->



​	<!-- tab Redis -->

​			{% post_link database/redis/安装环境 %}

​			{% post_link database/redis/基础指令 %}

​			{% post_link database/redis/数据类型 %}

​			{% post_link database/redis/缓存更新策略 %}

​		<!-- endtab -->



​		<!-- tab Neo4j -->

​			{% post_link database/neo4j/start %}

​			{% post_link database/neo4j/cypher %}

​			{% post_link database/neo4j/cypher_opt %}

​		<!-- endtab -->

​	

​		<!-- tab IoTDB -->

​			{% post_link database/iotdb/start %}

​			{% post_link database/iotdb/sql %}

​		<!-- endtab -->



​	{% endtabs %}	

<!-- endtab -->



<!-- tab Spring -->

{% post_link framework/spring/springbean生命周期和作用域 %}

{% post_link framework/spring/spring-IOC-DI %}

{% post_link framework/spring/spring-aop %}

{% post_link framework/spring/spring-拦截器 %}

{% post_link framework/spring/spring-cache %}

<!-- endtab -->



<!-- tab Springboot -->

{% post_link framework/springboot/springboot-validation %}

{% post_link framework/springboot/springboot-mybatis %}

{% post_link framework/springboot/springboot-jpa %}

{% post_link framework/springboot/springboot-websocket %}

<!-- endtab -->



<!-- tab Springcloud -->

{% post_link framework/springcloud/README %}

{% post_link framework/springcloud/搭建微服务工程 %}

- 服务注册
  - {% post_link framework/springcloud/service-registration/eureka %}
  - {% post_link framework/springcloud/service-registration/consul %}
- 服务调用
  - {% post_link framework/springcloud/service-call/Ribbon %}
  - {% post_link framework/springcloud/service-call/Feign %}
  - {% post_link framework/springcloud/service-call/OpenFeign %}
- 服务降级
  - {% post_link framework/springcloud/service-degradation/hystrix %}
- 服务网关
  - {% post_link framework/springcloud/service-gateway/zuul %}
  - {% post_link framework/springcloud/service-gateway/gateway %}
- 配置中心
  - {% post_link framework/springcloud/service-config/config %}
- 消息总线
  - {% post_link framework/springcloud/service-bus-message/bus %}
  - {% post_link framework/springcloud/service-bus-message/stream %}
- 链路跟踪
  - {% post_link framework/springcloud/service-link-tracking/sleuth %}
- 服务部署
  - {% post_link framework/springcloud/service-deployment/jenkins %}
  - {% post_link framework/springcloud/service-deployment/Alibaba-Cloud-Toolkit %}
- 服务监控
  - {% post_link framework/springcloud/service-monitoring/list %}
  - {% post_link framework/springcloud/service-monitoring/springboot-admin %}
  - {% post_link framework/springcloud/service-monitoring/SkyWalking %}
  - {% post_link framework/springcloud/service-monitoring/elastic-apm %}
- Spring Cloud Alibaba
  - {% post_link framework/springcloud/alibaba/README %}
  - {% post_link framework/springcloud/alibaba/nacos %}
  - {% post_link framework/springcloud/alibaba/sentinel %}

<!-- endtab -->



<!-- tab Network -->

{% post_link network/网络分层协议 %}

{% post_link network/传输层 %}

{% post_link network/网络层IP %}

<!-- endtab -->



<!-- tab Netty -->
{% post_link netty/java-io %}

{% post_link netty/java-nio %}

{% post_link netty/netty-thread %}

{% post_link netty/netty-sync %}

{% post_link netty/netty-heartbeat %}

{% post_link netty/netty-handler %}

{% post_link netty/TCP粘包拆包 %}

{% post_link netty/netty-sample %}

<!-- endtab -->





<!-- tab Message Center -->

​	{% tabs messageCenter-tab %}	

​		<!-- tab kafka -->

​			{% post_link framework/kafka/message-center/start %}

​			{% post_link framework/kafka/message-center/dashboard %}

​			{% post_link framework/kafka/message-center/partition %}

​			{% post_link framework/kafka/message-center/spring-kafka %}

​		<!-- endtab-->

​	{% endtabs%}

<!-- endtab -->



<!-- tab linux -->

{% post_link linux/centos7-常用命令 %}

{% post_link linux/centos7-常用配置 %}

{% post_link linux/centos7-软件&环境安装 %}

{% post_link linux/centos7-单机部署FastDFS %}

{% post_link linux/centos7-排查CPU占用高JAVA代码 %}

{% post_link linux/shell脚本 %}

{% post_link linux/centos7-定时任务 %}

<!-- endtab -->



<!-- tab Docker -->

{% post_link docker/start %}

{% post_link docker/Dockerfile %}

{% post_link docker/springboot %}

{% post_link docker/compose %}

<!-- endtab -->



<!-- tab Development Tool -->

​	{% tabs dtools %}

​	<!-- tab macbook -->
​		{% post_link tools/macbook/UsefulSettings %}

​		{% post_link tools/macbook/apps %}

​	<!-- endtab -->



​	<!-- tab idea -->

​		{% post_link tools/idea/必备插件 %}

​		{% post_link tools/idea/常用配置 %}

​		{% post_link tools/idea/leetcode %}

​		{% post_link tools/idea/问题记录 %}

​	<!-- endtab -->



​	<!-- tab maven -->
​   {% post_link tools/maven/maven %}
​   {% post_link tools/maven/pom_help %}
​	<!-- endtab -->

​	

​	<!-- tab git -->

​		{% post_link tools/git/git_command %}


​	<!-- endtab -->



​	<!-- tab gitlab -->

​    {% post_link tools/git/gitlab_start %}  

​	<!-- endtab -->



​	<!-- tab Jenkins -->

​    {% post_link tools/jenkins/install %}

​	<!-- endtab -->



​	<!-- tab sonarqube -->

​		SonarQube是一个用于代码质量管理的开源平台，用于管理源代码的质量。同时  SonarQube 还对大量的持续集成工具提供了接口支持，可以很方便地在持续集成中使用  SonarQube。SonarQube 支持插件扩展，拥有丰富的插件。

官网：https://www.sonarqube.org/	

 - 7.6 
   - {% post_link tools/sonarqube/7.6/start %}
   - {% post_link tools/sonarqube/7.6/gitlab-sonar %}
 - 8.9
   - {% post_link tools/sonarqube/8.9/start %}
   - {% post_link tools/sonarqube/8.9/gitlab-sonar %}

<!-- endtab -->



​	<!-- tab 接口测试 -->

​		{% post_link tools/interfacetest/postman %}

​		{% post_link tools/interfacetest/apipost %}

​	<!-- endtab -->



​	{% endtabs %}

<!-- endtab -->



<!-- tab Frontend -->
{% post_link frontend/html5&css3/selectors %}
{% post_link frontend/html5&css3/box %}
{% post_link frontend/html5&css3/float %}
{% post_link frontend/html5&css3/position %}
<!-- endtab -->



{% endtabs %}







