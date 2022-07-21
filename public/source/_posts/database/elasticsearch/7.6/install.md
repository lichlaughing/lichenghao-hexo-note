---
title: Elasticsearch7.6.2 安装部署
categories: elasticsearch
tags:
  - elasticsearch
keywords: elasticsearch
abbrlink: '476e8838'
date: 2022-05-15 12:35:00
---
## 单机部署

下载地址：https://www.elastic.co/cn/downloads/past-releases#elasticsearch

解压后`elasticsearch-7.6.2-linux-x86_64.tar.gz` 执行运行 `[root@lch bin]# ./elasticsearch`

**第 1 个错误：**

!>future versions of Elasticsearch will require Java 11

建议使用 jdk11 ，但是我大部分应用还是 jd8，可以设置为 es 自带的 jdk。修改启动文`elasticsearch`件如下： 增加两块。

```properties
# 增加 JAVA
export JAVA_HOME=/opt/module/elasticsearch/elasticsearch-7.6.2/jdk
export PATH=$JAVA_HOME/bin:$PATH


source "`dirname "$0"`"/elasticsearch-env

if [ -z "$ES_TMPDIR" ]; then
  ES_TMPDIR=`"$JAVA" -cp "$ES_CLASSPATH" org.elasticsearch.tools.launchers.TempDirectory`
fi

ES_JVM_OPTIONS="$ES_PATH_CONF"/jvm.options
ES_JAVA_OPTS=`export ES_TMPDIR; "$JAVA" -cp "$ES_CLASSPATH" org.elasticsearch.tools.launchers.JvmOptionsParser "$ES_JVM_OPTIONS"`

# 增加 jdk判断 
if [ -x "$JAVA_HOME/bin/java" ]; then
        JAVA="$JAVA_HOME/bin/java"
else
        JAVA=`which java`
fi
```

**第 2 个错误：**

!>OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.

这是提醒 UseConcMarkSweepGC 收集器在 jdk9 就开始被废弃了。

修改 jvm.options

```properties
# 内存整小点
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms256M
-Xmx256M

# 替换 UseConcMarkSweepGC为 G1GC
## GC configuration
#8-13:-XX:+UseConcMarkSweepGC
8-13:-XX:+UseG1GC
8-13:-XX:CMSInitiatingOccupancyFraction=75
8-13:-XX:+UseCMSInitiatingOccupancyOnly
```

**第 3 个错误：**

配置文件必须修改 elasticsearch.yml：

比如可能报错：

```properties
at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
```

增加或者修改如下配置项目。

```yaml
cluster.name: elasticsearch
node.name: node-1
# Path to directory where to store the data (separate multiple locations by comma):
#
#path.data: /path/to/data
path.data: /opt/module/elasticsearch/elasticsearch-7.6.2/data
#path.logs: /path/to/logs
path.logs: /opt/module/elasticsearch/elasticsearch-7.6.2/logs
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["127.0.0.1", "[::1]"]
cluster.initial_master_nodes: ["node-1"]
```

其他问题：

创建 ES 用户，来启动服务，还有其他的一些配置可以参考  [ES5.5 环境安装](db/elasticsearch/5.5/环境安装.md)

最终可以启动服务了

```bash
[es@lch bin]$ ./elasticsearch -d
```

测试访问 : http://192.168.1.240:9200/

```properties
{
    "name": "node-1",
    "cluster_name": "elasticsearch",
    "cluster_uuid": "4CPaIvYqQRCqjgE_lfqOEA",
    "version": {
        "number": "7.6.2",
        "build_flavor": "default",
        "build_type": "tar",
        "build_hash": "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
        "build_date": "2020-03-26T06:34:37.794943Z",
        "build_snapshot": false,
        "lucene_version": "8.4.0",
        "minimum_wire_compatibility_version": "6.8.0",
        "minimum_index_compatibility_version": "6.0.0-beta1"
    },
    "tagline": "You Know, for Search"
}
```



参考文章：

https://www.jianshu.com/p/6ddeae547f45?utm_campaign=shakespeare



## 集群部署

XXX



