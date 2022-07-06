---
title: Docker 安装及常用命令
categories: docker
tags:
  - docker
keywords: docker
abbrlink: 3607b27a
date: 2022-05-25 12:30:00
---
Docker

官网：https://www.docker.com/

文档：https://docs.docker.com/



## 安装

安装文档：https://docs.docker.com/engine/install/centos/

卸载历史

```bash
[root@lch ~]# sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

设置 yum

```bash
[root@lch ~]# sudo yum install -y yum-utils
[root@lch ~]# sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

安装

```bash
[root@lch ~]# sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin

或者 指定版本安装
[root@lch ~]# sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io docker-compose-plugin
```

测试安装

```bash
[root@lch ~]# sudo docker run hello-world
```

## 安装私有镜像仓库

私有镜像仓库，参考文档：https://mp.weixin.qq.com/s/3X6vVdWmjmWCyiLm35jpVw

下载镜像

```bash
docker pull registry:2
```

使用Docker容器运行Registry服务，需要添加环境变量`REGISTRY_STORAGE_DELETE_ENABLED=true`开启删除镜像的功能；修改Docker Daemon的配置文件，文件位置为`/etc/docker/daemon.json`，由于Docker默认使用HTTPS推送镜像，而我们的镜像仓库没有支持，所以需要添加如下配置，改为使用HTTP推送；

```properties
{
  "insecure-registries": ["192.168.1.240:5000"]
}
```

重启下 docker

```bash
systemctl daemon-reload && systemctl restart docker
```

开启注册服务

```properties
docker run -p 5000:5000 --name registry2 \
--restart=always \
-e REGISTRY_STORAGE_DELETE_ENABLED="true" \
-d registry:2
```

镜像仓库可视化

下载`docker-registry-ui`的Docker镜像；

```bash
docker pull joxit/docker-registry-ui:static
```

运行`docker-registry-ui`服务；

```bash
docker run -p 8280:80 --name registry-ui \
--link registry2:registry2 \
-e REGISTRY_URL="http://registry2:5000" \
-e DELETE_IMAGES="true" \
-e REGISTRY_TITLE="Registry2" \
-d joxit/docker-registry-ui:static
```

下载个测试镜像

```bash
[root@lch ~]# docker pull hello-world
```

打上私有仓库的标签

```bash
[root@lch ~]# docker tag hello-world 192.168.1.240:5000/hello-world:v1.0
```

推送到私有镜像仓库

```bash
[root@lch ~]# docker push 192.168.1.240:5000/hello-world:v1.0
The push refers to repository [192.168.1.240:5000/hello-world]
e07ee1baac5f: Pushed 
v1.0: digest: sha256:f54a58bc1aac5ea1a25d796ae155dc228b3f0e11d046ae276b39c4bf2f13d8c4 size: 525
```

