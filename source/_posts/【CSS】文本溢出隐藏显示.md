---
title: '【CSS】文本溢出隐藏显示...'
date: 2024-01-17 14:52:28
tags:
  - CSS
---

## 单行文本隐藏

```css
.line-hidden {
  white-space: nowrap; // 不换行
  overflow: hidden; // 查出部分隐藏
  text-overflow: ellipsis; // 溢出部分显示为...
}
```

## 多行文本隐藏

```css
.multiline-hidden {
  overflow: hidden; //溢出隐藏
  text-overflow: ellipsis; //省略号
  display: -webkit-box; // 弹性盒模型
  -webkit-box-orient: vertical; //设置弹性盒子的子元素的排列方式
  -webkit-line-clamp: 2;// 设置显示文本的行数
}
```
