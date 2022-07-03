---
title: butterfly主题下写作模板
keywords: butterfly 模板
categories: 其他
tags:
  - butterfly
  - template
abbrlink: 4b1484b
date: 2022-01-01 00:00:00
---

标签： https://butterfly.js.org/posts/2df239ce/

# 文本

```markdown
{% pullquote left/right %}
出师表/前出师表
{% endpullquote %}
```

{% pullquote left %}
出师表/前出师表
{% endpullquote %}

先帝创业未半而中道崩殂，今天下三分，益州疲弊，此诚危急存亡之秋也。然侍卫之臣不懈于内，忠志之士忘身于外者，盖追先帝之殊遇，欲报之于陛下也。诚宜开张圣听，以光先帝遗德，恢弘志士之气，不宜妄自菲薄，引喻失义，以塞忠谏之路也。宫中府中，俱为一体，陟罚臧否，不宜异同。若有作奸犯科及为忠善者，宜付有司论其刑赏，以昭陛下平明之理，不宜偏私，使内外异法也。侍中、侍郎郭攸之、费祎、董允等，此皆良实，志虑忠纯，是以先帝简拔以遗陛下。愚以为宫中之事，事无大小，悉以咨之，然后施行，必能裨补阙漏，有所广益。

{% pullquote right%}
出师表/前出师表
{% endpullquote %}

先帝创业未半而中道崩殂，今天下三分，益州疲弊，此诚危急存亡之秋也。然侍卫之臣不懈于内，忠志之士忘身于外者，盖追先帝之殊遇，欲报之于陛下也。诚宜开张圣听，以光先帝遗德，恢弘志士之气，不宜妄自菲薄，引喻失义，以塞忠谏之路也。宫中府中，俱为一体，陟罚臧否，不宜异同。若有作奸犯科及为忠善者，宜付有司论其刑赏，以昭陛下平明之理，不宜偏私，使内外异法也。侍中、侍郎郭攸之、费祎、董允等，此皆良实，志虑忠纯，是以先帝简拔以遗陛下。愚以为宫中之事，事无大小，悉以咨之，然后施行，必能裨补阙漏，有所广益

# 引用

格式：

```markdown
{% blockquote [author[, source]] [link] [source_link_title] %}
content
{% endblockquote %}
```

{% tabs quote %}
<!-- tab 普通引用 -->

```markdown
{% blockquote %}
先帝创业未半而中道崩殂，今天下三分，益州疲弊，此诚危急存亡之秋也。然侍卫之臣不懈于内，忠志之士忘身于外者，盖追先帝之殊遇，欲报之于陛下也。诚宜开张圣听，以光先帝遗德，恢弘志士之气，不宜妄自菲薄，引喻失义，以塞忠谏之路也
{% endblockquote %}
```

{% blockquote %}
先帝创业未半而中道崩殂，今天下三分，益州疲弊，此诚危急存亡之秋也。然侍卫之臣不懈于内，忠志之士忘身于外者，盖追先帝之殊遇，欲报之于陛下也。诚宜开张圣听，以光先帝遗德，恢弘志士之气，不宜妄自菲薄，引喻失义，以塞忠谏之路也
{% endblockquote %}

<!-- endtab -->
<!-- tab 引用一本书的内容 -->

```markdown
{% blockquote 《向上生长》, 九边 %}
简单的欲望，只需要放纵就可以实现，而高级的欲望，放纵是实现不了的，需要的是自律和克制。
{% endblockquote %}
```

{% blockquote 《向上生长》, 九边 %}
简单的欲望，只需要放纵就可以实现，而高级的欲望，放纵是实现不了的，需要的是自律和克制。
{% endblockquote %}

<!-- endtab -->
<!-- tab 引用一个网站的内容 -->

```markdown
{% blockquote @DevDocs https://twitter.com/devdocs/status/356095192085962752 %}
NEW: DevDocs now comes with syntax highlighting. http://devdocs.io
{% endblockquote %}
```

