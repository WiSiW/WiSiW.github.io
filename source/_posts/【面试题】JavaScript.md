---
title: 【面试题】JavaScript
date: 2021-02-04 17:20:55
tags:
	- JavaScript
	- 面试题
cover_picture: /images/javascript.png
---

# 一、数据类型

## 1.JavaScript有哪些数据类型，它们的区别？

Null、Undefined、Boolean、String、Number、Object

基本数据类型：Boolean、String、Number，因为简单、数据量小、大小固定、频繁调用，所以保存在**栈**中

引用数据类型：Object、Array、Function，因为结构复杂、占据空间大、大小不固定，所以保存在**堆**中，栈中之保存了指针，指向数据在栈中的地址

## 2.数据类型检测的方式有哪些

1.  typeOf，只能检查基本数据类型和函数，**null返回Object，因为null的类型标签为0，而Object的类型标签为000**
2.  instanceOf，只能检查引用类型，用来检测当前对象在其原型链中是否存在一个构造函数**prototype属性**
3.  constructor，能判断全部类型，因为是通过对象的原型判断，所以当原型被修改，就不能判断了
4.  Object.prototype.toString.call()，能判断全部类型

>   为什么不适用被检测对象的```toString（）```?因为Array、Function等作为Objec的实例，都重写了```toString()```方法

## 3.判断数组的方式有哪些

1.  ```Object.prototype.toString.call()```
2.  ```obj.__proto__ === Array.prototype```
3.  ```Array.isArray```
4.  ```obj instanceof Array```

## 4.null和undefined区别

null表示空对象

undefined表示未定义

## 5.typeof null 的结果是什么，为什么？

## 6.intanceof 操作符的实现原理及实现

使用```Object.getPrototypeOf()```与```while(true)```去判断构造函数的```prototype```是否在被检测对象的原型链上

## 7.为什么0.1+0.2 ! == 0.3，如何让其相等

## 8.如何获取安全的 undefined 值？

## 9.typeof NaN 的结果是什么？

## 10.isNaN 和 Number.isNaN 函数的区别？

## 11.```==``` 操作符的强制类型转换规则？

## 12.其他值到字符串的转换规则？

## 13.其他值到数字值的转换规则？

## 14.其他值到布尔类型的值的转换规则？

## 15.|| 和 && 操作符的返回值？

## 16.Object.is() 与比较操作符 ```===```  ```==```的区别？

```Object.is()```一般情况下与```===```相同，但它处理了一些特殊情况，比如```Object.is(-0,+0)  // false```和```Object.is(NaN,NaN) // true```

```==```如果两边的类型不一致，会进行强制类型转换后再进行比较

```===```会先比较类型

## 17.什么是 JavaScript 中的包装类型？

## 18.JavaScript 中如何进行隐式类型转换？

## 19.+操作符什么时候用于字符串的拼接？

## 20.为什么会有BigInt的提案？

## 21.object.assign和扩展运算法是深拷贝还是浅拷贝，两者区别

# 二、ES6

## 1.let、const、var的区别

块级作用域：解决了内层变量可能覆盖外层变量、用来计数的循环变量泄露为全局变量

变量提升：const、let只能在声明后使用

全局属性：const、let不会挂载在全局变量上，浏览器的全局变量是window，node的全局变量是global

重复声明：let、const不能重复声明

暂时性死区：let、const在声明之前，不可用

初始值设置：const使用必须设置初始值

指针指向：let可以更改指针指向，const不可以

# JavaScript为什么要进行变量提升，它导致了什么问题？

变量提升：无论在函数中处于何处声明的变量，都被提升到了函数的前端，可以在变量声明前访问而不会报错

本质原因：js引擎在代码执行前有一个解析的过程，创建饿了执行上下文，初始化了一些代码执行时需要用到的对象。当访问一个变量时，会到当前执行上下文的作用域链中去查找，而作用域链首端指向的当前执行上下文的变量对象，这个变量对象是执行上下文的一个属性，包含了函数的形参、所有函数和变量声明，这个对象是在代码解析的时候创建的

