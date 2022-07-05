---
title: centos7-单机部署FastDFS
categories: linux
tags:
  - linux
abbrlink: 51293c90
date: 2022-04-01 12:00:00
---
FastDFS(Fast Distributed File System)是一款开源轻量级分布式文件系统，记录个人使用单机部署过程。

> fdfs 相关安装包：

![](https://blog.lichenghao.cn/upload/2022/07/20200715133918.png)

> nginx 安装包 : nginx.1.120.tar.gz

上传到服务器，位置如下所示(位置随便)

```java
drwxr-xr-x 2 root root   4096 Jan  6 09:02 fastDFS
-rw-r--r-- 1 root root 980831 Jan  6 09:05 nginx-1.12.0.tar.gz
[root@lichcloud soft]# pwd
/home/soft
[root@lichcloud soft]#
```

---

# 配置基本环境

## 安装 libevent

Libevent 是一个用 C 语言编写的、轻量级的开源高性能事件通知库

```shell
[root@lichcloud soft]# yum -y install libevent
Loaded plugins: fastestmirror
Setting up Install Process
Determining fastest mirrors
epel/metalink                                                                                                    | 9.5 kB     00:00
 * base: mirrors.tuna.tsinghua.edu.cn
 * epel: mirrors.huaweicloud.com
 * extras: mirrors.163.com
 * updates: mirrors.huaweicloud.com
base                                                                                                             | 3.7 kB     00:00
epel                                                                                                             | 3.2 kB     00:00
epel/primary                                                                                                     | 3.2 MB     00:00
epel                                                                                                                        12508/12508
extras                                                                                                           | 3.4 kB     00:00
extras/primary_db                                                                                                |  27 kB     00:00
updates                                                                                                          | 3.4 kB     00:00
updates/primary_db                                                                                               | 2.4 MB     00:00
Resolving Dependencies
--> Running transaction check
---> Package libevent.x86_64 0:1.4.13-4.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================================================================
 Package                         Arch                          Version                                Repository                   Size
========================================================================================================================================
Installing:
 libevent                        x86_64                        1.4.13-4.el6                           base                         66 k

Transaction Summary
========================================================================================================================================
Install       1 Package(s)

Total download size: 66 k
Installed size: 227 k
Downloading Packages:
libevent-1.4.13-4.el6.x86_64.rpm                                                                                 |  66 kB     00:00
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : libevent-1.4.13-4.el6.x86_64                                                                                         1/1
  Verifying  : libevent-1.4.13-4.el6.x86_64                                                                                         1/1

Installed:
  libevent.x86_64 0:1.4.13-4.el6

Complete!
[root@lichcloud soft]#
```

## 1.2 libfastcommon-1.0.7 编译安装

解压 libfastcommon-1.0.7.tar.gz，进入目录，进行 make 编译,如果报错如下，表示需要安装 gcc 的依赖包

```java
[root@lichcloud libfastcommon-1.0.7]# ./make.sh
./make.sh: line 14: gcc: command not found
./make.sh: line 15: ./a.out: No such file or directory
cc -Wall -D_FILE_OFFSET_BITS=64 -g -DDEBUG_FLAG -DOS_LINUX -DIOEVENT_USE_EPOLL -c -fPIC -o hash.lo hash.c
make: cc: Command not found
make: *** [hash.lo] Error 127
[root@lichcloud libfastcommon-1.0.7]#
```

安装 gcc

```java
yum install gcc
```

执行./make.sh install 进行安装

```shell
[root@lichcloud libfastcommon-1.0.7]# ./make.sh install
mkdir -p /usr/lib64
install -m 755 libfastcommon.so /usr/lib64
mkdir -p /usr/include/fastcommon
install -m 644 common_define.h hash.h chain.h logger.h base64.h shared_func.h pthread_func.h ini_file_reader.h _os_bits.h sockopt.h sched_thread.h http_func.h md5.h local_ip_func.h avl_tree.h ioevent.h ioevent_loop.h fast_task_queue.h fast_timer.h process_ctrl.h fast_mblock.h connection_pool.h /usr/include/fastcommon
[root@lichcloud libfastcommon-1.0.7]#
```

将配置文件 libfastcommon.so 拷贝到 32 位目录下一份

```shell
[root@lichcloud lib64]# ll libfast*
-rwxr-xr-x 1 root root 284000 Jan  6 09:18 libfastcommon.so
[root@lichcloud lib64]# pwd
/usr/lib64
[root@lichcloud lib64]# cp libfastcommon.so /usr/lib
[root@lichcloud lib64]#
```

# 安装 tracker 服务

## 编译安装 tracker 服务

解压 fastdfs-5.05.tar.gz

```shell
[root@lichcloud fastDFS]# tar -zxvf fastdfs-5.05.tar.gz
```

解压后的目录：

```java
[root@lichcloud fastDFS]# cd fastdfs-5.05
[root@lichcloud fastdfs-5.05]# ll
total 132
drwxrwxr-x 3 root root  4096 Nov 22  2014 client
drwxrwxr-x 2 root root  4096 Nov 22  2014 common
drwxrwxr-x 2 root root  4096 Nov 22  2014 conf
-rw-rw-r-- 1 root root 35067 Nov 22  2014 COPYING-3_0.txt
-rw-rw-r-- 1 root root  2802 Nov 22  2014 fastdfs.spec
-rw-rw-r-- 1 root root 31386 Nov 22  2014 HISTORY
drwxrwxr-x 2 root root  4096 Nov 22  2014 init.d
-rw-rw-r-- 1 root root  7755 Nov 22  2014 INSTALL
-rwxrwxr-x 1 root root  5813 Nov 22  2014 make.sh
drwxrwxr-x 2 root root  4096 Nov 22  2014 php_client
-rw-rw-r-- 1 root root  2380 Nov 22  2014 README.md
-rwxrwxr-x 1 root root  1768 Nov 22  2014 restart.sh
-rwxrwxr-x 1 root root  1680 Nov 22  2014 stop.sh
drwxrwxr-x 4 root root  4096 Nov 22  2014 storage
drwxrwxr-x 2 root root  4096 Nov 22  2014 test
drwxrwxr-x 2 root root  4096 Nov 22  2014 tracker
[root@lichcloud fastdfs-5.05]#
```

编码 ./make.sh

```java
太多省略
```

安装 ./make.sh install

```shell
[root@lichcloud fastdfs-5.05]# ./make.sh install
mkdir -p /usr/bin
mkdir -p /etc/fdfs
cp -f fdfs_trackerd /usr/bin
if [ ! -f /etc/fdfs/tracker.conf.sample ]; then cp -f ../conf/tracker.conf /etc/fdfs/tracker.conf.sample; fi
mkdir -p /usr/bin
mkdir -p /etc/fdfs
cp -f fdfs_storaged  /usr/bin
if [ ! -f /etc/fdfs/storage.conf.sample ]; then cp -f ../conf/storage.conf /etc/fdfs/storage.conf.sample; fi
mkdir -p /usr/bin
mkdir -p /etc/fdfs
mkdir -p /usr/lib64
cp -f fdfs_monitor fdfs_test fdfs_test1 fdfs_crc32 fdfs_upload_file fdfs_download_file fdfs_delete_file fdfs_file_info fdfs_appender_test fdfs_appender_test1 fdfs_append_file fdfs_upload_appender /usr/bin
if [ 0 -eq 1 ]; then cp -f libfdfsclient.a /usr/lib64; fi
if [ 1 -eq 1 ]; then cp -f libfdfsclient.so /usr/lib64; fi
mkdir -p /usr/include/fastdfs
cp -f ../common/fdfs_define.h ../common/fdfs_global.h ../common/mime_file_parser.h ../common/fdfs_http_shared.h ../tracker/tracker_types.h ../tracker/tracker_proto.h ../tracker/fdfs_shared_func.h ../storage/trunk_mgr/trunk_shared.h tracker_client.h storage_client.h storage_client1.h client_func.h client_global.h fdfs_client.h /usr/include/fastdfs
if [ ! -f /etc/fdfs/client.conf.sample ]; then cp -f ../conf/client.conf /etc/fdfs/client.conf.sample; fi
[root@lichcloud fastdfs-5.05]#
```

安装完毕后，相关配在/usr/bin 目录下

```shell
[root@lichcloud bin]# ll fdfs*
-rwxr-xr-x 1 root root 260972 Jan  6 09:29 fdfs_appender_test
-rwxr-xr-x 1 root root 260701 Jan  6 09:29 fdfs_appender_test1
-rwxr-xr-x 1 root root 251029 Jan  6 09:29 fdfs_append_file
-rwxr-xr-x 1 root root 253521 Jan  6 09:29 fdfs_crc32
-rwxr-xr-x 1 root root 251088 Jan  6 09:29 fdfs_delete_file
-rwxr-xr-x 1 root root 251919 Jan  6 09:29 fdfs_download_file
-rwxr-xr-x 1 root root 251629 Jan  6 09:29 fdfs_file_info
-rwxr-xr-x 1 root root 264245 Jan  6 09:29 fdfs_monitor
-rwxr-xr-x 1 root root 891607 Jan  6 09:29 fdfs_storaged
-rwxr-xr-x 1 root root 267404 Jan  6 09:29 fdfs_test
-rwxr-xr-x 1 root root 266581 Jan  6 09:29 fdfs_test1
-rwxr-xr-x 1 root root 379654 Jan  6 09:29 fdfs_trackerd
-rwxr-xr-x 1 root root 251967 Jan  6 09:29 fdfs_upload_appender
-rwxr-xr-x 1 root root 253065 Jan  6 09:29 fdfs_upload_file
[root@lichcloud bin]# pwd
/usr/bin
[root@lichcloud bin]#
```

还有部分配置文件在 /etc/fdfs 下

```shell
[root@lichcloud bin]# cd /etc/fdfs/
[root@lichcloud fdfs]# ll
total 20
-rw-r--r-- 1 root root 1461 Jan  6 09:29 client.conf.sample
-rw-r--r-- 1 root root 7829 Jan  6 09:29 storage.conf.sample
-rw-r--r-- 1 root root 7102 Jan  6 09:29 tracker.conf.sample
[root@lichcloud fdfs]#
```

将安装文件中配置文件拷贝到/etc/fdfs

```shell
-rw-rw-r-- 1 root root 23981 Nov 22  2014 anti-steal.jpg
-rw-rw-r-- 1 root root  1461 Nov 22  2014 client.conf
-rw-rw-r-- 1 root root   858 Nov 22  2014 http.conf
-rw-rw-r-- 1 root root 31172 Nov 22  2014 mime.types
-rw-rw-r-- 1 root root  7829 Nov 22  2014 storage.conf
-rw-rw-r-- 1 root root   105 Nov 22  2014 storage_ids.conf
-rw-rw-r-- 1 root root  7102 Nov 22  2014 tracker.conf
[root@lichcloud conf]# cp * /etc/fdfs/
[root@lichcloud conf]# pwd
/home/soft/fastDFS/fastdfs-5.05/conf
[root@lichcloud conf]#
```

## 配置 tracker.conf

### 修改 base_path 路径

```shell
# the base path to store data and log files
base_path=/fastdfs/tracker
```

### 创建 2.2.1 修改的路径及其他路径

然后直接新建 client 和 storage 目录，以后要用的，这里一起创建了。

```shell
[root@lichcloud /]# mkdir /fastdfs/tracker -p

[root@lichcloud fastdfs]# cd /fastdfs/
[root@lichcloud fastdfs]# ll
total 12
drwxr-xr-x 2 root root 4096 Jan  6 09:42 client
drwxr-xr-x 2 root root 4096 Jan  6 09:42 storage
drwxr-xr-x 2 root root 4096 Jan  6 09:41 tracker
[root@lichcloud fastdfs]#
```

## 启动/重启 tracker 服务

```shell
fdfs_trackerd /etc/fdfs/tracker.conf
```

```shell
[root@lichcloud fastdfs]# cd /usr/bin
[root@lichcloud bin]# ll fdfs*
-rwxr-xr-x 1 root root 260972 Jan  6 09:29 fdfs_appender_test
-rwxr-xr-x 1 root root 260701 Jan  6 09:29 fdfs_appender_test1
-rwxr-xr-x 1 root root 251029 Jan  6 09:29 fdfs_append_file
-rwxr-xr-x 1 root root 253521 Jan  6 09:29 fdfs_crc32
-rwxr-xr-x 1 root root 251088 Jan  6 09:29 fdfs_delete_file
-rwxr-xr-x 1 root root 251919 Jan  6 09:29 fdfs_download_file
-rwxr-xr-x 1 root root 251629 Jan  6 09:29 fdfs_file_info
-rwxr-xr-x 1 root root 264245 Jan  6 09:29 fdfs_monitor
-rwxr-xr-x 1 root root 891607 Jan  6 09:29 fdfs_storaged
-rwxr-xr-x 1 root root 267404 Jan  6 09:29 fdfs_test
-rwxr-xr-x 1 root root 266581 Jan  6 09:29 fdfs_test1
-rwxr-xr-x 1 root root 379654 Jan  6 09:29 fdfs_trackerd
-rwxr-xr-x 1 root root 251967 Jan  6 09:29 fdfs_upload_appender
-rwxr-xr-x 1 root root 253065 Jan  6 09:29 fdfs_upload_file
[root@lichcloud bin]# fdfs_trackerd /etc/fdfs/tracker.conf
[root@lichcloud bin]#
```

重启

```shell
[root@lichcloud bin]# fdfs_trackerd /etc/fdfs/tracker.conf restart
waiting for pid [32761] exit ...
starting ...
[root@lichcloud bin]#
```

# 安装 storage 服务

## 修改 storage.conf

**base_path:基础路径**
**group_name:分组**
**store_path:文件存储路径**
**tracker_server:tracker 服务地址，我用的是云服务器，所以需要填写云服务内网地址**

```shell
# the base path to store data and log files
base_path=/fastdfs/storage


# the name of the group this storage server belongs to
#
# comment or remove this item for fetching from tracker server,
# in this case, use_storage_id must set to true in tracker.conf,
# and storage_ids.conf must be configed correctly.
group_name=lich


# store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
store_path0=/fastdfs/storage


# tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
tracker_server=172.16.0.85:22122

```

## 启动/重启 storage 服务

```java
[root@lichcloud bin]# fdfs_storaged /etc/fdfs/storage.conf
[root@lichcloud bin]#
```

重启

```shell
[root@lichcloud bin]# fdfs_storaged /etc/fdfs/storage.conf restart
waiting for pid [16884] exit ...
starting ...
[root@lichcloud bin]#
```

# 配置 client

## 修改 client.conf

```shell
# the base path to store log files
base_path=/fastdfs/client

# tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
tracker_server=172.16.0.85:22122

```

## 上传图片测试

命令：
/usr/bin/fdfs_test /etc/fdfs/client.conf upload /home/xueren.png

```shell
[root@lichcloud fdfs]# /usr/bin/fdfs_test /etc/fdfs/client.conf upload /home/xueren.png
This is FastDFS client test program v5.05

Copyright (C) 2008, Happy Fish / YuQing

FastDFS may be copied only under the terms of the GNU General
Public License V3, which may be found in the FastDFS source kit.
Please visit the FastDFS Home Page http://www.csource.org/
for more detail.

[2019-01-06 10:14:53] DEBUG - base_path=/fastdfs/client, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group:
        server 1. group_name=, ip_addr=172.16.0.85, port=23000

group_name=lich, ip_addr=172.16.0.85, port=23000
storage_upload_by_filename
group_name=lich, remote_filename=M00/00/00/rBAAVVwxZJ6AXN1xAACBjAPi9Xc274.png
source ip address: 172.16.0.85
file timestamp=2019-01-06 10:14:54
file size=33164
file crc32=65205623
example file url: http://172.16.0.85/lich/M00/00/00/rBAAVVwxZJ6AXN1xAACBjAPi9Xc274.png
storage_upload_slave_by_filename
group_name=lich, remote_filename=M00/00/00/rBAAVVwxZJ6AXN1xAACBjAPi9Xc274_big.png
source ip address: 172.16.0.85
file timestamp=2019-01-06 10:14:54
file size=33164
file crc32=65205623
example file url: http://172.16.0.85/lich/M00/00/00/rBAAVVwxZJ6AXN1xAACBjAPi9Xc274_big.png
[root@lichcloud fdfs]#
```

# 安装配置 nginx

## 解压解压软件

fastdfs-nginx-module_v1.16.tar.gz

```shell
[root@lichcloud fastDFS]# cd fastdfs-nginx-module
[root@lichcloud fastdfs-nginx-module]# ll
total 12
-rw-rw-r-- 1 500 500 2342 May  4  2014 HISTORY
-rw-rw-r-- 1 500 500 1733 May  4  2014 INSTALL
drwxrwxr-x 2 500 500 4096 May  4  2014 src
[root@lichcloud fastdfs-nginx-module]# ^C
[root@lichcloud fastdfs-nginx-module]#
```

## 修改 src 下的 conf,去掉 local

```shell
ngx_addon_name=ngx_http_fastdfs_module
HTTP_MODULES="$HTTP_MODULES ngx_http_fastdfs_module"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_fastdfs_module.c"
CORE_INCS="$CORE_INCS /usr/local/include/fastdfs /usr/local/include/fastcommon/"
CORE_LIBS="$CORE_LIBS -L/usr/local/lib -lfastcommon -lfdfsclient"
CFLAGS="$CFLAGS -D_FILE_OFFSET_BITS=64 -DFDFS_OUTPUT_CHUNK_SIZE='256*1024' -DFDFS_MOD_CONF_FILENAME='\"/etc/fdfs/mod_fastdfs.conf\"'"
```

修改后：

```shell
ngx_addon_name=ngx_http_fastdfs_module
HTTP_MODULES="$HTTP_MODULES ngx_http_fastdfs_module"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_fastdfs_module.c"
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
CORE_LIBS="$CORE_LIBS -L/usr/lib -lfastcommon -lfdfsclient"
CFLAGS="$CFLAGS -D_FILE_OFFSET_BITS=64 -DFDFS_OUTPUT_CHUNK_SIZE='256*1024' -DFDFS_MOD_CONF_FILENAME='\"/etc/fdfs/mod_fastdfs.conf\"'"
```

## 安装相关依赖

pcre: PCRE(Perl Compatible Regular Expressions)是一个 Perl 库，包括 perl 兼容的正则表达式库

zlib: zlib 是提供数据压缩用的函式库

openssl: 在计算机网络上，OpenSSL 是一个开放源代码的软件库包，应用程序可以使用这个包来进行安全通信，避免窃听，同时确认另一端连接者的身份。这个包广泛被应用在互联网的网页服务器上。

```shell
[root@lichcloud src]# yum install gcc-c++

[root@lichcloud src]# yum install pcre pcre-devel

[root@lichcloud src]# yum install zlib zlib-devel

[root@lichcloud src]# yum install openssl openssl-devel
```

## 解压 nginx 安装包

```shell
drwxr-xr-x 6 1001 1001   4096 Jan  6 10:30 auto
-rw-r--r-- 1 1001 1001 277049 Apr 12  2017 CHANGES
-rw-r--r-- 1 1001 1001 421985 Apr 12  2017 CHANGES.ru
drwxr-xr-x 2 1001 1001   4096 Jan  6 10:30 conf
-rwxr-xr-x 1 1001 1001   2481 Apr 12  2017 configure
drwxr-xr-x 4 1001 1001   4096 Jan  6 10:30 contrib
drwxr-xr-x 2 1001 1001   4096 Jan  6 10:30 html
-rw-r--r-- 1 1001 1001   1397 Apr 12  2017 LICENSE
drwxr-xr-x 2 1001 1001   4096 Jan  6 10:30 man
-rw-r--r-- 1 1001 1001     49 Apr 12  2017 README
drwxr-xr-x 9 1001 1001   4096 Jan  6 10:30 src
[root@lichcloud nginx-1.12.0]#
```

## 修改 config 配置

**注意修改--add-module=/home/soft/fastDFS/fastdfs-nginx-module/src**

```shell
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi \
--add-module=/home/soft/fastDFS/fastdfs-nginx-module/src
```

## 在 nginx config 下执行即可

```shell
drwxr-xr-x 6 1001 1001   4096 Jan  6 10:30 auto
-rw-r--r-- 1 1001 1001 277049 Apr 12  2017 CHANGES
-rw-r--r-- 1 1001 1001 421985 Apr 12  2017 CHANGES.ru
drwxr-xr-x 2 1001 1001   4096 Jan  6 10:30 conf
-rwxr-xr-x 1 1001 1001   2481 Apr 12  2017 configure
drwxr-xr-x 4 1001 1001   4096 Jan  6 10:30 contrib
drwxr-xr-x 2 1001 1001   4096 Jan  6 10:30 html
-rw-r--r-- 1 1001 1001   1397 Apr 12  2017 LICENSE
drwxr-xr-x 2 1001 1001   4096 Jan  6 10:30 man
-rw-r--r-- 1 1001 1001     49 Apr 12  2017 README
drwxr-xr-x 9 1001 1001   4096 Jan  6 10:30 src
[root@lichcloud nginx-1.12.0]# ./configure \
> --prefix=/usr/local/nginx \
> --pid-path=/var/run/nginx/nginx.pid \
> --lock-path=/var/lock/nginx.lock \
> --error-log-path=/var/log/nginx/error.log \
> --http-log-path=/var/log/nginx/access.log \
> --with-http_gzip_static_module \
> --http-client-body-temp-path=/var/temp/nginx/client \
> --http-proxy-temp-path=/var/temp/nginx/proxy \
> --http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
> --http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
> --http-scgi-temp-path=/var/temp/nginx/scgi \
> --add-module=/home/soft/fastDFS/fastdfs-nginx-module/src
checking for OS
 + Linux 2.6.32-696.16.1.el6.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 4.4.7 20120313 (Red Hat 4.4.7-23) (GCC)
checking for gcc -pipe switch ... found
checking for -Wl,-E switch ... found
checking for gcc builtin atomic operations ... found
checking for C99 variadic macros ... found
checking for gcc variadic macros ... found
checking for gcc builtin 64 bit byteswap ... found
checking for unistd.h ... found
checking for inttypes.h ... found
checking for limits.h ... found
checking for sys/filio.h ... not found
checking for sys/param.h ... found
checking for sys/mount.h ... found
checking for sys/statvfs.h ... found
checking for crypt.h ... found
checking for Linux specific features
checking for epoll ... found
checking for EPOLLRDHUP ... found
checking for EPOLLEXCLUSIVE ... not found
checking for O_PATH ... not found
checking for sendfile() ... found
checking for sendfile64() ... found
checking for sys/prctl.h ... found
checking for prctl(PR_SET_DUMPABLE) ... found
checking for sched_setaffinity() ... found
checking for crypt_r() ... found
checking for sys/vfs.h ... found
checking for nobody group ... found
checking for poll() ... found
checking for /dev/poll ... not found
checking for kqueue ... not found
checking for crypt() ... not found
checking for crypt() in libcrypt ... found
checking for F_READAHEAD ... not found
checking for posix_fadvise() ... found
checking for O_DIRECT ... found
checking for F_NOCACHE ... not found
checking for directio() ... not found
checking for statfs() ... found
checking for statvfs() ... found
checking for dlopen() ... not found
checking for dlopen() in libdl ... found
checking for sched_yield() ... found
checking for SO_SETFIB ... not found
checking for SO_REUSEPORT ... found
checking for SO_ACCEPTFILTER ... not found
checking for SO_BINDANY ... not found
checking for IP_BIND_ADDRESS_NO_PORT ... not found
checking for IP_TRANSPARENT ... found
checking for IP_BINDANY ... not found
checking for IP_RECVDSTADDR ... not found
checking for IP_PKTINFO ... found
checking for IPV6_RECVPKTINFO ... found
checking for TCP_DEFER_ACCEPT ... found
checking for TCP_KEEPIDLE ... found
checking for TCP_FASTOPEN ... not found
checking for TCP_INFO ... found
checking for accept4() ... found
checking for eventfd() ... found
checking for int size ... 4 bytes
checking for long size ... 8 bytes
checking for long long size ... 8 bytes
checking for void * size ... 8 bytes
checking for uint32_t ... found
checking for uint64_t ... found
checking for sig_atomic_t ... found
checking for sig_atomic_t size ... 4 bytes
checking for socklen_t ... found
checking for in_addr_t ... found
checking for in_port_t ... found
checking for rlim_t ... found
checking for uintptr_t ... uintptr_t found
checking for system byte ordering ... little endian
checking for size_t size ... 8 bytes
checking for off_t size ... 8 bytes
checking for time_t size ... 8 bytes
checking for AF_INET6 ... found
checking for setproctitle() ... not found
checking for pread() ... found
checking for pwrite() ... found
checking for pwritev() ... found
checking for sys_nerr ... found
checking for localtime_r() ... found
checking for posix_memalign() ... found
checking for memalign() ... found
checking for mmap(MAP_ANON|MAP_SHARED) ... found
checking for mmap("/dev/zero", MAP_SHARED) ... found
checking for System V shared memory ... found
checking for POSIX semaphores ... not found
checking for POSIX semaphores in libpthread ... found
checking for struct msghdr.msg_control ... found
checking for ioctl(FIONBIO) ... found
checking for struct tm.tm_gmtoff ... found
checking for struct dirent.d_namlen ... not found
checking for struct dirent.d_type ... found
checking for sysconf(_SC_NPROCESSORS_ONLN) ... found
checking for openat(), fstatat() ... found
checking for getaddrinfo() ... found
configuring additional modules
adding module in /home/soft/fastDFS/fastdfs-nginx-module/src
 + ngx_http_fastdfs_module was configured
checking for PCRE library ... found
checking for PCRE JIT support ... not found
checking for zlib library ... found
creating objs/Makefile

Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/var/run/nginx/nginx.pid"
  nginx error log file: "/var/log/nginx/error.log"
  nginx http access log file: "/var/log/nginx/access.log"
  nginx http client request body temporary files: "/var/temp/nginx/client"
  nginx http proxy temporary files: "/var/temp/nginx/proxy"
  nginx http fastcgi temporary files: "/var/temp/nginx/fastcgi"
  nginx http uwsgi temporary files: "/var/temp/nginx/uwsgi"
  nginx http scgi temporary files: "/var/temp/nginx/scgi"

[root@lichcloud nginx-1.12.0]#
```

## 配置完毕后，直接 make 编译，make install 安装

安装完毕后，安装目录在/usr/local/nginx,复制 fastdfs-nginx-module/src/mod_fastdfs.conf 配置文件到 etc/fdfs 下面

```shell
[root@lichcloud src]# cp mod_fastdfs.conf /etc/fdfs/
[root@lichcloud src]#
```

## 修改 mod_fastdfs.conf 配置文件

base_path: 文件目录
tracker_server: tracker_server 地址
group_name: group 分组名字

```shell
# the base path to store log files
base_path=/fastdfs/tmp

# FastDFS tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
# valid only when load_fdfs_parameters_from_tracker is true
tracker_server=172.16.0.85:22122

# the group name of the local storage server
group_name=lich

# if the url / uri including the group name
# set to false when uri like /M00/00/00/xxx
# set to true when uri like ${group_name}/M00/00/00/xxx, such as group1/M00/xxx
# default value is false
url_have_group_name = true


# store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
# must same as storage.conf
store_path0=/fastdfs/storage
#store_path1=/home/yuqing/fastdfs1
```

## 修改 nginx.conf

```shell
/usr/local/nginx/conf
[root@lichcloud conf]# vim nginx.conf
```

增加一个 server，我是放在云服务器上，所以这里目前要写公网的 ip

```shell
 server{

        listen          88;
        server_name     114.116.88.220;

        location        /lich/M00{

                ngx_fastdfs_module;

                }

        }
```

## ngix 启动检查

```shell
-rwxr-xr-x 1 root root 3709238 Jan  6 10:35 nginx
[root@lichcloud sbin]# ./nginx -t
ngx_http_fastdfs_set pid=7681
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: [emerg] mkdir() "/var/temp/nginx/client" failed (2: No such file or directory)
nginx: configuration file /usr/local/nginx/conf/nginx.conf test failed
[root@lichcloud sbin]#

```

报错创建路径即可

```shell
[root@lichcloud sbin]# mkdir /var/temp/nginx/client -p
[root@lichcloud sbin]#
```

再次检查

```shell
[root@lichcloud sbin]# ./nginx -t
ngx_http_fastdfs_set pid=8843
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@lichcloud sbin]#
```

## 启动 nginx

```shell
./nginx
```

重载

```shell
[root@lichcloud sbin]# ./nginx -s reload
```

浏览器访问 nginx 测试
![](images/20200715134504.png)

访问图片地址：
http://114.116.88.220:88/lich/M00/00/00/rBAAVVwxZJ6AXN1xAACBjAPi9Xc274.png
![](https://blog.lichenghao.cn/upload/2022/07/20200715134638.png)

**如果访问不到，注意查看云服务器的安全组，端口是否开放**

# 配置开机自启动

```shell
vim /etc/rc.d/rc.local
```

添加下列脚本

```shell
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart

/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart

/usr/local/nginx/sbin/nginx
```

**注：如果 tracker 和 storage 安装在不同的位置，则需要在不同位置的文件中添加脚本**
