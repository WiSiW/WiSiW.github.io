---
title: 【JavaScript】手写函数
date: 2021-02-02 15:32:39
tags:
    - JavaScript
    - 手写
cover_picture: /images/javascript.png
---

# call()&&apply()&&bind()

## 1.call()

```javascript
function call(Fn, obj, ...arg) {
    if (obj === undefined || obj === null) {
        obj = window;
    }
    // 为obj添加临时方法,this自动指向obj
    obj.temp = Fn;
    // 调用temp方法
    let result = obj.temp(...arg);
    // 删除临时方法
    delete obj.temp;
    // 返回执行结果
    return result;
}
```



## 2.apply()

```javascript
function call(Fn, obj, arg) {
    if (obj === undefined || obj === null) {
        obj = window;
    }
    // 为obj添加临时方法,this自动指向obj
    obj.temp = Fn;
    // 调用temp方法
    let result = obj.temp(...arg);
    // 删除临时方法
    delete obj.temp;
    // 返回执行结果
    return result;
}
```

## 3.bind()

```javascript
function bind(Fn, obj, ...arg) {
    // 返回一个新函数
    return function (...arg2) {
        // 通过call调用原函数, 并指定this为obj, 实参为args与args2
        return call(Fn, obj, ...arg, ...arg2)
    }
}
```

# 函数节流与防抖

## 1.函数节流throttle()

```javascript
function throttle(callback, wait) {
    let start = 0;
    return function (e) {
        let now = Date.now();
        if (now - start > wait) {
            callback.call(this, e)
            start = now;
        }
    }
}
```

## 2.函数防抖debounce()

```javascript
function debounce(callback,time) {
    let timeId = null;
    return function(e) {
        if(timeId !== null){
            clearTimeout(timeId);
        }
        timeId = setTimeout(function () {
            callback.call(this,e);
            timeId = null;
        },time)
    }
}
```

# 数组相关

## 1.map()

**返回一个由回调函数的返回值组成的新数组**

```javascript
function map(arr,callback) {
    // 声明空数组
    let result = [];
    for (let i = 0;i<arr.length;i++){
        // 将callback的执行结果添加到数组中
        result.push(callback(arr[i],i))
    }
    return result;
}
```

## 2.reduce()

**从左到右为每个数组元素执行一次回调函数，并把上次回调函数的返回值放在一个暂存器中传给下次回调函数，并返回最后一次回调函数的返回值**

```javascript
function reduce(arr, callback, initValue) {
    let result = initValue;
    // 执行callback
    for (let i = 0; i < arr.length; i++) {
        // 将回调函数的执行结果赋值给result
        result = callback(result, arr[i]);
    }
    return result;
}
```

## 3.filter()

**将所有在过滤函数中返回 `true` 的数组元素放进一个新数组中并返回**

```javascript
function filter(arr, callback) {
    // 声明空数组
    let result = [];
    for (let i = 0; i < arr.length; i++) {
        // 判断回调函数的返回值
        if (callback(arr[i], i)) {
            result.push(arr[i])
        }
    }
    return result;
}
```

## 4.find()

**找到第一个满足测试函数的元素并返回那个元素的值，如果找不到，则返回 `undefined`**

```javascript
function find(arr, callback) {
    // 遍历数组
    for (let i = 0; i < arr.length; i++) {
        // 判断执行结果
        if (callback(arr[i], i)) {
            // 返回当前正在遍历的元素
            return arr[i];
        }
    }
    // 没有遇到满足条件的元素，返回undefined
    return undefined;
}
```

##  5.findIndex()

**找到第一个满足测试函数的元素并返回那个元素的索引，如果找不到，则返回 `-1`**

```javascript
function findIndex(arr, callback) {
    // 遍历数组
    for (let i = 0; i < arr.length; i++) {
        // 判断回调函数返回值
        if (callback(arr[i], i)) {
            return i
        }
    }
    // 找不到返回-1
    return -1
}
```

## 6.every()

**如果数组中的每个元素都满足测试函数，则返回 `true`，否则返回 `false`**

```javascript
function every(arr, callback) {
    for (let i = 0; i < arr.length; i++) {
        // 只有一个结果为false, 直接返回false
        if (!callback(arr[i], i)) {
            return false
        }
    }
    return true
}
```

## 7.some()