JS引擎在拿到变量或者一个函数时，会有两步操作，即解析和执行

解析：检查语法，并对函数进行预编译，解析时会创建一个**全局执行上下文环境**，先把代码中即将执行的变量和函数声明拿出来，变量先赋值为undefined，函数先声明好可使用，在一个函数执行之前，也会创建一个**函数执行上下文环境**，比全局执行上下文多出this

为什么？

**提高性能，让函数可以在执行时预先为变量分配栈空间**：在代码执行前，只对代码执行一次语法检查和预编译，统计声明了哪些变量、创建了哪些函数，并对代码进行压缩，去除注释和不必要的空白，这样每次执行函数时都可以为该函数分配栈空间（不需要再去解析一遍获取代码中声明了哪些变量，创建了哪些函数）

**容错性更好**

## 2.const对象的属性可以修改吗

## 3.如果new一个箭头函数的会怎么样

## 4.箭头函数与普通函数的区别

简洁

没有自己的this，只会在自己的作用域上一层继承this，因此箭头函数的this指向在定义时就已经确定了，之后不会改变

this指向不能被修改

不能作为构造函数使用，因为没有自己的this

没有自己的arguments

没有原型链prototype

不能作为构造函数，也不能使用yeild关键字

## 5.箭头函数的this指向哪⾥？

是捕获其所在上下⽂的 this 值，作为⾃⼰的 this 值，并且由于没有属于⾃⼰的this，所以是不会被new调⽤的，这个所谓的this也不会被改变

```javascript
// ES6 
const obj = { 
  getArrow() { 
    return () => { 
      console.log(this === obj); 
    }; 
  } 
}
```



```javascript
// ES5，由 Babel 转译
var obj = { 
   getArrow: function getArrow() { 
     var _this = this; 
     return function () { 
        console.log(_this === obj); 
     }; 
   } 
};
```



## 6.扩展运算符的作用及使用场景

## 7.Proxy 可以实现什么功能？

## 8.对对象与数组的解构的理解

## 9.如何提取高度嵌套的对象里的指定属性？

## 10.对 rest 参数的理解

## 11.ES6中模板语法与字符串处理

# 三、JavaScript基础

## 1.new操作符的实现原理

创建一个对象

设置当前对象的原型，将对象的原型设置为函数的```prototype```对象

修改函数的this指向当前对象

判断函数的返回值类型，如果是值，返回创建的对象。如果是引用类型，返回引用类型的对象

````javascript
function myNew() {
  let obj = null;
  let constructor = Array.prototype.shift.call(arguments);
  let res = null;
  if (typeOf constructor !== "function") {
    console.error("type errror");
    return;
  }
  obj = Object.create(constructor.prototype);
  res = constructor.apply(obj, arguments);
  let flag = res && ( typeOf res === "object" || typeOf res === "function");
  return flag ? res : obj
}
````



## 2.map和Object的区别

|          | Map                                      | Object                                                 |
| -------- | ---------------------------------------- | ------------------------------------------------------ |
| 键的命名 | 默认情况不包含任何键，只包含显式插入的键 | 有一个原型，原型链上的键名可能会与在对象设置的键名冲突 |
| 键的类型 | 任意值，包括函数、对象或任意基本类型     | 必须是```String```或```Symbol```                       |
| 键的顺序 | 有序，当迭代时，以插入的顺序返回键值     | 无序                                                   |
| size     | 通过```size```属性获取                   | 只能手动计算                                           |
| 迭代     | 可以直接被迭代                           | 需要先获取键然后才能迭代                               |
| 性能     | 适用于频繁增删键值对的情况               |                                                        |



## 3.map和weakMap的区别

## 4.JavaScript有哪些内置对象

**全局的对象（global objects）**

1.  值属性：
2.  函数属性
3.  基本对象
4.  数字和日期
5.  字符串
6.  可索引的集合对象
7.  使用键的集合对象
8.  矢量集合
9.  结构化数据
10.  控制抽象对象
11.  反射
12.  国际化
13.  WebAssembly
14.  其他

