---
title: Html5&css3 定位
tags:
  - frontend
  - html5
  - css3
  - position
categories: frontend
keywords: interfacetest
abbrlink: 62df8c24
date: 2022-05-07 13:50:00
---



定位是一种更高级的布局手段，可以把元素放到页面的任意位置，通过 position 来开启定位，可选值有：

- static：默认值，没有开启定位
- relative：相对定位
- absolute：绝对定位
- fixed：固定定位
- sticky：粘滞定位



## 相对定位 （relative）

开启相对定位

```css
.box2 {
      width: 200px;
      height: 200px;
      background-color: green;
      position: relative; 
    }
```

开启相对定位后，还需要设置偏移量（offset）后，元素才会改变位置。

偏移量（offset）：元素开启定位后，可以通过偏移量设置元素的位置。可选值为：top 、left、 bottom、right 。

相对定位的特点：

- 元素开启相对对位后，如果不设置偏移量，元素不会变化。
- 相对对位是参照于元素在文档流中的位置进行定位 （可以通过 left:0;top:0 来判断原点的位置）。
- 相对定位会提升元素的层级。
- 相对定位不会使元素脱离文档流。不会改变元素的性质。



<details>
<summary> 相对定位演示代码（点击展开）</summary>

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>相对定位</title>
  <style>
    .box1 {
      width: 200px;
      height: 200px;
      background-color: aqua;
    }

    /* 相对定位 */
    .box2 {
      width: 200px;
      height: 200px;
      background-color: green;
      position: relative;
      left: 200px;
      top: -200px;
    }

    .box3 {
      width: 200px;
      height: 200px;
      background-color: yellow;
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



## 绝对定位（absolute）

```css
  .box2 {
      width: 200px;
      height: 200px;
      background-color: green;
      position: absolute;
    }
```



绝对定位的特点：

- 元素开启相对对位后，如果不设置偏移量，元素位置不会变化（其他属性可能已经变了）。
- 脱离文档流。
- 绝对定位会改变元素的性质，行内变成块，块的宽高被内容撑开。
- 绝对定位会使元素提升层级。
- 元素相对于其包含块进行定位。
  - `包含块（containing block）`：
    - 正常情况下：离当前元素最近的祖先块元素。
    - 绝对定位情况下：离当前最近的开启了定位的祖先元素。
- 如果祖先都没有开启定位，那么相对于根元素 html（初始包含块） 进行定位。



<details>
<summary> 绝对定位示例代码（点击展开）</summary>

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>绝对定位</title>
  <style>
    .box1 {
      width: 100px;
      height: 200px;
      background-color: aqua;
    }

    .box2 {
      width: 200px;
      height: 200px;
      background-color: green;
      position: absolute;
      top: 0;
      left: 0;
    }

    .box3 {
      width: 300px;
      height: 200px;
      background-color: yellow;

    }

    .box4 {
      width: 400px;
      height: 300px;
      background-color: pink;

    }

    .box5 {
      width: 500px;
      height: 400px;
      background-color: blue;
    }
  </style>
</head>

<body>
  <div class="box1">1</div>
  <div class="box4">
    4
    <div class="box5">
      5
      <div class="box2">
        2
      </div>
    </div>
  </div>

  <div class="box3">3</div>
</body>

</html>
```

</details>





## 固定定位（fixed）

固定定位也是一种特殊的绝对定位，大部分特点和绝对定位一样。

与绝对定位不同的是固定定位永远参照于浏览器的`视口(浏览器的可视窗口)`进行定位。



<details>
<summary> 固定定位示例（点击展开）</summary>

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>固定定位</title>
  <style>
    body {
      height: 1000px;
    }

    .box1 {
      width: 100px;
      height: 200px;
      background-color: aqua;
    }

    .box2 {
      width: 200px;
      height: 200px;
      background-color: green;
      position: fixed;
      top: 0;
      left: 0;
    }

    .box3 {
      width: 300px;
      height: 200px;
      background-color: yellow;

    }

    .box4 {
      width: 400px;
      height: 300px;
      background-color: pink;

    }

    .box5 {
      width: 500px;
      height: 400px;
      background-color: blue;
    }
  </style>
</head>

<body>
  <div class="box1">1</div>
  <div class="box4">
    4
    <div class="box5">
      5
      <div class="box2">
        2
      </div>
    </div>
  </div>

  <div class="box3">3</div>
</body>

</html>
```

</details>

## 粘滞定位（sticky）

粘滞定位和相对定位的性质基本一样，不同的是粘滞定位可以在元素到达某个位置时将其固定。兼容性不好。



<details>
<summary> 粘滞定位示例（点击展开）</summary>

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>粘滞定位</title>
  <style>
    body {
      height: 2000px;
    }

    .box1 {
      width: 100px;
      height: 200px;
      background-color: aqua;
      position: sticky;
      top: 0;
    }
  </style>
</head>

<body>
  <div class="box1">1</div>
</body>

</html>
```



</details>