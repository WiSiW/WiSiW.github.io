---
title: 【JavaScript】功能代码
date: 2021-09-23 15:05:51
tags:
    - JavaScript
    - 功能代码
cover_picture: /images/javascript.png
---

# 获取数组深度
```javascript
const arr = [1,2,[3,[4,5]],[1,2,3,[1,2,[1,[2,2],2],3]]]
console.log(getArrDepth(arr)) // 5
function getArrDepth (arr, num = 0) {
    if (Array.isArray(arr)) {
        num+=1
        const depArr = []
        arr.map((v) => {
            depArr.push(getArrDepth(v, num))
        })
        num = depArr.sort((a,b) => b - a)[0]
    }
    return num
}
```

# 数额转化
```javascript

```

# 时长转化
```javascript
// 时长转xx天xx小时xx分钟 （默认分钟单位）
function timeStamp(time, isSecond = false) {
  let StatusMin = isSecond ? time / 60 : time
  const day = parseInt(StatusMin / 60 / 24)
  const hour = parseInt((StatusMin / 60) % 24)
  let min = parseInt(StatusMin % 60)
  if (StatusMin > 0 < 60) {
    min = Math.ceil(StatusMin % 60)
  }
  let res = ''
  if (day > 0) {
    res = day + '天'
  }
  if (hour > 0) {
    res += hour + '小时'
  }
  if (min > 0) {
    res += parseFloat(min) + '分钟'
  }
  return res || '--'
}
```
