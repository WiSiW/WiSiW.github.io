---
title: 【Echarts】graph连线设置label
date: 2023-01-30 18:03:34
tags:
    - Echarts
---
```javascript
{
    edgeLabel: {
        normal: {
            show: true,
            formatter: () => {
            return "标题";
            },
        },
    },
}
```