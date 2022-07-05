---
title: centos7-常用命令
categories: linux
tags:
  - linux
abbrlink: bdf70cd5
date: 2022-04-01 12:00:00
---
Linux 常用的一些命令整理，例如查看日志，查看文件内容，编辑文件等等

## 用户管理

查看系统有哪些用户 ( 用户基本信息存放在/etc/passwd文件中 )

```shell
[root@lch ~]# cut -d : -f 1 /etc/passwd
root
bin
daemon
......
```



{% tabs user %}

<!-- tab 添加用户 -->

添加用户：useradd [参数] 用户名

```shell
[root@lch ~]# useradd libai
```

可用参数：

| 参数 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| -c   | 添加备注文字                                                 |
| -d   | 新用户每次登陆时所使用的家目录                               |
| -e   | 用户终止日期，日期的格式为YYYY-MM-DD                         |
| -f   | 用户过期几日后永久停权。当值为0时用户立即被停权，而值为-1时则关闭此功能，预设值为-1 |
| -g   | 指定用户对应的用户组，不指定组会默认创建和用户名相同的组     |
| -G   | 定义此用户为多个不同组的成员                                 |
| -m   | 用户目录不存在时则自动创建                                   |
| -M   | 不建立用户家目录，优先于/etc/login.defs文件设定              |
| -n   | 取消建立以用户名称为名的群组                                 |
| -r   | 建立系统帐号                                                 |
| -u   | 指定用户id                                                   |

<!-- endtab -->

<!-- tab 删除用户 -->

删除用户： userdel [参数] 用户名

```shell
[root@lch ~]# userdel libai
```

可选参数：

| 参数 | 说明                           |
| ---- | ------------------------------ |
| -f   | 强制删除用户账号               |
| -r   | 删除用户主目录及其中的任何文件 |
| -h   | 显示命令的帮助信息             |

<!-- endtab -->

<!-- tab 修改用户密码 -->

修改用户密码： passwd [参数] 用户名

```shell
[root@lch ~]# passwd libai
更改用户 libai 的密码 。
新的 密码：
无效的密码： 密码未通过字典检查 - 过于简单化/系统化
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
```

可用参数：

| 参数 | 说明                         |
| ---- | ---------------------------- |
| -d   | 删除已有密码                 |
| -l   | 锁定用户的密码值，不允许修改 |
| -u   | 解锁用户的密码值，允许修改   |
| -e   | 下次登陆强制修改密码         |
| -k   | 用户在期满后能仍能使用       |
| -S   | 查询密码状态                 |

<!-- endtab -->

{% endtabs %}





## 查看文件



{% tabs look %}

<!-- tab head -->

`head fileName`：从头查看文件，默认会显示10行 

```shell
[root@VM-0-2-centos arthas]# head arthas.log 
Arthas server agent start...
2021-06-19 17:37:31 [arthas-binding-thread] INFO  c.t.arthas.core.util.ArthasBanner -Current arthas version: 3.5.1, recommend latest version: 3.5.1
2021-06-19 17:37:31 [arthas-binding-thread] INFO  c.t.arthas.core.util.ArthasBanner -Current arthas version: 3.5.1, recommend latest version: 3.5.1
....
```

`head -n num fileName`：通过参数指定显示前N行 

```shell
[root@VM-0-2-centos arthas]# head -n 2 arthas.log 
Arthas server agent start...
2021-06-19 17:37:31 [arthas-binding-thread] INFO  c.t.arthas.core.util.ArthasBanner -Current arthas version: 3.5.1, recommend latest version: 3.5.1
```

`head -n -num fileName`：如果参数为负数表示后N行以外的所有行

```shell
[root@VM-0-2-centos arthas]# head -n -10 arthas.log 
```

<!-- endtab -->

<!-- tab tail -->

`tail fileName`：从文件末尾开始查看文件，默认会显示后10行

```shell
# 查看文件的后10行内容
[root@VM-0-2-centos arthas]# tail arthas.log 
2021-06-19 17:49:49 [arthas-command-execute] INFO  c.t.arthas.core.advisor.Enhancer -enhance matched classes: [class Test03]
.....
```

` tail -n num fileName 和 tail -n -num fileName 一样`： 表示最后num行数据

```shell
# 查看文件的后两行内容
[root@VM-0-2-centos arthas]# tail -n 2 arthas.log 
2021-06-19 17:57:27 [arthas-NettyWebsocketTtyBootstrap-4-1] INFO  c.a.a.d.i.n.h.logging.LoggingHandler -[id: 0x2c0166e2, L:/127.0.0.1:8563] UNREGISTERED
2021-06-19 17:57:27 [as-shutdown-hooker] INFO  c.t.a.core.server.ArthasBootstrap -as-server destroy completed.
```

