---
title: 【Node】CommonJS的模块规范
date: 2022-11-04 01:03:36
tags:
	- Node
cover_picture: /images/node.png
---

# 1.模块引用

```require()```这个方法接受模块标识，以此引入一个模块的API到当前上下文中

```javascript
// 示例代码
var common = require('common')
```

# 2.模块定义

对于```1.```中的```require()```方法引入的外部模块中，使用```exports```对象用于导出当前模块的方法或者变量

```javascript
// 示例代码
exports.name = 'common'

exports.changeName = function(name) => {
  return this.name
}
```

# 3.模块标识

模块标识其实就是传递给```require()```的参数，它必须是符合小驼峰命名的字符串，以```.```、```..```开头，或者绝对路径。它可以没有文件名后缀。

**模块的定义十分简单，接口也十分简洁。它的意义在于将类聚的方法和变量等限定在私有的作用域中，同时支持接入和导出功能以顺畅地连接上下游依赖。**


每个模块具有独立的空间，互不干扰。