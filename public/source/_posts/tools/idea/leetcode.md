---
title: IntelliJ IDEA 配置 Leetcode 进行疯狂的刷题！
tags:
  - idea
  - leetcode
categories: 
  - tools
  - idea
keywords: IntelliJ IDEA Leetcode
cover: https://blog.lichenghao.cn/upload/2022/07/B118E8A4-leetcode.png
abbrlink: 697634be
date: 2022-07-09 18:25:38
---

以前这个插件并不是很好用。（所以通常是把代码拷贝到编辑器中进行编写，然后在拷贝到浏览器中，进行运行）但是！现在这个插件很丝滑，适合摸鱼疯狂刷题了！

IntelliJ IDEA 的版本信息

> IntelliJ IDEA 2022.1.1 (Ultimate Edition)
> Build #IU-221.5591.52, built on May 11, 2022
> Licensed to chenghao.li

LeetCode Editor 版本

> 8.1

建议新建个项目，专门用来存放我们 leecode 相关的代码。

然后下载插件,直接在插件管理中搜索`LeetCode Editor` 下载安装；或者在插件网站下载离线安装包 https://plugins.jetbrains.com/plugin/12132-leetcode-editor 进行离线的安装。

安装完毕后，在右下角就会显示 leetcode 侧边栏；或者在菜单栏中依次点击 view ——> Tools Windows ——> leetcode 进入目录列表页面。

点击上方 ⚙ 进入配置详情页面；

![](https://blog.lichenghao.cn/upload/2022/07/09183713-leetcode.png)



- URL：国内的话选择 leetcode.cn。

- CodeType：用什么语言就选择什么语言。

- LoginName：登录的用户名。

- Password：登录的密码，但是用用户名和密码登录的方式好像并不好使，下面会说用 cookie 登录的方式。

- JCEF：√，勾选后，我使用的版本支持这个插件，点击登录的时候可以加载 leetcode 登录页面进行登录。

- TempFilePath：下载的代码存放的位置。建议选择自己新建的项目中的 src 下的具体包的位置。

- Custom Template：√，勾选使用自定义的代码模板，默认的模板不太方便，我们希望题目下载下来后能直接在 IDEA 中测试运行。

- Code FileName：代码文件的名称,效果大概是：`[3]longest-substring-without-repeating-characters.java`。

  ```java
  [$!{question.frontendQuestionId}]${question.titleSlug}
  ```

- Code Template：自定义的模板，可以参考我使用的模板如下：

  ```java
  package cn.lichenghao.leetcode.editor.cn;
  ${question.content}
  class $!velocityTool.camelCaseName(${question.titleSlug}){
      public static void main(String[] args){
          Solution solution = new $!velocityTool.camelCaseName(${question.titleSlug})().new Solution();
      }
  ${question.code}
  }
  ```

  

  这样配置完毕后的效果如下所示：可以直接测试运行。(默认 Solution 是没有返回值的，这个需要手动填上)。
  
  ![](https://blog.lichenghao.cn/upload/2022/07/09184109-leetcode.png)



配置完毕后，点击登录即可。(如果你的 IDEA 版本不支持 JCEF 的话，可以看下官网给的说明：[登录配置](https://github.com/shuzijun/leetcode-editor/blob/master/doc/LoginHelp_ZH.md))

![](https://blog.lichenghao.cn/upload/2022/07/09190138-leetcode.png)

登录成功后，就可以选择题库，进行刷题了。

![](https://blog.lichenghao.cn/upload/2022/07/09190418-leetcode.png)



