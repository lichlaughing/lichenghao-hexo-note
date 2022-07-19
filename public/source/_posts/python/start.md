---
title: Python 环境安装及基础操作
tags:
  - python
categories: python
keywords: python
abbrlink: f0754d4e
date: 2022-07-15 10:08:39
cover: https://blog.lichenghao.cn/upload/2022/07/3F93FBD9-python.png
---

## 安装 Python

Python下载：https://www.python.org/downloads/ 安装稳定版本。

修改pip源：

国内的镜像源有很多

```properties
清华大学 ：https://pypi.tuna.tsinghua.edu.cn/simple/
阿里云：http://mirrors.aliyun.com/pypi/simple/
中国科学技术大学 ：http://pypi.mirrors.ustc.edu.cn/simple/
华中科技大学：http://pypi.hustunique.com/
豆瓣源：http://pypi.douban.com/simple/
腾讯源：http://mirrors.cloud.tencent.com/pypi/simple
华为镜像源：https://repo.huaweicloud.com/repository/pypi/simple/
```



{% tabs pip %}
<!-- tab window -->

- 在 C:\Users\用户名 目录下创建pip文件夹
- 在pip文件夹下创建一个txt文件，更名为pip.ini
- 文件夹内容如下所示：

```properties
[global]
index-url=https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=https://pypi.tuna.tsinghua.edu.cn/simple
```

验证下结果：

```bash
C:\Users\chenghao.li>pip config list
global.index-url='https://pypi.tuna.tsinghua.edu.cn/simple'
install.trusted-host='https://pypi.tuna.tsinghua.edu.cn/simple'
```

<!-- endtab -->

<!-- tab linux -->

Mac 版本修改也和下面的一样。

- 在用下创建 .pip 文件夹
- 在 .pip 文件夹下创建配置文件 pip.conf
- 配置文件内容如下：

```properties
[global]
index-url=https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=https://pypi.tuna.tsinghua.edu.cn/simple
```

验证结果：

```bash
➜  ~ pip config list
global.index-url='https://pypi.tuna.tsinghua.edu.cn/simple'
install.trusted-host='https://pypi.tuna.tsinghua.edu.cn/simple'
```

<!-- endtab -->


{% endtabs %}

如果不设置 pip 源也可以在每次安装模块的时候指定也可以，这使用起来就比较麻烦了。

```shell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host=https://pypi.tuna.tsinghua.edu.cn/simple requests
```



## 安装开发环境

可以使用 Python 自带的 IDLE，或者使用 VSCODE 或者使用 PyCharm

## Hello World

```python
print("hello world")
```





