---
title: springcloud 服务监控 Elastic APM
categories: springcloud
tags:
  - springcloud
  - Elastic
  - APM
abbrlink: 6c9e8453
date: 2022-02-02 12:00:00
---
## 入门

Elastic APM是基于Elastic Stack构建的应用性能监控（APM）系统。

官方文档：https://www.elastic.co/guide/en/apm/server/7.6/getting-started-apm-server.html

### 下载&部署相关软件

Elasticsearch， Kibana ，APM-Server  (版本：7.6.2)

相关软件的下载地址：https://www.elastic.co/cn/downloads/past-releases#elasticsearch



安装 ES  - [ES7.6.2-环境安装](db/elasticsearch/7.6.2/install.md)

安装 Kibana 

解压安装文件，修改配置文件,kibana.yml

```yaml
#配置端口号
server.port: 5601
#配置网络访问地址
server.host: "0.0.0.0"
#配置es链接地址(es集群,可以用逗号分隔)
elasticsearch.hosts: ["http://localhost:9200"]
```

后台启动即可

```sh
[es@lch kibana-7.6.2]$ nohup bin/kibana &
```

安装 APM-Server

解压后，修改配置文件  (如果和 ES 在同一个服务器下就不用修改了)

```yacas
apm-server:
  host: "192.168.1.240:8200"
output.elasticsearch:
  hosts: ["localhost:9200"]
```

启动APM Server即可，启动成功 APM Server 将在`8200`端口运行；

```bash
# 前台启动
[root@lch apm-server-7.6.2]# ./apm-server -e
# 后台启动
[root@lch apm-server-7.6.2]# nohup ./apm-server -e &
```

可以直接访问 ip:8200，如下表示启动成功

```json
{
    "build_date": "2020-02-26T04:23:50Z",
    "build_sha": "8827b43d24eaf42391ea815bb9fa12bdef5d6bce",
    "version": "7.6.2"
}
```

还可以在 kibana 中检测下，home —> Add APM —>Check APM Server status (http://ip:5601/app/kibana#/home/tutorial/apm)

![](https://blog.lichenghao.cn/upload/2022/07/10092709.png)



### 写个启动&关闭脚本

启动 `start.sh ` 放在安装目录下：

```shell
nohup ./apm-server -e > apmserver.log 2>&1 &
```

关闭脚本 `stop.sh`

```shell
#!/bin/bash

ID=`ps -ef | grep apm-server  | grep -v grep | awk '{print $2}'`
for id in $ID
do
        kill -9 $id
        echo "kill $id"
done
```

使用前，别忘记授权 `chmod +x xxxx.sh `



### SpringBoot集成APM Agent

增加依赖

```xml
<dependency>
                <groupId>co.elastic.apm</groupId>
                <artifactId>apm-agent-attach</artifactId>
                <version>1.17.0</version>
            </dependency>
```

增加 apm server 配置文件 elasticapm.properties

```properties
# 配置服务名称
service_name=SECOND-PRODUCER-SERVICE
# 配置应用所在基础包
application_packages=cn.lichenghao
# 配置APM Server的访问地址
server_urls=http://192.168.1.240:8200
```

然后在启动类绑定代理即可，`ElasticApmAttacher.attach();`

```java
/**
 * 生产者
 *
 * @author chenghao.li
 */
@SpringBootApplication
@EnableDiscoveryClient
public class SecondProduder7001Application {
    public static void main(String[] args) {
        // agent
        ElasticApmAttacher.attach();
        SpringApplication.run(SecondProduder7001Application.class, args);
    }

    @Value("${server.port}")
    private String port;

    /**
     * 演示请求
     */
    @RestController
    public class SecondProducerController {
        @GetMapping(value = "/second-producer/{id}")
        public String producer(@PathVariable String id) {
            int a = 1/0;
            return "second-producer-service : " + port + "\t id:" + id;
        }
    }
}
```

然后通过 kibana 查看。

![](https://blog.lichenghao.cn/upload/2022/07/12173340.png)