## 5.常用的正则表达式有哪些？

## 6.对JSON的理解

## 7.JavaScript脚本延迟加载的方式有哪些？

1.  defer，让脚本的加载和文档的解析同步解析，然后在文档解析完成后再执行这个脚本，保证页面渲染不被阻塞，执行顺序可测，但在一些浏览器中不会
2.  async，使脚本异步加载，不会阻塞页面的渲染进程，但是当脚本加载完成后会立即执行，这个时候如果文档没有解析完成会阻塞，渲染顺序不可测
3.  动态创建DOM
4.  setTimeout
5.  脚本最后加载

## 8.JavaScript 类数组对象的定义？

## 9.数组有哪些原生方法？

## 10.Unicode、UTF-8、UTF-16、UTF-32的区别？

## 11.常见的位运算符有哪些？其计算规则是什么？

## 12.为什么函数的 arguments 参数是类数组而不是数组？如何遍历类数组?

## 13.什么是 DOM 和 BOM？

## 14.对类数组对象的理解，如何转化为数组

## 15.escape、encodeURI、encodeURIComponent 的区别

## 16.对AJAX的理解，实现一个AJAX请求

```javascript
let xhr = new XMLHttpRequest();
xhr.open("GET", url, true);
xhr.onreadystatechange = function() {
  if (this.readyState !== 4) return;
  if (this.status === 200) {
    handle(this.response);
  } else {
    console.error(this.statusText);
  }
}
xhr.onerror = function() {
  console.log(this.statusText);
}
// 设置相应数据类型
xhr.responseType = "json"
// 设置请求头信息
xhr.setRequestHeader("Accept", "application/json");
xhr.send(null)
```

## 17.

## 18.什么是尾调用，使用尾调用有什么好处？

## 19.ES6模块与CommonJS模块有什么异同？

## 20.常见的DOM操作有哪些

## 21.use strict是什么意思 ? 使用它区别是什么？

## 22.如何判断一个对象是否属于某个类？

## 23.强类型语言和弱类型语言的区别

## 24.解释性语言和编译型语言的区别

（1）**解释型语言** 使用专门的解释器对源程序逐行解释成特定平台的机器码并立即执行。是代码在执行时才被解释器一行行动态翻译和执行，而不是在执行之前就完成翻译。解释型语言不需要事先编译，其直接将源代码解释成机器码并立即执行，所以只要某一平台提供了相应的解释器即可运行该程序。其特点总结如下

-   解释型语言每次运行都需要将源代码解释称机器码并执行，效率较低；
-   只要平台提供相应的解释器，就可以运行源代码，所以可以方便源程序移植；
-   JavaScript、Python等属于解释型语言。

（2）**编译型语言** 使用专门的编译器，针对特定的平台，将高级语言源代码一次性的编译成可被该平台硬件执行的机器码，并包装成该平台所能识别的可执行性程序的格式。在编译型语言写的程序执行之前，需要一个专门的编译过程，把源代码编译成机器语言的文件，如exe格式的文件，以后要再运行时，直接使用编译结果即可，如直接运行exe文件。因为只需编译一次，以后运行时不需要编译，所以编译型语言执行效率高。其特点总结如下：

-   一次性的编译成平台相关的机器语言文件，运行时脱离开发环境，运行效率高；
-   与特定平台相关，一般无法移植到其他平台；
-   C、C++等属于编译型语言。

**两者主要区别在于：** 前者源程序编译后即可在该平台运行，后者是在运行期间才编译。所以前者运行速度快，后者跨平台性好。

## 25.for...in和for...of的区别

for...in遍历的是对象的键名，for...of遍历的是对象的键值

```javascript
var list = [1,2,3]
for(var i in list) {
  console.log(i)
}
// 0，1，2
for(var i in list) {
  console.log(i)
}
// 1，2，3
```

for...in会遍历对象的整个原型链

对于数组for...in会返回数组中所有可枚举的属性（包括原型链上可枚举的属性），for...in只会返回数组的下标对应的属性值

