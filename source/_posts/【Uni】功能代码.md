---
title: 【Uni】功能代码
date: 2021-09-28 11:00:02
tags:
    - Uni
    - 功能代码
---

# 锚点定位
```javascript
const topTabList = [
    { name: '锚点一', point: 0 },
    { name: '锚点二', point: 0 },
    { name: '锚点三', point: 0 },
]
getScrollPoint(className) {
    const systemInfo = uni.getSystemInfoSync()
    let query = ''
    // 在微信小程序中，如果把骨架屏放入组件中使用的话，需要调in(this)上下文为父组件才有效
    // #ifdef MP-WEIXIN
    query = uni.createSelectorQuery().in(this.$parent)
    // #endif
    // #ifndef MP-WEIXIN
    query = uni.createSelectorQuery()
    // #endif
    query.selectAll(`.${className}`).boundingClientRect().exec((res) => {
        res[0].forEach((v, i) => {
            topTabList[i].point = v.top - v.height - 20 - (systemInfo.platform === 'ios' ? 44 : 48)
        })
    })
}
```
