---
title: jenkins 部署微服务
categories: springcloud
tags:
  - springcloud
  - jenkins
abbrlink: 198a8951
date: 2022-07-12 15:08:30
---
部署基本的服务：

Gitlab 安装：{% post_link tools/git/gitlab_start %}  

Jenkins 安装： {% post_link tools/jenkins/install %}

Docker服务 安装： {% post_link docker/start %}



实现的效果，将构建脚本直接放在项目中，jenkins 一键打包即可。（包括：build.sh、Dockerfile）

maven 聚合项目的目录如下所示：

```properties
├── api
│   ├── pom.xml
│   ├── src
│   │   └── main
│   └── target
│       ├── classes
│       ├── maven-archiver
│       └── maven-status
├── appdemo
│   ├── pom.xml
│   ├── src
│   │   ├── main
│   │   └── Test
│   └── target
│       ├── classes
│       ├── maven-archiver
│       └── maven-status
........... 还有很多
├── pom.xml
└── README.md
```

将脚本放在 src/test/resource下

build.sh 脚本内容

```shell
#! /bin/bash

# jenkins 数据目录
jenkinsHome=/root/.jenkins/workspace
# 构建的名称
buildName=org-appdemo
# 项目名称
projectName=appdemo
# jar包的名称
jarName=appdemo-0.0.1-SNAPSHOT.jar
# docker image name
dockerImageName=org/appdemo
# container name
dockerContainerName=appdemo01

# 停掉容器并删除
for i in `docker ps | grep $dockerContainerName |awk '{print $1}'`
do
	docker stop $i
done
for i in `docker ps -a | grep $dockerContainerName |awk '{print $1}'`
do
	docker rm -f $i
done

# 删除镜像
for i in `docker images|grep $dockerContainerName |awk '{print $1}'`
do
	docker rmi $i
done
echo "清除历史镜像和容器完毕!"
# 构建镜像
echo "当前位置:"
pwd
cd $projectName/target
cp $jarName classes
cd classes
chmod +x build.sh
docker build -t $dockerImageName:latest .

# 运行镜像，生成容器实例
docker run -itd --name $dockerContainerName --network=host -p 8010:8010 $dockerImageName
```

Dockerfile 

```dockerfile
FROM frolvlad/alpine-java:jdk8-slim
ADD appdemo-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-jar","-Dspring.profiles.active=test","/app.jar"]
```

然后在 jenkins 新建个构建，其他的都一样，只需要在 Post Steps 中调用下我们的 shell 脚本即可

```shell
sh $WORKSPACE/appdemo/target/classes/build.sh
```





> 脚本的大概执行流程：
>
> - 杀死历史的镜像和容器
> - 将生成的 jar 包放在 Dockerfile 所在目录中
> - 脚本授权
> - 构建镜像，运行容器
> - 完毕