**```for...in```主要用于遍历对象，不适用于数组，```for...of```可以用来遍历所有包含迭代器接口的数据结构，返回各项的值**

## 26.如何使用for...of遍历对象

类数组对象

```javascript
var obj = {
    0:'one',
    1:'two',
    length: 2
};
obj = Array.from(obj);
for(var k of obj){
    console.log(k)
}
```



非类数组对象

```javascript
//方法一：
var obj = {
    a:1,
    b:2,
    c:3
};

obj[Symbol.iterator] = function(){
	var keys = Object.keys(this);
	var count = 0;
	return {
		next(){
			if(count<keys.length){
				return {value: obj[keys[count++]],done:false};
			}else{
				return {value:undefined,done:true};
			}
		}
	}
};

for(var k of obj){
	console.log(k);
}


// 方法二
var obj = {
    a:1,
    b:2,
    c:3
};
obj[Symbol.iterator] = function*(){
    var keys = Object.keys(obj);
    for(var k of keys){
        yield [k,obj[k]]
    }
};

for(var [k,v] of obj){
    console.log(k,v);
}
```



## 27.ajax、axios、fetch的区别

**AJAX**：是一种创建交互式网页应用的网页开发技术，是在一种无需重新加载整个页面的情况下，能够更新部分网页的技术，通过后台与服务器进行少量数据交换，可以是网页实现一步更新

缺点：1）本身针对MVC编程，不符合MVVM规范；2）基于远程XHR开发，XHR本身架构不够清晰；3）不符合关注分离的原则；4）配置和调用方式非常混乱，而且基于时间的一步模型不友好

**Fetch**：基于ES6的promise对象设计，是原生js

优点：1）语法简洁，更加语义化；2）基于标准Promise实现，支持async/await；3）更加底层，提供丰富的API；4）脱离了XHR，是基于ES6的新规范实现的

缺点：1）只对网络请求报错，第400、500都当做成功的请求，不会reject，只有网络请求错误导致请求不能完成是，才会reject；2）默认不带cookie，需要添加配置项；3）不支持abort，不支持超时控制，setTimeout和Promise.reject的实现的超时控制并不能阻止请求过程在后台的运行；4）没办法原生监测请求的进度，XHR可以

**Axios**：是基于Promise封装的http请求客户端

1）不同端发起不同请求（浏览器端发起XMLHttpRequest请求，Node端发起http请求；2）支持Promise的API；3）监听请求和返回；4）对请求和返回进行转化；5）取消请求；6）自动转换json数据；7）客户端支持抵御XSRF攻击

## 28.数组的遍历方法有哪些

## 29.forEach和map方法有什么区别

# 四、原型与原型链

## 1.对原型、原型链的理解

js中是使用构造函数来新建一个对象，在每个构造函数内部都有一个prototype属性，它的属性值是一个对象，包含了可以由该创造函数的所有实例共享的属性和方法。当使用构造函数创建一个新的对象后，在这个对象内部将包含一个指针，指向构造函数的prototype属性对应的值，这个指针被称为原型，可以使用```Object.getPrototype()```来获取

当访问一个对象的属性时，如果当前对象内部不存在这个属性，那么就会去它的原型对象中去寻找这个属性，而这个原型对象又会有自己的原型，一直找下去的链路成为 **原型链**。原型链的尽头一般都是```Object.prototype.__proto__```为null

js对象是通过引用来传递的，创建的每个对象实体中并没有一份属于自己的原型副本，因此当修改原型时，与之相关的对象也会继承这一改变

## 2.原型修改、重写

## 3.原型链指向

## 4.原型链的终点是什么？如何打印出原型链的终点？

```Object.prototype.__proto__```

## 5.如何获得对象非原型链上的属性？

