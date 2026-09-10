---
title: 【JavaScript】<script>延迟执行
date: 2022-11-06 05:33:51
tags:
---

# why

当脚本放在```<head>```标签中，并且初始化需要调用```<body>```中的节点时，由于此时```<body>```未被加载，所以脚本会运行错误

# how

+ 将```<script>```至于```<body>```之后，等节点被加载后，再执行脚本

```html
<body>
    <div id="root"></div>
</body>
<script type="text/javascript">
    console.log(docment.body.id)
</script>
```

+ ```<script```添加```defer```参数

```html
<head>
    <script type="text/javascript" defer>
        console.log(docment.body.id)
    </script>
</head>
```

+ ```window.onload()```

```window.onload()```将会在网页加载完毕后立即执行
