---
title: Macbook Pro 实用设置
categories: macbook
tags:
  - macbook
keywords: macbook
abbrlink: b62b2404
date: 2022-05-25 12:30:00
---
## 取消系统更新提示

系统偏好设置——>软件更新——>高级，把勾选的都取消掉。

![](https://blog.lichenghao.cn/upload/2022/07/20213952.png)

然后在控制台执行，小红点就会取消。

```shell
取消小红点：
defaults write com.apple.systempreferences AttentionPrefBundleIDs 0
killall Dock
```

`但是如果你再次点击了软件更新，那么红点还会出来。再执行一次。我不建议彻底取消更新提示，毕竟有大的更新的时候最好更新下。`



```shell
忽略大版本更新提示：
sudo softwareupdate --ignore "macOS Monterey"
忽略小版本更新的方法：
sudo softwareupdate --ignore "macOS Monterey 12.3 Update"
恢复更新提示：
sudo softwareupdate --reset-ignored
```







## 使用 Win 键盘的键位设置

如果你从 Win转到了 Mac，那么你把 Win 键盘插入到 Mac 电脑上需要调整一下键位。

如果你使用 Mac 电脑了，我建议你去熟悉 Mac 的键位顺序，而不是在 Mac上调整为 Win 的键位顺序。

当你插上键盘后，Mac 会自动识别你的键盘，例如我使用的是 Cherry MX BOARD 3.0S。这个时候打字是没有问题了，它会把我们常用的键位都映射好。除了这几个特殊的↓

- 键盘上 Shift 键会被 mac 识别为 Shift

- 键盘上 Ctrl 键会被 mac 识别为 Control

- 键盘上 Win 键会被 mac 识别为 Command

- 键盘上 Atl  键会被 mac 识别为 Option

所以你左下角的三个键分别为：Control Command Option。

其中 Command 和 Option 是和 Mac键盘正好反着的。Mac 下键的顺序是 Control Option Command。所以我们来调整下顺序保持和 Mac 键盘一致。

依次打开，系统偏好设置——>键盘——>右下角“修饰键”

![](https://blog.lichenghao.cn/upload/2022/07/12.54.53.png)

将Option和 Command键调整为如下所示，其他不变即可。

![](https://blog.lichenghao.cn/upload/2022/07/13.00.54.png)



















