---
title: 【CSS】样式功能
date: 2021-11-16 16:00:41
tags:
    - CSS
cover_picture: /images/css.png
---



# 点闪烁

![点闪烁](/images/icon_point_twinkle.gif)

```html
<html>
  <head>
    <style>
      .point {
        background: #06DFF9;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
      }
      .point-level1 {
        width: 10px;
        height: 10px;
      	border-radius: 50%;
      }
      .point-level2 {
        width: 20px;
        height: 20px;
      	border-radius: 50%;
      	opacity: 0.45;
        animation: twinkle2 1s linear infinite;
      }
      @keyframes twinkle2 {
        0% {
          width: 10px;
          height: 10px;
        }
        100% {
          width: 20px;
          height: 20px;
        }
      }
      .point-level3 {
        width: 30px;
        height: 30px;
      	border-radius: 50%;
      	opacity: 0.35;
        animation: twinkle3 1s linear infinite;
      }
      @keyframes twinkle3 {
        0% {
          width: 10px;
          height: 10px;
        }
        100% {
          width: 30px;
          height: 30px;
        }
      }
      .point-level4 {
        width: 40px;
        height: 40px;
      	border-radius: 50%;
      	opacity: 0.25;
        animation: twinkle4 1s linear infinite;
      }
      @keyframes twinkle4 {
        0% {
          width: 10px;
          height: 10px;
        }
        100% {
          width: 40px;
          height: 40px;
        }
      }
    </style>
  </head>
  <body>
    <div class="point point-level1"></div>
    <div class="point point-level2"></div>
    <div class="point point-level3"></div>
    <div class="point point-level4"></div>
  </body>
</html>
```

