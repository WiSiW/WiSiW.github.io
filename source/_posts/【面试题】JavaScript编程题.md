---
title: 【面试题】JavaScript编程题
date: 2021-02-18 12:09:21
tags:
	- 面试题
	- 编程题
cover_picture: /images/javaScript.png
---

# 柯里化

## 实现 add(1)(2)(3)

```javascript
function add () {
  const numberList = Array.from(arguments);

  // 进一步收集剩余参数
  const calculate = function() {
    numberList.push(...arguments);
    return calculate;
  }

  // 利用 toString 隐式转换，最后执行时进行转换
  calculate.toString = function() {
    return numberList.reduce((a, b) => a + b, 0);
  }

  return calculate;
}

// 实现一个 add 方法，使计算结果能够满足以下预期
console.log(add(1)(2)(3)); // 6
console.log(add(1, 2, 3)(4)); // 10;
console.log(add(1)(2)(3)(4)(5)); // 15;
```



## 实现 compose(foo, bar, baz)('start')

```javascript
function foo(...args) {
  console.log(args[0]);
  return 'foo';
}
function bar(...args) {
  console.log(args[0]);
  return 'bar';
}
function baz(...args) {
  console.log(args[0]);
  return 'baz';
}

function compose() {
  // 闭包元素 - 函数列表
  const list = Array.from(arguments);

  // 闭包元素 - 函数列表执行位置
  let index = -1;

  // 闭包元素 - 上一个函数的返回
  let prev = '';

  // 返回闭包函数
  const doNext = function() {
    index++; // 索引值累加
    // 一开始没有上一个元素时，获取第二个括号的值
    if (!prev) {
      prev = arguments[0];
    }
    // 设置前一个结果为当前函数返回
    prev = list[index](prev);
    // 递归调用
    if (index < list.length - 1) {
      doNext(index + 1);
    }
  };

  // 第一次返回闭包函数
  return doNext;
}

compose(foo, bar, baz)('start');
```



## 选择题

```javascript
function test() {
  var n = 4399;

  function add() {
    n++;
    console.log(n);
  }

  return {
    n,
    add
  };
};

var result = test();
var result2 = test();

result.add(); // 输出啥
result.add(); // 输出啥
console.log(result.n); // 输出啥
result2.add(); // 输出啥
// 4400 4401 4399 4400
```



```javascript
function Foo() {
  var i = 0;
  return function() {
    console.log(i++);
  }
}

var f1 = Foo();
var f2 = Foo();

f1();
f1();
f2();
// 0 1  0
```

