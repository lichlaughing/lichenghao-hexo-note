---
title: Git 常用命令
categories:
  - tools
  - git
tags:
  - git
keywords: git
abbrlink: 5fddf106
date: 2022-05-26 12:40:00
cover: https://blog.lichenghao.cn/upload/2022/07/73239168-git.png
---
## 常用命令

| 命令名称                             | 作用           |
| ------------------------------------ | -------------- |
| git config --global user.name 用户名 | 设置用户签名   |
| git config --global user.email 邮箱  | 设置用户签名   |
| git init                             | 初始化本地库   |
| git status                           | 查看本地库状态 |
| git add 文件名                       | 添加到暂存区   |
| git commit -m "日志信息" 文件名      | 提交到本地库   |
| git reflog                           | 查看历史记录   |
| git reset --hard 版本号              | 版本穿梭       |

### 设置用户签名

```sh
git config --global user.name 用户名
git config --global user.email 邮箱
```

### 查看本地库状态

空的本地库查看

```sh
➜  git_test git:(master) git status
位于分支 master

尚无提交

无文件要提交（创建/拷贝文件并使用 "git add" 建立跟踪）
```

有未提交的文件

```sh
➜  git_test git:(master) git status
位于分支 master

尚无提交

未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）
	hello.txt

提交为空，但是存在尚未跟踪的文件（使用 "git add" 建立跟踪）
```

有添加到暂存区的文件

```sh
➜  git_test git:(master) ✗ git status
位于分支 master

尚无提交

要提交的变更：
  （使用 "git rm --cached <文件>..." 以取消暂存）
	新文件：   hello.txt
```

### 提交文件查看状态

```sh
git_test git:(master) ✗ git commit -m "first commit" hello.txt
[master（根提交） eeb90bb] first commit
 1 file changed, 4 insertions(+)
 create mode 100644 hello.txt
➜  git_test git:(master)
```

```sh
➜  git_test git:(master) ✗ git status
位于分支 master
无文件要提交，干净的工作区
```

### 修改文件查看状态

```sh
➜  git_test git:(master) ✗ git status
位于分支 master
尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git restore <文件>..." 丢弃工作区的改动）
	修改：     hello.txt

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
```

提交后查看状态

```sh
➜  git_test git:(master) ✗ git commit -m "second commit" hello.txt
[master d32d8e4] second commit
 1 file changed, 2 insertions(+), 1 deletion(-)
➜  git_test git:(master) git status
位于分支 master
无文件要提交，干净的工作区
```

### 查看历史版本

```sh
➜  git_test git:(master) git reflog

d32d8e4 (HEAD -> master) HEAD@{0}: commit: second commit
eeb90bb HEAD@{1}: commit (initial): first commit
(END)
```

### 版本切换

回退到上个版本

```sh
➜  git_test git:(master) git reset --hard eeb90bb
HEAD 现在位于 eeb90bb first commit

```

查看版本记录

```sh
eeb90bb (HEAD -> master) HEAD@{0}: reset: moving to eeb90bb
d32d8e4 HEAD@{1}: commit: second commit
eeb90bb (HEAD -> master) HEAD@{2}: commit (initial): first commit
```

### 取消文件的暂存

也就是取消文件的 commit 操作

```sh
➜  git_test git:(master) ✗ git restore --staged hello.txt
```

### 取消文件跟踪

取消文件的 add 操作

```sh
➜  git_test git:(master) git rm --cache hello.txt
rm 'hello.txt'
```

## 分支操作

| 命令名称            | 作用                         |
| ------------------- | ---------------------------- |
| git branch 分支名   | 创建分支                     |
| git branch -v       | 查看分支                     |
| git checkout 分支名 | 切换分支                     |
| git merge 分支名    | 把指定的分支合并到当前分支上 |

### 查看分支

```sh
➜  git_test git:(master) git branch -v

```

### 创建分支

```sh
➜  git_test git:(master) git branch hot-fix

➜  git_test git:(master) git branch -v

  hot-fix fefedb1 update
* master  fefedb1 update
```

### 切换分支

```sh
➜  git_test git:(master) git checkout hot-fix
切换到分支 'hot-fix'

➜  git_test git:(hot-fix) git branch -v
* hot-fix fefedb1 update
  master  fefedb1 update
```

### 合并分支

