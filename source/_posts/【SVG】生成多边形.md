---
title: '【SVG】生成多边形'
date: 2024-03-23 15:51:18
tags:
  - SVG
---

```javascript
const getSin = (deg) => {
  return Math.sin(deg * Math.PI / 180)
}
const getCos = (deg) => {
  return Math.cos(deg * Math.PI / 180)
}
const getPoints = (c, n, l) => {
  if (n < 3) return;
  const res = [];
  const θ = 360 / n;
  for(let i = 1;i < n + 1;i++) {
    let deg = (n%2 == 0 ? θ/2 : 0) + (i - 1)*θ;
    res.push([c[0] + l * getSin(deg), c[1] - l * getCos(deg)])
  }
  return res;
}
// 绘制中心在[50, 50]，中心距顶点30的五边形
const res = getPoints([50, 50], 5, 30);
// [ [ 50, 20 ], [ 78.53169548885461, 40.729490168751575 ], [ 67.6335575687742, 74.27050983124842 ], [ 32.366442431225806, 74.27050983124843 ], [ 21.46830451114539, 40.72949016875158 ] ]

// 生成SVG

const fs = require("fs");
const buildSVG = (list) => {
  let path = `M ${list[0][0]} ${list[0][1]}`;
  for(let i = 1; i < list.length; i ++) {
    const it = list[i];
    path += ` L ${it[0]} ${it[1]}`
  }
  path += `L ${list[0][0]} ${list[0][1]}`;
  const svg =  `<svg width="100" height="100">
    <path d="${path}" fill="#fff" stroke="#000" stroke-width="2" >
  </svg>`;
  fs.writeFileSync("index.html", svg)
}

buildSVG(res);
```
