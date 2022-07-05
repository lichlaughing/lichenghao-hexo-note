---
title: Alibaba-Cloud-Toolkit 部署微服务
categories: springcloud
tags:
  - springcloud
  - Alibaba-Cloud-Toolkit
abbrlink: d9702737
date: 2022-02-02 12:00:00
---
##   IDEA 插件部署微服务

### 两分钟部署一个微服务

插件地址：https://plugins.jetbrains.com/plugin/11386-alibaba-cloud-toolkit

使用该插件将微服务做成镜像，发布到指定服务器运行对应的容器。（前提是已经有了一个可以运行的 springboot 项目）。

插件安装完毕后，在下边工具类 `Alibaba Cloud View` 添加一个用来部署服务的 host 使用测试连接验证。

![](https://blog.lichenghao.cn/upload/2022/07/27134911.png)

新建一个部署

<!-- panels:start  -->

<!-- div:left-panel -->

通过工具栏新增一个部署

![](https://blog.lichenghao.cn/upload/2022/07/27135357.png)

<!-- div:right-panel -->

或者在 run 中新增一个也可以

![](https://blog.lichenghao.cn/upload/2022/07/27135638.png)

<!-- panels:end -->

File : 选择 maven，打包后上传。

Target host ：需要填写部署的目的服务器，也就是我们上边新增的 host 。

Target Directory：放在服务器的哪个位置。

After deploy：部署完毕后，要执行什么东西。

Before launch：mvn 打包的相关命令。命令不需要写 mvn。

示例：

![](https://blog.lichenghao.cn/upload/2022/07/27140452.png)

其中的部署脚本 deploy.sh 是中 docker 相关的如下所示：

```shell
docker build -t mytest .
docker rm -f mytest
docker run -itd --name mytest -p 8091:8091 mytest
```

对应的 Dockerfile

```dockerfile
FROM java:8
ADD deploytest-1.0-SNAPSHOT.jar app.jar
EXPOSE 8091
RUN bash -c 'touch /app.jar'
# 指定docker容器启动时运行jar包
ENTRYPOINT ["java", "-jar","/app.jar"]
```

需要提前把 deploy.sh 和 Dockerfile 放在指定目录下。别忘了给 deploy.sh 授权。

```shell
[root@lch test]# ll
-rwxr-xr-x. 1 root root       94 3月  27 13:36 deploy.sh
-rw-r--r--. 1 root root      173 3月  27 13:36 Dockerfile
```

验证：

点击右上角运行我们的部署，结果如下所示则表示，服务打成镜像，正确运行容器。

![](https://blog.lichenghao.cn/upload/2022/07/27141040.png)



### 注意事项

!>如果是 maven  聚合项目，需要注意下 mvn 命令的工作空间！

![](https://blog.lichenghao.cn/upload/2022/07/27141449.png)