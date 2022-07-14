---
title: Git Gitlab 私有部署
categories:
  - tools
  - git
tags:
  - git
  - gitlab
keywords: 'git,gitlab'
abbrlink: 2ff242c3
date: 2022-05-26 12:40:00
cover: https://blog.lichenghao.cn/upload/2022/07/DE697120-gitlab.png
---
## 部署 gitlab

部署参考文档：https://gitlab.cn/install/

安装包地址：https://packages.gitlab.com/gitlab/gitlab-ce ，Centos7 使用 el/ 7版本下载。（ gitlab-ce-12.9.4-ce.0.el7.x86_64.rpm）

配置必须的依赖和防火墙的拦截。

```shell
sudo yum install -y curl policycoreutils-python openssh-server perl
sudo systemctl enable sshd
sudo systemctl start sshd

sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
```

安装

```shell
[root@lch ~]# rpm -i gitlab-ce-12.9.4-ce.0.el7.x86_64.rpm 
....
Thank you for installing GitLab!
.....
```

- 默认安装路径：/opt/gitlab
- 配置文件的路径：/etc/gitlab/gitlab.rb 
- 会安装一个服务 gitlab-ctl 

修改服务地址，修改配置文件 gitlab.rb 

```properties
# external_url 'http://gitlab.example.com'
external_url 'http://192.168.1.240:7788'
```

修改仓库及仓库备份的路径，修改配置文件 gitlab.rb 

gitlab 默认的仓库地址为 /var/opt/gitlab/git-data/repositories 。如果需要修改如下所示：

```properties
###!   path that doesn't contain symlinks.**
# git_data_dirs({
#   "default" => {
#     "path" => "/mnt/nfs-01/git-data"
#    }
# })

git_data_dirs({
   "default" => {
     "path" => "/git-data"
    }
 })
```

备份路径通常也是默认在系统盘 /var/opt/gitlab/backups 。如果需要修改则如下所示：

```properties
# gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
gitlab_rails['backup_path'] = "/git-data/backups"
```

修改完毕后，重新配置下

```shell
[root@lch gitlab]# gitlab-ctl reconfigure
...等一会
Chef Client finished, 543/1462 resources updated in 07 minutes 40 seconds
gitlab Reconfigured!
```

重启服务

```shell
[root@lch gitlab]# gitlab-ctl restart
```

访问配置好的地址：http://192.168.1.240:7788/

![](https://blog.lichenghao.cn/upload/2022/07/29134458.png)

第一次打开需要修改密码，默认用户为 root 。

最终登录后：

![](https://blog.lichenghao.cn/upload/2022/07/29134839.png)



如果打开报错 502 的话：

- 日志目录赋权：chmod -R 777 /var/log/gitlab
- 查看是否有端口的占用





## 部署 gitlab-runner

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



## 自动部署一个 docsify 服务

需求：

比如我们通过 nodejs 启动了一个 docsify 文档服务。我们把代码托管在 gitlab 中，希望每次修改代码都能直接生效。因为 docsify 具有热部署的功能，所以我们只需要把文件拷贝到服务所在的目录即可。

执行：

假如我们的 docsify 服务启动在目录 /opt/digital-twins-development-doc 下，

那么注册一个 gitlab-runner ，然后配置 .gitlab-ci.yml

```yaml
stages:
  - doc_deploy

start:
  stage: doc_deploy
  tags:
    - docdeploy
  script:
    - yes| cp -rf . /usr/local/digital-twins-development-doc || exit 0
```

就是把文件拷贝到指定目录，然后返回 0，表示成功

效果就是：

前提是 docsify 已经正常运行起来，通过这个 gitlab-runner 去自动更新文件，因为热部署所以可以实施更新。

