---
title: Shell 脚本
categories: linux
tags:
  - linux
abbrlink: dc3000b8
date: 2022-04-01 12:00:00
---
## 关闭一个 java 进程

如果使用相对路径启动一个 java 程序，通过 jps 查看进程的时候只能看到一个 java

```bash
[root@lch zipkin-server]# nohup java -jar zipkin-server-2.23.16-exec.jar > log.file 2>&1 &
[1] 38016
```

通过 jps 查看进程，发现 38016 进程对应的名字就是一个 jar，如果我启动多个那么就是一堆 jar

```bash
[root@lch zipkin-server]# jps
86929 QuorumPeerMain
38016 jar
87779 Kafka
86962 QuorumPeerMain
87444 Kafka
38154 Jps
85034 Bootstrap
88122 Kafka
90330 KafkaEagle
87005 QuorumPeerMain
```

如果想要显示名称的话，使用绝对路径启动 jar。

```bash
[root@lch zipkin-server]# nohup java -jar /opt/module/zipkin-server/zipkin-server-2.23.16-exec.jar > log.file 2>&1 &
[1] 38546
```

然后再查看进程，发现就显示指定的 jar 名称了。进程 ID 38546。

```bash
[root@lch zipkin-server]# jps
86929 QuorumPeerMain
87779 Kafka
38546 zipkin-server-2.23.16-exec.jar
86962 QuorumPeerMain
38563 Jps
87444 Kafka
85034 Bootstrap
88122 Kafka
90330 KafkaEagle
87005 QuorumPeerMain
```

杀死这样的一个进程，需要先查询进程 ID，然后 kill，写成 shell 脚本：

stopjava.sh  

```bash
#!/bin/bash

ID=`ps -ef | grep $1  | grep java | awk '{print $2}'`
for id in $ID
do
        kill -9 $id
        echo "kill $id"
done
```

解释下：

- grep  $1 : 过滤给定参数的进程
- grep java : 因为是 java 进程再过滤一次
- awk：处理每一行的字段内的数据。上面的意思就是把查询结果的第 2 个字段(进程 ID )赋值给 ID

使用，给定需要杀死的进程的名称即可。

```bash
# 执行之前，授权别忘了
[root@lch zipkin-server]# ./stopjava.sh zipkin
kill 38546
Killed
```



## 批量删除 Docker 容器

批量删除包含 "digital" 的容器脚本`delImages.sh`，所以前提是容器名称包含 digital。可以考虑给容器增加分类前缀或者后缀。

```bash
sudo docker rmi -f $(docker images | grep digital | awk '{print $3}')
```

还可以把关键字 "digital" 当成参数拿出来，如下脚本 delImages.sh

```bash
sudo docker rmi -f $(docker images | grep $1 | awk '{print $3}')
```

使用参数删除

```bash
[root@lch ~]# ./delImages.sh digital
```