**如果数组中至少有一个元素满足测试函数，则返回 true，否则返回 false**

```javascript
function some(arr, callback) {
    for (let i = 0; i < arr.length; i++) {
        // 只有一个结果为true, 直接返回true
        if (callback(arr[i], i)) {
            return true
        }
    }
    return false
}
```

## 8.数组去重

### 8.1.利用forEach()和indexOf()

**本质是双重遍历, 效率差些**

```javascript
function unique1 (arr) {
    const result = []
    arr.forEach(item => {
        if (result.indexOf(item)===-1) {
            result.push(item)
        }
    })
    return result
}
```

### 8.2.利用forEach() + 对象容器

**只需一重遍历, 效率高些**

```javascript
function unique2 (arr) {
    const result = []
    const obj = {}
    arr.forEach(item => {
        if (!obj.hasOwnProperty(item)) {
            obj[item] = true
            result.push(item)
        }
    })
    return result
}
```

### 8.3.利用ES6语法: from + Set 或者 ... + Set

```javascript
function unique3 (arr) {
    return [...new Set(arr)]
}
```

### 8.4.不使用多余空间

```javascript
function unique4(arr) {
    for (let i = 0; i < arr.length; i++) {
        for (let j = i + 1; j < arr.length; j++) {
            if (arr[i] === arr[j]) {
                arr.splice(j, 1);
                j--;
            }
        }
    }
    return arr;
}
```

## 9.数组合并

```javascript
function concat(arr,...value) {
    const array = [...arr]
    value.forEach(item => {
        if(Array.isArray(item)){
            array.push(...item)
        }else{
            array.push(item)
        }
    })
    return array;
}
```

## 10.数组切片

```javascript
function slice(arr, start, end) {
    // 如果当前数组为[]，返回[]
    if (arr.length === 0) {
        return []
    }
    // 如果开始位置大雨数组长度，返回[]
    start = start || 0
    if (start >= arr.length) {
        return []
    }
    // 如果结束位置大于数组长度，end更新为数组长度
    end = end || arr.length
    if (end > arr.length) {
        end = arr.length;
    }
    // 如果结束位置小于等于开始位置，返回[]
    if (end <= start) {
        return []
    }
    let result = [];
    // 取出下标在[start,end)之间的元素，放入result中
    for (let i = start; i < end; i++) {
        result.push(arr[i])
    }
    return result;
}
```

## 11.数组扁平化

##11.1.递归 + concat()

```javascript
function flatten1(arr) {
    let result = [];
    for (let i = 0; i < arr.length; i++) {
        if (Array.isArray(arr[i])) {
            // 如果是数组，遍历
            result = result.concat(flatten1(arr[i]))
        } else {
            // 不是数组，合并入result中
            result = result.concat(arr[i])
        }
    }
    return result;
}
```

## 11.2. ... + some() + concat()

```javascript
function flatten2(arr) {
    let result = [].concat(...arr)
    // 如果数组中存在第二级，展开合并
    while (result.some(item => Array.isArray(item))) {
        result = [].concat(...result)
    }
}
```

## 12.数组分块

```javascript
function chunk(arr, size = 1) {
    let result = [];
    let temp = [];
    // 遍历数组
    arr.forEach(item => {
        if (temp.length === 0) {
            // 如果临时数组为空，压入结果数组中
            result.push(temp)
        }
        // 将元素压入临时数组中
        temp.push(item)
        if (temp.length === size) {
            // 长度满足后，清空temp
            temp = []
        }
    })
    return result;
}
```

## 13.数组求差集

```javascript
function difference(arr1,arr2=[]) {
    // 如果arr1为[]，返回[]
    if(arr1.length === 0){
        return [];
    }
    // 如果arr2为[]，通过slice深拷贝arr1返回
    if(arr2.length === 0){
        return arr1.slice();
    }
    return  arr1.filter(item => !arr2.includes(item))
}
```

## 14.删除数组中部分元素

### 14.1.pull([1,3,5,3,7], 2, 7, 3, 7) ===> 原数组变为[1, 5], 返回值为[3,3,7]

```javascript
function pull(arr, ...values) {
    if (arr.length === 0 || values.length === 0) {
        return [];
    }
    let result = [];
    for (let i = 0; i < arr.length; i++) {
        if (values.indexOf(arr[i]) !== -1) {
            arr.splice(i,1);
            result.push(arr[i]);
            i--;
        }
    }
    return result;
}
```



