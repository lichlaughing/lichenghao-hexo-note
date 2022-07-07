---
title: Sonarqube8.9 与 gitlab 集成
categories: sonarqube
tags:
  - sonarqube
  - gitlab
keywords: sonarqube
abbrlink: bfe6f3bc
date: 2022-05-26 13:50:00
cover: https://blog.lichenghao.cn/upload/2022/07/A889F46E-sonarqube.png
---
Gitlab 集成，先安装 Gitlab

## SonarQube与GitLab账号打通

Gitlab 单点登录到 sonarqube 。

Gitlab 管理员登录，在控制中心——>应用——>新增应用

- 名称：sornaqube
- Redirect URI：sornaqube 访问地址 + /oauth2/callback/gitlab
- Trusted：√

其余的如下图所示：

![](https://blog.lichenghao.cn/upload/2022/07/29142616.png)

提交后会得到 ApplicationID 和 Secret

![](https://blog.lichenghao.cn/upload/2022/07/29142817.png)



登录 sonarqube，在配置，ALM 集成中，配置 gitlab

- Enabled：√
- GitLab URL：http://192.168.1.240:7788/
- Application ID：上面得到
- Secret：上面得到
- Allow users to sign-up：√

在 通用 下配置 Server base URL 和 gitlab 中配置保持一致：http://192.168.1.240:9000

配置好了重新登录页面：

![](https://blog.lichenghao.cn/upload/2022/07/29143202.png)





## gitlab-runner 安装配置

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

需要的地址和令牌在 gitlab 中的管理中心，Runner 中。

![](https://blog.lichenghao.cn/upload/2022/07/29151040.png)

配置完毕后，在下面就会出现刚才配置的 runner 

![](https://blog.lichenghao.cn/upload/2022/07/29151257.png)

点击右侧编辑，将该作业指定给一个项目。

![](https://blog.lichenghao.cn/upload/2022/07/29154917.png)

然后测试个流水作业：

在测试项目中，设置——>CI/CD 中启用该 Runner 

![](https://blog.lichenghao.cn/upload/2022/07/29154314.png)

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





## 集成 SonarQube 完成代码的检查

需要下载 sonar-scanner 插件，sonar-scanner 需要和 gitlab-runner 安装在同一服务器上，这样gitlab-runner才可以调用sonar-scanner的命令进行分析。

下载地址：https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/

例如：sonar-scanner-cli-4.6.2.2472-linux.zip

解压到指定目录

```shell
[root@lch ~]# unzip sonar-scanner-cli-4.6.2.2472-linux.zip -d /opt/module/
```

修改配置文件 sonar-scanner.properties

```properties
#----- Default SonarQube server
sonar.host.url=http://192.168.1.240:9000
sonar.login=admin 
sonar.password=admin@123
#----- Default source code encoding
sonar.sourceEncoding=UTF-8

```

增加环境变量

```shell
[root@lch conf]# vim /etc/profile

# sonar-scanner
export SONAR_SCANNER_HOME=/opt/module/sonar-scanner-4.6.2.2472-linux
export PATH=${SONAR_SCANNER_HOME}/bin:${PATH}

[root@lch conf]# source /etc/profile
```



修改 gitlab 的作业配置文件

```yaml
stages:
  - first
  - sonar-scanner

job1:
  stage: first
  tags:
    - test
  only:
    - dev
  script:
    - echo "hi iam gitlab-ci"
sonarqube_scan_job:
  stage: sonar-scanner
  tags:
    - test
  only:
    - dev
  script:
    - sonar-scanner -Dsonar.projectName=$CI_PROJECT_NAME -Dsonar.projectKey=$CI_PROJECT_NAME  -Dsonar.language=java -Dsonar.java.binaries=. -Dsonar.host.url=http://192.168.1.240:9000  -Dsonar.login=admin -Dsonar.password=admin@123
```



然后提交代码触发作业

![](https://blog.lichenghao.cn/upload/2022/07/29161249.png)

同时在 sonarqube 中可以看到

![](https://blog.lichenghao.cn/upload/2022/07/29161352.png)

























