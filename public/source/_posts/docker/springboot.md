---
title: Docker Springboot 打包成镜像
categories: docker
tags:
  - docker
  - springboot
keywords: docker
abbrlink: dfef8849
date: 2022-05-25 12:30:00
---
## Springboot 打包成镜像发布到镜像仓库

（私有镜像仓库见安装章节）

使用打包插件 docker-maven-plugin

```xml
 <build>
        <finalName>${project.name}-${project.version}</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>1.5.3.RELEASE</version>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>io.fabric8</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.33.0</version>
                <executions>
                    <!--如果想在项目打包时构建镜像添加-->
                    <execution>
                        <id>build-image</id>
                        <phase>package</phase>
                        <goals>
                            <goal>build</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <!-- Docker 远程管理地址-->
                    <dockerHost>http://192.168.1.240:2375</dockerHost>
                    <!-- Docker 推送镜像仓库地址-->
                    <pushRegistry>http://192.168.1.240:5000</pushRegistry>
                    <images>
                        <image>
                            <!--由于推送到私有镜像仓库，镜像名需要添加仓库地址-->
                            <name>192.168.1.240:5000/digital-twins/${project.name}:${project.version}</name>
                            <!--定义镜像构建行为-->
                            <build>
                                <!--定义基础镜像-->
                                <from>java:8</from>
                                <args>
                                    <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
                                </args>
                                <!--定义哪些文件拷贝到容器中-->
                                <assembly>
                                    <!--定义拷贝到容器的目录-->
                                    <targetDir>/</targetDir>
                                    <!--只拷贝生成的jar包-->
                                    <descriptorRef>artifact</descriptorRef>
                                </assembly>
                                <!--定义容器启动命令-->
                                <entryPoint>["java", "-jar","/${project.build.finalName}.jar"]</entryPoint>
                                <!--定义维护者-->
                                <maintainer>chenghao.li</maintainer>
                            </build>
                        </image>
                    </images>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

然后执行 mvn 的插件，docker:build ，docker:push 推送到镜像仓库。

镜像推送成功后，可以写个启动脚本。start.sh 。

```shell
sudo docker run -itd --name model-server -p 8091:8091 镜像名称
```

停止脚本 stop.sh 。

```shell
sudo docker rm -f model-server
```