`tail -n +num fileName`： 如果参数num前面有+的话表示：文件从num行开始后的所有行（包括num行）

```shell
# 查看文件从第10行开始到结束的内容
[root@VM-0-2-centos arthas]# tail -n +10 arthas.log 
```

`tail -n num -f fileName` :  如果当文件内容增加后，自动显示最新的文件内容

```shell
# 查看后100行内容，有新增更新
[root@VM-0-2-centos arthas]# tail -n 100 -f arthas.log 
```

<!-- endtab -->

<!-- tab more -->

more 命令可以一页一页的查看文件，通过 空格(space)下一页 和 b 上一页

```sh
[root@localhost arthas]# more arthas.log 
```

常用参数：

| 参数说明                                         | 示例                                                 |
| ------------------------------------------------ | ---------------------------------------------------- |
| -num 一次显示的行数                              | [root@localhost arthas]# more -10 arthas.log         |
| +num 从第几行开始显示                            | [root@localhost arthas]# more +100 arthas.log        |
| -c 从顶部清屏开始显示                            | [root@localhost arthas]# more -c arthas.log          |
| -s 多个空行合并成一行                            | [root@localhost arthas]# more -c -s arthas.log       |
| +/pattern 寻该字串然后从该字串前两行之后开始显示 | [root@localhost arthas]# more +/completed arthas.log |

除了显示文档内容，more还可以配合grep查找文档的内容

```sh
[root@localhost arthas]# more +10 arthas.log | grep -i success
```

<!-- endtab -->

<!-- tab less -->

less 与 more 类似，less 可以随意浏览文件，支持翻页和搜索，支持向上翻页和向下翻页。

- -m 显示类似more命令的百分比
- -N 显示每行的行号

- 空格键 滚动一页
- 回车键 滚动一行
- [pagedown]： 向下翻动一页
- [pageup]： 向上翻动一页
- q：退出

```bash
[root@VM-16-3-centos log]# less yum.log
```

```bash
# 显示百分比,行号
[root@VM-16-3-centos log]# less -m -N yum.log
....
     20 Feb 14 14:38:01 Installed: libidn-devel-1.28-4.el7.x86_64
     21 Feb 14 14:38:21 Installed: kernel-devel-3.10.0-1160.53.1.el7.x86_64
     22 Feb 14 14:38:21 Installed: 1:gmp-devel-6.0.0-15.el7.x86_64
yum.log 29%

```

<!-- endtab -->

<!-- tab cat -->

cat 命令用于连接文件并打印到标准输出设备上。

比如将文件显示行号然后输出

```sh
[root@localhost arthas]# cat -n arthas.log 
```

其他参数：

| 参数说明                                | 示例                                       |
| --------------------------------------- | ------------------------------------------ |
| -n 从1开始每行编号                      | [root@localhost arthas]# cat -n arthas.log |
| -b 同-n，但是空行不编号                 | [root@localhost arthas]# cat -b arthas.log |
| -s 连续两行以上的空白行换为一行的空白行 | [root@localhost arthas]# cat -s arthas.log |

除了查看文件外，还可以

把文档A内容加上行号后输入B这个文档里：

```sh
[root@localhost lich]# cat -n A.txt > B.txt
```

清空某个文件

```sh
[root@localhost lich]# cat /dev/null > B.txt 
```

<!-- endtab -->

{% endtabs %}



## Vim

一般的命令大家都知道，比如一般的 `vim hello.txt` 进入一般模式，然后 `i`进入编辑模式，然后 `shift+:`进入命令模式，然后 `wq`保存并关闭，或者`!q`强制关闭等等,这些都是经常使用的。



{% tabs vim %}

<!-- tab 光标操作 -->


常用

| 按键 | 功能                   |
| ---- | ---------------------- |
| gg   | 移动光标至文件首行     |
| G    | 移动光标至文件末尾     |
| ^    | 光标移至当前行的首字符 |
| $    | 光标移至当前行的尾字符 |

<!-- endtab -->

<!-- tab 复制粘贴 -->

复制一行 

在一般模式下，按两次 y,就可以复制光标所在的一行，在光标所在的位置按 p 就可以粘贴方才复制的这一行。