```sh
➜  git_test git:(master) git merge hot-fix
更新 fefedb1..4919661
Fast-forward
 hello.txt | 4 +++-
 1 file changed, 3 insertions(+), 1 deletion(-)
```

### 冲突合并

两个分支对同一个文件做了不同的修改，git 无法决定使用哪个人的修改，造成冲突。

演示

先修改 master 分支

```sh
➜  git_test git:(master) vim hello.txt
➜  git_test git:(master) ✗ git add hello.txt
➜  git_test git:(master) ✗ git commit -m "master update" hello.txt
[master 7488ace] master update
 1 file changed, 2 insertions(+), 1 deletion(-)

```

再修改 hot-fix 分支

```sh
 ➜  git_test git:(master) git checkout hot-fix
切换到分支 'hot-fix'
➜  git_test git:(hot-fix) vim hello.txt
➜  git_test git:(hot-fix) ✗ git add hello.txt
➜  git_test git:(hot-fix) ✗ git commit -m "hot-fix update" hello.txt
[hot-fix 456f9e2] hot-fix update
 1 file changed, 2 insertions(+), 1 deletion(-)
```

回到 master 分支 合并分支，这时候会产生冲突了

```sh
 ➜  git_test git:(hot-fix) git checkout master
切换到分支 'master'
➜  git_test git:(master) git merge hot-fix
自动合并 hello.txt
冲突（内容）：合并冲突于 hello.txt
自动合并失败，修正冲突然后提交修正的结果。

// 此时在查看状态

➜  git_test git:(master) ✗ git status
位于分支 master
您有尚未合并的路径。
  （解决冲突并运行 "git commit"）
  （使用 "git merge --abort" 终止合并）

未合并的路径：
  （使用 "git add <文件>..." 标记解决方案）
	双方修改：   hello.txt

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
```

手动修改冲突

```properties
hello you
hello you
hello you
hello you
how are you

hot fix add
<<<<<<< HEAD
master update
=======
hot fix update
>>>>>>> hot-fix

将需要留下来的手动修改即可，修改后如下所示

hello you
hello you
hello you
hello you
how are you

hot fix add
master update
hot fix update
```

然后提交修改即可。**注意修改冲突后，提交的时候不需要带文件名！**

同时合并只会修改 mater 分支的内容，hot-fix 分支的内容不会修改。

```sh
➜  git_test git:(master) ✗ git add hello.txt
➜  git_test git:(master) ✗ git commit -m "merge hello"
```

## 远程仓库

用 GITHUB 做测试

### 查看/创建远程仓库别名

查看别名

```sh
➜  git_test git:(master) git remote -v
```

创建别名

```sh
➜  git_test git:(master) git remote add git_test https://github.com/lichlaughing/git_test.git
```

### 推送

将本地 master 分支推送到远程仓库

```sh
➜  git_test git:(master) git push git_test https://github.com/lichlaughing/git_test.git master
```

### 克隆远程仓库

```sh
git clone https://github.com/lichlaughing/git_test.git master
```

### 拉取远程仓库内容

git pull 远程库地址别名 远程分支名

```sh
git pull git_test master
```

### Github 免密登录

通过 ssh 免密登录 github 进行上面的操作

```sh
--运行命令生成.ssh 秘钥目录
[注意：这里-C 这个参数是大写的 C]
$ ssh-keygen -t rsa -C lichenghao.vip@qq.com
```

输入命令然后一顿回车就行了。

然后在家目录下的.ssh 下就会生成两个文件

```sh
➜  ~ ll .ssh
total 24
-rw-------  1 lichenghao  staff   3.3K 11  7  2020 id_rsa
-rw-r--r--  1 lichenghao  staff   743B 11  7  2020 id_rsa.pub
```

复制 id_rsa.pub 文件内容，登录 GitHub，点击用户头像 →Settings→SSH and GPG keys ，（ [🔑 点击我新建 ssh key](https://github.com/settings/ssh/new) ）

新建 ssh key 将刚才生成的文件`id_rsa.pub`里面的内容拷贝进去，添加说明即可。

🧪 测试下

```bash
$ ssh -T git@github.com
Hi lichlaughing! You've successfully authenticated, but GitHub does not provide shell access.
```

这样就在 Git 就可以使用 SSH 来进行代码的拉取和提交了 🌻

## 场景记录
