---
title: 【架构】SystemJS VS Webpack
date: 2021-08-13 11:07:02
tags:
    - 前端工程化
    - SysemJS
---

 功能 | SystemJS|Webpack | 说明
 --|--|--|--|--|
| 自启动HTML页面 | 支持 | 支持 | 需服务器支持（lite-server或webpack-dev-server）
| 模块化开发 | 支持 | 支持 | TypeScipt语言支持模块化开发
| 严格类型检测 | 支持 | 支持 | 通过TypeScript的tsconfig.json配置获取
| 断点调试 | 支持 | 支持 | 需要浏览器调试插件
| 解决跨域问题 | 支持 | 支持 | 需服务器支持（lite-server或webpack-dev-server）
| 热更新  | 支持（在SystemJS中，修改HTML或CSS源码并保存后，会立即自动重载刷新页面） | 需要css-loader、style-loader、html-webpack-plugin等插件 | 需服务器支持（lite-server或webpack-dev-server）
| 自动重新编译（转译）TypeScript | 支持 | 支持 | SystemJS和Webpack内置了该功能，但都需要手动刷新页面运行更新后的代码
| 打包压缩源码 | 不支持 | 支持 | SystemJS是加载器，二Webpack是打包器

**最大的不同**：

在SystemJs中，index.html引入的是

```html
<script src="node_modules/systemjs/system.js"></script>
<script src="node_modules/typescript/lib/typescript.js"></script>
```

在Webpack中，index.html引入的是

```html
<script src="./dist/bundle.js"></script>
```

由上可见，SystemJS是在页面载入时进行TypeScript源码的编译（转译），因此它是一个加载器

而Webpack是在后台进行编译（转译）和打包操作的，生成bundle.js。HTML直接执行打包后的JS资源。

因此就运行效率及内存消耗来说，Webpack更具优势
