---
title: Html5&css3 浮动
categories: frontend
tags:
  - frontend
  - html5
  - css3
  - float
keywords: interfacetest
abbrlink: 56f329ab
date: 2022-05-07 13:50:00
---
## 浮动特点

浮动可以让一个元素在其父元素的左侧或者右侧移动。通过 `float` 来设置浮动。

float 可选值：

- none : 默认值，元素不浮动
- left：向左浮动
- right：向右浮动

**浮动的特点：**

- 元素设置浮动后，水平布局的等式不需要强制成立了。（子元素的：margin-left + border-left + padding-left + with + padding-right + border-right + margin-right = 父元素内容区的宽度 。）

- 元素设置浮动后，会完全脱离文档流。不再占用文档流中的位置。在它下面的文档流中的元素会自动向上移动。(示例 1)
- 浮动元素不会从父元素中移出。
- 浮动元素向左或者向右浮动，不会超过它前面的其他浮动元素。
- 如果浮动元素的上边没有一个浮动的块元素，则浮动元素无法上移。
- 浮动元素不会超过它上边的浮动的兄弟元素，最多和它一样高。（示例 2）
- 浮动元素不会盖住文字，文字会环绕在浮动元素的周围。（可以用来设置文字环绕图片的效果）
- 元素设置浮动以后，将会从文档流中脱离，其元素的一些特点也会发生变化。也就是`元素脱离文档流的特点：`
  - 块元素：不在独占一行。
  - 块元素：宽度和高度默认都被内容撑开。
  - 行内元素：变成块元素，特点和块元素一样。
  - 脱离文档流后，所有元素都一样，不需要在区分行内元素或是块元素了！



<details>
<summary> 示例 1（点击展开）</summary>


- 元素设置浮动后，会完全脱离文档流。不再占用文档流中的位置。在它下面的文档流中的元素会自动向上移动。

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>浮动</title>
  <style>
    .box1 {
      width: 200px;
      height: 200px;
      background-color: aqua;
      float: left;
    }

    .box2 {
      width: 200px;
      height: 200px;
      background-color: red;
      float: left;
    }

    .box3 {
      width: 200px;
      height: 200px;
      background-color: green;
      float: left;
    }
  </style>
</head>

<body>
  <div class="box1"></div>
  <div class="box2"></div>
  <div class="box3"></div>
</body>

</html>
```



</details>



<details>
<summary> 示例 2（点击展开）</summary>


- 浮动元素不会超过它上边的浮动的兄弟元素，最多和它一样高。

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>浮动,浮动元素不会超过它上边的浮动的兄弟元素，最多和它一样高</title>
  <style>
    body {
      width: 600px;
      border: 1px black solid;
    }

    .box1 {
      width: 420px;
      height: 200px;
      background-color: aqua;
      float: left;
    }

    .box2 {
      width: 300px;
      height: 200px;
      background-color: red;
      float: left;
    }

    .box3 {
      width: 100px;
      height: 200px;
      background-color: green;
      float: right;
    }
  </style>
</head>

<body>
  <div class="box1"></div>
  <div class="box2"></div>
  <div class="box3"></div>
</body>

</html>
```



</details>

## 高度塌陷

高度塌陷：在浮动布局中，父元素的高度默认由子元素撑开的，当子元素浮动后会脱离文档流，将无法撑起父元素的高度，导致父元素的高度丢失。

<details>
<summary> 高度塌陷示例代码（点击展开）</summary>

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>子浮动导致父元素高度塌陷</title>
  <style>
    .out {
      background-color: aqua;
      border: 10px black solid;
    }

    .in {
      width: 200px;
      height: 200px;
      background-color: red;
      float: left;
    }

    .other {
      width: 300px;
      height: 100px;
      background-color: green;
    }
  </style>
</head>

<body>
  <div class="out">
    <div class="in"></div>
  </div>
  <div class="other"></div>

</body>

