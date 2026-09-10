---
title: '【功能模块】图片转base64'
date: 2024-02-27 20:37:54
tags:
  - 功能模块
  - 图片
---

# NodeJS

使用```fs.readFileSync```

```javascript
const path = require("path");
const fs = require("fs");

const res = fs.readFileSync(path.resolve("./logo.png"))

console.log(`data:image/png;base64,${res.toString("base64")}`)
```

# Brower

使用```FileReader```

```javascript
<img style="width: 50px;" id="img">
<input id="fileupload" type="file">

<script>
    const el = document.querySelector("#fileupload");
    const img = document.querySelector("#img");

    const reader = new FileReader();
    reader.addEventListener("load", function(e){
        console.log(e.result)
        img.src = reader.result;
    });
    el.addEventListener("change", function(e){
        const file = e.target.files[0];
        reader.readAsDataURL(file);
    })
</script>
```