![](https://blog.lichenghao.cn/upload/2022/07/01085816.png)



## 常用命令

### 镜像命令

查看相关：

```bash
[root@lch ~]# docker images

REPOSITORY                       TAG       IMAGE ID       CREATED             SIZE
app                              v1.0      d45bee24a299   About an hour ago   839MB
registry                         2         773dbf02e42e   5 days ago          24.1MB
192.168.1.240:5000/hello-world   v1.0      feb5d9fea6a5   8 months ago        13.3kB

# 所有镜像，包括中间镜像
[root@lch ~]# docker images -a  

# 只显示镜像 ID
[root@lch ~]# docker images -q

# 显示镜像的摘要信息
[root@lch ~]# docker images --digests 

# 显示镜像的完整信息
[root@lch ~]# docker images --no-trunc
REPOSITORY                       TAG       IMAGE ID                                                                  CREATED             SIZE
app                              v1.0      sha256:d45bee24a2994afb9032315c5f006dd6a3ddce304a3306726395ea274e7ba044   About an hour ago   839MB
```

搜索相关：

```bash
# 查询镜像 
[root@lch ~]# docker search hello-world
NAME                                       DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
hello-world                                Hello World! (an example of minimal Dockeriz…   1754      [OK]
kitematic/hello-world-nginx                A light-weight nginx container that demonstr…   151
```



下载镜像

docker pull xxx:tag  不指定 tag就下载最新的。

```bash
[root@lch ~]# docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
Digest: sha256:80f31da1ac7b312ba29d65080fddf797dd76acfb870e677f390d5acba9741b17
Status: Image is up to date for hello-world:latest
docker.io/library/hello-world:latest
```

删除镜像

```bash
# docker rmi XXX （镜像名字或者唯一镜像ID)
[root@lch ~]# docker rmi hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:80f31da1ac7b312ba29d65080fddf797dd76acfb870e677f390d5acba9741b17

# 强制删除（如果镜像正在运行）
[root@lch ~]# docker rmi -f XXX 

# 删除多个
[root@lch ~]# docker rmi -f hello-world hello

# 删除全部镜像
[root@lch ~]# docker rmi -f $(docker images -qa)
```

### 容器命令

#### 查看容器

```bash
# 查看运行的容器
[root@lch ~]# docker ps 
CONTAINER ID   IMAGE                             COMMAND                  CREATED        STATUS        PORTS                                       NAMES
2a4040d2b800   joxit/docker-registry-ui:static   "/docker-entrypoint.…"   15 hours ago   Up 15 hours   0.0.0.0:8280->80/tcp, :::8280->80/tcp       registry-ui
487bc206b516   registry:2                        "/entrypoint.sh /etc…"   15 hours ago   Up 15 hours   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   registry2

# 查看所有的容器
[root@lch ~]# docker ps -a

# 查看最近运行的容器
[root@lch ~]# docker ps -a -l
```



#### 启动容器

```bash
# 以ubuntu 镜像启动一个容器，参数为以命令行模式进入该容器：
[root@lch ~]# docker pull ubuntu
[root@lch ~]# docker run -it ubuntu /bin/bash
root@c8dfa687c0cd:/#  

# 启动容器并且指定一个名字
[root@lch ~]# docker run -it --name u2  ubuntu /bin/bash
root@fb3261067505:/# 
```

- **-i**: 交互式操作。
- **-t**: 终端。
- **ubuntu**: ubuntu 镜像。
- **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。
- exit: 退出容器



#### 启动已停止的容器

```bash
# 查询出容器的列表
[root@lch ~]# docker ps -a
# 启动停止的容器
[root@lch ~]# docker start c8dfa687c0cd 
```

#### 停止一个运行的容器

```bash
[root@lch ~]# docker stop c8dfa687c0cd 
c8dfa687c0cd
```

#### 重启一个容器

```bash
[root@lch ~]# docker restart c8dfa687c0cd 
```

#### 后台运行一个容器

```bash
[root@lch ~]# docker run -itd --name ubuntu ubuntu /bin/bash
5f9d7d938e833ec82acc4c0cb832412b93007411b0933c900d322312c70aa4d5
```

#### 进入一个后台运行中的容器

- **docker attach**
- **docker exec**：推荐使用 docker exec 命令，此命令退出容器终端，不会导致容器停止。



attach

```bash
# 退出后，容器停止
[root@lch ~]# docker attach 5f9d7d938e83 
root@5f9d7d938e83:/# exit
exit

[root@lch ~]# docker ps -a
CONTAINER ID   IMAGE                             COMMAND                  CREATED          STATUS                       PORTS                                       NAMES
5f9d7d938e83   ubuntu                            "/bin/bash"              3 minutes ago    Exited (0) 20 seconds ago  
```

exec

```bash
# 后台启动容器
[root@lch ~]# docker run -itd ubuntu /bin/bash
fddc7d0333a54eca6adce3495c97d1468a65210aaeafaf2efda66911a93d28c3
[root@lch ~]# docker ps -a -l
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
fddc7d0333a5   ubuntu    "/bin/bash"   24 seconds ago   Up 22 seconds             hopeful_volhard
# 进入容器
[root@lch ~]# docker exec -it fddc7d0333a5 /bin/bash
root@fddc7d0333a5:/# exit
exit
# 退出容器后，容器依然在运行
[root@lch ~]# docker ps -a -l
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
fddc7d0333a5   ubuntu    "/bin/bash"   49 seconds ago   Up 48 seconds             hopeful_volhard

```



#### 运行容器，映射端口

- **-P:** 随机端口映射，容器内部端口**随机**映射到主机的端口
- **-p:** 指定端口映射，格式为：**主机(宿主)端口:容器端口**

```bash
[root@lch ~]# docker run -itd --name mynginx -p 8080:80 nginx
349b6b4e2ed53ced7771492f8634f94fa717f829689eeeddb20bc0b9b41dacd6
```

使用大 P，随机端口映射。

```bash
[root@lch ~]# docker run -itd --name mynginx -P nginx
d4bb332de7e4384d4568bc1692261bf11bdefab9f1f3c40c85bd9bbabb7481fb

[root@lch ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                     NAMES
d4bb332de7e4   nginx     "/docker-entrypoint.…"   21 seconds ago   Up 20 seconds   0.0.0.0:49153->80/tcp, :::49153->80/tcp   mynginx
```



#### 运行容器，目录映射

```bash
[root@lch ~]# docker run -itd --name mynginx -p 8080:80 --privileged=true -v /root/html:/usr/share/nginx/html nginx 
```

> --privileged=true 关闭安全权限，否则你容器操作文件夹可能没有权限。

然后访问 ip:8080就会看到 nginx 的欢迎页。这个时候我们在宿主机中html目录下新增 index.html 内容为 hello world 。替换默认的欢迎页。

```bash
[root@lch html]# echo "hello world" > index.html
```

然后刷新网页，就会发现欢迎页变成了 `hello world`。

#### 删除容器

```bash
# docker rm 容器ID(或者名字)  只能删除未运行的容器
[root@lch ~]# docker rm fddc7d0333a5 
Error response from daemon: You cannot remove a running container fddc7d0333a54eca6adce3495c97d1468a65210aaeafaf2efda66911a93d28c3. Stop the container before attempting removal or force remove

# docker rm -f 容器ID(或者名字)  强制删除
[root@lch ~]# docker rm -f  fddc7d0333a5 
fddc7d0333a5

# 删除多个容器
[root@lch ~]# docker rm -f 5f5965c92591 60f0738a8325 
5f5965c92591
60f0738a8325

# 批量删除所有的容器
[root@lch ~]# docker rm -f $(docker ps -aq)

# 批量删除状态为 exited 容器
[root@lch ~]# docker rm $(sudo docker ps -qf status=exited)

# 例如删除包含"some"的镜像
[root@lch ~]# docker rmi -f $(docker images | grep some | awk '{print $3}')
```

> 通常给批量删除写个脚本，参考： [批量删除 Docker 容器](tool/linux/shell.md?id=批量删除-docker-容器)



#### 查看容器日志

查看最近的 n 条日志

`docker logs --tail=n 容器ID/容器名称`

```bash
# 查看最近的 1000 条日志
[root@lch digitalTwins]# docker logs --tail=1000 model-server
```

查看实时日志

`docker logs -f 容器ID/容器名称 `

```bash
[root@lch digitalTwins]# docker logs -f model-server
```