</html>
```

</details>



## BFC

BFC (Block Format Context) ：块级格式化环境，是 CSS 的一个隐含的属性。开启 BFC 的元素会变成一个独立的布局区域。

开启 BFC 的元素有以下特点：

1. 元素不会被浮动的元素覆盖。
2. 子元素和父元素的外边距不会重叠。——>[盒子外边距重叠问题](/front/html5&css3/box?id=盒子的外边距折叠)
3. 可以包含浮动的子元素。

开启元素 BFC 的方式：

1. 设置元素浮动 （脱离文档流！）
2. 元素设置为行内块元素。（宽度丢失！）
3. 将元素的 overflow 设置一个非 visible 的值。
4. ......

## 解决高度塌陷

### clear

作用：清除浮动元素对当前元素的影响。

可选值：

- left : 清除左侧浮动元素对当前元素的影响。
- right : 清除右侧浮动元素对当前元素的影响。
- both：清除两侧影响最大的元素对当前元素的影响。

原理：

设置清除浮动后，浏览器会自动为当前元素设置一个上外边距，使其位置不受其他元素影响。

测试代码：

<details>
<summary> clear 清除浮动对当前元素的影响示例（点击展开）</summary>

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>clear 清除浮动对当前元素的影响</title>
  <style>
    .out {
      width: 300px;
      height: 100px;
      background-color: aqua;
      float: left;
    }

    .out2 {
      width: 300px;
      height: 300px;
      background-color: yellow;
      float: right;
    }

    .other {
      width: 400px;
      height: 200px;
      background-color: green;
      clear: both;
    }
  </style>
</head>

<body>
  <div class="out"></div>
  <div class="out2"></div>
  <div class="other"></div>
</body>

</html>
```



</details>

**利用 clear 来解决高度塌陷**

测试代码：

<details>
<summary> clear 解决高度塌陷示例（点击展开）</summary>

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>clear 解决高度塌陷</title>
  <style>
    /* 借助 box3 利用 clear 解决父元素高度塌陷 */
    /* 利用结构解决布局 */
    .box1 {
      background-color: yellow;
    }

    .box2 {
      width: 100px;
      height: 100px;
      background-color: green;
      float: left;
    }

    .box3 {
      clear: both;
    }
  </style>
</head>

<body>
  <div class="box1">
    <div class="box2"></div>
    <div class="box3"></div>
  </div>
</body>

</html>
```



</details>

### clear 进阶示例

上个示例利用结构解决布局，可以把结构替换成伪元素。

<details>
<summary> clear 解决高度塌陷进阶示例（点击展开）</summary>

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>clear 解决高度塌陷进阶示例</title>
  <style>
    /* 借助 box3 利用 clear 解决父元素高度塌陷 */
    /* 利用伪元素解决布局 */
    .box1 {
      background-color: yellow;
    }

    .box1::after {
      content: "";
      clear: both;
      display: block;
    }

    .box2 {
      width: 100px;
      height: 100px;
      background-color: green;
      float: left;
    }
  </style>
</head>

<body>
  <div class="box1">
    <div class="box2"></div>
  </div>
</body>

</html>
```



</details>



### .clearfix 

经典代码：可以同时解决高度塌陷和外边距重叠的问题。

```css
.clearfix::before,
.clearfix::after {
  content: "";
  display: table;
  clear: both;
}
```

测试代码：

<details>
<summary> clearfix 解决高度塌陷和外边距重叠问题示例（点击展开）</summary>

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>clearfix 解决高度塌陷和外边距重叠问题</title>
  <style>
    .box1 {
      background-color: yellow;
      width: 200px;
      height: 200px;
    }

    .box2 {
      width: 100px;
      height: 100px;
      background-color: green;
      margin-top: 100px;
    }

    .clearfix::before,
    .clearfix::after {
      content: "";
      display: table;
      clear: both;
    }
  </style>
</head>

<body>
  <div class="box1 clearfix">
    <div class="box2"></div>
  </div>
</body>

</html>
```

</details>

