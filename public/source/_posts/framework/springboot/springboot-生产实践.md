---
title: Springboot 生产最佳实践！
tags:
  - springboot
categories: springboot
keywords: springboot 生产实践
abbrlink: a44b4238
date: 2022-07-11 11:19:45
---



## 三方库管理

项目中多多少少会用到第三方好用的包，比如 guava、hutool 通常我们会把这些通用的包放在最上面的父工程中，这样其他的模块就可以复用这些插件，但是随着项目的开发，父工程的pom文件就会变的越来越长越复杂。

```xml
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>cn.hutool</groupId>
                <artifactId>hutool-all</artifactId>
                <version>5.8.0</version>
            </dependency>
            <dependency>
                <groupId>com.google.guava</groupId>
                <artifactId>guava</artifactId>
                <version>30.0-jre</version>
            </dependency>
           	......
        </dependencies>
    </dependencyManagement>
```

而且随着插件越来越多，如果我们要维护版本的话，也能麻烦。所以希望能统一的把这些包管理起来。其实 springboot 早就想到这些了。比如我们用 spring-cloud 做微服务开发的时候，用到的 springboot 的那些依赖，我们是以 pom 的形式引入的。（而不是依次指定版本引入）这样的话我们就可以使用该版本下的所有相关依赖。

```xml
<dependencyManagement>
   <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>${spring-boot.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
     	......
	</dependencies>
</dependencyManagement>
```

我们也可以参考这个形式，将我们使用到的三方包整理到一个项目中，然后其他需要的地方引入使用即可。

**实现也很容易，如下所示：**

第一步：

新建个 maven 工程,用来维护我们的三方包，pom 如下所示：

随便写了几个三方包

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.lichenghao</groupId>
    <artifactId>third-package</artifactId>
    <version>1.0.0.RELEASE</version>

    <name>third-package</name>
    <packaging>pom</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>cn.hutool</groupId>
                <artifactId>hutool-all</artifactId>
                <version>5.8.0</version>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.10</version>
            </dependency>
            <dependency>
                <groupId>com.google.guava</groupId>
                <artifactId>guava</artifactId>
                <version>30.0-jre</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.18.24</version>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
                <version>1.7.30</version>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-core</artifactId>
                <version>1.2.3</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

第二步：

将项目 install 进本地 maven 仓库

```properties
 mvn install -Dmaven.test.skip=true  
```

然后在项目的父工程中引入这个依赖

```xml
 <dependencyManagement>
   <dependencies>
     <dependency>
       <groupId>cn.lichenghao</groupId>
       <artifactId>third-package</artifactId>
       <version>1.0.0.RELEASE</version>
       <type>pom</type>
       <scope>import</scope>
     </dependency>
   </dependencies>
</dependencyManagement>
```

第三步：

最后在子模块中就可以使用了

```xml
 <dependencies>
   <dependency>
     <groupId>com.google.guava</groupId>
     <artifactId>guava</artifactId>
   </dependency>
</dependencies>
```

> - 如果有 maven 私服的话，打包上传到私服中，这样大家就可以一起用了~
> - 引入的依赖要指定好scope

