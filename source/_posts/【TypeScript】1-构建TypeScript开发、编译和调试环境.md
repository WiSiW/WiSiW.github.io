---
title: 【TypeScript】1.构建TypeScript开发、编译和调试环境
date: 2022-11-06 05:24:11
tags:
    - TypeScript
    - 使用
---

# 1. 安装环境

## 1.1 安装Node
[官网](https://nodejs.org/)
安装Node.js时，一般会提供两个版本：LTS版和Current版。其中，LTS是Long Term Support（官方长期支持版）的缩写，在生产环境中，请使用LTS版

## 1.2 全局安装ts

```
sudo npm install -g typescript
```

# 2. 使用

## 2.1 手动编译（一般用法）
例如将当前路径下的```main.ts```转译为```main.js```
使用
```
tsc main.ts
```
当前路径下会生成```main.js```文件

## 2.2 编译器

实现自动编译

### 2.2.1 生成```tsconfig.json```文件

```
tsc --init
```

### 2.2.2 自动编译

```json
{
    "compilerOptions" : {
        "target" : "es5" ,
        "module" : "commonjs" ,
        "strict" : true ,
        "esModuleInterop" : true,
        "watch" : true // 是否开启监控.ts文件变化，实现自动编译
    }
}
```
