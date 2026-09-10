---
title: 【CSS】边框动画
date: 2021-12-09 13:46:12
tags:
    - CSS
cover_picture: /images/css.png
---

[源码](https://github.com/WiSiW/frontend/tree/master/CSS/borderAnimation)

![边框动画](/images/border_animation.gif)

# 元素外框

```html
<div class="border-box"></div>
```



```css
body {
  background: #000000;
}
.border-box {
  width: 200px;
  height: 200px;
  box-shadow: 16px 14px 20px #000000;
  border-radius: 10px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  justify-content: center;
  align-items: center;
}
.border-box:before {
  content: '';
  width: 150%;
  height: 150%;
  position: absolute;
  border-radius: 10px;
}
// 内部元素
.border-box:after {
  content: 'Animation';
  width: 190px;
  height: 190px;
  line-height: 190px;
  text-align: center;
  background: #000000;
  color: red;
  position: absolute;
  border-radius: 10px;
  letter-spacing: 5px;
  box-shadow: inset 20px 20px 20px #000000;
}
```



# 添加外框颜色

```css
.border-box:before {
  /* ... */
  background-image: conic-gradient(#ff0052 20deg, transparent 120deg);
}
```



# 外框旋转

![边框动画](/images/border_animation.gif)

```css
.border-box:before {
  /* ... */
  animation: rotate 2s linear infinite;
}
```



# 遮挡动画外框

```css
.border-box {
  /* ... */
  overflow: hidden;
}
```



# 全部代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Border Animation</title>
  <style>
    body {
      background: #000000;
    }
    .border-box {
      width: 200px;
      height: 200px;
      box-shadow: 16px 14px 20px #000000;
      border-radius: 10px;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      display: flex;
      justify-content: center;
      align-items: center;
      overflow: hidden;
    }
    .border-box:before {
      content: '';
      background-image: conic-gradient(#ff0052 20deg, transparent 120deg);
      width: 150%;
      height: 150%;
      position: absolute;
      border-radius: 10px;
      animation: rotate 2s linear infinite;
    }
    .border-box:after {
      content: 'Animation';
      width: 190px;
      height: 190px;
      line-height: 190px;
      text-align: center;
      background: #000000;
      color: red;
      position: absolute;
      border-radius: 10px;
      letter-spacing: 5px;
      box-shadow: inset 20px 20px 20px #000000;
    }
    @keyframes rotate {
      0% {
        transform: rotate(0deg);
      }
      100% {
        transform: rotate(-360deg);
      }
    }
  </style>
</head>
<body>
  <div class="border-box"></div>
</body>
</html>
```

