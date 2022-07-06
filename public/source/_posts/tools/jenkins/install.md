---
title: Jenkins
categories: jenkins
tags:
  - jenkins
keywords: jenkins
abbrlink: 9a7f448e
date: 2022-05-26 12:50:00
---
jenkins

官网：https://www.jenkins.io/

Github：https://github.com/jenkinsci/jenkins

其他：http://www.jenkins.org.cn/



## 安装 JDK

省略....

## 安装 Maven

下载 maven

https://maven.apache.org/download.cgi

https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/

配置环境变量

```properties
export MAVEN_HOME=/usr/local/apache-maven-3.6.3
export PATH=${PATH}:${MAVEN_HOME}/bin
```

验证

```bash
[root@lch ~]#  mvn -v
```

## 安装 Git

下载 git

https://mirrors.edge.kernel.org/pub/software/scm/git/

移除旧版本

```bash
[root@lch ~]sudo yum remove git
[root@lch ~]sudo yum remove git-*
```

安装编译工具

```bash
--安装gcc
yum install gcc

--安装g++
yum install gcc-c++

--安装编译所需的包
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
yum install gcc perl-ExtUtils-MakeMaker
```

编译后安装

```bash
./configure --prefix=/opt/xxx/xx  # 自定义安装目录
make && make install  
```

配置环境变量

```bash
export GIT_HOME=/usr/local/git 
export PATH=$PATH:$GIT_HOME/bin
```

验证

```bash
[root@lch ~]# git --version
git version 1.8.3.1
```



## 安装jenkins

下载 war 包，

https://www.jenkins.io/download/#stable-lts

上传到服务器，编写启动脚本 start.sh

```sh
nohup java -jar jenkins.war --httpPort=8000 &
```

查看密码：

```bash
[root@lch secrets]# cat /root/.jenkins/secrets/initialAdminPassword 
015d83c548274fb49d6fb564453fe35b
```

![](https://blog.lichenghao.cn/upload/2022/07/31182422.png)

然后安装推荐的插件.......



这样执行完毕后，会在当前用户下新建一个隐藏的.jenkis 文件夹作为工作目录。

除此之外，既然是 war 包就可以使用 tomcat来部署，下载 tomcat .......

将 jenkins.war 放在在 webapp/ROOT下（将 ROOT 下的文件删除了即可）

解压 war包

```bash
[root@lch ROOT]# jar -xvf jenkins.war 
```

修改 tomcat 端口为 9000 启动服务即可。然后继续查看密码，配置等等。

