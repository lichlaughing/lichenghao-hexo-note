---
title: Html5&css3 选择器
categories: frontend
tags:
  - frontend
  - html5
  - css3
keywords: interfacetest
abbrlink: 86fff6eb
date: 2022-05-07 13:50:00
---
## 常用选择器

### 元素选择器

作用：根据标签名选中指定的元素

语法：标签名 {}

示例： [在线测试](https://www.runoob.com/runcode)

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>选择器</title>

    <style>
      /* 元素选择器 */
      p {
        color: blue;
      }

      h1 {
        color: red;
      }
    </style>
  </head>

  <body>
    <h1>陆游</h1>
    <p>古人学问无遗力，</p>
    <p>少壮工夫老始成，</p>
    <p>纸上得来终觉浅，</p>
    <p>绝知此事要躬行。</p>
  </body>
</html>
```

### ID 选择器

作用：根据元素的 ID 值选中元素

语法：#id {}

示例： [在线测试](https://www.runoob.com/runcode)

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>选择器</title>

    <style>
      /* 元素选择器 */
      p {
        color: blue;
      }

      h1 {
        color: red;
      }

      /* ID 选择器 */
      #first {
        color: green;
      }
    </style>
  </head>

  <body>
    <h1>陆游</h1>
    <p id="first">古人学问无遗力，</p>
    <p>少壮工夫老始成，</p>
    <p>纸上得来终觉浅，</p>
    <p>绝知此事要躬行。</p>
  </body>
</html>
```

> 保持元素的 ID 全局唯一，避免出现问题

### 类选择器

作用：根据元素的 class 属性选中一组元素

语法：.className {}

示例： [在线测试](https://www.runoob.com/runcode)

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>选择器</title>

    <style>
      /* 元素选择器 */
      p {
        color: blue;
      }

      h1 {
        color: red;
      }

      /* ID 选择器 */
      #first {
        color: green;
      }

      .second {
        color: pink;
      }
    </style>
  </head>

  <body>
    <h1>陆游</h1>
    <p id="first">古人学问无遗力，</p>
    <p class="second">少壮工夫老始成，</p>
    <p class="second">纸上得来终觉浅，</p>
    <p>绝知此事要躬行。</p>
  </body>
</html>
```

### 通配选择器

作用：选中所有

语法：\* {}

示例： [在线测试](https://www.runoob.com/runcode)

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>选择器</title>

    <style>
      /* 元素选择器 */
      p {
        color: blue;
      }

      h1 {
        color: red;
      }

      /* ID 选择器 */
      #first {
        color: green;
      }

      /* 类选择器 */
      .second {
        color: pink;
      }

      /* 通配选择器 */
      * {
        font-size: 50px;
      }
    </style>
  </head>

  <body>
    <h1>陆游</h1>
    <p id="first">古人学问无遗力，</p>
    <p class="second">少壮工夫老始成，</p>
    <p class="second">纸上得来终觉浅，</p>
    <p>绝知此事要躬行。</p>
  </body>
</html>
```

## 复合选择器

### 交集选择器

作用：选中同时符合多个条件的元素

语法：选择器1选择器2.....

示例： 

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>复合选择器</title>
  <style>
    .red {
      color: red;
    }

    /* 交集选择器 */
    div.red {
      font-size: 100px;
    }

    .a.b.c {
      color: blueviolet;
    }
  </style>
</head>

<body>
  <div class="red">我是DIV</div>
  <div class="red2">我是DIV</div>
  <p class="red">我是P</p>
  <p class="a b c">我是P2</p>
</body>

</html>
```

!>交集选择器中，如果有元素选择器，那么必须以元素选择器开头

### 并集选择器（分组）

作用：同时选择多个选择器对应的元素

语法：选择器1,选择器2.....

示例： 

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>复合选择器</title>
  <style>
    .red {
      color: red;
    }

    /* 交集选择器 */
    div.red {
      font-size: 100px;
    }

    .a.b.c {
      color: blueviolet;
    }

    /* 并集选择器 */
    h1,
    h2 {
      color: green;
    }
  </style>
</head>

<body>
  <div class="red">我是DIV</div>
  <div class="red2">我是DIV</div>
  <p class="red">我是P</p>
  <p class="a b c">我是P2</p>

  <h1 class="h1">我是H1</h1>
  <H2 class="h2">我是H2</H2>
</body>

</html>
```

?>可多个选择器组合，比如 #a1,h1,span,div.red{}



## 关系选择器

### 元素的关系类型

- 父元素：直接包含子元素的元素叫做父元素。
- 子元素：直接被父元素包含的元素叫做子元素。
- 祖先元素：直接或者间接包含后代元素的元素叫做祖先元素。父元素也是祖先元素。
- 后代元素：直接或者间接被祖先元素包含的元素叫做后代元素。子元素也是后代元素。
- 兄弟元素：拥有相同父元素的元素是兄弟元素。



### 子元素选择器

作用：选中指定父元素的指定子元素

语法：父元素>子元素

示例： 

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>关系选择器</title>
  <style>
    /* 给div的子元素设置颜色 */
    div>span {
      color: red;
    }
  </style>
</head>

<body>
  <div>
    我是第一个个DIV

    <p>我是第一个DIV中的P
      <span>我是P中的span</span>
    </p>

    <span>我是DIV中的span</span>
  </div>

  <span>我是DIV外面的span</span>
</body>

</html>
```



### 后代元素选择器

作用：选中指定元素内的指定后台元素

语法：祖先 后代

示例： 

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>关系选择器</title>
  <style>
    /* 给div的子元素设置颜色 */
    div>span {
      color: red;
    }

    /* div 内的所哟span字体都设置大小 */
    div span {
      font-size: 50px;
    }
  </style>
</head>

<body>
  <div>
    我是第一个个DIV

    <p>我是第一个DIV中的P
      <span>我是P中的span</span>
    </p>

    <span>我是DIV中的span</span>
  </div>

  <span>我是DIV外面的span</span>
</body>

</html>
```



> 选择子元素或者后代元素都可以多层使用比如：

```html
 /* 可以多层嵌套 */
div>p>span {
	background-color: green;
}
```



### 下一个兄弟选择器

作用：选择下一个兄弟

语法：前一个 + 下一个

示例： 

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>关系选择器</title>
  <style>
    /* 给div的子元素设置颜色 */
    div>span {
      color: red;
    }

    /* div 内的所哟span字体都设置大小 */
    div span {
      font-size: 50px;
    }

    /* 可以多层嵌套 */
    div>p>span {
      background-color: green;
    }

    /* P后面的第一个兄弟 */
    p+span {
      color: orange;
    }

  </style>
</head>

<body>
  <div>
    我是第一个个DIV

    <p>我是第一个DIV中的P
      <span>我是P中的span</span>
    </p>

    <span>我是DIV中的span</span>
    <span>我是DIV中的span2</span>
    <span>我是DIV中的span3</span>
  </div>

  <span>我是DIV外面的span</span>
</body>

</html>
```

### 下面所有的兄弟选择器

作用：选择下一个兄弟

语法：兄 ~ 弟

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>关系选择器</title>
  <style>
    /* 给div的子元素设置颜色 */
    div>span {
      color: red;
    }

    /* div 内的所哟span字体都设置大小 */
    div span {
      font-size: 50px;
    }

    /* 可以多层嵌套 */
    div>p>span {
      background-color: green;
    }

    /* P后面的第一个兄弟 */
    p+span {
      color: orange;
    }

    /* p后面所有的兄弟 */
    p~span {
      color: pink;
    }
  </style>
</head>

<body>
  <div>
    我是第一个个DIV

    <p>我是第一个DIV中的P
      <span>我是P中的span</span>
    </p>

    <span>我是DIV中的span</span>
    <span>我是DIV中的span2</span>
    <span>我是DIV中的span3</span>
  </div>

  <span>我是DIV外面的span</span>
</body>

</html>
```





## 属性选择器

- [属性名] 选择含有指定属性的元素。
- [属性名=属性值] 选择含有指定属性和属性值的元素。
- [属性名^=属性值] 选择属性值以指定值开头的元素。
- [属性名$=属性值] 选择属性值以指定值结尾的元素。
- [属性名*=属性值] 选择属性值含有某值的元素。

示例：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>属性选择器</title>
  <style>
    /* 任何包含title属性的元素 */
    *[title] {
      font-size: 20px;
    }

    /* 选择p含有属性title的元素 */
    p[title] {
      color: red;
    }

    /* 选择属性等于某个值的元素 */
    p[title=a] {
      color: green;
    }

    /* 属性值以 s-开头的元素 */
    p[title^=s-] {
      color: orange;
    }

    /* 属性值以 -d结尾的元素 */
    p[title$=-d] {
      color: pink;
    }

    /* 选择属性值含有a的元素。 */
    p[title*=a] {
      background-color: gray;
    }
  </style>
</head>

<body>
  <h1 title="ti">陆游</h1>
  <p title="a">古人学问无遗力，</p>
  <p title="s-ab">少壮工夫老始成，</p>
  <p title="s-ac">纸上得来终觉浅，</p>
  <p title="j-d">绝知此事要躬行。</p>
  <p title="k-d">绝知此事要躬行2。</p>
</body>

</html>
```

## 伪类选择器

伪类：不存在的类，用于描述一个元素的特殊状态，比如：第一个子元素，被点击的元素，最后一个元素。

### 选中某个子元素

一般情况下使用冒号：`:`开头，例如：:first-child

- :first-child 选中第一个子元素
- :last-child  选中最后一个子元素
- :nth-child(n) 选中第 n 个子元素，如果参数就是 n 的话表示，选中所有的子元素；2n(或者 event) 表示选中偶数元素；2n+1(或者 odd) 表示奇数元素

示例

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>伪类&伪元素选择器</title>
  <style>
    /* 选择第一个子元素 */
    ul>li:first-child {
      color: red;
    }

    /* 选择最后一个子元素 */
    ul>li:last-child {
      color: yellow;
    }

    /* 选中第N个子元素，如果参数为n则选中所有的子元素 */
    ul>li:nth-child(2) {
      color: green;
    }
  </style>
</head>

<body>
  <ul>
    <span>下面是一首诗</span>
    <li>陆游</li>
    <li>古人学问无遗力，</li>
    <li>少壮工夫老始成，</li>
    <li>纸上得来终觉浅，</li>
    <li>绝知此事要躬行。</li>
  </ul>
</body>

</html>
```

上面的伪类选择器都是按照所有子元素排序选择的。可以验证下：

```html
/* ul所有子元素的第一个 */
ul>*:first-child {
color: pink;
}
```

### 选中某个类型的子元素

以下的伪类选择器是按照子元素的类型进行排序选择：

- :first-of-type  
- :last-of-type
- :nth-of-type(2)

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>伪类&伪元素选择器</title>
  <style>
    /* 同类型的子元素的第一个 */
    ul>li:first-of-type {
      color: blue;
    }

    /* 同类型的子元素的最后一个 */
    ul>li:last-of-type {
      color: aquamarine;
    }

    /* 同类型的子元素的第n个 */
    ul>li:nth-of-type(2) {
      color: #002fa7;
    }
  </style>
</head>

<body>
  <ul>
    <span>下面是一首诗</span>
    <li>陆游</li>
    <li>古人学问无遗力，</li>
    <li>少壮工夫老始成，</li>
    <li>纸上得来终觉浅，</li>
    <li>绝知此事要躬行。</li>
  </ul>
</body>

</html>
```

### 排除某个子元素

:not()  否定选择器

示例：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>伪类&伪元素选择器</title>
  <style>
    /* 除了 ul 下的第二个子元素在，其他的 li都设置字体大小 */
    ul>li:not(li:nth-child(2)) {
      font-size: 50px;
    }
  </style>
</head>

<body>
  <ul>
    <span>下面是一首诗</span>
    <li>陆游</li>
    <li>古人学问无遗力，</li>
    <li>少壮工夫老始成，</li>
    <li>纸上得来终觉浅，</li>
    <li>绝知此事要躬行。</li>
  </ul>
</body>

</html>
```

### 超链接的伪类

- :link  正常的连接，没有访问过的连接
- :visited 访问过的连接 ，只能修改颜色，其他的改不了
- :hover 鼠标移入
- :active 鼠标点击



### 伪元素

表示页面中一些特殊的并不真实存在的元素（特殊的位置）。使用 :: 开头。

- ::first-letter  选中第一个字母
- ::first-line  选中第一行
- ::selection  鼠标选中
- ::before   在之前，通常和 content 配合使用
- ::after  在之后，通常和 content 配合使用

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>伪类&伪元素选择器</title>
  <style>
    /* 伪元素 */
    /* 选择第一个首字母 */
    p::first-letter {
      font-size: 100px;
    }

    /* 选择第一行 */
    p::first-line {
      color: green;
    }

    /* 鼠标选中 */
    p::selection {
      color: rebeccapurple;
    }

    /* 在前面 */
    p::before {
      color: red;
      content: "Hello! ";
    }

    /* 在后面 */
    p::after {
      content: " over!";
      color: red;
    }
  </style>
</head>
<body>
  <!-- 伪元素 -->
  <p>
    good good study ,day day up!
  </p>
</body>
</html>
```



## 选择器权重

样式冲突：

​	当通过不同的选择器选中相同的元素，并且为相同的样式设置了不同的值，便发生了样式冲突。

优先级：从高到低↑

1. 内联样式
2. ID 选择器
3. 类和伪类选择器
4. 元素选择器
5. 统配
6. 继承

可以通过 `!important` 关键字，放在样式后边，就可以提升到最高优先级，超过内联样式。

