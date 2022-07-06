---
title: kafka Partition 分区
categories: kafka
tags:
  - kafka
  - Partition
keywords: 'kafka,分区,消息中心'
abbrlink: dbf4ed3c
date: 2022-02-10 12:00:00
---
## Partition

再说 Topic , kafka根据 Topic 对消息进行分类，发布到 kafka 集群的消息需要指定一个 Topic。

Topic 是一个逻辑概念，它被分成多个 Partition（Partition 是最小的存储单元）每个 Partition 是一个单独的 log 文件，存储Topic的部分数据。

为什么要使用分区？

一个 Topic 可以被多个消费者消费，如果 Topic 都在一个 Broker 上，那么支持的消费者数量就会有限（受服务器 IO ，网络等影响）。如果使用分区，那么消费者可以去不同的分区去消费，这样提高了消费性能。Partition log文件会受到所在机器的文件系统大小的限制，分区之后可以将不同的分区放在不同的机器上，相当于对数据做了分布式存储，理论上一个 Topic 可以处理任意数量的数据。

## Offset（偏移量）

log 文件中的每一条记录都会被分配一个唯一的序号，称之为`Offset（偏移量）`，它是一个由 kafka 维护的递增的，不可变的数字。

当一条消息记录写入到 Partition 的时候，它被追加到 log 文件的末尾，并分配偏移量。

![](https://blog.lichenghao.cn/upload/2022/07/06112204.png)

?>从上图可以得到结论，站在 Topic 角度看消息是无序的，但是从每一个 Partition 中来看，消息是有序的。如果让 Topic 的消息有序，是不是可以只设置一个 Partition ?



## Replication

和 ES 的副本有些类型，同一分区下的所有副本保存相同的消息，这些副本保存在不同的 Broker上，从而避免节点宕机导致的服务不可用。

副本分成两类：领导者副本（Leader Replica）和追随者副本（Follower Replica）。每个分区在创建时都要选举一个副本，称为领导者副本，其余的副本自动称为追随者副本。其中领导者负责读写，追随者异步拉取信息。

当领导者副本挂掉了，或者说领导者副本所在的Broker宕机时，Kafka依托于ZooKeeper提供的监控功能能够实时感知到，并立即开启新一轮的领导者选举，从追随者副本中选一个作为新的领导者。老Leader副本重启回来后，只能作为追随者副本加入到集群中。所以追随者就是个备胎！

![](https://blog.lichenghao.cn/upload/2022/07/06141805.png)



### AR & ISR & OSR 

所有的副本（replicas）统称为Assigned Replicas，即AR。

ISR：In-sync Replicas，副本集合。

ISR 中的副本都是与 Leader 同步的副本，不在 ISR 中的副本被认为没有与 Leader 同步 （也许还在同步中）。（注意：Leader Replica 天生就在 ISR 中）

什么样的 Follower Replica 能进入 ISR ?

Broker 端有个参数 `replica.lag.time.max.ms` ,表示 Follower Replica 能够落后 Leader Replica 的最长时间，默认 10s。超过阈值都会把 Follower Replica 剔除出 ISR, 存入OSR（Outof-Sync Replicas）列表。

所以：AR = ISR + OSR。

### 优先副本选举

假设现在有三个节点的 Kafka Cluster，我们有一个 topic 为 "topic-test", 三个分区，每个分区一主二从，那么示意图如下所示：

![](https://blog.lichenghao.cn/upload/2022/07/06153138.png)

对于同一个分区，同一个 broker 上不会出现它的副本。Leader Replica 所在的节点为 Leader 节点，Follower Replica 所在的节点为 Follower 节点。所以上图中：

- 分区 0 的 AR节点 (Leader Replica，Follower Replica 节点) 为 [broker 1,broker 0, broker 2]；
- 分区 1 的 AR节点 (Leader Replica，Follower Replica 节点) 为 [2,0,1]
- 分区 2 的 AR节点 (Leader Replica，Follower Replica 节点) 为 [0,1,2]

这个时候如果分区 1 的 Leader 节点 broker 2 死掉了，那么就会进行副本的选举，副本在 broker0 和 broker1 上，那么可能 broker1 上的副本成了分区 1 的 Leader Replica节点，然后 broker2 复活后变成了分区 1 的Follower Replica 节点，如下所示：

![](https://blog.lichenghao.cn/upload/2022/07/06155406.png)

这样的话，并没有实现负载均衡的效果，把压力都给到了 broker1。

**为了解决上述负载不均衡的情况，kafka支持了`优先副本选举`，优先副本指的是一个分区所在的 AR 集合的第一个副本**。

分区 1 的 AR节点 (Leader Replica，Follower Replica 节点) 为 [2,0,1]，所以分区 1 的优先副本为 broker2，针对上述的情况，只要再触发一次`优先副本选举`就能保证分区负载均衡。(kafka支持自动优先副本选举功能，默认每5分钟触发一次优先副本选举操作)。

## Partition重分配

xxx

