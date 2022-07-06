---
title: Docker-Dockerfile
categories: docker
tags:
  - docker
  - Dockerfile
keywords: 'docker,Dockerfile'
abbrlink: e3a7e431
date: 2022-05-25 12:30:00
---
## Docker commit

在使用 Dockerfile构建镜像前，Docker还提供了一个命令可以构建镜像 `docker commit`

```bash
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```

先运行个 nginx 或者 tomcat 都可以 

```bash
[root@lch ~]# docker run -d --name myNginx -p 80:80 nginx
```

启动后访问就能看到熟悉的

```properties
Welcome to nginx!
```

然后进入容器，修改下 nginx 默认页面的内容

```bash
[root@lch ~]# docker exec -it myNginx /bin/bash
root@83312c28e42e:/# echo '<h1>Hello World</h2>'>/usr/share/nginx/html/index.html 
root@83312c28e42e:/# exit
exit
```

再次访问页面就变成了我们的 `Hello World` 

![](https://blog.lichenghao.cn/upload/2022/07/02094123.png)

然后 commit 镜像为自定义镜像

```bash
[root@lch ~]# docker commit --author "chenghao.li" myNginx nginx:v1.1
sha256:3a47715011129ccea78dbfaf2c37b1468dcf887dafafba0a845c2be915a96751
```

然后就可以使用啦！

!>使用 `docker commit` 意味着所有对镜像的操作都是黑箱操作，生成的镜像也被称为 **黑箱镜像**，这种黑箱镜像的维护工作是非常痛苦的，因为不知道里面修改了啥东西，所以不建议使用，推荐使用 Dockerfile。



## Dockerfile

官网文档：https://docs.docker.com/engine/reference/builder/

Dockerfile是一个文本文档，其中包含用户可以在命令行上调用以组装图像的所有命令。说白了就是一堆执行的命令。

先看一个 Dockerfile长什么样子 （DockerHub 上搜索 ubuntu ，官网的 Dockerfile ）

```properties
FROM scratch
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /

# a few minor docker-specific tweaks
# see https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap
RUN set -xe \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L40-L48
	&& echo '#!/bin/sh' > /usr/sbin/policy-rc.d \
	&& echo 'exit 101' >> /usr/sbin/policy-rc.d \
	&& chmod +x /usr/sbin/policy-rc.d \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L54-L56
	&& dpkg-divert --local --rename --add /sbin/initctl \
	&& cp -a /usr/sbin/policy-rc.d /sbin/initctl \
	&& sed -i 's/^exit.*/exit 0/' /sbin/initctl \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L71-L78
	&& echo 'force-unsafe-io' > /etc/dpkg/dpkg.cfg.d/docker-apt-speedup \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L85-L105
	&& echo 'DPkg::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' > /etc/apt/apt.conf.d/docker-clean \
	&& echo 'APT::Update::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' >> /etc/apt/apt.conf.d/docker-clean \
	&& echo 'Dir::Cache::pkgcache ""; Dir::Cache::srcpkgcache "";' >> /etc/apt/apt.conf.d/docker-clean \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L109-L115
	&& echo 'Acquire::Languages "none";' > /etc/apt/apt.conf.d/docker-no-languages \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L118-L130
	&& echo 'Acquire::GzipIndexes "true"; Acquire::CompressionTypes::Order:: "gz";' > /etc/apt/apt.conf.d/docker-gzip-indexes \
	\
# https://github.com/docker/docker/blob/9a9fc01af8fb5d98b8eec0740716226fadb3735c/contrib/mkimage/debootstrap#L134-L151
	&& echo 'Apt::AutoRemove::SuggestsImportant "false";' > /etc/apt/apt.conf.d/docker-autoremove-suggests

# delete all the apt list files since they're big and get stale quickly
RUN rm -rf /var/lib/apt/lists/*
# this forces "apt-get update" in dependent images, which is also good
# (see also https://bugs.launchpad.net/cloud-images/+bug/1699913)

# make systemd-detect-virt return "docker"
# See: https://github.com/systemd/systemd/blob/aa0c34279ee40bce2f9681b496922dedbadfca19/src/basic/virt.c#L434
RUN mkdir -p /run/systemd && echo 'docker' > /run/systemd/container

CMD ["/bin/bash"]
```

### 入门构建

构建个简单的 Dockerfile，来构建上面修改的 nginx 镜像。

```bash
[root@lch ~]# mkdir myNginx
[root@lch ~]# cd myNginx/
[root@lch ~]# vim Dockerfile
```

Dockerfile  文件内容如下所示：

```Dockerfile
FROM nginx
RUN echo echo '<h1>Hello World V2</h2>'>/usr/share/nginx/html/index.html
```

执行镜像构建

```bash
[root@lch myNginx]# docker build -t mynginx:v2 .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM nginx
 ---> 0e901e68141f
Step 2/2 : RUN echo echo '<h1>Hello World V2</h2>'>/usr/share/nginx/html/index.html
 ---> Running in 8cefa7c01113
Removing intermediate container 8cefa7c01113
 ---> a3b675c0582b
Successfully built a3b675c0582b
Successfully tagged mynginx:v2

```

查看下自定义镜像

```bash
[root@lch myNginx]# docker images
REPOSITORY                                                   TAG              IMAGE ID       CREATED          SIZE
mynginx                                                      v2               a3b675c0582b   26 seconds ago   142MB

```

运行下自定义镜像 （不要介意，多打了个 echo  - -!）

```bash
[root@lch myNginx]# docker run -d --name myNginx -p 80:80 mynginx:v2
a1efe44cc9cd49d2459e389a1856db3974c986d61bd4488cc3eae54b3e95bdc1
```

![](https://blog.lichenghao.cn/upload/2022/07/02101543.png)



### 构建指令

| 指令           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| ARG            | 唯一能放在 FROM 上面的指令，可以用来声明一个常量             |
| FROM           | FROM 指定的内容为镜像的基础镜像                              |
| RUN            | 用于执行命令和提交执行的结果                                 |
| CMD            | 执行命令，例如执行一个 sh 脚本                               |
| LABEL          | 给镜像打标签🏷                                                |
| ~~MAINTAINER~~ | deprecated                                                   |
| EXPOSE         | EXPOSE指令通知Docker容器在运行时侦听指定的网络端口           |
| ENV            | 设置容器环境变量                                             |
| ADD            | 复制新文件、目录或远程文件URL，并将它们添加到映像文件系统中  |
| COPY           | 和 ADD 差不多，但是只能拷贝宿主机的文件，不能拷贝远程的文件  |
| ENTRYPOINT     | 入口点允许您配置将作为可执行文件运行的容器                   |
| VOLUME         | 目录挂载                                                     |
| USER           | 用户指令设置运行映像时要使用的用户名（或UID）和用户组（或GID） |
| WORKDIR        | WORKDIR指令为Dockerfile中的任何RUN、CMD、ENTRYPOINT、COPY和ADD指令设置工作目录 |
| SHELL          |                                                              |



#### FROM

FROM 是 Dockerfile 的必备命令，用来指定基础镜像。

指令格式：

```dockerfile
FROM [--platform=<platform>] <image> [AS <name>]
或者
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
或者
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

示例 (nginx镜像的开头部分)

```dockerfile
FROM debian:bullseye-slim

LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"

ENV NGINX_VERSION   1.21.6
ENV NJS_VERSION     0.7.3
ENV PKG_RELEASE     1~bullseye
......
```

官方给出了一些常用的基础镜像

- 应用镜像，如 `nginx`、`redis`、`mongo`、`mysql`、`httpd`、`php`、`tomcat` 等。

- 语言镜像：方便开发、构建、运行各种语言应用的编程语言镜像，如 `node`、`oraclejdk`，`openjdk`、`python`、`ruby`、`golang` 等。

- 系统镜像：更为基础的操作系统镜像，如 `ubuntu`、`debian`、`centos`、`fedora`、`alpine` 等，这些操作系统的软件库为我们提供了更广阔的扩展空间。

- 空白镜像：`scratch` 这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。比如 ubuntu的 Dockerfile 

  - ```dockerfile
    FROM scratch
    ADD ubuntu-bionic-oci-amd64-root.tar.gz /
    CMD ["bash"]
    ```

#### RUN

RUN 指令将在当前镜像上加新的一层，可执行任何命令和提交结果，生成的一层镜像将用于 Dockfile 中的后续步骤。

它有两种形式：

- `RUN <command>`    linux下默认使用 `/bin/sh -c`  window下使用`cmd /S /C`
- `RUN ["executable", "param1", "param2"]`  必须使用双引号，不能使用单引号

示例

```dockerfile
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
 
RUN ["/bin/bash", "-c", "echo hello"]
```

如果使用不同的 shell 的话，可以将如下定义

```dockerfile
RUN ["/bin/bash", "-c", "echo hello"]
```



#### CMD

CMD指令运行一个可执行的文件并提供参数，可以为ENTRYPOINT指定参数。

有三种命令格式：

- `CMD ["executable","param1","param2"]`  首选模式，运行一个可执行脚本，并提供参数
- `CMD ["param1","param2"]`  可以为 ENTRYPOINT 提供参数
- `CMD command param1 param2`   类似 "/bin/sh -c" 的方法执行的命令。

!>在一个 Dockerfile文件中，如果有多个 CMD 命令，只有最后一个生效！



假如启动容器，调用开源的接口打印 "一句话" 那么该 Dockerfile可以如下所示：

```dockerfile
FROM ubuntu:18.04
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
CMD ["curl","-s","https://v1.hitokoto.cn/?encode=text"]
```

构建镜像 myword

```bash
[root@lch myCmdTest2]# docker build -t myword .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM ubuntu:18.04
 ---> c6ad7e71ba7d
Step 2/3 : RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
 ---> Using cache
 ---> 06d160bc6260
Step 3/3 : CMD ["curl","-s","https://v1.hitokoto.cn/?encode=text"]
 ---> Running in fca440bc07b1
Removing intermediate container fca440bc07b1
 ---> 5b7a09b3267e
Successfully built 5b7a09b3267e
Successfully tagged myword:latest
```

执行下对应的容器

```bash
[root@lch myCmdTest2]# docker run myword
我的辫子长在头上，诸君的辫子长在心里。
```

!>如果启动容器指定了参数，那么会覆盖到 CMD 中的参数，如下所示，给了 /bin/bash参数，那么 CMD 的命令就失效了。

```bash
[root@lch myCmdTest2]# docker run myword /bin/bash
[root@lch myCmdTest2]# 
```

上面的 CMD 命令其实就是我们调用 curl 去请求一个接口然后打印了得到的结果。如果莫一天我换接口了或者我想打印其他类型的一句话，需要去修改对应的 Dockerfile 然后重新发布镜像。这样做没问题就是比较繁琐，那么可不可以更简便些？比如使用下面的 ENTRYPOINT。

#### ENTRYPOINT

进入点，可以让你的容器功能表现得像一个可执行程序一样。

它也有两种命令格式：

- ENTRYPOINT ["executable", "param1", "param2"]  
- ENTRYPOINT command param1 param2

我们把上面 CMD 中的 Dockerfile 修改下：

```dockerfile
FROM ubuntu:18.04
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
ENTRYPOINT ["curl"]
```

构建成 myword:v2.0

```bash
[root@lch myCmdTest2]# docker build -t myword:v2.0 .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM ubuntu:18.04
 ---> c6ad7e71ba7d
Step 2/3 : RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
 ---> Using cache
 ---> 06d160bc6260
Step 3/3 : ENTRYPOINT ["curl"]
 ---> Running in a08e46eebcdc
Removing intermediate container a08e46eebcdc
 ---> 680778f03581
Successfully built 680778f03581
Successfully tagged myword:v2.0
```

再次运行新的镜像，这次我们需要手动提供参数。

```bash
[root@lch myCmdTest2]# docker run myword:v2.0 -s https://v1.hitokoto.cn/?encode=text
红茶的温度和女人心在任何时代都是难以琢磨呢。

# 展示不同类型的一句好
[root@lch myCmdTest2]# docker run myword:v2.0 -s https://v1.hitokoto.cn/?encode=text&c=c
人生充斥着谎言,我又岂能独善其身!
```



我们上面说过，CMD 可以为 ENTRYPOINT 提供参数，那么我们可以再次修改下：

```dockerfile
FROM ubuntu:18.04
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
ENTRYPOINT ["curl","-s"]
CMD ["https://v1.hitokoto.cn/?encode=text"]
```

> 把通用的参数放在 ENTRYPOINT 中，把可变的参数放在 CMD 中。

重新构建镜像 myword:v3.0

```bash
[root@lch myCmdTest2]# docker build -t myword:v3.0 .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM ubuntu:18.04
 ---> c6ad7e71ba7d
Step 2/4 : RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
 ---> Using cache
 ---> 06d160bc6260
Step 3/4 : ENTRYPOINT ["curl","-s"]
 ---> Running in ccaa8eba21a6
Removing intermediate container ccaa8eba21a6
 ---> bd844726645e
Step 4/4 : CMD ["https://v1.hitokoto.cn/?encode=text"]
 ---> Running in a4386f93735a
Removing intermediate container a4386f93735a
 ---> cacad0b7a90e
Successfully built cacad0b7a90e
Successfully tagged myword:v3.0
```

运行个容器测试下，我们可以在docker run 中指定参数覆盖掉 CMD 中的参数。

```bash
[root@lch myCmdTest2]# docker run myword:v3.0
如果我贏了，你就是我的人了

[root@lch myCmdTest2]# docker run myword:v3.0 https://v1.hitokoto.cn/?encode=text&c=c
若是我笑得皮开又肉绽，就莫问，离合悲欢。
```



#### EXPOSE

EXPOSE 指令通知 Docker 容器在运行时侦听指定的网络端口。

例如下所示：（如果不指定协议，默认是 TCP 协议）

```dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
```

指定端口后，我们在运行的容器的时候就可以和宿主机做个端口映射，

```bash
docker run -p 80:80/tcp -p 80:80/udp ...
```

例如我们运行下 nginx (默认暴露的是 80 端口)，我们用 8080 端口做个映射。

```bash
[root@lch ~]# docker run -itd --name mynginx -p 8080:80 nginx
```

![](https://blog.lichenghao.cn/upload/2022/07/04160851.png)



#### ENV

设置环境变量

```dockerfile
ENV <key>=<value> ...
```

例如：

```dockerfile
ENV MY_NAME="John Doe"
ENV MY_DOG=Rex\ The\ Dog
ENV MY_CAT=fluffy
```

这个环境变量是构建镜像过程中的环境变量，可以使用在当前镜像层以及后序的镜像层。但是环境变量不能被 CMD 命令使用。

比如我们构建一个镜像，在文件中写入 "hello world" 。Dockerfile 如下所示：

```dockerfile
FROM ubuntu:18.04
ENV myDir=/root/mydir
RUN mkdir $myDir 
RUN echo "hello world" > $myDir/hello.txt
```

构建镜像

```bash
[root@lch myEnvTest]# docker build -t myenv .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM ubuntu:18.04
 ---> c6ad7e71ba7d
Step 2/4 : ENV myDir=/root/mydir
 ---> Running in b9928b614c68
Removing intermediate container b9928b614c68
 ---> ce66f4954e16
Step 3/4 : RUN mkdir $myDir
 ---> Running in 15a0a0970b6a
Removing intermediate container 15a0a0970b6a
 ---> f82d4a30ac4d
Step 4/4 : RUN echo "hello world" > $myDir/hello.txt
 ---> Running in 126074a17a94
Removing intermediate container 126074a17a94
 ---> 71bf39f9c42f
Successfully built 71bf39f9c42f
Successfully tagged myenv:latest
```

运行容器查看我们的环境变量是否生效了

```bash
[root@lch myEnvTest]# docker run -it myenv /bin/bash
root@f45d89e5e313:/# cd /root/mydir/
root@f45d89e5e313:~/mydir# ls
hello.txt
root@f45d89e5e313:~/mydir# cat hello.txt 
hello world
```



#### ADD

添加文件到容器的指定位置，这里的文件可以是本地的文件，也可以是远程的某个文件；

命令格式：

```dockerfile
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

示例

```dockerfile
# 添加 hom 开头的文件到 /mydir 目录下
ADD hom* /mydir/

# 支持正则表达式
# 添加文件，这个文件可以是 home.txt、homm.txt、hom1.txt ...
ADD hom?.txt /mydir/

# 添加远程文件到指定目录
ADD https://archive.apache.org/dist/XX /mydir
```

如果添加的文件想要给其他用户（非 root）的话，那么参数中的 chown 就派上用场了

```dockerfile
# 指定用户和用户组
COPY --chown=myuser:mygroup /dir /container/dir
```

示例：做个镜像，拷贝远程文件到指定目录

```dockerfile
FROM ubuntu:18.04
RUN  adduser myuser
ADD  --chown=myuser:myuser https://repo1.maven.org/maven2/ch/qos/logback/logback-classic/1.2.10/logback-classic-1.2.10.jar  /usr/local
```

构建镜像

```bash
[root@lch myAddTest]# docker build -t myadd .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM ubuntu:18.04
 ---> c6ad7e71ba7d
Step 2/3 : RUN  adduser myuser
 ---> Running in 957afffb068a
Adding user `myuser' ...
Adding new group `myuser' (1000) ...
Adding new user `myuser' (1000) with group `myuser' ...
Creating home directory `/home/myuser' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: Retype new UNIX password: passwd: Authentication token manipulation error
passwd: password unchanged
Use of uninitialized value $answer in chop at /usr/sbin/adduser line 591.
Use of uninitialized value $answer in pattern match (m//) at /usr/sbin/adduser line 592.
Try again? [y/N] Changing the user information for myuser
Enter the new value, or press ENTER for the default
        Full Name []:   Room Number []:         Work Phone []:  Home Phone []:  Other []: Use of uninitialized value $answer in chop at /usr/sbin/adduser line 621.
Use of uninitialized value $answer in pattern match (m//) at /usr/sbin/adduser line 622.
Is the information correct? [Y/n] Removing intermediate container 957afffb068a
 ---> c104e1b8881a
Step 3/3 : ADD  --chown=myuser:myuser https://repo1.maven.org/maven2/ch/qos/logback/logback-classic/1.2.10/logback-classic-1.2.10.jar  /usr/local
Downloading [==================================================>]  233.8kB/233.8kB

 ---> 32436a08a136
Successfully built 32436a08a136
Successfully tagged myadd:latest

```

运行容器查看结果，文件已经拷贝过去，并且属于我们创建的用户。

```bash
[root@lch myAddTest]# docker run -it myadd /bin/bash
root@673d8c98fe64:/# cd /usr/local/
root@673d8c98fe64:/usr/local# ll
total 232
....
-rw-------. 1 myuser myuser 233809 logback-classic-1.2.10.jar
....
root@673d8c98fe64:/usr/local# 
```

#### COPY

copy 可以理解为简单版本的 ADD 。它和 ADD 的区别在于，COPY 不能使用远程地址，如果使用了远程地址会报错。

```bash
COPY failed: source can't be a URL for COPY
```

#### VOLUME

容器运行应该尽量保证容器存储层不发生读写，对于需要保存的数据，应该保存在 volume （目录卷）中。在 Dockerfile 中通过 VOLUME 指令，指定某些目录挂载为匿名卷，这样用户即便 docker run 不挂载目录卷也能正常运行容器。Docker 会在指定目录下帮你映射一个目录，比如：/var/lib/docker/volumes (不同版本的 Docker可能位置不同)，以容器运行ID 区分开。可以通过命令查看，也可以直接进入目录查看。

```bash
[root@lch ~]# docker volume ls
DRIVER    VOLUME NAME
local     3b0845329035cb1d0a485cb6b541bbdcd5c4c9d9a2257833ca1ab8e530815a2f
local     7dce6f8faff9ba4e14a308d776c08a36166dfe71fda664193dae9ac904895f64
```

所以，Dockerfile中的命令VOLUME，只会在容器的指定位置创建目录，docker 会默认帮你挂载一个目录在

`/var/lib/docker/volumes/{volumeId}` 。这样做的好处是：比如你用容器启动了一个 mysql，但是你运行容器的时候没指定目录映射。如果你把容器删除了，那么里面存储的数据可能就丢失了。mysql 的 Dockerfile一定通过VOLUME指定了匿名卷，这样你就可以去指定目录下找回删除掉的数据。

那么如果有数据需要存储的话，还是建议做目录挂载哦，`docker run -v /xxx:/xxx`

命令格式也很简单：

```dockerfile
VOLUME ["/data"]
```

测试下，匿名挂载的话，在指定目录是否有共享的文件。

Dockerfile

```dockerfile
FROM ubuntu:18.04
RUN mkdir /volume 
RUN echo "hello world" > /volume/hello.txt
VOLUME /volume
```

构建镜像，运行容器测试，在指定 volume 下的_data目录下，有我们映射的文件。

```bash
[root@lch volumes]# cd 890631a6f79b5a4f14e97ccf7ac29524008fbf30dfd8eed01df9b01ffa6c88fc/
[root@lch 890631a6f79b5a4f14e97ccf7ac29524008fbf30dfd8eed01df9b01ffa6c88fc]# ll
总用量 0
drwxr-xr-x. 2 root root 23 6月   5 13:08 _data
[root@lch 890631a6f79b5a4f14e97ccf7ac29524008fbf30dfd8eed01df9b01ffa6c88fc]# cd _data/
[root@lch _data]# ll
总用量 4
-rw-r--r--. 1 root root 12 6月   5 11:34 hello.txt
[root@lch _data]# cat hello.txt 
hello world
[root@lch _data]# 
```