### 14.2.pullAll([1,3,5,3,7], [2, 7, 3, 7]) ===> 数组1变为[1, 5], 返回值为[3,3,7]

```javascript
function pullAll(arr, values) {
    if (!values || !Array.isArray(values)) {
        return [];
    }
    return pull(arr, ...values)
}
```

## 16.获取数组部分元素

### 16.1.drop(array, count)

**drop([1,3,5,7], 2) ===> [5, 7]**

```javascript
function drop(arr, count = 1) {
    if (arr.length === 0 || count >= arr.length) {
        return [];
    }
    return arr.filter((item, i) => i >= count)
}
```

### 16.2.dropRight(array, count)

**dropRight([1,3,5,7], 2) ===> [1, 3]**

```javascript
function dropRight(arr, count = 1) {
    if (arr.length === 0 || count >= arr.length) {
        return [];
    }
    return arr.filter((item, i) => i < arr.length - count)
}
```



# 对象相关

## new

根据构造函数实例化对象

```javascript
function New(Fn, ...args) {
    // 1.创建一个新对象
    const obj = {};
    // 2.修改函数内部的this指向新对象，并执行
    const result = Fn.call(obj, ...args)
    obj.__proto__ = Fn.prototype;
    // 3.返回新对象
    return result instanceof Object ? result : obj;
}
```

## instanceOf

```javascript
function InstanceOf(obj,type) {
    // 获取obj的隐式原型对象
    let protoObj = obj.__proto__;
    // 遍历原型链
    while (protoObj){
        // 检测obj的原型对象和type的原型对象是否相等
        if (protoObj === type.__proto__){
            return true;
        }
        // 不相等，向上查找原型对象
        protoObj = protoObj.__proto__;
    }
    return false;
}
```

## 对象合并

```javascript
function mergeObj(...objs) {
    let result = {};
    // 遍历对象
    objs.forEach(obj => {
        // 获取对象的所有key并遍历
        Object.keys(obj).forEach(key => {
            if (result.hasOwnProperty(key)) {
                // 存在，则合并属性
                result[key] = [].concat(result[key], obj[key])
            } else {
                // 如果结果对象中不存在，则添加属性
                result[key] = obj[key];
            }
        })
    })
    return result;
}
```

## 对象/数组拷贝

### 浅拷贝

#### 1）...

```javascript
function clone1(target) {
    // 判断是否为对象
    if (target !== null && typeof target === 'Object') {
        if (Array.isArray(target)) {
            return [...target]
        } else {
            return {...target}
        }
    } else {
        return target;
    }
}
```

#### 2）for...in

```javascript
function clone2(target) {
    // 判断是否为对象
    if (target !== null && typeof target === 'Object') {
        const result = Array.isArray(target) ? [] : {};
        // 遍历target数据
        for (let key in target) {
            // 判断target中是否包含该属性
            if (target.hasOwnProperty(key)) {
                // 将该属性写入result中
                result[key] = target[key]
            }
        }
        return result;
    } else {
        return target;
    }
}
```

### 深拷贝

#### 例子

```javascript
let obj = {
    a: 1, b: ['e', 'f'], c: {d: 20}, g: function () {
    }
}
obj.b.push(obj.c)
obj.c.d = obj.b
deepClone3(obj)
```

#### 1）'JSON.parse(JSON.stringify())'

**缺点：1.无法克隆函数属性；2.无法解决循环引用**

```javascript
function deepClone1(target) {
    return JSON.parse(JSON.stringify(target));
}
```

#### 2）递归拷贝，解决无法克隆函数属性

```javascript
function deepClone2(target) {
    if (target !== null && typeof target === 'object') {
        const result = Array.isArray(target) ? [] : {};
        for (let key in target) {
            if (target.hasOwnProperty(key)) {
                result[key] = deepClone2(target[key])
            }
        }
        return result;
    } else {
        return target;
    }
}
```

#### 3）使用缓存容器，解决无法循环应用问题

