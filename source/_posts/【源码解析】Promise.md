---
title: 【源码解析】Promise
date: 2021-01-27 15:45:29
categories: 前端
tags:
	- Promise
	- 源码解析
	- JavaScript
cover_picture: /images/promise.png
---



# 构造函数

```javascript
/**
 * Promise构造函数
 * excutor：执行器函数（同步执行）
 * @param excutor
 */
function Promise(executor) {
    const _this = this;
    _this.status = "pending"; // 给Promise对象指定status属性，初始值为pending
    _this.data = undefined; // 给Promise对象指定一个用于存储结果数据的属性
    _this.callbacks = []; // 每个元素的结构：{onResolve(){},onReject(){}}

    function resolve(value) {
        // 如果状态已经修改，返回
        if (_this.status !== "pending") return;
        // 修改状态为fullFilled
        _this.status = "fullFilled";
        // 保存value数据
        _this.data = value;
        if (_this.callbacks.length > 0) {
            setTimeout(function () {
                // 放入队列中执行所有成功的回调
                _this.callbacks.forEach(callbackObj => {
                    callbackObj.onResolved(value)
                })
            })
        }
    }

    function reject(reason) {
        // 如果状态已经修改，返回
        if (_this.status !== "pending") return;
        // 修改状态为fullFilled
        _this.status = "rejected";
        // 保存value数据
        _this.data = reason;
        if (_this.callbacks.length > 0) {
            setTimeout(function () {
                // 放入队列中执行所有成功的回调
                _this.callbacks.forEach(callbackObj => {
                    callbackObj.onRejected(reason)
                })
            })
        }
    }

    // 立即同步执行excutor
    try {
        executor(resolve, reject)
    } catch (reason) {
        // 如果执行器抛出异常，Promise对象变为rejected状态
        reject(reason)
    }
}

/**
 * Promise原型对象的then()，指定成功和失败的回调函数，返回一个新的Promise对象
 * @param onResolved
 * @param onRejected
 */
Promise.prototype.then = function (onResolved, onRejected) {

    onResolved = typeof onResolved === 'function' ? onResolved : value => value; // 向后传递成功的value
    // 指定默认的失败回调（实现错误/异常传递的关键点）
    onRejected = typeof onRejected === 'function' ? onRejected : reason => {
        throw reason
    } // 向后传递失败的reason

    const _this = this;
    /**
     调用指定回调函数执行
     */
    return new Promise((resolve, reject) => {
        function handler(callback) {
            /**
             1. 如果抛出异常，return的Promise就会失败，reason就是error
             2. 如果回调函数返回的不是Promise，return的Promise就会成功，value就是返回的值
             3. 如果回调函数返回的是Promise，return的Promise结果就是这个Promise的结果
             */
            try {
                const result = callback(_this.data);
                if (result instanceof Promise) {
                    result.then(
                        value => resolve(value), // 当result成功时，让return的Promise也成功
                        reason => reject(reason) // 当result失败时，让return的Promise也失败
                    )
                    // 简洁写法
                    // result.then(resolve,reject)
                } else {
                    resolve(result)
                }
            } catch (error) {
                reject(error)
            }
        }

        // 当前状态还是pending状态，将回调函数保存
        if (_this.status === "pending") {
            _this.callbacks.push({
                onResolved() {
                    handler(onResolved)
                },
                onRejected() {
                    handler(onRejected)
                }
            })
        } else if (_this.status === "fullFilled") {
            // 如果当前是fullFilled状态，异步执行onResolve并改变return的promise状态
            setTimeout(() => {
                handler(onResolved)
            })
        } else {
            // 如果当前是rejected状态，异步执行onReject并改变return的promise状态
            setTimeout(() => {
                handler(onRejected)
            })
        }
    })
};

/**
 * Promise原型对象的catch()，指定失败的回调函数，返回一个新的Promise对象
 * @param onRejected
 */
Promise.prototype.catch = function (onRejected) {
    return this.then(undefined, onRejected)
};

/**
 * Promise函数对象的resolve方法，返回一个指定value的成功的Promise
 * @param value
 */
Promise.resolve = function (value) {
    // 返回一个成功/失败的Promise
    return new Promise((resolve, reject) => {
        // value是promise
        if (value instanceof Promise) {
            value.then(resolve, reject)
        } else {
            resolve(value)
        }
    })
}

/**
 * Promise函数对象的reject方法，返回一个指定reason的失败的Promise
 * @param reason
 */
Promise.reject = function (reason) {
    // 返回一个失败的promise
    return new Promise((resolve, reject) => {
        reject(reason)
    })
}

/**
 * Promise函数对象的all方法，返回一个Promise，只有当所有的Promise都成功时才成功，否则只要有一个失败的就失败
 * @param promises
 */
Promise.all = function (promises) {
    // 保存所有成功value的数组
    const values = new Array(promises.length);
    // 用来保存成功Promise的数量
    let count = 0;
    return new Promise((resolve, reject) => {
        // 遍历获取每个Promise的结果
        promises.forEach((p, index) => {
            p.then(
                value => {
                    // 成功了数量+1
                    count++;
                    // p成功，将成功的value保存入values
                    values[index] = value;
                    // 如果全部成功了，将return的Promise状态改为成功
                    if (count === promises.length) resolve(values);
                },
                reason => {
                    // 只要一个失败了，return的Promise就会失败
                    reject(reason)
                }
            )
        })
    })
}

/**
 * Promise函数对象的race方法，返回一个Promise，只有当所有的Promise都成功时才成功，否则只要有一个失败的就失败
 * @param promises
 */
Promise.race = function (promises) {
    return new Promise((resolve, reject) => {
        // 遍历promises获取每个promise的结果
        promises.forEach((p, index) => {
            p.then(
                value => {
                    // 一旦成功了，就将返回的Promise状态变为成功
                    resolve(value)
                },
                reason => {
                    // 一旦失败了，就将返回的Promise状态变为失败
                    reject(reason)
                }
            )
        })
    })
}

/**
 * 返回一个Promise对象，在指定的时间后才确定结果
 * @param value
 * @param time
 */
Promise.resolveDelay = function (value, time) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (value instanceof Promise) {
                value.then(resolve, reject)
            } else {
                resolve(value)
            }
        }, time)
    })
}

/**
 * 返回一个Promise对象，在指定的时间后才确定失败
 * @param reason
 * @param time
 */
Promise.rejectDelay = function (reason, time) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            reject(reason)
        })
    }, time)
}
window.Promise = Promise;
```

