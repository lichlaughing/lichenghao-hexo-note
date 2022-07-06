---
title: spring kafka 集成示例
categories: kafka
tags:
  - kafka
  - spring
keywords: 'kafka,消息中心'
abbrlink: 7d28f865
date: 2022-02-10 12:00:00
---
集成 Spring 落地使用，记录 Springboot使用。

文档先行：https://docs.spring.io/spring-kafka/docs/current/reference/html/#preface

## 基础环境

前提条件：

- [集群搭建&部分概念](framework/kafka/message-center/install)
- [可视化监控](framework/kafka/message-center/dashboard)

### 版本

测试工程版本：

父工程给出版本依赖，完整 pom 如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>M1</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>kafka</module>
    </modules>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.12.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.18.12</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

测试项目继承依赖即可，完整 pom 如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>M1</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <artifactId>kafka</artifactId>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-test</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
        </dependency>
    </dependencies>
</project>
```

### 配置文件

```yaml
server:
  port: 8099
spring:
  application:
    name: kafka-test
  kafka:
    bootstrap-servers: 192.168.1.240:9091,192.168.1.240:9092,192.168.1.240:9093
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

给定主启动类

```java
/**
 * 入口
 *
 * @author chenghao.li
 */
@SpringBootApplication
public class KafkaApplication {
    public static void main(String[] args) {
        SpringApplication.run(KafkaApplication.class, args);
    }
}
```

### 一个注解实现消费者

```java
/**
 * 消费者
 *
 * @author chenghao.li
 */
@Component
public class FirstConsumer {
    
    @KafkaListener(id = "group-first", topics = "topic-first")
    public void consumer(String msg) {
        System.out.println("消费：" + msg);
    }
}
```

### 测试消费

启动项目，在 kafka-eagle 中添加主题 `topic-first` 发送个 `hello world ` 就能看到效果了！