```javascript
function deepClone3(target, map = new Map()) {
    if (target !== null && typeof target === 'object') {
        // 判断传入的map中存不存在对象的克隆
        let result = map.get(target)
        if (result) {
            return target;
        }
        result = Array.isArray(target) ? [] : {};
        map.set(target, result)
        for (let key in target) {
            if (target.hasOwnProperty(key)) {
                result[key] = deepClone3(target[key], map)
            }
        }
        return result;
    } else {
        return target;
    }
}
```

#### 4）性能优化

**数组: while | for | forEach() 优于 for-in | keys()&forEach() **

**对象: for-in 与 keys()&forEach() 差不多**

```javascript
function deepClone4(target, map = new Map()) {
    if (target !== null && target === "object") {
        let result = map.get(target);
        if (result) {
            return result;
        }
        if (Array.isArray(target)) {
            result = [];
            map.set(target, result);
            target.forEach((item, i) => {
                result[i] = deepClone4(item[i], map)
            })
        } else {
            result = {};
            map.set(target, result);
            Object.keys(target).forEach(key => {
                result[key] = deepClone4(target[key], map)
            })
        }
        return result;
    }else{
        return target;
    }
}
```

# 字符串相关

## 倒序

```javascript
function reverseString(str) {
    // return str.split('').reverse().join('')
    // return [...str].reverse().join('')
    return Array.from(str).reverse().join('')
}
```

## 检测是否回文

```javascript
function palindrome(str) {
    return str === reverseString(str)
}
```

## 截取字符串，以...隐藏多余字符串

```javascript
function truncate(str, size) {
    return str.length > size ? str.slice(0, size) + '...' : str
}
```

# 手写DOM事件监听

事件捕获与事件冒泡

![事件捕获 事件冒泡](data:image/png;base64,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)

## 事件委托函数

**将多个子元素的同类事件监听委托给(绑定在)共同的一个父组件上**

好处：

- 减少内存占用(事件监听回调从n变为
- 动态添加的内部元素也能响应

```javascript
function addEventListener(el, type, Fn, selector) {
    if (typeof el === 'string') {
        el = document.querySelector(el);
    }
    if (!selector) {
        // 如果没有selector，普通事件绑定
        el.addEventListener(type, Fn);
    } else {
        // 否则是代委托的事件绑定
        el.addEventListener(type, function (e) {
            // 获取真正发生事件的目标元素
            const target = e.target;
            // 判断目标元素是否与selector相同
            if (target.matches(selector)) {
                // 调用处理事件的回调函数Fn，绑定当前this，参数为e
                Fn.call(target, e)
            }
        })
    }
}
```

# 事件总线

```javascript
const eventBus = {callbacks: {}}

// 注册事件监听
eventBus.on = function (type, callback) {
    const callbacks = this.callbacks[type];
    if (!callbacks) {
        this.callbacks[type] = [callback];
    } else {
        callbacks.push(callback);
    }
}

// 触发事件监听
eventBus.emit = function (type, data) {
    const callbacks = this.callbacks[type];
    if (callbacks && callbacks.length > 0) {
        this.callbacks[type].forEach(cb => {
            cb(data)
        })
    }
}

// 移除事件监听
eventBus.off = function (type) {
    if (!type) {
        this.callbacks = {};
    } else {
        delete this.callbacks[type]
    }
}
```

# 消息订阅与发布

```javascript
const sub = {
    callbacks: {},
    id: 0
}

// 订阅
sub.subscribe = function (type, callback) {
    const token = "token_" + this.id;
    const callbacks = this.callbacks[type];
    if (callbacks) {
        callbacks[token] = callback
    } else {
        this.callbacks[type] = {
            [token]: callback
        }
        console.log(this.callbacks)
    }
    this.id++;
    return token;
}

// 发布
sub.publish = function (type, data) {
    const callbacks = this.callbacks[type];
    if (callbacks) {
        Object.values(this.callbacks).forEach(cb => {
            cb(data)
        })
    }
}

// 取消
sub.unsubscribe = function (flag) {
    if (flag === undefined) {
        this.callbacks = {};
    } else if (typeof flag === "string") {
        if (flag.indexOf("token_") === 0) {
            const callbacks = Object.values(this.callbacks).find(callback => callback.hasOwnProperty(flag));
            if (callbacks) {
                delete callbacks[flag]
            }
        } else {
            delete this.callbacks[flag]
        }
    } else {
        throw new Error("传入参数错误")
    }
}
```



