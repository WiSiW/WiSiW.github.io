---
title: '【SVG】复用图像元素'
date: 2024-03-21 17:23:42
tags:
  - SVG
---

```use```元素在SVG文档内取得目标节点，并在其他地方复用它们。它的效果等同于这些节点被深克隆到一个不可见的DOM中，然后黏贴到```use```元素的位置。

```html
<svg width="200" height="200" viewBox="-15 -15 30 30">
  <circle fill="#e5c39c" r="6" />
  <line id="ray" stroke="#e5c39c" stroke-width="2" stroke-linecap="round" x1="0" y1="11" x2="0" y2="14"></line>
  <use href="#ray" transform="rotate(45)" />
  <use href="#ray" transform="rotate(90)" />
  <use href="#ray" transform="rotate(135)" />
  <use href="#ray" transform="rotate(180)" />
  <use href="#ray" transform="rotate(225)" />
  <use href="#ray" transform="rotate(270)" />
  <use href="#ray" transform="rotate(315)" />
</svg>
```
