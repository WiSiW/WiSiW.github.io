---
title: 【CSS】tips
date: 2021-12-09 13:46:12
tags:
    - CSS
cover_picture: /images/css.png
---


# 1.函数

## 1.1自定义变量

### 1.1.1声明与使用

#### 1）在选择器中声明变量

CSS选择器不能是数字开头，JS中的变量是不能直接数值的，但是，在CSS变量中，这些限制通通没有。 但不能包含`$`，`[`，`^`，`(`，`%`等字符，普通字符局限在只要是“数字`[0-9]`”“字母`[a-zA-Z]`”“下划线`_`”和“短横线`-`”这些组合，但是可以是中文，日文或者韩文

```css
:root {
  --global-color: red;
}
```

`:root`匹配的是文档的根元素（<html>标签）所以常用于声明全局的CSS变量，也可以在其他选择器中

```css
body {
  --global-color: red;
}
```

声明变量时需要在变量名前添加`--`以表名此为变量

#### 2）使用，`var()`读取变量

```css
.main {
  color: var(--global-color);
}
```

#### 3)组合使用

使用变量时，可以组合使用，比如进行字符串拼接，如下

```css
:root {
  --content-text: '内容'
}
span::before {
  content: 'before'var(--content-text)
}
```

当变量时数值时，必须使用`calc()`计算

```css
:root {
  -padding: 20;
}
body {
  padding: cal(var(--padding) * 1px)
}
```



#### 2）在行内声明变量

```html
<style>
  .circle {
    width: 30px;
    height: 30px;
    background: var(--clr);
  }
</style>
```

`style`的选择器只能读取自身或者父元素声明在行内的变量

```html
<div style="--clr: red">
  <div class="circle"></div>
</div>
```

or

```html
<div>
  <div class="circle" style="--clr: red"></div>
</div>
```



#### 3)默认值`var(custom-property-name, value)`

`var()`第二个参数作为第一个参数为空或者错误时的值

````css
:root {
  --clr: #fff
}
body {
  /* 未声明--color，因此使用第二个参数#000*/
  color: var(--color, #000);
  /* 变量类型不对，因此使用第二个参数12px*/
  font-size: var(--colr, 12px)
}
````



#### 4）作用域

`:root{}`用来声明全局变量

声明局部变量需要在某个特定的选择器下声明

```css
:root {
  /* --clr全局可用 */
  --clr: #f0f0f0;
}
.main {
  /* 此--clr只能在class为main的元素及其子孙元素中使用*/
  --clr: #fff;
}
```



#### 5）继承

```html
<style>
  .main {
    --clr: #f0f0f0;
  }
  .content1 {
    --clr: #000;
    color: var(--clr);
  }
  .content2 {
    color: var(--clr);
  }
</style>
<div class="main">
  <div class="content1">继承父元声明的变量</div>
  <div class="content2">使用自己声明的变量</div>
</div>
```



## 1.3 计算属性值`calc()`

```css
.main {
  width: calc(100% - 100px)
}
```

-   运算符前后需要保留空格，如`width: calc(100%-100px)`就不会起效，需要在`-`前后添加空格才会起效
-   支持`+,-,*,/`运算
-   计算时使用标准的数学运算优先级规则

## 1.4

## 1.5

## 1.6

