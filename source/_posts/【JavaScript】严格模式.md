---
title: 【JavaScript】严格模式
date: 2022-11-05 21:29:33
tags:
---

# 目的：

+ 消除 JavaScript 语法中一些不合理、不严谨的地方；
+ 消除代码中一些不安全的地方，保证代码的安全运行；
+ 提高 JavaScript 程序的运行效率；
+ 为以后新版本的 JavaScript 做好铺垫。

**目前，主流浏览器包括 IE10 及其之后的版本都已支持严格模式**

注意："use strict";或'use strict';指令只有在整个脚本第一行或者函数第一行时才能被识别，除了 IE9 以及更低的版本外，所有的浏览器都支持该指令。

# 相较于普通模式

## 1.不允许使用未声明的变量

```javascript
"use strict";
v = 1;        // 此处报错：Uncaught ReferenceError: v is not defined
for(i = 0; i < 2; i++) { // 此处报错：Uncaught ReferenceError: i is not defined
}
```

## 2.不允许删除变量或函数

```javascript
"use strict";
var person = {name: "Peter", age: 28};
delete person;  // 此处报错：Uncaught SyntaxError: Delete of an unqualified identifier in strict mode.
function sum(a, b) {
    return a + b;
}
delete sum;  // 此处报错：Uncaught SyntaxError: Delete of an unqualified identifier in strict mode.
```

## 3.函数中不允许同名的参数

```javascript
"use strict";
function square(a, a) {     // 此处报错：Uncaught SyntaxError: Duplicate parameter name not allowed in this context
    return a * a;
}
```

## 4.eval语句的作用域是独立的

```javascript
"use strict";
eval("var x = 5; console.log(x);");
console.log(x);     // 此处报错：Uncaught ReferenceError: x is not defined
```

## 5.不允许使用with语句

```javascript
"use strict";
var radius1 = 5;
var area1 = Math.PI * radius1 * radius1;
var radius2 = 5;
with(Math) {        // 此处报错：Uncaught SyntaxError: Strict mode code may not include a with statement
    var area2 = PI * radius2 * radius2;
}
```

## 6.不允许写入只读属性

```javascript
"use strict";
var person = {name: "Peter", age: 28};
Object.defineProperty(person, "gender", {value: "male", writable: false});
person.gender = "female"; // 此处报错：Uncaught TypeError: Cannot assign to read only property 'gender' of object '#<Object>'
```

## 7.不允许使用八进制数

```javascript
"use strict";
var x = 010; // 此处报错：Uncaught SyntaxError: Octal literals are not allowed in strict mode.
console.log(parseInt(x));
```

## 8.不能在if语句中声明函数

```javascript
"use strict";
//如果在if语句中声明函数，则会产生语法错误
if (true) {
    function demo() { // 此处报错：Uncaught ReferenceError: demo is not defined
        console.log(111);
    }
}
demo();
```

## 9.禁止使用this表示全局对象

```javascript
"use strict";
var name = "name";
function demoTest() {
    console.log(this);
}
demoTest();
```
