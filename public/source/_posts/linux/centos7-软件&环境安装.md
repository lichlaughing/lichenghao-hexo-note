---
title: centos7-软件&环境安装
categories: linux
tags:
  - linux
abbrlink: 65a818d4
date: 2022-04-01 12:00:00
---
## Node.js

下载文件 ：`官网下载地址：https://nodejs.org/en/download/`

```bash
[root@iz ~]# wget https://nodejs.org/dist/v12.18.4/node-v12.18.4-linux-x64.tar.xz
```

解压改名后放到`/usr/local`下

```bash
[root@iz ~]# tar -xvf node-v12.18.4-linux-x64.tar.xz
[root@iz ~]# mv node-v12.18.4-linux-x64 nodejs
[root@iz ~]# mv nodejs /usr/local/
```

添加软连接使`node` `npm`命令生效

```bash
[root@iz local]# ln -s /usr/local/nodejs/bin/npm /usr/local/bin/
[root@iz local]# ln -s /usr/local/nodejs/bin/node /usr/local/bin/
```

测试命令

```bash
[root@iz local]# node -v
v12.18.4
[root@iz local]# npm -v
6.14.6
```

默认 npm 使用国外的镜像地址。npm 安装插件过程：从[http://registry.npmjs.org](http://registry.npmjs.org/)下载对应的插件包（该网站服务器位于国外，所以经常下载缓慢或出现异常），解决办法就是使用`cnpm`

```bash
[root@iz local]# npm get registry
https://registry.npmjs.org/
```

切换国内镜像源

```shell
[root@lch local]# npm get registry
https://registry.npmjs.org/

[root@lch local]# npm config set registry https://registry.npm.taobao.org
[root@lch local]# npm get registry
https://registry.npm.taobao.org/
```

或者使用 cnpm

`cnpm`使用淘宝的镜像源。这是一个完整 `npmjs.org` 镜像，你可以用此代替官方版本(只读)，同步频率目前为 10 分钟 一次以保证尽量与官方服务同步。官网说明地址：http://npm.taobao.org

安装 cnpm

```BASH
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

添加软链接

```bash
[root@iz local]# ln -s /usr/local/nodejs/bin/cnpm /usr/local/bin/
```

-v

```bash
[root@iz bin]# cnpm -v
cnpm@6.1.1 (/usr/local/nodejs/lib/node_modules/cnpm/lib/parse_argv.js)
npm@6.14.8 (/usr/local/nodejs/lib/node_modules/cnpm/node_modules/npm/lib/npm.js)
node@12.18.4 (/usr/local/nodejs/bin/node)
npminstall@3.27.0 (/usr/local/nodejs/lib/node_modules/cnpm/node_modules/npminstall/lib/index.js)
prefix=/usr/local/nodejs
linux x64 3.10.0-514.26.2.el7.x86_64
registry=https://r.npm.taobao.org
```



安装 Hexo 测试

```bash
[root@iz ~]# npm install -g hexo-cli
```

添加软链接

```bash
[root@iz bin]# ln -s /usr/local/nodejs/bin/hexo /usr/local/bin/
```

-v

```bash
[root@iz8vb17kkcn9okl08gwullz bin]# hexo -v
hexo-cli: 4.2.0
os: Linux 3.10.0-514.26.2.el7.x86_64 linux x64
node: 12.18.4
v8: 7.8.279.23-node.39
uv: 1.38.0
zlib: 1.2.11
brotli: 1.0.7
ares: 1.16.0
modules: 72
nghttp2: 1.41.0
napi: 6
llhttp: 2.1.2
http_parser: 2.9.3
openssl: 1.1.1g
cldr: 37.0
icu: 67.1
tz: 2019c
unicode: 13.0
```

`opt`下新建文件夹`hexo_blog`存放 blog 文件,名称随便起。

```bash
[root@iz opt]# mkdir hexo_blog
```

进入文件夹初始化`hexo init` `npm install`

```bash
[root@iz hexo_blog]# hexo init
......
INFO  Start blogging with Hexo!
[root@iz hexo_blog]# npm install

//最终得到文件列表
-rw-r--r--   1 root root  2439 9月  28 15:57 _config.yml
drwxr-xr-x 167 root root  4096 9月  28 16:01 node_modules
-rw-r--r--   1 root root   577 9月  28 15:57 package.json
-rw-r--r--   1 root root 55015 9月  28 15:59 package-lock.json
drwxr-xr-x   2 root root  4096 9月  28 15:57 scaffolds
drwxr-xr-x   3 root root  4096 9月  28 15:57 source
drwxr-xr-x   3 root root  4096 9月  28 15:57 themes
```

启动服务，访问 `http://localhost:4000`

```bash
[root@iz hexo_blog]# hexo s
INFO  Validating config
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```

如果是云服务器访问不到，安全组放行端口，防火墙放行端口

```bash
[root@iz ~]# firewall-cmd --zone=public --add-port=4000/tcp --permanent
[root@iz ~]# firewall-cmd --reload
```

> 附：firewall 防火墙的相关命令

```bash
查看版本： firewall-cmd --version
查看帮助： firewall-cmd --help
显示状态： firewall-cmd --state
查看所有打开的端口： firewall-cmd --zone=public --list-ports
添加端口放行:	firewall-cmd --zone=public --add-port=123456/tcp --permanent
更新防火墙规则： firewall-cmd --reload
查看区域信息: firewall-cmd --get-active-zones
查看指定接口所属区域： firewall-cmd --get-zone-of-interface=eth0
拒绝所有包：firewall-cmd --panic-on
取消拒绝状态： firewall-cmd --panic-off
查看是否拒绝： firewall-cmd --query-panic
```



## 搭建 python3.x 环境

查看历史版本：

```bash
[root@iz8vb17kkcn9okl08gwullz ~]# python -V
Python 2.7.5
```

更新下 yum 源(可不操作)

```
yum update
```

安装相关依赖

```bash
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make libffi-devel
```

安装 wget,用于下载安装包

```bash
yum install wget
```

下载源码包,需要哪个版本改成哪个版本，官网下载地址：https://www.python.org/downloads/

```
wget https://www.python.org/ftp/python/3.8.6/Python-3.8.6.tgz
```

解压并安装

```bash
# 解压压缩包
tar -zxvf Python-3.8.6.tgz
# 进入文件夹
cd Python-3.8.6
# 配置安装位置
./configure prefix=/usr/local/python3
# 安装
make && make install
```

没有错误提示，并且如下所示，表示安装成功，在`/usr/local/python3` 可以看到安装文件

```
.........
Looking in links: /tmp/tmpowzioxl7
Processing /tmp/tmpowzioxl7/setuptools-49.2.1-py3-none-any.whl
Processing /tmp/tmpowzioxl7/pip-20.2.1-py2.py3-none-any.whl
Installing collected packages: setuptools, pip
  WARNING: The script easy_install-3.8 is installed in '/usr/local/python3/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
  WARNING: The scripts pip3 and pip3.8 are installed in '/usr/local/python3/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed pip-20.2.1 setuptools-49.2.1
```

添加软连接

```bash
#添加python3的软链接
ln -s /usr/local/python3/bin/python3.8 /usr/bin/python3
#添加 pip3 的软链接
ln -s /usr/local/python3/bin/pip3.8 /usr/bin/pip3
```

此时测试，python 还是 2.x 版本，python3 为 3.x 版本

```bash
[root@iz8vb17kkcn9okl08gwullz ~]# python -V
Python 2.7.5
[root@iz8vb17kkcn9okl08gwullz ~]# python3 -V
Python 3.8.6
```

修改 pip 源为国内的源

如果使用阿里云的云服务器默认就是阿里云的源，可以替换成如下国内的，清华的比较稳定

豆瓣 ··············· http://pypi.douban.com/

华中理工大学 ········ http://pypi.hustunique.com/

山东理工大学 ········ http://pypi.sdutlinux.org/

中国科学技术大学 ···· http://pypi.mirrors.ustc.edu.cn/

阿里云 ············· http://mirrors.aliyun.com/pypi/simple/

清华大学 ··········· https://pypi.tuna.tsinghua.edu.cn/simple/

步骤：

- 创建.pip 文件夹

```bash
mkdir ~/.pip
```

- 创建 pip.conf 配置文件

```bash
touch ~/.pip/pip.conf
```

- 修改 pip.conf 配置文件，修改内容如下：

```properties
[global]
index-url=http://mirrors.aliyun.com/pypi/simple
[install]
trusted-host=mirrors.aliyun.com
```

测试

```bash
[root@iz8vb17kkcn9okl08gwullz ~]# pip3 install lxml
Looking in indexes: https://mirrors.aliyun.com/pypi/simple
Collecting lxml
  Downloading https://mirrors.aliyun.com/pypi/packages/b7/51/fe28c128985b6a38fbcd091110e1e53846bf900cf143b04a071f2c1c81e7/lxml-4.5.2-cp38-cp38-manylinux1_x86_64.whl (5.4 MB)
     |████████████████████████████████| 5.4 MB 63.1 MB/s
Installing collected packages: lxml
Successfully installed lxml-4.5.2
```

例如安装`requests`模块

```bash
[root@iz8vb17kkcn9okl08gwullz ~]# pip3 install requests
Looking in indexes: https://mirrors.aliyun.com/pypi/simple
Collecting requests
  Downloading https://mirrors.aliyun.com/pypi/packages/45/1e/0c169c6a5381e241ba7404532c16a21d86ab872c9bed8bdcd4c423954103/requests-2.24.0-py2.py3-none-any.whl (61 kB)
     |████████████████████████████████| 61 kB 581 kB/s
Collecting chardet<4,>=3.0.2
  Downloading https://mirrors.aliyun.com/pypi/packages/bc/a9/01ffebfb562e4274b6487b4bb1ddec7ca55ec7510b22e4c51f14098443b8/chardet-3.0.4-py2.py3-none-any.whl (133 kB)
     |████████████████████████████████| 133 kB 7.5 MB/s
Collecting urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1
  Downloading https://mirrors.aliyun.com/pypi/packages/9f/f0/a391d1463ebb1b233795cabfc0ef38d3db4442339de68f847026199e69d7/urllib3-1.25.10-py2.py3-none-any.whl (127 kB)
     |████████████████████████████████| 127 kB 71.3 MB/s
Collecting idna<3,>=2.5
  Downloading https://mirrors.aliyun.com/pypi/packages/a2/38/928ddce2273eaa564f6f50de919327bf3a00f091b5baba8dfa9460f3a8a8/idna-2.10-py2.py3-none-any.whl (58 kB)
     |████████████████████████████████| 58 kB 10.1 MB/s
Collecting certifi>=2017.4.17
  Downloading https://mirrors.aliyun.com/pypi/packages/5e/c4/6c4fe722df5343c33226f0b4e0bb042e4dc13483228b4718baf286f86d87/certifi-2020.6.20-py2.py3-none-any.whl (156 kB)
     |████████████████████████████████| 156 kB 72.2 MB/s
Installing collected packages: chardet, urllib3, idna, certifi, requests
  WARNING: The script chardetect is installed in '/usr/local/python3/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed certifi-2020.6.20 chardet-3.0.4 idna-2.10 requests-2.24.0 urllib3-1.25.10
[root@iz8vb17kkcn9okl08gwullz ~]#
```

其他：

使用`requests.get("xxxx")`报错如下：

```
requests.exceptions.SSLError: HTTPSConnectionPool(host='www.baidu.com', port=443): Max retries exceeded with url: /baidusitemap.xml (Caused by SSLError(SSLError(1, '[SSL: SSLV3_ALERT_HANDSHAKE_FAILURE] sslv3 alert handshake failure (_ssl.c:1124)')))
```

网上很多解决方式：

首先保证你访问的网址是可用的！！！

第一种：去掉 ssl 证书的验证 ，或者指定证书

`requests.get("xxxx",verify=False)`

`requests.get('https://google.com', verify='/path/to/certfile')`

第二种：

安装三个模块：

```bash
pip install cryptography
pip install pyOpenSSL
pip install certifi
```



## Nginx

### 单机部署

下载：地址：http://nginx.org/download/ 选择不同的版本即可

```sh
wget http://nginx.org/download/nginx-1.20.2.tar.gz
```

安装依赖

```sh
yum -y install gcc pcre-devel zlib-devel openssl openssl-devel
```

解压编译安装

```sh
[root@localhost ~]# tar -zxvf nginx-1.20.2.tar.gz
[root@localhost ~]# cd nginx-1.20.2
# 配置
[root@localhost nginx-1.20.2]]# ./configure --prefix=/opt/nginx
# 编译安装
[root@localhost nginx-1.20.2]# make
[root@localhost nginx-1.20.2]# make install
```

启动服务

```sh
# 进入目录
[root@localhost ~]# cd /opt/nginx/sbin
# 启动服务
[root@localhost sbin]# ./nginx 
# 重新加载配置
[root@localhost sbin]# ./nginx -s reload
# 停止服务
[root@localhost sbin]# ./nginx -s stop
# 检查配置是否正确
[root@localhost sbin]# ./nginx -t
```

访问服务地址：http://172.16.127.3/

![](https://blog.lichenghao.cn/upload/2022/07/30180444.png)

配置 systemctl 服务

增加配置文件

```sh
vim /etc/systemd/system/nginx.service
```

内容如下，路径修改为自己的。

```properties
[Unit]
Description=The Nginx  Server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/opt/nginx/logs/nginx.pid
ExecStart=/opt/nginx/sbin/nginx
ExecReload=/opt/nginx/sbin/nginx -s reload
ExecStop=/opt/nginx/sbin/nginx -s stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target

```

服务

```sh
systemctl start nginx.service　#（启动nginx服务）
systemctl stop nginx.service　#（停止nginx服务）
systemctl enable nginx.service #（设置开机自启动）
systemctl disable nginx.service #（停止开机自启动）
systemctl status nginx.service #（查看服务当前状态）
systemctl restart nginx.service　#（重新启动服务）
systemctl list-units --type=service #（查看所有已启动的服务）
```

如果你通过nginx 启动了服务，先关闭掉后。systemctl 才好使。



## Mysql



### 下载包

首先到官网下载对应版本的 RPM 包，下载地址：https://dev.mysql.com/downloads/mysql/ 选择**Red Hat Enterprise Linux**下载 RPM 包。得到一个这样一个包：mysql-5.7.34-1.el7.x86_64.rpm-bundle.tar

先删除系统自带的

```sh
yum remove -y mariadb*
```

解压包得到许多 RPM 包：

```sh
mysql-community-client-5.7.34-1.el7.x86_64.rpm
mysql-community-common-5.7.34-1.el7.x86_64.rpm
mysql-community-embedded-5.7.34-1.el7.x86_64.rpm
mysql-community-embedded-compat-5.7.34-1.el7.x86_64.rpm
mysql-community-embedded-devel-5.7.34-1.el7.x86_64.rpm
mysql-community-libs-5.7.34-1.el7.x86_64.rpm
mysql-community-libs-compat-5.7.34-1.el7.x86_64.rpm
mysql-community-server-5.7.34-1.el7.x86_64.rpm
mysql-community-test-5.7.34-1.el7.x86_64.rpm
```

### 安装

执行安装服务端：

```sh
[root@localhost mysql]# rpm -ivh mysql-community-server-5.7.34-1.el7.x86_64.rpm
```

直接安装服务端可能安装不成功，会提示你安装的依赖关系，按照依赖关系依次安装即可。

通常情况下安装顺序是：

mysql-community-common-5.7.34-1.el7.x86_64.rpm

mysql-community-libs-5.7.34-1.el7.x86_64.rpm

mysql-community-client-5.7.34-1.el7.x86_64.rpm

mysql-community-server-5.7.34-1.el7.x86_64.rpm

都安装完毕后，启动服务。

```sh
[root@localhost mysql]# service mysqld start
```

### 修改密码

使用 root 登录

```sh
[root@localhost mysql]# mysql -u root
```

默认密码在日志文件中，注意别复制空格

```bash
[root@localhost ~]# grep 'temporary password' /var/log/mysqld.log
 A temporary password is generated for root@localhost: 0uCarwKy,hhu
```

登录成功后，修改密码

```sql
mysql> set password for root@localhost  = password("Root@123456");
```

如果你的密码过于简单可能会提示你

```sql
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

如果没有找到默认密码的话可以使用暴力的方式，给 root 设置默认密码

修改配置文件：/etc/my.cnf ,在[mysqld]下增加：skip-grant-tables 如下所示：（该配置可以让 root 不使用密码就可以登录数据库）

```properties
[mysqld]
character-set-server=utf8
lower_case_table_names=1
skip-grant-tables  # 主要是这个跳过认证
```

然后重启服务

```sh
[root@localhost ~]# service mysqld restart
```

这个时候 root 不使用密码就可登录，到了输入密码的地方直接回车

```sh
[root@localhost ~]# mysql -u root -p
```

然后修改 root 密码：

```sh
flush privileges;
set password for root@localhost=password('你的密码'); #重启数据库即可
```

修改完毕后，将刚才配置文件中增加的`skip-grant-tables`给去掉，重启数据库即可。

### 开启远程访问

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;
flush privileges;
```

或者

```sql
 update user set host = '%' where user = 'root';
 flush privileges;
```

### 大小写敏感

配置文件增加：不区分表名大小写

```properties
[mysqld]
character-set-server=utf8
lower_case_table_names=1
```





## Rabbitmq

首先查看 erlang 和 rabbitmq 的版本兼容情况

https://www.rabbitmq.com/which-erlang.html#compatibility-matrix

### RPM 安装

erlang rpm 下载地址：https://github.com/rabbitmq/erlang-rpm/releases

rabbitmq rpm 下载地址：https://www.rabbitmq.com/install-rpm.html

！注意选择的版本，高版本的需要 centos8

运行安装命令即可：

```bash
rpm -ivh xxx
```

```bash
//启动服务
systemctl start rabbitmq-server
//重启
systemctl restart rabbitmq-server
//查看服务状态
systemctl status rabbitmq-server
//启用web插件
rabbitmq-plugins enable rabbitmq_management
//添加管理员
rabbitmqctl add_user 用户名 密码
rabbitmqctl set_user_tags 用户 administrator
rabbitmqctl set_permissions -p / 用户 ".*" ".*" ".*"
```

如果需要 rabbitmq 的配置文件，下载模板

配置文件地址：https://github.com/rabbitmq/rabbitmq-server/blob/master/docs/rabbitmq.conf.example

重命名后放在`/etc/rabbitmq/rabbitmq.conf`，没有路径自己创建即可



启动服务后，使用默认的账号登录 guest/guest 即可。

可能会提示 `User can only log in via localhost`  因为默认的用户不让远程访问。我们可以新建个账号给上权限即可。如上面的添加管理员。







### 编译安装

#### 安装 erlang

下载地址：https://erlang.org/download/otp_src_23.1.tar.gz

删除历史 erlang `rpm -e --nodeps erlang`

解压文件

```bash
[root@iz8vb17kkcn9okl08gwullz ~]# tar -zxvf otp_src_23.1.tar.gz
```

进入根目录执行配置，报错缺什么依赖就装什么依赖

```
./configure --prefix=/usr/local/erlang --without-javac
```

编译安装

```bash
make && make install
```

配置环境变量

```bash
echo 'export PATH=$PATH:/usr/local/erlang/bin' >> /etc/profile
source /etc/profile
```

验证

```bash
[root@iz8vb17kkcn9okl08gwullz erlang]# erl
Erlang/OTP 23 [erts-11.1] [source] [64-bit] [smp:1:1] [ds:1:1:10] [async-threads:1] [hipe]

Eshell V11.1  (abort with ^G)
1>
```

#### 安装 rabbitmq

官网：https://www.rabbitmq.com/

下载安装包：https://github.com/rabbitmq/rabbitmq-server/releases

```bash
[root@iz8vb17kkcn9okl08gwullz ~]# wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.9/rabbitmq-server-generic-unix-3.8.9.tar.xz
```

或者

```bash
[root@iz8vb17kkcn9okl08gwullz ~]# wget https://hub.fastgit.org/rabbitmq/rabbitmq-server/releases/download/v3.8.9/rabbitmq-server-generic-unix-3.8.9.tar.xz
```

解压，没有 xz 命令安装`yum install -y xz`

```bash
[root@iz8vb17kkcn9okl08gwullz ~]# xz -d rabbitmq-server-generic-unix-3.8.9.tar.xz

[root@iz8vb17kkcn9okl08gwullz ~]# tar -xvf rabbitmq-server-generic-unix-3.8.9.tar
```

移动到`/user/local`下

```bash
[root@iz8vb17kkcn9okl08gwullz local]# mv rabbitmq_server-3.8.9/ /usr/local/
```

配置环境变量

```bash
echo 'export PATH=$PATH:/usr/local/rabbitmq_server-3.8.9/sbin' >> /etc/profile

//刷新环境变量
source /etc/profile
```

创建配置文件,配置内容可以放空，走默认配置，或者从 github 上下载一个默认的

配置文件地址：https://github.com/rabbitmq/rabbitmq-server/blob/master/docs/rabbitmq.conf.example

重命名后放在`/etc/rabbitmq/rabbitmq.conf`，没有路径自己创建即可

启动服务

```bash
rabbitmq-server -detached
```

停止服务

```bash
rabbitmqctl stop
```

状态

```bash
rabbitmqctl status
```

开启 web 插件

```bash
rabbitmq-plugins enable rabbitmq_management
```

添加管理员用户

```bash
rabbitmqctl add_user 用户名 密码

rabbitmqctl set_user_tags 用户 administrator

rabbitmqctl set_permissions -p / 用户 ".*" ".*" ".*"
```

查看管理页面：

http://ip:15672/

---

卸载

```bash
# 卸载erlang
yum list | grep erlang
yum -y remove erlang-*
rm -rf /usr/local/erlang

# 卸载RabbitMQ
yum list | grep rabbitmq
yum -y remove rabbitmq-server.noarch
```





## PostgreSQL

### 在线安装 

在官网下载页面，选择对应的版本和操作系统，会给出 rpm 的安装脚本，安装即可。如下安装脚本所示：

```shell
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum install -y postgresql14-server
sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
sudo systemctl enable postgresql-14
sudo systemctl start postgresql-14
```



### 离线安装

演示版本：postgresql-14.4

在下载页面选择好版本后，在页面的最下有个`Direct RPM download` 点击[direct download](https://yum.postgresql.org/rpmchart/)  就可以选择对应的版本下载离线的 rpm 包。

比如：

![](https://blog.lichenghao.cn/upload/2022/07/28105833.png ':size=20%')



![](https://blog.lichenghao.cn/upload/2022/07/28105910.png ':size=20%')

把下面的每个连接中的包都下载一个。

![](https://blog.lichenghao.cn/upload/2022/07/28105933.png ':size=20%')

比如选择第一个版本的包：

![](https://blog.lichenghao.cn/upload/2022/07/28110006.png ':size=20%')



下载完毕后，开始安装，如果单个安装会报依赖错误，比如：

```shell
[root@lch PostgreSQL14.4]# rpm -ivh postgresql14-14.4-1PGDG.rhel7.x86_64.rpm
警告：postgresql14-14.4-1PGDG.rhel7.x86_64.rpm: 头V4 DSA/SHA1 Signature, 密钥 ID 442df0f8: NOKEY
错误：依赖检测失败：
libpq.so.5()(64bit) 被 postgresql14-14.4-1PGDG.rhel7.x86_64 需要
postgresql14-libs(x86-64) = 14.4-1PGDG.rhel7 被 postgresql14-14.4-1PGDG.rhel7.x86_64 需要
```

直接安装全部：

```shell
[root@lch PostgreSQL14.4]# yum -y install postgresql14-*
```

- 会创建一个默认的 linux 用户 postgres

- 会自动安装一个服务 `postgresql-14`

- 最终的安装目录在 `/usr/pgsql-14`

- 默认的数据目录在 `/var/lib/pgsql`

进入安装目录，初始化数据库，会创建默认数据库： postgres，默认用户：postgres/postgres

```shell
[root@lch bin]# ./postgresql-14-setup initdb
Initializing database ... OK
```

修改配置文件，开启远程访问, 配置文件路径：/var/lib/pgsql/14/data/pg_hba.conf

```shell
# 修改为 0.0.0.0/0
# IPv4 local connections:
host    all             all             0.0.0.0/0            scram-sha-256
```

启动服务

```shell
[root@lch ~]# systemctl start postgresql-14
```

修改用户 postgres 的密码的一种方式：

```shell
# 切换用户
[root@lch ~]# su postgres
# 进入安装目录，执行脚本
bash-4.2$ ./psql 
postgres=#
# 修改密码
postgres= # \password postgres 
# 退出
postgres=# \q
```

Navicat 测试登录即可。



