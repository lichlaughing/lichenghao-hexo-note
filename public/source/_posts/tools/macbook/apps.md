---
title: Macbook Pro 推荐 APP
categories: macbook
tags:
  - macbook
  - app
keywords: macbook
abbrlink: 33a0b12c
date: 2022-05-25 12:30:00
cover: https://blog.lichenghao.cn/upload/2022/07/1B1E0B07-macbook.jpg
---
必备 APP

## VMware Fusion

Mac 下使用VMware Fusion ，安装 Centos7，并且配置静态 IP 上网。

现在安装过程省略，网上教程很多。安装完毕后，接下来配置网络NAT 模式上网。

安装完毕后，一般情况下默认就是 NAT 模式，如果不是的话，点击窗口上方的按钮设置为 NAT 模式。

同时将网络适配器设置为"与我的 Mac 共享"。

![](https://blog.lichenghao.cn/upload/2022/07/112011.png)



进入 VM 的配置文件目录，`/Library/Preferences/VMware Fusion/vmnet8`

```bash
➜  vmnet8 ls
dhcpd.conf     dhcpd.conf.bak nat.conf       nat.conf.bak   nat.mac
```

查看 nat.conf 配置文件，可以得到 gateway ip和 netmask后面需要用到。

``` bash
➜  vmnet8 cat nat.conf
[host]

# Use MacOS network virtualization API
useMacosVmnetVirtApi = 1

# NAT gateway address
ip = 172.16.127.1  
netmask = 255.255.255.0
```

然后登录虚拟机，进入网卡配置目录

```bash
[root@localhost ~]# cd /etc/sysconfig/network-scripts/
[root@localhost network-scripts]# ls
ifcfg-ens33  ifdown-isdn      ifdown-tunnel  ifup-isdn    ifup-Team
......
```

修改 ifcfg-ens33 如下所示：

```properties
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static  //修改为 static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=9cd8fc87-2d21-48a3-9546-4a7788889dd8
DEVICE=ens33
ONBOOT=yes  // 自启动
IPADDR=172.16.127.3 // 除了172.16.127.1 外，172.16.127.x 一般都可以
GATEWAY=172.16.127.1 // 上面查询到的
NETMASK=255.255.255.0 // 上面查询到的
```

> [root@localhost network-scripts]# ifconfig
> -bash: ifconfig: 未找到命令
>
> 查看下网络配置，因为缺少工具，`net-tools.x86_64`。网络配置完毕后，可以 yum 安装下

然后重启网络即可

```bash
[root@localhost network-scripts]# service network restart;
Restarting network (via systemctl):                        [  确定  ]
```

测试下：

```bash
[root@localhost network-scripts]# ping baidu.com
PING baidu.com (220.181.38.251) 56(84) bytes of data.
64 bytes from 220.181.38.251 (220.181.38.251): icmp_seq=1 ttl=42 time=16.1 ms
64 bytes from 220.181.38.251 (220.181.38.251): icmp_seq=2 ttl=42 time=14.6 ms
.....
```

安装工具net-tools.x86_64。

```bash
[root@localhost network-scripts]# yum install -y net-tools
.....

[root@localhost network-scripts]# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.127.3  netmask 255.255.255.0  broadcast 172.16.127.255
        inet6 fd15:4ba5:5a2b:1008:7edf:f1ba:f599:7352  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::e86:bbff:8ea0:8b  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:3d:1c:e1  txqueuelen 1000  (Ethernet)
        RX packets 53537  bytes 30257363 (28.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 42318  bytes 5227531 (4.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

ssh工具连接，如果连接失败，一定是防火墙开着，放开端口或者关闭防火墙即可。





## zsh 打造个性化终端

### 安装

建议使用 iterm2 替换系统默认的终端：https://iterm2.com/

linux 下的命令解释器有很多，默认是 bash，除此之外还有 csh dash ksh sh zsh ... 通过命令可以查看系统当前有的命令解释器

```bash
cat /etc/shells 

/bin/bash
/bin/csh
/bin/dash
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```

查看当前系统启用的命令解释器

```bash
~ echo $SHELL
/bin/zsh
```

其中 zsh 最为好用，同时有个开源的项目对其增强了，那就是 `on my zsh` ,项目地址：https://github.com/ohmyzsh/ohmyzsh

![](https://blog.lichenghao.cn/upload/2022/07/ohmyzsh.png)

拥有众多的主题和插件等，灰常强大👍🏻，看项目的 Star就知道这个项目有多牛逼 o(￣▽￣)ｄ！



安装

```bash
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

或者

$ sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

> 如果慢的话，通过其他方式下载下来，执行其中的 install 即可。



将项目中的`.zshrc` 复制到我们用户目录下一份，复制完毕后如下所示（安装后存在的话就不用了）

```bash
➜  ~ ll -a
-rw-r--r--   1 lichenghao  staff   3.9K  1 21 22:40 .zshrc
```



更改默认的命令解释器

```bash
chsh -s /bin/zsh
```



### 修改主题

官方给的主题地址：https://github.com/ohmyzsh/ohmyzsh/wiki/Themes

修改配置文件，对应的主题名称即可

```xml
# Set name of the theme to load --- if set to "random", it will
# load a random theme each time oh-my-zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
ZSH_THEME="robbyrussell"
```



安装其他主题

例如：https://github.com/romkatv/powerlevel10k  官方给的安装文档很清楚

第一步 安装字体：MesloLGS NF Regular.ttf

第二步 安装主题：powerlevel10k  Oh My Zsh 方式安装    https://github.com/romkatv/powerlevel10k#oh-my-zsh

第三步 配置主题：p10k configure

最终结果样式：

![](https://blog.lichenghao.cn/upload/2022/07/27111441.png)



### 个性化插件

官方插件列表：https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins

推荐其他插件，安装给的文档安装即可。

 `zsh-autosuggestions`  GitHub  https://github.com/zsh-users/zsh-autosuggestions

`zsh-syntax-highlighting` GitHub  https://github.com/zsh-users/zsh-syntax-highlighting





## Alfred

本地搜索及应用快速启动工具，支持自定义开发 workflow。总之很强大。

官网：https://www.alfredapp.com/

Workflow 站点：

- https://www.alfredworkflows.store/workflows
- https://www.alfredforum.com/
- https://www.packal.org/





