``hasOwnProperty```

# 五、执行上下文/作用域链/闭包

## 1.对闭包的理解

有权访问另一函数作用域中变量的函数

用途：

1.  在函数外部能够访问函数内部的变量，创建私有变量，vue的判断是否是html标签
2.  将已经结束的函数上下文中的变量对象继续保存在内存中



## 2.对作用域、作用域链的理解

全局作用域

最外层函数和最外层函数外面定义的变量拥有全局作用域

所有未定义直接赋值的变量自动声明为全局作用域

所有window对象的属性拥有全局作用域

全局作用域有很大的弊端，过多的全局作用域变量会污染全局命名空间，容易引起命名冲突。



函数作用域

函数作用域声明在函数内部的变零，一般只有固定的代码片段可以访问到

作用域是分层的，内层作用域可以访问外层作用域，反之不行



块级作用域

使用ES6中新增的let和const指令可以声明块级作用域，块级作用域可以在函数中创建也可以在一个代码块中的创建（由`{ }`包裹的代码片段）

let和const声明的变量不会有变量提升，也不可以重复声明

在循环中比较适合绑定块级作用域，这样就可以把声明的计数器变量限制在循环内部。



**作用域链：** 在当前作用域中查找所需变量，但是该作用域没有这个变量，那这个变量就是自由变量。如果在自己作用域找不到该变量就去父级作用域查找，依次向上级作用域查找，直到访问到window对象就被终止，这一层层的关系就是作用域链。

作用域链的作用是**保证对执行环境有权访问的所有变量和函数的有序访问，通过作用域链，可以访问到外层环境的变量和函数。**



作用域链的本质上是一个指向变量对象的指针列表。变量对象是一个包含了执行环境中所有变量和函数的对象。作用域链的前端始终都是当前执行上下文的变量对象。全局执行上下文的变量对象（也就是全局对象）始终是作用域链的最后一个对象。

## 3.对执行上下文的理解

执行上下文分为 **全局执行上下文**和 **函数执行上下文**，

全局执行上下文：js执行时，会创建一个全局的window对象，并且设置this的值等于这个全局对象，任何不在函数内部的都是全局执行上下文，一个程序中只会存在一个全局执行上下文

函数执行上下文：函数执行时，会创建一个新的执行上下文



**执行上下栈**：js引擎使用执行上下栈来管理执行上下文

当执行代码时，首先遇到全局代码，会创建一个全局执行上下文并且压入执行栈中，每当遇到一个函数调用，就会为该函数创建一个新的执行上下文并且压入栈顶，引擎会执行位于上下文栈顶的函数，当函数执行完毕后，执行上下文从栈中弹出，继续执行下一个上下文，当所有代码都执行完毕后，从栈中弹出全局执行上下文



**创建执行上下文**：

创建执行上下文有两个阶段：**创建阶段**和**执行阶段**

**1）创建阶段**

（1）this绑定

-   在全局执行上下文中，this指向全局对象（window对象）
-   在函数执行上下文中，this指向取决于函数如何调用。如果它被一个引用对象调用，那么 this 会被设置成那个对象，否则 this 的值被设置为全局对象或者 undefined

（2）创建词法环境组件

-   词法环境是一种有**标识符——变量映射**的数据结构，标识符是指变量/函数名，变量是对实际对象或原始数据的引用。
-   词法环境的内部有两个组件：**加粗样式**：环境记录器:用来储存变量个函数声明的实际位置**外部环境的引用**：可以访问父级作用域

（3）创建变量环境组件

-   变量环境也是一个词法环境，其环境记录器持有变量声明语句在执行上下文中创建的绑定关系。

**2）执行阶段** 此阶段会完成对变量的分配，最后执行完代码。

**简单来说执行上下文就是指：**

在执行一点JS代码之前，需要先解析代码。解析的时候会先创建一个全局执行上下文环境，先把代码中即将执行的变量、函数声明都拿出来，变量先赋值为undefined，函数先声明好可使用。这一步执行完了，才开始正式的执行程序。

在一个函数执行之前，也会创建一个函数执行上下文环境，跟全局执行上下文类似，不过函数执行上下文会多出this、arguments和函数的参数。

-   全局上下文：变量定义，函数声明
-   函数上下文：变量定义，函数声明，`this`，`arguments`



# 六、this/call/apply/bind

## 1.对this对象的理解

this 是执行上下文中的一个属性，它指向最后一次调用这个方法的对象。在实际开发中，this 的指向可以通过四种调用模式来判断。

-   **函数调用模式**，当一个函数不是一个对象的属性时，直接作为函数来调用时，this 指向全局对象。
-   **方法调用模式**，如果一个函数作为一个对象的方法来调用时，this 指向这个对象。
-   **构造器调用模式**，如果一个函数用 new 调用时，函数执行前会新创建一个对象，this 指向这个新创建的对象。
-    **apply 、 call 和 bind 调用模式**，这三个方法都可以显示的指定调用函数的 this 指向。其中 apply 方法接收两个参数：一个是 this 绑定的对象，一个是参数数组。call 方法接收的参数，第一个是 this 绑定的对象，后面的其余参数是传入函数执行的参数。也就是说，在使用 call() 方法时，传递给函数的参数必须逐个列举出来。bind 方法通过传入一个对象，返回一个 this 绑定了传入对象的新函数。这个函数的 this 指向除了使用 new 时会被改变，其他情况下都不会改变。

## 2.call() 和 apply() 的区别？

```apply()```只接受两个参数，第一个参数代表函数体内this对象的指向，第二个参数为一个带下标的集合，可以为数组或类数组，作为参数传递给被调用的函数

```call()```接受的参数不固定，但第一个参数代表函数体内this对象的指向，之后的参数作为被调用函数的参数

## 3.实现call、apply 及 bind 函数

```javascript
function call(context) {
  if (typeOf this !== 'function') {
    console.error("this is not a function")
    return
  }
  context = context || window
  const args = [...arguments]slice[1]
  context.fn = this
  const res = context.fn(args)
  delete context.fn()
  return res
}
```



# 七、异步编程

## 1.异步编程的实现方式？

## 2.setTimeout、Promise、Async/Await 的区别

## 3.对Promise的理解

Promise是异步编程的一种解决方案，它是一个对象，可以获取异步操作的消息，他的出现大大改善了异步编程的困境，避免了地狱回调，它比传统的解决方案回调函数和事件更合理和更强大。

所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理

## 4.Promise的基本用法

## 5.Promise解决了什么问题

## 6.Promise.all和Promise.race的区别的使用场景

## 7.对async/await 的理解

## 8.await 到底在等啥？

## 9.async/await的优势

## 10.async/await对比Promise的优势

代码读起来更加同步，Promise虽然摆脱了回调地狱，但是then的链式调⽤也会带来额外的阅读负担

Promise传递中间值⾮常麻烦，⽽async/await⼏乎是同步的写法，⾮常优雅

错误处理友好，async/await可以⽤成熟的try/catch，Promise的错误捕获⾮常冗余

调试友好，Promise的调试很差，由于没有代码块，你不能在⼀个返回表达式的箭头函数中设置断点，如果你在⼀个.then代码块中使⽤调试器的步进(step-over)功能，调试器并不会进⼊后续的.then代码块，因为调试器只能跟踪同步代码的每⼀步。

## 11.async/await 如何捕获异常

## 12.并发与并行的区别？

-   并发是宏观概念，我分别有任务 A 和任务 B，在一段时间内通过任务间的切换完成了这两个任务，这种情况就可以称之为并发。
-   并行是微观概念，假设 CPU 中存在两个核心，那么我就可以同时完成任务 A、B。同时完成多个任务的情况就可以称之为并行。

## 13.什么是回调函数？回调函数有什么缺点？如何解决回调地狱问题？

## 14.setTimeout、setInterval、requestAnimationFrame 各有什么特点？

# 八、面向对象

## 1.对象创建的方式有哪些？

1.  工厂模式
2.  构造函数模式
3.  原型模式
4.  组合使用构造函数和原型模式
5.  动态原型模式
6.  寄生构造函数模式

## 2.对象继承的方式有哪些？

1.  原型链继承，但因为原型被所有实例对象共享，在修改引用类型的数据时，会造成污染
2.  构造函数继承，通过在子类型的函数中调用超类型的构造函数来实现
3.  组合继承，将原型继承和构造函数继承组合起来
4.  原型式继承
5.  寄生式继承
6.  寄生式组合继承

# 九、垃圾回收与内存泄漏

## 1.浏览器的垃圾回收机制

## 2.哪些情况会导致内存泄漏









# 基础

## 1.null 和 undefined 的区别

- `null` 表示 `无` 的对象，也就是此处不应该有值；而 `undefined` 表示未定义。
- 在转换数字的时候， `Number(null)` 为 `0`，而 `Number(undefined)` 为 `NaN`



- `null`：

1. 作为函数的参数，表示该函数的参数不是对象。
2. 作为对象原型链的终点。 `Object.prototype.__proto__ === null`

- `undefined`：

1. 变量被声明但是没有赋值，等于 `undefined`。
2. 调用函数时，对应的参数没有提供，也是 `undefined`。
3. 对象没有赋值，这个属性的值为 `undefined`。
4. 函数没有返回值，默认返回 `undefined`。

# new关键字
新建一个对象obj
把obj的和构造函数通过原型链连接起来
将构造函数的this指向obj
如果该函数没有返回对象，则返回this


## 2.事件流



## 3.typeof 和 instanceof 的区别

- `typeof`：对某个变量类型的检测，基本类型除了 `null` 之外，都能正常地显示为对应的类型，引用类型除了函数会显示为 `function`，其他都显示为 `object`。
- `instanceof` 主要用于检测某个构造函数的原型对象在不在某个对象的原型链上。



#### 为什么`"object"`?

因为 JavaScript 早起版本是 32 位系统，为了性能考虑使用低位存储变量的类型信息，`000` 开头代表是对象然而 `null` 表示为全零，所以它错误判断为 `object`

## 4.一句话描述this

对于函数而言，指向最后调用函数的那个对象，是函数运行时内部自动生成的一个内部对象，只能在函数内部使用；对于全局而言，`this` 指向 `window`

- 普通函数中 `this` 的指向，是 `this` **执行时**的上下文
- 箭头函数中 `this` 的指向，是 `this` **定义时**的上下文



#### 改变this的指向？

1）保存指针`let that = this`

2）使用箭头函数：因为箭头函数不会创建其自身的执行上下文，函数中的this取决于外部函数，即谁调用它`this`就继承谁

3）call/apply/bind

## 5.执行上下文



## 闭包

### 是什么？

在JavaScript中，根据作用域规则，内部函数总是可以访问外部函数声明的变量。

当通过调用一个外部函数返回一个内部函数后，即使该外部函数已经执行结束了，但是内部函数应用外部函数的变量依旧保存在内存中，这些变量的集合称为闭包

**函数B在函数A中调用了A中的变量，B就称之为A的闭包**

### 优缺点

- 优点

1）缓存持久化

2）实现柯里化

- 缺点

1）内存消耗：闭包产生的变量无法被销毁

2）性能问题：闭包内部变量优先级高于外部变量，所以需要作用域链要多查找一级，影响查找速度

## 柯里化

### 是什么？

通过将接收多个参数换成一个参数，每次调用返回新函数的技术

1. 通过闭包管理
2. 支持链式调用
3. 每次运行返回一个 `function`

### 为什么？

- 参数复用

```javascript
// 校验数字
let numberReg = /[0-9]+/g;

