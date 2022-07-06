---
title: kafka 集群部署&基本概念和操作
categories: kafka
tags:
  - kafka
keywords: 'kafka,消息中心'
abbrlink: dc53a8f5
date: 2022-02-10 12:00:00
---
Kafka

官网：https://kafka.apache.org/

开源地址：https://github.com/apache/kafka

使用的版本：kafka_2.12-3.0.0

## 集群搭建

集群示意图

![](https://blog.lichenghao.cn/upload/2022/07/06101635.png)

### zk集群

修改zookeeper 配置文件

修改内容为：dataDir，dataLogDir，集群服务，完成配置如下所示：

```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/opt/zookeeper-3.8/data
dataLogDir=/opt/zookeeper-3.8/dataLog
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

## Metrics Providers
#
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpHost=0.0.0.0
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true

server.1= 192.168.200.128:2888:3888
server.2= 192.168.200.129:2888:3888
server.3= 192.168.200.130:2888:3888
```

然后在 dataDir 所在目录下增加一个myid文件，文件内容为 server.n= xxx.xxx.x.xxx:2888:3888 中的n。

```properties
[root@bogon zookeeper-3.8]# cat data/myid 
1
```

然后将该节点的zookeeper分发到另外的服务器上。注意修改myid文件内容。

启动zk集群，分别启动每个节点即可。

```sh
[root@bogon zookeeper-3.8]# bin/zkServer.sh start
```

> 如果没有启动起来，那么可能存在的问题是，这个节点以前作为单节点启动过。删除data下面的除了myid以外的其他文件。然后再次重启即可。

验证启动，QuorumPeerMain 是其守护进程。

```sh
[root@bogon zookeeper-3.8]# jps
1888 QuorumPeerMain
2132 Jps
```

也可以利用可视化工具查看，https://github.com/vran-dev/PrettyZoo

![](https://blog.lichenghao.cn/upload/2022/07/102041.jpg)



### kafka集群



修改配置文件`server.properties`

修改内容包括：broker.id 唯一即可，log.dirs 默认在临时文件中，zookeeper.connect 增加集群节点

完整如下：

```properties
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# see kafka.server.KafkaConfig for additional details and defaults

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=1

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://192.168.200.128:9092

# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
#advertised.listeners=PLAINTEXT://your.host.name:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/opt/kafka/kafka-logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=192.168.200.128:2181,192.168.200.129:2181,192.168.200.130:2181/kafka

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=18000


############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0
```

然后分发到其他节点上。`!注意修改broker.id`

启动每个节点即可。

```sh
[root@bogon kafka]# bin/kafka-server-start.sh -daemon config/server.properties
```

查看进程

```sh
[root@bogon kafka]# jps
1888 QuorumPeerMain
2601 Jps
2511 Kafka
```

同时查看zk节点

![](https://blog.lichenghao.cn/upload/2022/07/103810.jpg)



## 基础起步



### 基本概念

参考文档：

https://segmentfault.com/a/1190000040633029

https://blog.csdn.net/lzxlfly/article/details/80672284

kafka是apache基金会管理的开源流处理平台，不单单是个消息中间件！下面是对于消息队列的使用。



#### 主题（Topic）

生产者使用不用的主题将将不同的数据区分开来，不同的消费者则可以根据不同的主题消费不同的数据。



#### 分区（partition）

如果一个主题下的数据特别的多，那么就会考虑分割数据。将一个主题拆分成不同的分区，多个分区之间互不干扰，来提高吞吐量。kafka对于分区的拆分有很多的策略，例如：随机，轮训，hash等。



#### 副本（Replication）

要想保证数据尽可能不丢失，就的把数据多整几份到其他的节点上，复制出来的数据就叫做副本。

这里有几个概念先了解：

**HW：** high-water，一个特殊的offset，只有在这个offset以下的消息才能被消费者(consumer)读到，高水位的具体值取决于主从副本数据同步的状态，这里不再展开。（offset：消息在文件中的位置）
**ISR:** in-sync-replica，处于同步状态的副本集合，是指副本数据和主副本数据相差在一定范围（时间范围或数量范围）之内的副本，当然主副本肯定是一直在ISR中的。 当主副本挂了之后，新的主副本将从ISR中被选出来接替它的工作。

**OSR:** 和IRS相对应 out-sync-replica，其实就是指那些不在ISR中的副本。



副本的功能就是给主副本当备用，只有主副本才会对外提供读和写，当主副本挂了立马从副本中再选出一个当主副本。

#### Broker

翻译过来就是经纪人，就是用来管理这些主从副本的。broker将数据的不同副本分发到不同的节点上，这个跟ES很像。以及其他的管理工作。



#### 消费者组（Consumer-group）



同一个分区同时只能被同一个消费者消费。同一个消费者组的消费者可以消费同一个topic下的不同分区。

为了提高吞吐量，一般会设置多个消费者来消费不同的分区，这些消费者组成消费者组，他们拥有相同的 Goup-id。



### 基本操作

#### 创建Topic

```sh
[root@bogon kafka]# bin/kafka-topics.sh --create --zookeeper 192.168.200.128:2181,192.168.200.129:2181,192.168.200.130:2181/kafka --partitions 3  --replication-factor 3 --topic kfk_hello
```

结果 （提示不建议用 . 和 _）

```sh
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic kfk_hello.

```

> 这里需要注意，如果在kafka配置文件中配置的zookeeper链接指定了节点，那么这里创建也需要加上。如上的/kafka。不然会报错：
>
> ```sh
> Error while executing topic command : Replication factor: 2 larger than available brokers: 0.
> ```



#### 列出Topic列表

```sh
[root@bogon kafka]# bin/kafka-topics.sh --list --zookeeper 192.168.200.128:2181,192.168.200.129:2181,192.168.200.130:2181/kafka
kfk_hello
```



#### 查看Topic信息

```sh
[root@bogon kafka]# bin/kafka-topics.sh --describe --zookeeper 192.168.200.128:2181,192.168.200.129:2181,192.168.200.130:2181/kafka --topic kfk_hello
Topic: kfk_hello	TopicId: YdV2sn_CTk6QOrzgAMFlxA	PartitionCount: 3	ReplicationFactor: 3	Configs: 
	Topic: kfk_hello	Partition: 0	Leader: 1	Replicas: 1,2,3	Isr: 1
	Topic: kfk_hello	Partition: 1	Leader: 2	Replicas: 2,3,1	Isr: 2,3,1
	Topic: kfk_hello	Partition: 2	Leader: 3	Replicas: 3,1,2	Isr: 3,1,2

```



#### 删除指定的Topic

```sh
[root@bogon kafka]# bin/kafka-topics.sh --delete --zookeeper 192.168.200.128:2181,192.168.200.129:2181,192.168.200.130:2181/kafka --topic kfk_hello
Topic kfk_hello is marked for deletion.
Note: This will have no impact if delete.topic.enable is not set to true.
```



#### 生产者生产

```sh
[root@bogon kafka]# bin/kafka-console-producer.sh -broker-list 192.168.200.128:9092,192.168.200.129:9092,192.168.200.130:9092 --topic kfk_hello
>123
>456

```

如果报错 `ERROR Error when sending message to topic kfk_hello with key: null, value: 4 bytes with error: (org.apache.kafka.clients.producer.internals.ErrorLoggingCallback)`

检查下配置文件 其中的listeners是否打开，没有请打开。

```properties
listeners=PLAINTEXT://192.168.200.128:9092
```



#### 消费者消费

```sh
[root@bogon kafka]# bin/kafka-console-consumer.sh --bootstrap-server 192.168.200.128:9092,192.168.200.129:9200,192.168.200.130:9092 --from-beginning --topic kfk_hello

123
456
```







































