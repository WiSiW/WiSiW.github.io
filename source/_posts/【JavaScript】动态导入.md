---
title: 【JavaScript】动态导入
date: 2022-10-28 23:09:17
tags:
    - JavaScript
---

将变量作为标识符，并动态导入

```javascript
const jsFile = new Blob(['export default true'], { type: 'application/javscript'});
const jsURL = URL.createObjectURL(jsFile);

import(jsURL).then(module => {
    console.log(module.default); // true
})
```