// 校验小写字母
let stringReg = /[a-z]+/g;

// currying 后
function curryingCheck(reg) {
  return function(txt) {
    return reg.test(txt);
  }
}

// 校验数字
let checkNumber = curryingCheck(numberReg);
let checkString = curryingCheck(stringReg);

// 使用
console.log(checkNumber('13888888888')); // true
console.log(checkString('jsliang')); // true
```



- 提前确认
- 延迟运行

## 作用域与作用域链

- 作用域：



- 作用域链：本质上是一个指向变量对象的指针列表

## 变量提升与函数提升

函数提升是为了解决相互递归的问题，大体上可以解决自上而下的顺序问题

## 原型与原型链

### 是什么？

- 原型：在JavaScript中，每当定义一个对象时，对象中都会包含一些预定义的属性。

其中每个函数对象都有一个`prototype`属性，这个属性的指向被称为这个函数对象的原型对象（简称原型）

- 原型链：查找原型上存在的某个属性的的链式路径

如果某个实例对象不存在的某个属性，那么JavaScript会去改构造函数的原型上去找；如果原型上没有找到，那么会继续往`Object`的原型上，如果`Object`的原型上还是没有，会返回`undefined`

### 为什么？

为了节约内存空间

## `__proto__`与`prototype`的关系

每个 JavaScript 对象（普通对象和函数对象）都具有一个属性 `__proto__`，这个属性会指向该对象的原型。

## 堆与栈



## Event Loop（事件循环）

### 为什么？

因为JavaScript是单线程语言，一次只能执行一件程序。单线程在程序执行的时候，所走的程序路径按照连续顺序排下来，依次执行，只有前面的程序执行结束才会执行接下来的程序。

如果遇到一些需要等待的程序，比如说`setTimeOut`就会造成延迟

因此，JavaScript为了协调时间、用户交互、脚本、渲染、网络等，就使用事件循环（Event Loop）

### 是什么？

从`script`开始读取，不断循环，从“任务队列”中读取执行事件的过程。

- 执行过程

1. 一开始整个脚本`script`作为一个宏任务执行
2. 执行过程中，**同步代码**直接执行，**宏任务**进入宏任务队列，**微任务**进入微任务队列
3. 当前宏任务执行出对，检查微任务列表是否存在，有则一次执行，直至全部执行完毕
4. 执行浏览器的UI线程的渲染工作
5. 检查是否有`web worker`任务，有则执行
6. 执行完本轮的宏任务，回到步骤2，依次循环，知道宏任务和微任务为空



## 宏任务、微任务

### 是什么？

事件循环的异步队列，是为了解决单线程阻塞的问题

**宏任务队列可以有多个，但是微任务队列只有一个**

- 宏任务：`script`、`setTimeout`、`setInterval`、`setImmediate`、`I/O`、`UI rendering`

- 微任务：`MutationObserver`、`Promise.then()/catch()`、fetch/axios（以Promise为基础开发）、V8垃圾回收过程、`process.nextTick`（Node独有）

## babel 编译原理

- `babylon` 将 `ES6/ES7` 代码解析成 `AST`
- `babel-traverse` 对 `AST` 进行遍历转译，得到新的 `AST`
- 新 `AST` 通过 `babel-generator` 转换成 `ES5`

## 浏览器渲染过程

![](data:image/png;base64,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)

### 1）创建DOM树

HTML解析器解析HTML元素，创建DOM树

### 2）创建CSS规则树

CSS解析器解析CSS文件和内联与行内样式，生成页面的样式表

### 3）创建Render树

将DOM树与CSS规则树关联起来，创建Render树

### 4）布局Layout

根据Render树，浏览器开始布局，为每个Render树上的节点确定一个在显示器出现的精确坐标

### 5）绘制Painting

在Render树和节点显示坐标的基础上，调用每个节点的`paint`方法，绘制出来



- 重绘：当元素样式的改变不影响布局时，只会对元素进行更新，此时只需要UI层面的额像素绘制，损耗较少

触发条件：background、color、visibility

- 回流：当元素尺寸、结构或者触发某些属性时，浏览器会重新渲染页面

触发条件：1）添加删除DOM元素；2）修改边框、边距、宽高等影响元素定位 的属性；3）浏览器窗口resize

**回流必定会发生重绘，重绘不一定会引发回流。**



```javascript
function a() {
    console.log(1)
    return "2"
}

function b() {
    try{
        console.log(3)
        return a()
    }catch (e) {
        console.log(5)
        return "6"
    }finally {
        console.log(7)
        return "8"
    }
    return "9"
}
console.log(b()) // 3 1 7 8
// finally有return  那么return的就是finally的
```