{% blockquote @DevDocs https://twitter.com/devdocs/status/356095192085962752 %}
NEW: DevDocs now comes with syntax highlighting. http://devdocs.io
{% endblockquote %}

<!-- endtab -->

<!-- tab 引用网页内容 -->

```markdown
{% blockquote 李成浩 https://www.lichneghao.com.cn %}
纸上得来终觉浅，绝知此事要躬行
{% endblockquote %}
```

{% blockquote 李成浩 https://www.lichneghao.com.cn %}
纸上得来终觉浅，绝知此事要躬行
{% endblockquote %}

<!-- endtab -->

{% endtabs %}



# Tabs

默认选择第一个tab页

```markdown
{% tabs test1 %}
<!-- tab t1 -->
This is Tab 1.
<!-- endtab -->
<!-- tab t2 -->
This is Tab 2.
<!-- endtab -->
<!-- tab t3 -->
This is Tab 3.
<!-- endtab -->
{% endtabs %}
```

{% tabs test1 %}
<!-- tab t1 -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab t2 -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab t3 -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

指定默认页

```markdown
{% tabs test2, 3 %}
<!-- tab 第一个 -->
This is Tab 1.
<!-- endtab -->
<!-- tab 第二个 -->
This is Tab 2.
<!-- endtab -->
<!-- tab 第三个 -->
This is Tab 3.
<!-- endtab -->
{% endtabs %}
```

{% tabs test2, 3 %}
<!-- tab 第一个 -->
**第一个**
<!-- endtab -->

<!-- tab 第二个 -->
**第二个**
<!-- endtab -->

<!-- tab 第三个 -->
**第三个**
<!-- endtab -->
{% endtabs %}

tabs名字可以使用图标

```markdown
{% tabs test4 %}
<!-- tab 第一個Tab -->
**第一個Tab**
<!-- endtab -->

<!-- tab @fab fa-apple-pay -->
**图标**
<!-- endtab -->

<!-- tab 炸彈@fas fa-bomb -->
**名字+icon**
<!-- endtab -->
{% endtabs %}
```

{% tabs test4 %}
<!-- tab 第一個Tab -->
**第一個Tab**
<!-- endtab -->

<!-- tab @fab fa-apple-pay -->
**只有图标 沒有Tab名字**
<!-- endtab -->

<!-- tab 炸彈@fas fa-bomb -->
**名字+icon**
<!-- endtab -->
{% endtabs %}

# 按钮

{% tabs button %}
<!-- tab 默认-->

```markdown
{% btn 'https://www.lichenghao.com.cn/',Butterfly %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,,outline %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,larger %}
```

{% btn 'https://www.lichenghao.com.cn/',Butterfly %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,,outline %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,larger %}

<!-- endtab -->
<!-- tab 不同位置 -->

```markdown
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,block %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,block center larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,block right outline larger %}
```

{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,block %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,block center larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,block right outline larger %}

<!-- endtab -->
<!-- tab 不同颜色 -->

```markdown
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,blue larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,pink larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,red larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,purple larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,orange larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,green larger %}
```

{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,blue larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,pink larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,red larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,purple larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,orange larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,green larger %}

<!-- endtab -->

<!-- tab 不同边框 -->

```markdown
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline blue larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline pink larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline red larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline purple larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline orange larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline green larger %}
```

{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline blue larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline pink larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline red larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline purple larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline orange larger %}
{% btn 'https://www.lichenghao.com.cn/',Butterfly,far fa-hand-point-right,outline green larger %}

<!-- endtab -->

{% endtabs %}

# Note

{% tabs note %}
<!-- tab simple -->

```markdown
{% note simple %}
默认 提示
{% endnote %}

{% note default simple %}
default 提示
{% endnote %}

{% note primary simple %}
primary 提示
{% endnote %}

{% note success simple %}
success 提示
{% endnote %}

{% note info simple %}
info 提示
{% endnote %}

{% note warning simple %}
warning 提示
{% endnote %}

{% note danger simple %}
danger 提示
{% endnote %}
```

{% note simple %}
默认 提示
{% endnote %}

{% note default simple %}
default 提示
{% endnote %}

{% note primary simple %}
primary 提示
{% endnote %}

{% note success simple %}
success 提示
{% endnote %}

{% note info simple %}
info 提示
{% endnote %}

{% note warning simple %}
warning 提示
{% endnote %}

{% note danger simple %}
danger 提示
{% endnote %}

<!-- endtab -->
<!-- tab modern -->

```markdown
{% note modern %}
默认 提示
{% endnote %}

{% note default modern %}
default 提示
{% endnote %}

{% note primary modern %}
primary 提示
{% endnote %}

{% note success modern %}
success 提示
{% endnote %}

{% note info modern %}
info 提示
{% endnote %}

{% note warning modern %}
warning 提示
{% endnote %}

{% note danger modern %}
danger 提示
{% endnote %}
```

{% note modern %}
默认 提示
{% endnote %}

{% note default modern %}
default 提示
{% endnote %}

{% note primary modern %}
primary 提示
{% endnote %}

{% note success modern %}
success 提示
{% endnote %}

{% note info modern %}
info 提示
{% endnote %}

{% note warning modern %}
warning 提示
{% endnote %}

{% note danger modern %}
danger 提示
{% endnote %}

<!-- endtab -->
<!-- tab flat-->

```markdown
{% note flat %}
默认 提示
{% endnote %}

{% note default flat %}
default 提示
{% endnote %}

{% note primary flat %}
primary 提示
{% endnote %}

{% note success flat %}
success 提示
{% endnote %}

{% note info flat %}
info 提示
{% endnote %}

{% note warning flat %}
warning 提示
{% endnote %}

{% note danger flat %}
danger 提示
{% endnote %}
```

{% note flat %}
默认 提示
{% endnote %}

{% note default flat %}
default 提示
{% endnote %}

{% note primary flat %}
primary 提示
{% endnote %}

{% note success flat %}
success 提示
{% endnote %}

{% note info flat %}
info 提示
{% endnote %}

{% note warning flat %}
warning 提示
{% endnote %}

{% note danger flat %}
danger 提示
{% endnote %}

<!-- endtab -->

<!-- tab disabled-->

```markdown
{% note disabled %}
默认 提示
{% endnote %}

{% note default disabled %}
default 提示
{% endnote %}

{% note primary disabled %}
primary 提示
{% endnote %}

{% note success disabled %}
success 提示
{% endnote %}

{% note info disabled %}
info 提示
{% endnote %}

{% note warning disabled %}
warning 提示
{% endnote %}

{% note danger disabled %}
danger 提示
{% endnote %}

```

{% note disabled %}
默认 提示
{% endnote %}

{% note default disabled %}
default 提示
{% endnote %}

{% note primary disabled %}
primary 提示
{% endnote %}

{% note success disabled %}
success 提示
{% endnote %}

{% note info disabled %}
info 提示
{% endnote %}

{% note warning disabled %}
warning 提示
{% endnote %}

{% note danger disabled %}
danger 提示
{% endnote %}

<!-- endtab -->

<!-- tab no-icon-->

```markdown
{% note no-icon %}
默认 提示塊
{% endnote %}

{% note default no-icon %}
default 提示
{% endnote %}

{% note primary no-icon %}
primary 提示
{% endnote %}

{% note success no-icon %}
success 提示
{% endnote %}

{% note info no-icon %}
info 提示
{% endnote %}

{% note warning no-icon %}
warning 提示
{% endnote %}

{% note danger no-icon %}
danger 提示
{% endnote %}
```

{% note no-icon %}
默认 提示塊
{% endnote %}

{% note default no-icon %}
default 提示
{% endnote %}

{% note primary no-icon %}
primary 提示
{% endnote %}

{% note success no-icon %}
success 提示
{% endnote %}

{% note info no-icon %}
info 提示
{% endnote %}

{% note warning no-icon %}
warning 提示
{% endnote %}

{% note danger no-icon %}
danger 提示
{% endnote %}

<!-- endtab -->

<!-- tab simple-icon-->

```markdown
{% note 'fab fa-cc-visa' simple %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note blue 'fas fa-bullhorn' simple %}
2021年快到了....
{% endnote %}
{% note pink 'fas fa-car-crash' simple %}
小心开车 安全至上
{% endnote %}
{% note red 'fas fa-fan' simple%}
这是三片呢？还是四片？
{% endnote %}
{% note orange 'fas fa-battery-half' simple %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note purple 'far fa-hand-scissors' simple %}
剪刀石头布
{% endnote %}
{% note green 'fab fa-internet-explorer' simple %}
IE浏览器
{% endnote %}
```

{% note 'fab fa-cc-visa' simple %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note blue 'fas fa-bullhorn' simple %}
2021年快到了....
{% endnote %}
{% note pink 'fas fa-car-crash' simple %}
小心开车 安全至上
{% endnote %}
{% note red 'fas fa-fan' simple%}
这是三片呢？还是四片？
{% endnote %}
{% note orange 'fas fa-battery-half' simple %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note purple 'far fa-hand-scissors' simple %}
剪刀石头布
{% endnote %}
{% note green 'fab fa-internet-explorer' simple %}
IE浏览器
{% endnote %}

<!-- endtab -->

<!-- tab modern-icon-->

```markdown
{% note 'fab fa-cc-visa' modern %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note blue 'fas fa-bullhorn' modern %}
2021年快到了....
{% endnote %}
{% note pink 'fas fa-car-crash' modern %}
小心开车 安全至上
{% endnote %}
{% note red 'fas fa-fan' modern%}
这是三片呢？还是四片？
{% endnote %}
{% note orange 'fas fa-battery-half' modern %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note purple 'far fa-hand-scissors' modern %}
剪刀石头布
{% endnote %}
{% note green 'fab fa-internet-explorer' modern %}
IE浏览器
{% endnote %}
```

{% note 'fab fa-cc-visa' modern %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note blue 'fas fa-bullhorn' modern %}
2021年快到了....
{% endnote %}
{% note pink 'fas fa-car-crash' modern %}
小心开车 安全至上
{% endnote %}
{% note red 'fas fa-fan' modern%}
这是三片呢？还是四片？
{% endnote %}
{% note orange 'fas fa-battery-half' modern %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note purple 'far fa-hand-scissors' modern %}
剪刀石头布
{% endnote %}
{% note green 'fab fa-internet-explorer' modern %}
IE浏览器
{% endnote %}

<!-- endtab -->

<!-- tab flat-icon-->

```markdown
{% note 'fab fa-cc-visa' flat %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note blue 'fas fa-bullhorn' flat %}
2021年快到了....
{% endnote %}
{% note pink 'fas fa-car-crash' flat %}
小心开车 安全至上
{% endnote %}
{% note red 'fas fa-fan' flat%}
这是三片呢？还是四片？
{% endnote %}
{% note orange 'fas fa-battery-half' flat %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note purple 'far fa-hand-scissors' flat %}
剪刀石头布
{% endnote %}
{% note green 'fab fa-internet-explorer' flat %}
IE浏览器
{% endnote %}
```

{% note 'fab fa-cc-visa' flat %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note blue 'fas fa-bullhorn' flat %}
2021年快到了....
{% endnote %}
{% note pink 'fas fa-car-crash' flat %}
小心开车 安全至上
{% endnote %}
{% note red 'fas fa-fan' flat%}
这是三片呢？还是四片？
{% endnote %}
{% note orange 'fas fa-battery-half' flat %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note purple 'far fa-hand-scissors' flat %}
剪刀石头布
{% endnote %}
{% note green 'fab fa-internet-explorer' flat %}
IE浏览器
{% endnote %}

<!-- endtab -->

<!-- tab disabled-icon-->

```markdown
{% note 'fab fa-cc-visa' disabled %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note blue 'fas fa-bullhorn' disabled %}
2021年快到了....
{% endnote %}
{% note pink 'fas fa-car-crash' disabled %}
小心开车 安全至上
{% endnote %}
{% note red 'fas fa-fan' disabled %}
这是三片呢？还是四片？
{% endnote %}
{% note orange 'fas fa-battery-half' disabled %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note purple 'far fa-hand-scissors' disabled %}
剪刀石头布
{% endnote %}
{% note green 'fab fa-internet-explorer' disabled %}
IE浏览器
{% endnote %}
```

{% note 'fab fa-cc-visa' disabled %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note blue 'fas fa-bullhorn' disabled %}
2021年快到了....
{% endnote %}
{% note pink 'fas fa-car-crash' disabled %}
小心开车 安全至上
{% endnote %}
{% note red 'fas fa-fan' disabled %}
这是三片呢？还是四片？
{% endnote %}
{% note orange 'fas fa-battery-half' disabled %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note purple 'far fa-hand-scissors' disabled %}
剪刀石头布
{% endnote %}
{% note green 'fab fa-internet-explorer' disabled %}
IE浏览器
{% endnote %}

<!-- endtab -->

<!-- tab no-icon-icon-->

```markdown
{% note no-icon %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note blue no-icon %}
2021年快到了....
{% endnote %}
{% note pink no-icon %}
小心开车 安全至上
{% endnote %}
{% note red no-icon %}
这是三片呢？还是四片？
{% endnote %}
{% note orange no-icon %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note purple no-icon %}
剪刀石头布
{% endnote %}
{% note green no-icon %}
IE浏览器
{% endnote %}
```

{% note no-icon %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note blue no-icon %}
2021年快到了....
{% endnote %}
{% note pink no-icon %}
小心开车 安全至上
{% endnote %}
{% note red no-icon %}
这是三片呢？还是四片？
{% endnote %}
{% note orange no-icon %}
你是刷 Visa 还是 UnionPay
{% endnote %}
{% note purple no-icon %}
剪刀石头布
{% endnote %}
{% note green no-icon %}
IE浏览器
{% endnote %}

<!-- endtab -->

{% endtabs %}

# Tag-hide

{% tabs tag-hide %}
<!-- tab inline -->

格式：

```markdown
{% hideInline content,display,bg,color %}
参数：
content: 文本內容
display: 按钮提示的文字(可选)
bg: 按钮的背景顏色(可选)
color: 按钮文字的顏色(可选)
( display 不能包含英文逗號，可用&sbquo;)
```

示例：

```markdown
A和C谁高? {% hideInline 答案：A A比C低（ABCD）,查看答案,#FF7242,#fff %}

是谁在那边? {% hideInline 我 %}
```

A和C谁高? {% hideInline 答案：A A比C低（ABCD）,查看答案,#FF7242,#fff %}

是谁在那边? {% hideInline 我 %}

<!-- endtab -->
<!-- tab Block -->

```markdown
查看答案
{% hideBlock 查看答案 %}
这是一个答案
{% endhideBlock %}
```

查看答案
{% hideBlock 查看答案 %}
这是一个答案
{% endhideBlock %}

<!-- endtab -->
<!-- tab Toggle-->

```markdown
{% hideToggle Butterfly安裝方法 %}
在你的博客根目录
git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly
如果想要安裝比较新的dev分支，可以
git clone -b dev https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly
{% endhideToggle %}
```

{% hideToggle Butterfly安裝方法 %}
在你的博客根目录
git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly
如果想要安裝比较新的dev分支，可以
git clone -b dev https://github.com/jerryc127/hexo-theme-butterfly.git themes/Butterfly
{% endhideToggle %}

<!-- endtab -->
{% endtabs %}

# InlineImg

主題中的图片都是默认以块元素展示，如果你想以内联元素展示，可以使用这个。

```markdown
{% inlineImg [src] [height] %}

[src]      :    图片链接
[height]   ：    图片高度限制【可选】
```

示例：

```markdown
你看我長得漂亮不

![](https://i.loli.net/2021/03/19/2P6ivUGsdaEXSFI.png)

我覺得很漂亮 {% inlineImg https://i.loli.net/2021/03/19/5M4jUB3ynq7ePgw.png 150px %}
```

你看我長得漂亮不

![](https://i.loli.net/2021/03/19/2P6ivUGsdaEXSFI.png)

我覺得很漂亮 {% inlineImg https://i.loli.net/2021/03/19/5M4jUB3ynq7ePgw.png 150px %}