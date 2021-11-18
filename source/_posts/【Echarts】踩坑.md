---
title: 【Echarts】踩坑
date: 2021-11-16 10:45:43
tags:
    - echarts
    - 踩坑
cover_picture: /images/echarts.png
---



# label设置不同的样式

```javascript
options = {
  //...
  series: [
    {
      // ...
      label: {
        show: true,
        formatter: function(v) {
          return `{d|${v.percent}%}\n\n{b|${v.name}}`
        },
        rich: {
          d: {
            fontSize: 20,
            fontFamily: 'PingFang SC',
            fontWeight: 500,
            color: 'rgba(248, 181, 43, 1)'
          },
          b: {
            fontSize: 18,
            fontFamily: 'PingFang SC',
            fontWeight: 400,
            color: '#48AAFE'
          }
        }
      }
    }
  ]
}
```

**```{styleName|text content text content}```使用```|```隔开，前面是样式名，后面是值**



# 标签的视觉引导线消失的原因

1.  设置了```label.position```

