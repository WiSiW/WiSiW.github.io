---
title: 【VUE】局部样式的实现原理
date: 2021-09-30 10:11:07
tags:
    - 局部样式
    - vue-loader
cover_picture: /images/vue.jpeg
---

# 使用方式

通过在```style```标签上添加```scoped```标签实现样式局部化

```vue
<style scoped></style>
```

# 原理

```vue-loader```在解析```.vue```文件时，识别到```style```的```scoped```标签，会在生成的元素上绑定```[data-v-*]```标签并在css样式中添加```[data-v-*]```

比如：

```vue
<template>
  <div class="main">111</div>
</template>
<style scoped>
  .main {
    color: red;
  }
</style>
```

会被解析成

```html
<head>
  <style>
    .main[data-v-6542a5cf] {
      color: red;
    }
  </style>
</head>
<body>
  <div data-v-6542a5cf class="main">111</div>
</body>
```

这时```.main[data-v-6542a5cf]```的样式只会对```<div data-v-6542a5cf class="main">111</div>```起作用

主要起作用的就是```[data-v-*]```这个标签，这个其实是当前这个vue组件的根id，可以从```vue-loader```的源码中看到生成原理

https://github.com/vuejs/vue-loader/blob/3597f6d2b5dd6b4b47fbc30b8ff0902278444d1f/lib/utils/gen-id.js
https://github.com/search?q=repo%3Avuejs%2Fvue-loader+genId&type=issues

```javascript
// utility for generating a uid for each component file
// used in scoped CSS rewriting
var path = require('path')
var hash = require('hash-sum')
var cache = Object.create(null)
var sepRE = new RegExp(path.sep.replace('\\', '\\\\'), 'g')

module.exports = function genId (file, context, key) {
  var contextPath = context.split(path.sep)
  var rootId = contextPath[contextPath.length - 1]
  file = rootId + '/' + path.relative(context, file).replace(sepRE, '/') + (key || '')
  return cache[file] || (cache[file] = hash(file))
}
```

其实就是根据文件在文件中的相对地址和key生成的唯一id
