---
title: Sonarqube7.6 gitalb集成
categories: sonarqube
tags:
  - sonarqube
  - gitlab
keywords: sonarqube
abbrlink: de32e752
date: 2022-05-26 13:50:00
cover: https://blog.lichenghao.cn/upload/2022/07/A889F46E-sonarqube.png
---
Gitlab 集成，先安装 Gitlab 。



## gitlab-runner 安装配置

Gitlab Runner 分为三种：

- 共享 Runner(`Shared runners`)
- 专享 Runner(`Specific runners`)
- 分组 Runner(`Group Runners`)

rpm 安装包下载地址：https://packages.gitlab.com/runner/gitlab-runner?filter=rpms

下载和  gitlab 版本一致的安装包。比如 ：gitlab-runner-12.9.0-1.x86_64.rpm

安装

```shell
[root@lch ~]# yum install -y git
[root@lch ~]# rpm -i gitlab-runner-12.9.0-1.x86_64.rpm 
```

执行注册

```shell
[root@lch ~]# gitlab-runner register
```

根据提示需要提供 gitlab 服务地址，gitlab-ci的Token， description（这个runner的描述信息），tag-list（标签，gilab-ci配置的时候会用到，支持多个，逗号隔开），executor（执行器，这里选shell）。这些配置后期都可以修改！

需要的地址和令牌在 gitlab 中的管理中心，Runner 中。配置完毕后，在下面就会出现刚才配置的 runner 。

![](https://blog.lichenghao.cn/upload/2022/07/30114648.png)

然后测试个流水作业：

然后在项目根路径下新增 .gitlab-ci.yml 作业配置文件，

```yaml
stages:
  - first

job1:
  stage: first
  tags: 
    - test
  only:
    - dev
  script:
    - echo "hi i am gitlab-ci"

```

提交文件后，就会触发作业执行

![](https://blog.lichenghao.cn/upload/2022/07/29154609.png)

点进入就能看到任务的详情

![](https://blog.lichenghao.cn/upload/2022/07/29155116.png)



其他命令：

| **命令**                 | **描述**                                                     |
| ------------------------ | ------------------------------------------------------------ |
| gitlab-runner run        | 运行一个runner服务                                           |
| gitlab-runner register   | 注册一个新的runner                                           |
| gitlab-runner start      | 启动服务                                                     |
| gitlab-runner stop       | 关闭服务                                                     |
| gitlab-runner restart    | 重启服务                                                     |
| gitlab-runner status     | 查看各个runner的状态                                         |
| gitlab-runner unregister | 注销掉某个runner                                             |
| gitlab-runner list       | 显示所有运行着的runner                                       |
| gitlab-runner verify     | 检查已注册的运行程序是否可以连接到GitLab，但它不验证GitLab Runner服务是否正在使用运行程序。 |





## sonar-gitlab-plugin 集成 gitlab

通过在 sonarqube 中使用这个插件，完成 gitlab 每次提交或者合并都做代码的检查，并返回检查的结果。

插件地址：https://github.com/gabrie-allaigre/sonar-gitlab-plugin

作者说了灵感来自：https://github.com/gabrie-allaigre/sonar-gitlab-plugin

但是：

**Only SonarQube < 7.7, because preview mode is removed**

sonarQube 在 7.6 版本后不支持插件 `sonar-gitlab-plugin`，因为 preview mode is removed。若使用 7.6 之后的版本，将无法实现 sonarQube 与 GitLab 协作（除非使用付费版）。

所以 sonarqube选择7.6，sonar-gitlab-plugin 选择 Version 4.0。

### 插件安装

插件放在 sonarqube-7.6/extensions/plugins 下，然后重启服务。

在配置界面配置 gitlab 相关信息：

- GitLab url：http://192.168.1.240:7788
- GitLab User Token：在 gitlab 创建一个用户 sonarqube 用于和 sonarqube 的连接
- GitLab Ignore Certificate：√  忽略证书验证
- GitLab Max Global Issues：1000
- Set GitLab API version： V4
- Load rules information：√

### 配置 gitlab runner

同上，这次使用 shared 方式，任何项目都可以使用。

![](https://blog.lichenghao.cn/upload/2022/07/30153658.png)

在项目——>设置——>CD/CD 可以查看可用的 runner。

![](https://blog.lichenghao.cn/upload/2022/07/30153823.png)



### 配置环境变量

在项目——>设置——>CD/CD 中可以配置自己的环境变量。比如我们把 sonar 的服务地址，登录的用户名等设置为变量。

![](https://blog.lichenghao.cn/upload/2022/07/30152127.png)

### 配置 .gitlab-ci.yml 

在 gitlab 中注册一个执行器，用来做 sonar 代码的检查，然后在项目目录下增加 .gitlab-ci.yml 

```yaml
stages:
  - sonar_preview

sonar-check:
  stage: sonar_preview
  tags:
    - sonarScanner
  script:
    - mvn -Dmaven.test.skip=true --batch-mode verify sonar:sonar -Dsonar.host.url=$SONAR_URL -Dsonar.login=$SONAR_LOGIN_USER -Dsonar.password=$SONAR_LOGIN_PWD -Dsonar.analysis.mode=preview -Dsonar.gitlab.project_id=$CI_PROJECT_PATH -Dsonar.gitlab.commit_sha=$CI_COMMIT_SHA -Dsonar.gitlab.ref_name=$CI_COMMIT_REF_NAME
```

### 验证检测结果

测试，提交一点垃圾代码。

![](https://blog.lichenghao.cn/upload/2022/07/30152923.png)

查看统计信息

![](https://blog.lichenghao.cn/upload/2022/07/30153037.png)





注意📢：

如果是 maven 项目，要提前配置好 maven 及 相关环境变量。
