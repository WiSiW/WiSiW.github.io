---
title: VUE源码-数据响应式原理
date: 2021-01-26 17:55:19
tags:
	- VUE
	- 源码
---



# VUE源码-数据响应式原理

```javascript
var data = {
	name:'wsw',
	info:[
		27,
		{sex:1}
	]
}
function observe(data){
	if(Array.isArray(data)){
		data.__proto__ = arrayMethods;
		this.observerArray(data)
	}else{
		walk(data);
	}
}
function observerArray(value){
	for(let i = 0; i< value.length;i++){
		observe(value[i])
	}
}
function walk(data){
	let keys = Object.keys(data);
	keys.forEach((key) => {
		defineReactive(data,key,data[key])
	})
}
function defineReactive(data,key,value) {
    observe(value)
    Object.defineProperty(data,key,{
        get() {
            return value
        },
        set(newV) {
            if(newV == value)return;
            console.log("数据修改")
            observe(newV) 
            value = newV;
        }
    })
}
```

```javascript
// 重写数组的方法 push/shift/unshift/pop/reverse/sort/splice
let oldArrayMethods = Array.prototype;
let arrayMethods = Object.create(oldArrayMethods);
const methods = [
    'push',
    "shift",
    "unshift",
    "pop",
    "reverse",
    "sort",
    "splice"
]
methods.forEach( method => {
    arrayMethods[method] = function (...args) {
        const result = oldArrayMethods[method].apply(this,args);
        let inserted;
        let ob = this.__ob__;
        switch (method) {
            case 'push':
                break;
            case 'shift':
                break;
            case 'unshift':
                inserted = args;
                break;
            case 'pop':
                break;
            case 'reverse':
                break;
            case 'sort':
                break;
            case 'splice':
                inserted = args.splice(2)
                break;
            default:
                break;
        }
        if(inserted)ob.observerArray(inserted); // 当新增属性继续观测
        return result;
    }
})
```