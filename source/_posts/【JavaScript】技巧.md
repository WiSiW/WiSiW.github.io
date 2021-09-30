---
title: 【JavaScript】技巧
date: 2021-07-05 10:01:45
tags:
    - 技巧
    - JavaScript
cover_picture: /images/javascript.png
---

# 1.多个条件判断的if语句，使用includes

**在数组中存储需要判断的值，使用数组的includes方法**

```javascript
if( x === 'abc' || x === 'def' || x === 'ghi') {
   
}
if(['abc', 'def', 'ghi'].includes(x)) {
  
}
```

# 2.if...else使用三元表达式

**if...else的逻辑判断较为简单的时候，使用三元判断**