# class版本

```javascript
class Promise {
    constructor(executor) {
        const _this = this;
        _this.status = "pending"; // 给Promise对象指定status属性，初始值为pending
        _this.data = undefined; // 给Promise对象指定一个用于存储结果数据的属性
        _this.callbacks = []; // 每个元素的结构：{onResolve(){},onReject(){}}

        function resolve(value) {
            // 如果状态已经修改，返回
            if (_this.status !== "pending") return;
            // 修改状态为fullFilled
            _this.status = "fullFilled";
            // 保存value数据
            _this.data = value;
            if (_this.callbacks.length > 0) {
                setTimeout(function () {
                    // 放入队列中执行所有成功的回调
                    _this.callbacks.forEach(callbackObj => {
                        callbackObj.onResolved(value)
                    })
                })
            }
        }

        function reject(reason) {
            // 如果状态已经修改，返回
            if (_this.status !== "pending") return;
            // 修改状态为fullFilled
            _this.status = "rejected";
            // 保存value数据
            _this.data = reason;
            if (_this.callbacks.length > 0) {
                setTimeout(function () {
                    // 放入队列中执行所有成功的回调
                    _this.callbacks.forEach(callbackObj => {
                        callbackObj.onRejected(reason)
                    })
                })
            }
        }

        // 立即同步执行executor
        try {
            executor(resolve, reject)
        } catch (reason) {
            // 如果执行器抛出异常，Promise对象变为rejected状态
            reject(reason)
        }
    }

    then(onResolve, onReject) {
        onResolve = typeof onResolve === 'function' ? onResolve : value => value;
        onReject = typeof onReject === 'function' ? onReject : reason => throw reason;

        const _this = this;

        return new Promise((resolve, reject) => {
            function handle(callback) {
                try {
                    const result = callback(_this.data)
                    if (result instanceof Promise) {
                        result.then(
                            value => resolve(value),
                            reason => reject(reason)
                        )
                    } else {
                        resolve(result)
                    }
                } catch (error) {
                    reject(error)
                }
            }

            if (_this.status === "pending") {
                _this.callbacks.push({
                    onResolved() {
                        handle(onResolve)
                    },
                    onRejected() {
                        handle(onReject)
                    }
                })
            } else if (_this.status === "fullFilled") {
                setTimeout(() => {
                    handle(resolve)
                })
            } else if (_this.status === "rejected") {
                setTimeout(() => {
                    handle(reject)
                })
            }
        })
    }

    catch(onRejected) {
        return this.then(undefined, onRejected)
    }

    static resolve(value){
        return new Promise((resolve,reject)=>{
            if(value instanceof Promise){
                value.then(resolve,reject)
            }else{
                resolve(value)
            }
        })
    }

    static reject(reason){
        return new Promise((resolve,reject)=>{
            reject(reason)
        })
    }

    static all(promises) {
        return new Promise((resolve, reject) => {
            const values = new Array(promises.length);
            let count = 0;
            promises.forEach((p, index) => {
                p.then(
                    value => {
                        count++;
                        values[index] = value;
                        if (count === promises.length) resolve(values)
                    },
                    reject
                )
            })
        })
    }

    static race(promise) {
        return new Promise((resolve, reject) => {
            promise.forEach((p, index) => {
                p.then(
                    value => resolve(value),
                    reason => reject(reason)
                )
            })
        })
    }

    static resolveDelay(value, time) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                if (value instanceof Promise) {
                    value.then(resolve,reject)
                } else {
                    resolve(value)
                }
            }, time)
        })
    }

    static rejectDelay(reason, time) {
        return new Promise((resolve,reject)=>{
            setTimeout(()=>{
                reject(reason)
            },time)
        })
    }
}
```