![](https://blog.lichenghao.cn/upload/2022/07/18015-02.gif)


复制多行 

在 yy之前输入要复制的行数，比如 按完 5 再按 yy 表示复制从光标开始的五行数据，然后 p 粘贴就好了。

如果没有 5 行，就有几行就复制几行

![](https://blog.lichenghao.cn/upload/2022/07/18015-03.gif)

<!-- endtab -->

<!-- tab 删除 -->


 删除一行 
在光标所在的行，dd 就可以删除这一行

 删除多行 
在光标所在的行，5+dd 就可以删除5 行

<!-- endtab -->

<!-- tab 查找 -->

在命令行模式下 `:/查找的东西` , n下一个，N 上一个。这里可以使用通配符比如：*keyword等

<!-- endtab -->

<!-- tab 修改 -->

修改

i 进入修改模式

撤销

和 windows 中的 ctrl+z 功能一样。在一般模式下按 `u` 就可以了。

<!-- endtab -->

<!-- tab 显示行号 -->

显示/取消 行号

: set nu / :set nonu

<!-- endtab -->

{% endtabs %}



------


快捷键非常多，充分的利用了键盘上的每一个键位，官网给出了键位图，是英文的，有网友给翻译成了中文如下所示：

![](https://blog.lichenghao.cn/upload/2022/07/18015-04.png)





## Tcpdump

### 过滤器

{% tabs 过滤器 %}

<!-- tab Host过滤 -->

- 过滤某个主机的数据报文，该命令会抓取所有发往主机 `2.2.2.2` 或者从主机 `2.2.2.2` 发出的流量

```bash
[root@localhost ~]# tcpdump host 2.2.2.2
```

- 抓取eth0网卡，来自指定主机 upd 协议的数据 。（**如果不指定网卡那么默认取第一个网卡，不是所有网卡**）

```bash
[root@localhost ~]# tcpdump -i eth0 -n udp and src host 2.2.2.2
```

- 抓取所有送到指定主机的数据包

```bash
[root@localhost ~]# tcpdump -i eth0 dst host 2.2.2.2
```

- 截获主机192.168.4.56 和 主机185.222.58.151 或 185.222.58.151的通信。如果命令有问题可以给括号加上`\`转义

```bash
[root@localhost ~]# tcpdump host 192.168.4.56 and (210.27.48.2 or 210.27.48.3)
```

<!-- endtab -->

<!-- tab Network 过滤器 -->

Network 过滤器用来过滤某个网段的数据，可以使用四元组（x.x.x.x）、三元组（x.x.x）、二元组（x.x）和一元组（x）

抓取所有发往网段 `192.168.2.x` 或从网段 `192.168.2.x` 发出的流量

```sh
[root@localhost ~]# tcpdump net 192.168.2
```

同样可以指定源和目的

```sh
[root@localhost ~]# tcpdump src net 10
或者
[root@localhost ~]# tcpdump src net 172.16.0.0/12
```

<!-- endtab -->

{% endtabs %}



### 结果写入文件

写入文件后，就可以通过 [Wireshark ](https://www.wireshark.org/download.html)工具来分析流量数据，很方便。

- 抓取来自指定主机指定数量的数据包，通过参数 -c  来指定抓取的数量。

```sh
[root@localhost ~]# tcpdump -i eth0 -n -c 10 udp and src host 192.168.4.11
```

- 将数据写入到文件，通过wireshark 分析内容。

```sh
[root@localhost ~]# tcpdump -w dd.pacp -i eth0 -n -c 10 udp and src host 192.168.4.11
```

- 抓取目的ip port 端口的数据

```sh
[root@localhost ~]# tcpdump -w dataDst.pcap -i eth0 dst host ip and port port
```



参数说明：

| 参数 | 说明                      | 示例                                                         |
| ---- | ------------------------- | ------------------------------------------------------------ |
| i    | 指定网卡                  | [root@localhost ~]# tcpdump -i eth0                          |
| -n   | 将dns名称转化成ip形式     | [root@localhost ~]# tcpdump -w dd.pacp -i eth0 -n            |
| udp  | 指定协议，tcp 或者 udp    | [root@localhost ~]# tcpdump -i eth0 -n udp                   |
| host | 指定主机                  | [root@localhost ~]# tcpdump host 2.2.2.2                     |
| src  | 指定来源，指定目的则为dst | [root@localhost ~]# tcpdump -i eth0 -n udp and src host 2.2.2.2 |
| -c   | 指定抓取的数量            | [root@localhost ~]# tcpdump -w dd.pacp -i eth0 -n -c 10      |
| -w   | 写入文件                  | [root@localhost ~]# tcpdump -w dd.pacp -i eth0 -n -c 10      |



## Netstat

用于显示和网络相关的各种信息，例如网卡信息，端口监听，协议统计信息等等

例如：列出所有的端口信息

```sh
[root@localhost ~]# netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:sunrpc          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:mountd          0.0.0.0:*               LISTEN   
......
```

- 常见参数：

| 参数说明                                                     | 示例                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| -a (all)显示所有选项，默认不显示LISTEN相关                   | **[root@localhost ~]# netstat -a**<br/>Active Internet connections (servers and established)<br/>Proto Recv-Q Send-Q Local Address           Foreign Address         State      <br/>tcp        0      0 0.0.0.0:sunrpc          0.0.0.0:*               LISTEN     <br/>tcp        0      0 0.0.0.0:mountd          0.0.0.0:*               LISTEN   <br/> |
| -t (tcp)仅显示tcp相关选项                                    | **[root@localhost ~]# netstat -t**<br/>Active Internet connections (w/o servers)<br/>Proto Recv-Q Send-Q Local Address           Foreign Address         State      <br/>tcp        0      0 localhost:38426         localhost:websm         ESTABLISHED ...... |
| -u (udp)仅显示udp相关选项                                    | **[root@localhost ~]# netstat -u**<br/>Active Internet connections (w/o servers)<br/>Proto Recv-Q Send-Q Local Address           Foreign Address         State      <br/>udp        0      0 localhost.localdo:34387 dns.google:domain       ESTABLISHED |
| -n  直接使用IP地址，而不通过域名服务器，<br />右侧示例：显示tcp相关信息 | **[root@localhost ~]# netstat -tn**<br/>Active Internet connections (w/o servers)<br/>Proto Recv-Q Send-Q Local Address           Foreign Address         State      <br/>tcp        0    168 192.168.4.193:22        192.168.4.56:53580      ESTABLISHED |
| -l 仅列出有在 Listen (监听) 的服务                           | **[root@localhost ~]# netstat -l**<br/>Active Internet connections (only servers)<br/>Proto Recv-Q Send-Q Local Address           Foreign Address         State      <br/>tcp        0      0 0.0.0.0:sunrpc          0.0.0.0:*               LISTEN |
| -p 显示建立相关链接的PID和程序名称<br />如右侧：显示正在监听的程序 | **[root@localhost ~]# netstat -lp**<br/>Active Internet connections (only servers)<br/>Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    <br/>tcp        0      0 0.0.0.0:sunrpc          0.0.0.0:*               LISTEN      1/systemd |





- 常见的参数组合查询：

| 说明                                         | 示例                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| 列出所有 tcp 端口                            | **[root@localhost ~]# netstat -at**<br/>Active Internet connections (servers and established)<br/>Proto Recv-Q Send-Q Local Address           Foreign Address         State      <br/>tcp        0      0 0.0.0.0:sunrpc          0.0.0.0:*               LISTEN |
| 列出所有 udp 端口                            | [root@localhost ~]# netstat -au<br/>Active Internet connections (servers and established)<br/>Proto Recv-Q Send-Q Local Address           Foreign Address         State      <br/>udp        0      0 0.0.0.0:35318           0.0.0.0:* |
| 列出所有监听 tcp程序                         | **[root@localhost ~]# netstat -ltp**<br/>Active Internet connections (only servers)<br/>Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    <br/>tcp        0      0 0.0.0.0:sunrpc          0.0.0.0:*               LISTEN      1/systemd |
| 查看程序端口，同时也能看到程序是否正常运行了 | **[root@localhost ~]# netstat -anp** <br />tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2286/sshd |
| 网卡列表                                     | **[root@localhost ~]# netstat -i**<br/>Kernel Interface table<br/>Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg<br/>docker0   1500        0      0      0 0             0      0      0      0 BMU<br/>eth0      1500 96978401      5 403802 0      24447250      0      0      0 BMRU<br/>eth1      1500        0      0      0 0             0      0      0      0 BMU<br/>lo       65536 135319470      0      0 0      135319470      0      0      0 LRU |



## 解压缩

### zip & unzip

```shell
# 压缩并指定目录 
[root@lch ~]# zip -r /usr/local/hello.zip /usr/local/hello
# 解压并指定目录 
[root@lch ~]# unzip /usr/local/hello.zip -d /usr/local/hello
```

### tar

```shell
# 解压文件到当前目录
[root@lch software]# tar -zxvf nacos-server-1.1.4.tar.gz 
# 解压文件到指定目录
[root@lch software]# tar -zxvf nacos-server-1.1.4.tar.gz -C /root/
```



## 防火墙（firewall）

查看规则列表

```shell
[root@localhost ~]# firewall-cmd --zone=public --list-ports
3306/tcp 4321/tcp 80/tcp 1234/tcp
```

放行端口

```shell
# --permanent 永久放行
[root@localhost ~]# firewall-cmd --zone=public --add-port=1234/tcp --permanent
# 重载后生效
[root@localhost ~]# firewall-cmd --reload
```