![](https://blog.lichenghao.cn/upload/2022/07/07152311.png)

利用 Mock 模式生产者，生产消息。

![](https://blog.lichenghao.cn/upload/2022/07/07152343.png)



## 生产者 Producer

Springboot会自动装配一个包装类 `org.springframework.kafka.core.KafkaTemplate` ,里面有封装好的一些生产者的方法。

### sendDefault()

KafkaTemplate 有几个 sendDefault 方法

![](https://blog.lichenghao.cn/upload/2022/07/07160904.png)

使用这些方法的前提是需要配置默认的topic，可以在配置文件中指定

```yaml
server:
  port: 8099
spring:
  application:
    name: kafka-test
  kafka:
    bootstrap-servers: 192.168.1.240:9091,192.168.1.240:9092,192.168.1.240:9093
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    # 指定默认的Topic
    template:
      default-topic: topic-default
```

### send Callback

以 sendDefault 为例，几种回调的方式（非阻塞）

```java
/**
 * 生产者给默认的topic发送消息，几种回调的方式（非阻塞）
 *
 * @author chenghao.li
 */
@Component
public class DefaultProducer {

    @Resource
    private KafkaTemplate<String, String> kafkaTemplate;

    public void send(String msg) {
        ListenableFuture<SendResult<String, String>> res = kafkaTemplate.sendDefault(msg);
        CompletableFuture<SendResult<String, String>> completable = res.completable();
        completable.whenCompleteAsync((result, throwable) -> {
            if (throwable != null) {
                // KafkaProducerException
                System.out.println("发送失败：" + throwable.getMessage());
            } else {
                System.out.println(result);
            }
        });
    }

    public void send2(String msg) {
        ListenableFuture<SendResult<String, String>> res = kafkaTemplate.sendDefault(msg);
        res.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
            @Override
            public void onFailure(Throwable throwable) {
                // KafkaProducerException
                System.out.println("发送失败：" + throwable.getMessage());
            }

            @Override
            public void onSuccess(SendResult<String, String> result) {
                System.out.println(result.toString());
            }
        });
    }

    /**
     * Starting with version 2.5, you can use a KafkaSendCallback instead of a ListenableFutureCallback
     */
    public void send3(String msg) {
        ListenableFuture<SendResult<String, String>> res = kafkaTemplate.sendDefault(msg);
        res.addCallback(new KafkaSendCallback<String, String>() {
            @Override
            public void onFailure(KafkaProducerException e) {
                System.out.println("发送失败：" + e.getMessage());
            }

            @Override
            public void onSuccess(SendResult<String, String> result) {
                System.out.println(result);
            }
        });
    }

    /**
     * use a pair of lambdas
     */
    public void send4(String msg) {
        ListenableFuture<SendResult<String, String>> future = kafkaTemplate.sendDefault(msg);
        future.addCallback(result -> {
            // do something
        }, (KafkaFailureCallback<Integer, String>) ex -> {
            ProducerRecord<Integer, String> failed = ex.getFailedProducerRecord();
            System.out.println("发送失败：" + ex.getMessage());
        });
    }
}
```

当然还有阻塞的方式

```java
		/**
     * 阻塞
     */
    public void send5(String msg) {
        try {
            SendResult<String, String> result = kafkaTemplate.sendDefault(msg).get();
            // do something
        } catch (InterruptedException | ExecutionException e) {
            throw new RuntimeException(e);
        }
    }
```

### send()

生产消息需要几个参数：

```java
public class ProducerRecord<K, V> {

    private final String topic;
    private final Integer partition;
    private final Headers headers;
    private final K key;
    private final V value;
    private final Long timestamp;
  ......
}
```

参数说明：

- topic：消息主题

- value：消息内容

- partition：如果一个有效的partition属性数值被指定，那么在发送记录时partition属性数值就会被应用

- key：如果没有partition属性数值被指定，而一个key属性被声明的话，一个partition会通过key的hash而被选中

- 如果既没有key也没有partition属性数值被声明，那么一个partition将会被分配以轮询的方式。

- timestamp：如果用户没有提供时间戳，生产者将用当前时间标记记录。 Kafka 最终使用的时间戳取决于为主题配置的时间戳类型。

- - 如果主题配置为使用CreateTime ，则生产者记录中的时间戳将被 broker 使用。
  - 如果该主题被配置为使用LogAppendTime ，则生产者记录中的时间戳将被 broker 在将消息附加到其日志时使用本地时间覆盖。

对参数了解后，使用就变得简单了

```java
/**
 * 生产者，演示使用就不写回调了
 *
 * @author chenghao.li
 */
@Component
public class FirstProducer {

    @Resource
    private KafkaTemplate<String, String> kafkaTemplate;

    /**
     * 指定 topic 发送消息
     *
     * @param topic 主题
     * @param msg   消息
     */
    public void send(String topic, String msg) {
        // 指定 topic 发送消息
        kafkaTemplate.send(topic, msg);
    }

    /**
     * 指定分区发型消息
     *
     * @param topic     主题
     * @param partition 分区
     * @param msg       消息
     */
    public void send2(String topic, int partition, String k, String msg) {
        // 指定分区发送消息
        kafkaTemplate.send(topic, partition, k, msg);
    }

    /**
     * 指定好多信息
     *
     * @param topic     主题
     * @param partition 分区
     * @param timestamp 时间戳
     * @param k         key
     * @param msg       消息
     */
    public void send3(String topic, int partition, long timestamp, String k, String msg) {
        kafkaTemplate.send(topic, partition, timestamp, k, msg);
    }

    /**
     * 把上面的参数对象化
     *
     * @param record 消息参数
     */
    public void send4(ProducerRecord<String, String> record) {
        kafkaTemplate.send(record);
    }
}
```



## 消费者 Consumer

### 监听接口

消费者通过 `Message Listeners` 来监听消息，大概有八种类型：

```java

public interface MessageListener<K, V> { 

    void onMessage(ConsumerRecord<K, V> data);

}

public interface AcknowledgingMessageListener<K, V> { 

    void onMessage(ConsumerRecord<K, V> data, Acknowledgment acknowledgment);

}

public interface ConsumerAwareMessageListener<K, V> extends MessageListener<K, V> { 

    void onMessage(ConsumerRecord<K, V> data, Consumer<?, ?> consumer);

}

public interface AcknowledgingConsumerAwareMessageListener<K, V> extends MessageListener<K, V> { 

    void onMessage(ConsumerRecord<K, V> data, Acknowledgment acknowledgment, Consumer<?, ?> consumer);

}

public interface BatchMessageListener<K, V> { 

    void onMessage(List<ConsumerRecord<K, V>> data);

}

public interface BatchAcknowledgingMessageListener<K, V> { 

    void onMessage(List<ConsumerRecord<K, V>> data, Acknowledgment acknowledgment);

}

public interface BatchConsumerAwareMessageListener<K, V> extends BatchMessageListener<K, V> { 

    void onMessage(List<ConsumerRecord<K, V>> data, Consumer<?, ?> consumer);

}

public interface BatchAcknowledgingConsumerAwareMessageListener<K, V> extends BatchMessageListener<K, V> { 

    void onMessage(List<ConsumerRecord<K, V>> data, Acknowledgment acknowledgment, Consumer<?, ?> consumer);

}
```

### 监听容器

除此之外，提供了两个监听容器

- `KafkaMessageListenerContainer`   单线程接收所有的主题或者分区中的消息
- `ConcurrentMessageListenerContainer ` 一个或多个KafkaMessageListenerContainer实例，以提供多线程消费

### 注解监听

@KafkaListener



### 更新 offset 策略

默认情况下，消费者消费完毕后，是自动更新 offset的，对应的配置为：

```yaml
server:
  port: 8099
spring:
  application:
    name: kafka-test
  kafka:
    bootstrap-servers: 192.168.1.240:9091,192.168.1.240:9092,192.168.1.240:9093
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      # 自动提交 offset (从2.3开始，默认值为false)
      enable-auto-commit: true
      # 自动提交 offset 时间间隔（默认 5000 毫秒）
      auto-commit-interval: 1000
    # 指定默认的Topic
    template:
      default-topic: topic-default
```

上面的配置翻译过来就是：`默认5秒钟，一个 Consumer 将会提交它的 Offset 给 Kafka，或者每一次数据从指定的 Topic 取回时，将会提交最后一次的 Offset。`

如果 enable-auto-commit: false ，那么对 offset 的处理有几种模式：

**不使用事务：**

- `RECORD`：监听器处理完记录后返回时提交偏移量
- `BATCH`：监听器处理完 poll() 返回的所有记录后，提交偏移量
- `TIME`：poll() 的数据被消费者监听器（ListenerConsumer）处理之后，距离上次提交时间大于TIME时提交
- `COUNT`：poll() 的数据被消费者监听器（ListenerConsumer）处理之后，被处理record数量大于等于COUNT时提交
- `COUNT_TIME`：TIME | COUNT 有一个条件满足时提交
- `MANUAL`：poll() 的数据被消费者监听器（ListenerConsumer）处理之后, 手动调用Acknowledgment.acknowledge()后提交
- `MANUAL_IMMEDIATE`：手动调用Acknowledgment.acknowledge()后立即提交

**事务模式下：**

使用事务时，偏移量被发送到事务，语义等同于记录或批处理，具体取决于监听器的处理类型（RECORD 或 BATCH）。

## 创建 Topic

### 方式1：kafka-topics.sh

```bash
[root@bogon kafka]# bin/kafka-topics.sh --create --zookeeper 192.168.200.128:2181,192.168.200.129:2181,192.168.200.130:2181/kafka --partitions 3  --replication-factor 3 --topic kfk_hello
```

### 方式2：使用默认的 Topic

```yaml
server:
  port: 8099
spring:
  application:
    name: kafka-test
  kafka:
    bootstrap-servers: 192.168.1.240:9091,192.168.1.240:9092,192.168.1.240:9093
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    # 指定默认的Topic
    template:
      default-topic: topic-default
```

### 方式 3：自动 auto.create.topics.enable=true

修改 broker 的 server.properties 开启自动创建功能

```properties
auto.create.topics.enable=true
```

### 方式 4：手动代码创建

上面我们都是先创建 Topic 或者使用默认的 Topic 如果我想动态的新增 Topic 怎么办？

KafkaAdmin，服务启动后，springboot 自动装载了KafkaAdmin对象。通过查询文档，`KafkaAdminClient` 可以创建主题。那么可以通过 KafkaAdmin 来获取 KafkaAdminClient 实例。

```java
@Configuration
public class KafkaConfig {
    @Resource
    private KafkaAdmin kafkaAdmin;

    @Bean("kafkaAdminClient")
    public AdminClient admin() {
        return KafkaAdminClient.create(kafkaAdmin.getConfigurationProperties());
    }
}
```

动态创建主题

```java
@Resource
@Qualifier(value = "kafkaAdminClient")
private KafkaAdminClient kafkaAdminClient;

@RequestMapping("/createTopic/{topic}")
    public String createTopic(@PathVariable("topic") String topicName) {
        NewTopic topic = TopicBuilder.name(topicName).partitions(3).replicas(3).build();
        List<NewTopic> topicList = new ArrayList<>();
        topicList.add(topic);
        CreateTopicsOptions createTopicsOptions = new CreateTopicsOptions();
        kafkaAdminClient.createTopics(topicList, createTopicsOptions);
        return "success";
    }
```



