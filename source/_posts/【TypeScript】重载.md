---
title: 【TypeScript】重载
date: 2022-07-18 09:52:24
tags:
    - TypeScript
    - 重载
---
# demo

```ts
class OverLoad {
  // 重载签名
  load(param: string): void;
  load(param: Array<string>): void;

  // 实现签名
  load(param: string | Array<string>) {
    if (param instanceof Array<string>) {
      // do something
    } else {
      // do something
    }
  }
}

const OL = new OverLoad();
OL.load("a");
OL.load(["a", "b"]);
// OL.load(() => {}) // error
```

# 说明
## 1.重载签名是可调用的
虽然实现签名实现了函数行为，但是不能直接调用，只有重载签名是可调用的。
即使实现签名的入参为未在重载签名中定义的类型值，也不能调用重载签名函数
```ts
function load(param: string): void;
function load(param: number): void;

function load(param: any): void {
  // do something
}
load([1,2])
// 报错
/**
No overload matches this call.
  Overload 1 of 2, '(param: string): void', gave the following error.
    Argument of type 'number[]' is not assignable to parameter of type 'string'.
  Overload 2 of 2, '(param: number): void', gave the following error.
    Argument of type 'number[]' is not assignable to parameter of type 'number'.
**/
```

## 2.实现签名必须是通用的
重载签名不仅签名需要保持一致，返回值也许保持一致
```ts
function load(param: string): string;
function load(param: number): void;

function load(param: any): void {
  // do something
}
```
以上两个load的返回值不一致，因此不能实现重载

# 使用情况
## 不推荐对可选参数使用函数重载
```ts
function load(): void;
function load(param1: string): void;
function load(param1: string, param2: number): void;

function load(...args: any): void {
  // do something
}
```
## 推荐在实现签名中使用可选参数
```ts
function load(param1?: string, param2: number): void {}
```