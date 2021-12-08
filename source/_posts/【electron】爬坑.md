---
title: 【electron】爬坑
date: 2021-12-01 09:46:54
tags:
    - electron
    - 爬坑
---



# Q: process打印不出来

**A:** 需要在```main.js```中的```BrowserWindow```中添加配置

如果是16.X.X以下，那么

```javascript
const win = new BrowserWindow({
  // ...
  webPreferences: {
    // ...
    nodeIntegration: true
  }
})
```

如果是16.X.X，则

```javascript
const win = new BrowserWindow({
  // ...
  webPreferences: {
    // ...
    nodeIntegration: true,
    contextIsolation: false
  }
})
```

**W:**16.X.X以上版本的bug
