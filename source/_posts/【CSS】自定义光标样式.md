---
title: '【CSS】自定义光标样式'
date: 2024-03-31 03:02:23
tags:
  - CSS
---

```html
<style>
  .cursor {
    width: 50px;
    height: 50px;
    background-color: blue;
  }
  // 鼠标浮动到元素上时，会显示图片，如果图片不存在，将会使用auto
  .cursor:hover {
    cursor: url('custom-cursor.png'), auto;
  }
</style>
<div class="cursor"></div>
```
