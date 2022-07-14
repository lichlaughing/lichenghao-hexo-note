---
title: Jenkins 使用常见问题
tags:
  - jenkins
  - faq
categories: 
  - tools
  - jenkins
keywords: jenkins
cover: 'https://blog.lichenghao.cn/upload/2022/07/faq.png'
abbrlink: e1c81179
date: 2022-07-12 14:54:46
---



## 构建 maven 项目报错 Failed to parse POMs

构建 maven 项目的时候 maven 具体报错如下：

```properties
Parsing POMs
ERROR: Failed to parse POMs
hudson.maven.MavenEmbedderException: 1 problem was encountered while building the effective settings
[FATAL] Non-readable settings /opt/module/apache-maven-3.6.3: /opt/module/apache-maven-3.6.3 (是一个目录) @ /opt/module/apache-maven-3.6.3

	at hudson.maven.MavenEmbedder.<init>(MavenEmbedder.java:129)
	at hudson.maven.MavenEmbedder.<init>(MavenEmbedder.java:110)
	at hudson.maven.MavenEmbedder.<init>(MavenEmbedder.java:137)
	at hudson.maven.MavenUtil.createEmbedder(MavenUtil.java:211)
	at hudson.maven.MavenModuleSetBuild$PomParser.invoke(MavenModuleSetBuild.java:1324)
	at hudson.maven.MavenModuleSetBuild$PomParser.invoke(MavenModuleSetBuild.java:1124)
	at hudson.FilePath.act(FilePath.java:1200)
	at hudson.FilePath.act(FilePath.java:1183)
	at hudson.maven.MavenModuleSetBuild$MavenModuleSetBuildExecution.parsePoms(MavenModuleSetBuild.java:985)
	at hudson.maven.MavenModuleSetBuild$MavenModuleSetBuildExecution.doRun(MavenModuleSetBuild.java:689)
	at hudson.model.AbstractBuild$AbstractBuildExecution.run(AbstractBuild.java:522)
	at hudson.model.Run.execute(Run.java:1896)
	at hudson.maven.MavenModuleSetBuild.run(MavenModuleSetBuild.java:543)
	at hudson.model.ResourceController.execute(ResourceController.java:101)
	at hudson.model.Executor.run(Executor.java:442)
```



解决：

1. 在 Global Tool Configuration 中，最上面的 Maven 配置修改为：

- 默认 settings 提供 ：Use default maven settings
- 默认全局 settings 提供：Use default maven global settings

2. 在 Global Tool Configuration 下面添加自己的 maven 配置。

3. 然后在构建中，将 Build 中的高级选项配置修改：

- Settings file ：Use default maven settings
- Global Settings file ：Use default maven global settings

4. 修改完毕即可。

