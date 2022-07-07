---
title: Docker Compose
categories: docker
tags:
  - docker
  - compose
keywords: 'docker,compose'
abbrlink: '60641507'
date: 2022-05-25 12:30:00
cover: https://blog.lichenghao.cn/upload/2022/07/D7072609-docker.png
---
**Compose 是用于定义和运行多容器 Docker 应用程序的工具**。通过容器编排，可以让容器按照一定的顺序执行。

开源地址：https://github.com/docker/compose

文档：https://docs.docker.com/compose/

## 安装

去 GitHub 直接下载：https://github.com/docker/compose/releases

然后重命名放到 `/usr/local/bin` 下，授权即可。

```bash
# 重命名
[root@lch ~]# mv docker-compose-linux-x86_64 docker-compose
# 放到指定位置
[root@lch ~]# mv docker-compose /usr/local/bin/
# 授权
[root@lch ~]# chmod +x /usr/local/bin/docker-compose 
# 验证
[root@lch ~]# docker-compose -v
Docker Compose version v2.6.0
```

## 简单编排

首先我们需要有一个能正确执行的 Dockerfile 文件。这里我们用 skywalking 的 java agent 演示。

准备对应的文件

```shell
# jar包
digitaltwins-model-server-1.0.0-SNAPSHOT.jar
# skywalking java agent
digitaltwins-model-server-agent
# Dockerfile
model-server-dockerfile
```

这个时候 Dockerfile的文件名称：`model-server-dockerfile`

```dockerfile
FROM java:8
ADD digitaltwins-model-server-1.0.0-SNAPSHOT.jar /app.jar 
COPY digitaltwins-model-server-agent/ /opt/module/digitaltwins-model-server-agent
EXPOSE 8091
RUN bash -c 'touch /app.jar'
# 指定docker容器启动时运行jar包
ENTRYPOINT ["java","-javaagent:/opt/module/digitaltwins-model-server-agent/skywalking-agent.jar" ,"-jar","/app.jar"]
```

新建一个编排文件 `docker-compose.yml`

```yaml
version: "3"
services:
  model-server:
    build:
      context: .
      dockerfile: model-server-dockerfile
    image: model-server
    container_name: model-server
    ports:
      - "8091:8091"
    privileged: true
```

构建镜像，前台运行

```bash
[root@lch digitalTwins]# docker-compose up --build
```

构建镜像，后台运行

```bash
[root@lch digitalTwins]# docker-compose up -d
```

