---
title: 【Node】SystemJS安装
date: 2022-11-07 03:36:51
tags:
    - SystemJS
    - 安装
---
[SystemJS](https://github.com/systemjs/systemjs)
# 1. what

SystemJS是一个可配置的模块加载器（Configurable Module Loader），它能在浏览器或Node.js上动态加载模块，并且支持CommonJS、AMD、全局模块对象及ES 6模块等。

# 2. how

## 2-1. 构建运行环境
```
mkdir test
cd test
npm init -y
npm install typescript systemjs@0.19.47 --save
npm install lite-server --save
```

**注：SystemJS安装时使用@标记出具体的版本号，这是因为从SystemJS v0.2版开始将TypeScript加载变成插件方式了，而SystemJS v0.2之前的版本则直接内置TypeScript加载功能，为了简单起见，这里就直接使用SystemJS v0.19.47版了。**

## 2-2.使用SystemJS

### 2-2-1. 创建index.html

```html
// ./index.html
<head>
  <script src="node_modules/systemjs/dist/system.js"></script>
  <script src="node_modules/typescript/lib/typescript.js"></script>
  <script>
    System.config({
      transpiler : 'typescript' , // 要使用TypeScript编译（转译）器
      packages : {
        './ ': { defaultExtension : 'ts' } // 要转义package.json同一目录下的所有ts文件
      }
    });
    System
      .import('main.ts') // 要导入的ts入口文件名
      .then(null, console.error.bind(console));// 如果产生错误，将错误发送到浏览器控制台窗口
  </script>
</head>
```

### 2-2-2. 创建模块

```ts
// ./main.ts
import { name } from './src/common.ts';

alert(name);
```

```typescript
// ./src/common.ts
export const name = 'common';
```

## 2-3. 本地运行

### 2-3-1. 安装```lite-server```

```
npm install lite-server
```

### 2-3-2. 添加运行脚本

```json
// package.json
{
  "name": "test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "lite-server"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "lite-server": "^2.4.0",
    "systemjs": "^0.19.47",
    "typescript": "^3.4.1"
  }
}
```
### 2-3-3. 本地运行

```
npm run dev
```
