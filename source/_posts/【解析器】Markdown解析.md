---
title: 【解析器】Markdown解析
date: 2021-09-30 17:22:14
tags:
    - Markdown
---

解析基于```https://github.com/evilstreak/markdown-js```

暴露出去的方法为```toHTML```
```javascript
const mk = require('markdown').markdown
mk.toHTML('# 1.') // <h1>1.</h1>
```


