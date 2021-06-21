---
title: Vue面试题
categories: [前端,[Vue]]
date: 2020-01-25 15:17:13
tags:
	- VUE
	- 面试题
author: 不爽的麻雀
comments: true
---

# 生命周期

**VUE实例从创建到销毁的过程**

浏览器渲染有8个钩子，服务端渲染只有beforeCreate和created

## beforeCreate

当前阶段data、methods、computed、watch上的数据和方法都不能访问

**可以在这加个loading事件，在加载实例时触发**

## created

实例创建完成，当前阶段已经完成了数据观测，此时修改数据不会触发updated函数，可以在当前阶段获取一些初始数据，但DOM节点`$el`未创建，无法进行交互，但是可以通过$nextTick访问

**初始化完成时的事件写在这里，如在这结束loading事件，异步请求也适宜在这里调用**

## beforeMount

挂载之前，在这之前template模板已导入渲染函数编译，虚拟DOM已经创建完成，即将开始渲染，此时可以进行数据更改，不会触发updated

## mounted

挂载完成之后，真实DOM挂载完毕，数据完成双向绑定，可以访问到DOM节点，可以使用$ref对DOM进行操作

**获取到DOM节点;对数据统一处理**

## beforeUpdate

发生在更新之前，通过响应式数据更新触发，发生在虚拟DOM重新渲染之前，可以在当前阶段更改数据，不会造成重新渲染

## update

更新完成之后，当前节点组件DOM更新完成，避免在此阶段更改数据，因为可能会导致无限循环

## beforeDestory

实例销毁之前，当前阶段组件实例还存在，可以在当前阶段清除计时器、销毁父组件对子组件的重复监听

## destoryed

实例销毁之后，组件已被拆解，数据绑定被卸除，监听被移除，子实例被销毁



# 生命周期调用顺序

# 为什么 vue 组件中 data 必须是一个函数？

对象为引用类型，当复用组件时，由于数据对象都指向同一个 data 对象，当在一个组件中修改 data 时，其他重用的组件中的 data 会同时被修改；而使用返回对象的函数，由于每次返回的都是一个新对象（Object 的实例），引用地址不同，则不会出现这个问题。

# vue 中 v-if 和 v-show 有什么区别？

v-if 和 v-show 看起来似乎差不多，当条件不成立时，其所对应的标签元素都不可见，但是这两个选项是有区别的:

1、v-if 在条件切换时，会对标签进行适当的创建和销毁，而 v-show 则仅在初始化时加载一次，因此 v-if 的开销相对来说会比 v-show 大。

2、v-if 是惰性的，只有当条件为真时才会真正渲染标签；如果初始条件不为真，则 v-if 不会去渲染标签。v-show 则无论初始条件是否成立，都会渲染标签，它仅仅做的只是简单的 CSS 切换。

# v-for与v-if的优先级问题

1）v-for优先于v-if被解析

```javascript
// src/complier/codegen/index.js
// 会优先处理for，两者不同级的话生成不同的渲染函数
```
2）如果同时出现，每次渲染会先执行循环再判断条件，无论如何循环都不可避免，浪费性能
3）要避免这种情况，在外层嵌套```template```，在这一层进行v-if判断，然后在内部进行v-for

# Vue 组件中 data 为什么必须是函数？为什么根实例没有限制

```javascript
// src/score/instance/state.js-initData()
```
因为Vue组件可能会存在多个实例，如果使用对象形式定义data，则会导致他们在内存中指向向同一个对象，会造成数据污染。

采用函数形式，在initData时会将其作为工厂函数返回一个全新的data对象，有效避免了多实例之间的数据污染问题。

而根实例不存在该限制，以为根实例有且只能有一个。



# vue中key的作用和工作原理？

```javascript
// src/core/vdom/patch.js-updateChildren()
```

key的作用是为了高效的更新虚拟DOM，其原理是vue在patch过程中通过key精准判断两个节点是否是同一个，从而避免频繁地更新不同元素，使得整个patch过程更加高效，减少DOM的操作，提高性能

若不设置key还可能会在列表更新引发一些隐蔽的bug

vue在使用相同标签名元素的过渡切换时，也会使用到key属性，其目的是为了让vue可以区分它们，否则vue只会替换内部标签属性而不会触发过渡过程



# 虚拟DOM

## 为什么？

虚拟DOM本质上是用JavaScript对象描述的DOM节点，是对真实DOM的抽象。

浏览器获取到页面资源后，会对页面进行渲染，分别生成DOM树和CSS样式表，合并生成Render树，进行布局绘制。

用原生JS和JQuery操作真实DOM时，浏览器会从头到尾执行一遍渲染流程，触发回流和重绘，频繁地操作DOM，会有一定的性能问题，因此需要在`patch`过程中尽可能的一次性将差异更新到DOM中，但还是不可避免的去操作真实DOM，因此出现了虚拟DOM，使用一个JavaScript对象去描真实的DOM节点，在`patch`中使用diff算法比较新旧虚拟DOM，获得最小差异，再将结果反映到真实DOM上。因为在浏览器中，js操作要比操作真实DOM迅速许多，所以可以节约大量的时间。

## 是什么？

是一个用来描述真实DOM的JavaScript对象，在`patch`中使用Diff算法来获取最小更新，再反映到真实DOM中

## 优缺点？

- 优点

保证性能下限（虽然不能达到手动优化的最好结果，但是比起粗暴的DOM操作，性能要好很多）

无需手动操作DOM

跨平台（本质上是JavaScript对象，可以更方便地跨平台操作，例如服务端渲染、移动端开发）

- 缺点

无法进行极致优化（比不上在代码中进行手工优化）

# Diff算法？

diff算法是新旧虚拟DOM的比较算法，通过对两者进行比较，将变化的地方更新在真实DOM上。另外，也需要diff算法进行高效的对比过程，，从而降低时间复杂度O(n)

vue2.X中为了降低watcher粒度，每个组件只有一个watcher与之对应，只有引入diff才能精确找到发生变化的地方

执行时刻是组件实例执行其更新函数patch时，它会对比上一次渲染结果的oldVnode与新的渲染结果newVnode

遵循**深度优先，同层比较**的策略：

两个节点之间的比较会根据他们是否有子节点或者文本节点做不同的操作；

使用**列表比对**，对两组子节点的头尾进行四次比较（旧前与新前、旧后与新后、旧前与新后、旧后与新前）；如果没有找到相同节点，按照通用方法遍历查找；查找结束后按照情况处理剩余节点。使用key可以精确查找到相同节点，因此patch过程非常高效

# template编译

先通过while和栈转化为AST树，再得到render函数返回VNode

- 通过compile编译器把template编译成AST语法树
- AST经过generate得到render函数，render返回VNode

# **常用的事件修饰符**

- .stop: 阻止冒泡
- .prevent: 阻止默认行为
- .self: 仅绑定元素自身触发
- .once: 2.1.4 新增，只触发一次
- passive: 2.3.0 新增，滚动事件的默认行为 (即滚动行为) 将会立即触发，不能和.prevent 一起使用
- .sync 修饰符

从 2.3.0 起 vue 重新引入了.sync 修饰符，但是这次它只是作为一个编译时的语法糖存在。它会被扩展为一个自动更新父组件属性的 v-on 监听器。示例代码如下：

```javascript
<comp :foo.sync="bar"></comp>
```

会被扩展为：

```javascript
<comp :foo="bar" @update:foo="val => bar = val"></comp>
```

当子组件需要更新 foo 的值时，它需要显式地触发一个更新事件：

```javascript
this.$emit('update:foo', newValue)
```

# **vue 如何获取 dom**

先给标签设置一个 ref 值，再通过 this.$refs.domName 获取，例如：

```javascript
<div ref="test"></div>

const dom = this.$refs.test
```

# v-on 可以监听多个方法吗？

是可以的，来个例子：

```javascript
<input type="text" v-on="{ input:onInput,focus:onFocus,blur:onBlur, }">
```

# **assets 和 static 的区别**

这两个都是用来存放项目中所使用的静态资源文件。

两者的区别：

assets 中的文件在运行 npm run build 的时候会打包，简单来说就是会被压缩体积，代码格式化之类的。打包之后也会放到 static 中。

static 中的文件则不会被打包。

> 建议：将图片等未处理的文件放在 assets 中，打包减少体积。而对于第三方引入的一些资源文件如 iconfont.css 等可以放在 static 中，因为这些文件已经经过处理了。

# **slot 插槽**

很多时候，我们封装了一个子组件之后，在父组件使用的时候，想添加一些 dom 元素，这个时候就可以使用 slot 插槽了，但是这些 dom 是否显示以及在哪里显示，则是看子组件中 slot 组件的位置了。

# vue 初始化页面闪动问题

使用 vue 开发时，在 vue 初始化之前，由于 div 是不归 vue 管的，所以我们写的代码在还没有解析的情况下会容易出现花屏现象，看到类似于 {{message}} 的字样，虽然一般情况下这个时间很短暂，但是我们还是有必要让解决这个问题的。

首先：在 css 里加上以下代码

```javascript
[v-cloak] {
    display: none;
}
```

如果没有彻底解决问题，则在根元素加上 style=“display: none;” :style="{display: ‘block’}"

# 组件化

组件化使开发者使用小型、独立和可复用的组件构建大型应用

能大幅提高应用开发效率、测试性、复用性等

按使用分类为：页面组件、业务组件、通用组件

VUE的组件是基于配置的，平时编写的组件是组件配置而非组件，框架后续会生成其构造函数，基于VueComponent，扩展于Vue

vue中的组件化技术有：props、自定义事件、插槽等，主要用于组件通信、扩展等

合理的划分组件，有利于提升应用性能

组件应该是高内聚、低耦合

遵循单向数据流的原则



# 单向数据流

是什么？所有的prop使得	其父子prop之间形成了一个**单向下行绑定**（父级prop的更新会向下流动到子组件中，但反过来则不行）。

为什么？防止从子组件意外改变父级组件的状态，从而导致应用的**数据流向**混乱

好处？组件的prop有了单一来源，每次父级组件发生更新时，子组件的所有prop都会刷新为最新值。



如果想要子组件更新父组件的状态，只能通过**$emit**派发一个自定义事件，父组件接收后，由父组件自身修改



## 需要改变prop的情况的处理

### 1）prop用来传递初始值，组件将其作为一个本地data来处理

此时，需要定义一个本地data的属性并将此prop作为其初始值

```javascript
props: ['initialCounter'],
data: function () {
  return {
    counter: this.initialCounter//定义本地的data属性接收prop初始值
  }
}
```



### 2）prop以一种原始值传入，需要对其进行转换

此时，最好使用这个prop的值来定义一个计算属性

```javascript
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```

# computed与watch

#### 计算属性 computed：

- 支持缓存，只有依赖数据发生改变，才会重新进行计算
- 不支持异步，当 computed 内有异步操作时无效，无法监听数据的变化
- computed 属性值会默认走缓存，计算属性是基于它们的响应式依赖进行缓存的，也就是基于 data 中声明过或者父组件传递的 props 中的数据通过计算得到的值
- 如果一个属性是由其他属性计算而来的，这个属性依赖其他属性，是一个多对一或者一对一，一般用 computed
- 如果 computed 属性属性值是函数，那么默认会走 get 方法；函数的返回值就是属性的属性值；在 computed 中的，属性都有一个 get 和一个 set 方法，当数据变化时，调用 set 方法。

#### 侦听属性 watch：

- 不支持缓存，数据变，直接会触发相应的操作；
- watch 支持异步；
- 监听的函数接收两个参数，第一个参数是最新的值；第二个参数是输入之前的值；
- 当一个属性发生变化时，需要执行对应的操作；一对多；
- 监听数据必须是 data 中声明过或者父组件传递过来的 props 中的数据，当数据变化时，触发其他操作，函数有两个参数：

> immediate：组件加载立即触发回调函数执行

```javascript
watch: {
  firstName: {
    handler(newName, oldName) {
      this.fullName = newName + ' ' + this.lastName;
    },
    // 代表在wacth里声明了firstName这个方法之后立即执行handler方法
    immediate: true
  }
}
```

> deep: deep 的意思就是深入观察，监听器会一层层的往下遍历，给对象的所有属性都加上这个监听器，但是这样性能开销就会非常大了，任何修改 obj 里面任何一个属性都会触发这个监听器里的 handler

```javascript
watch: {
  obj: {
    handler(newName, oldName) {
      console.log('obj.a changed');
    },
    immediate: true,
    deep: true
  }
}
```

优化：我们可以使用字符串的形式监听

```javascript
watch: {
  'obj.a': {
    handler(newName, oldName) {
      console.log('obj.a changed');
    },
    immediate: true,
    // deep: true
  }
}
```

这样 Vue.js 才会一层一层解析下去，直到遇到属性 a，然后才给 a 设置监听函数。

# 组件通信

## 父-->子

## 子-->父

## 兄弟

# vue-loader

vue 文件的一个加载器，跟 template/js/style 转换成 js 模块

# **$nextTick 是什么？**

vue 实现响应式并不是数据发生变化后 dom 立即变化，而是按照一定的策略来进行 dom 更新。

> nextTick 是在下次 DOM 更新循环结束之后执行延迟回调，在修改数据之后使用 nextTick，则可以在回调中获取更新后的 DOM

# vue设计理念



# vue的双向绑定原理

观察者（observer）利用```Object.definePropertype()```拿到data依赖，遍历子集依赖，如果是数组，vue是通过拦截器重写了数组的原型链方法。set拿到所有的子依赖，告诉watcher（订阅者）每收集一个子依赖就new一个订阅者，最后订阅者被收集起来，dep就是个收集器，是个集合或者数组。

complier（编译器）匹配模板中的指令，进行赋值，render到当前页面中



vue.js 是采用数据劫持结合发布者-订阅者模式的方式，通过Object.defineProperty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调。

　　具体步骤：

　　第一步：需要 observe 的数据对象进行递归遍历，包括子属性对象的属性，都加上 setter 和 getter
这样的话，给这个对象的某个值赋值，就会触发setter，那么就能监听到了数据变化

　　第二步：compile解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，更新视图

　　第三步：Watcher订阅者是Observer和Compile之间通信的桥梁，主要做的事情是:

　　1、在自身实例化时往属性订阅器(dep)里面添加自己

　　2、自身必须有一个update()方法

　　3、待属性变动dep.notice()通知时，能调用自身的 update() 方法，并触发Compile中绑定的回调，则功成身退。

　　第四步：MVVM作为数据绑定的入口，整合Observer、Compile和Watcher三者，通过Observer来监听自己的model数据变化，通过Compile来解析编译模板指令，最终利用Watcher搭起Observer和Compile之间的通信桥梁，达到数据变化 -> 视图更新；视图交互变化(input) -> 数据model变更的双向绑定效果。

# 如何监听对象或者数组某个属性的变化

因为`Object.defineProperty()`的限制，监听不到数组某一项或者对象的某个属性值的变化。

使用`this.$set()`或者调用`splice()、 push()、pop()、shift()、unshift()、sort()、reverse()`

# 什么是依赖

需要用到数据的地方，称为依赖



Vue1.x，细粒度依赖，用到数据的DOM都是依赖

Vue2.X，中等粒度依赖，用到数据的组件是依赖



在```getter```中收集依赖，在```setter```中触发依赖

把依赖收集的代码封装成一个Dep类，专门用来管理依赖，每个Observer的实例，成员中都有一个Dep实例

Watcher是一个中介，数据发生变化时通过Watcher中转



Dep中收集的依赖是Watcher，只有Watcher触发的getter才会收集依赖，哪个Watcher触发了getter，就把哪个Watcher收集到Dep中

Dep使用了发布订阅模式，当数据发生变化时，会循环subs列表，把所有的Watcher都通知一遍

Watcher将自己设置到全局的指定位置window.target，然后读取数据，因为读取了数据，所以触发了这个数据的getter，在这个getter中就能获取当前正在读取数据的Watcher，并收集到Dep中



# Vue 中操作 data 中数组的方法中哪些可以触发视图更新，哪些不可以，不可以的话有什么解决办法？

`push ()、pop ()、shift ()、unshift ()、splice ()、sort ()、reverse ()` 这些方法会改变被操作的数组； `filter ()、concat ()、slice ()` 这些方法不会改变被操作的数组，返回一个新的数组； 以上方法都可以触发视图更新。

- 利用索引直接设置一个数组项，例：`this.array[index] = newValue`
- 直接修改数组的长度，例：`this.array.length = newLength`

以上两种方法不可以触发视图更新；

- 可以用 `this.$set(this.array,index,newValue)` 或 `this.array.splice(index,1,newValue)` 解决方法 1
- 可以用 `this.array.splice(newLength)` 解决方法 2

# 混入（mixin）

- 全局混入在项目中怎么用？

  在 main.js 中写入

  ```
      import Vue from 'vue';
      import mixins from './mixins';
      Vue.mixin(mixins);
  123
  ```

  之后，全局混入可以写在 mixins 文件夹中 index.js 中，全局混入会影响到每一个之后创建的 Vue 实例（组件）；

- 局部混入在项目中怎么用

  局部混入的注册，在 mixins 文件中创建一个 a_mixin.js 文件，然后再 a.vue 文件中写入

  ```javascript
  <script>
      import aMixin from 'mixins/a_mixin'
      export default{
          mixins:[aMixin],
      }
  </script>
  ```

  局部混入只会影响 a.vue 文件中创建的 Vue 实例，不会影响到其子组件创建的 Vue 实例；

- 组件的选项和混入的选项是怎么合并的

  - 数据对象【data 选项】，在内部进行递归合并，并在发生冲突时以组件数据优先；
  - 同名钩子函数将合并为一个数组，因此都将被调用。另外，混入对象的钩子将在组件自身钩子之前调用；
  - watch 对象合并时，相同的 key 合成一个对象，且混入监听在组件监听之前调用；
  - 值为对象的选项【filters 选项、computed 选项、methods 选项、components 选项、directives 选项】将被合并为同一个对象。两个对象键名冲突时，取组件对象的键值对。



# computed 中的属性名和 data 中的属性名可以相同吗？

不能同名，因为不管是 computed 属性名还是 data 数据名还是 props 数据名都会被挂载在 vm 实例上，因此这三个都不能同名。

```
if (key in vm.$data) {
    warn(`The computed property "${key}" is already defined in data.`, vm)
} else if (vm.$options.props && key in vm.$options.props) {
    warn(`The computed property "${key}" is already defined as a prop.`, vm)
}
12345
```

# 怎么强制刷新组件？

- this.$forceUpdate()。
- 组件上加上 key，然后变化 key 的值。

# watch 的属性和 methods 方法能用箭头函数定义吗？

不可以。`this` 会是 `undefind`, 因为箭头函数中的 this 指向的是定义时的 this，而不是执行时的 this，所以不会指向 Vue 实例的上下文。

# 给组件绑定自定义事件无效怎么解决？

加上修饰词.native。

# 怎么访问子组件的实例或者子元素？

先用 ref 特性为子组件赋予一个 ID 引用 `<base-input ref="myInput"></<base-input>`

- 比如子组件有个 focus 的方法，可以这样调用 `this.$refs.myInput.focus()`；
- 比如子组件有个 value 的数据，可以这样使用 `this.$refs.myInput.value`。

先用 ref 特性为普通的 DOM 元素赋予一个 ID 引用

```javascript
<ul ref="mydiv">
    <li class="item">第一个li</li>
    <li class="item">第一个li</li>
</ul>
console.log(this.$refs['mydiv'].getElementsByClassName('item')[0].innerHTML)//第一个li
```

# 怎么在子组件中访问父组件的实例？怎么在组件中访问到根实例？

使用 `this.$parent` 来访问

```javascript
this.$root
```

# 组件会在什么时候下被销毁？

- 没有使用 keep-alive 时的路由切换；
- `v-if='false'`；
- 执行 `vm.$destroy()`；

# is 这个特性你有用过吗？主要用在哪些方面？

- 动态组件

`<component :is="componentName"></component>`， `componentName` 可以是在本页面已经注册的局部组件名和全局组件名，也可以是一个组件的选项对象。 当控制 `componentName` 改变时就可以动态切换选择组件。

- is 的用法

有些 HTML 元素，诸如 `<ul>、<ol>、<table>` 和 `<select>`，对于哪些元素可以出现在其内部是有严格限制的。

而有些 HTML 元素，诸如 `<li>、<tr> 和 <option>`，只能出现在其它某些特定的元素内部。

```javascript
<ul>
    <card-list></card-list>
</ul>
```

所以上面 `<card-list></card-list>` 会被作为无效的内容提升到外部，并导致最终渲染结果出错。应该这么写：

```javascript
<ul>
    <li is="cardList"></li>
</ul>
```

# prop 验证的 type 类型有哪几种？

String、Number、Boolean、Array、Object、Date、Function、Symbol， 此外还可以是一个自定义的构造函数 Personnel，并且通过 instanceof 来验证 propwokrer 的值是否是通过这个自定义的构造函数创建的。

```javascript
function Personnel(name,age){
    this.name = name;
    this.age = age;
}
export default {
    props:{
        wokrer:Personnel
    }
}
```

# 在 Vue 事件中传入 `$event` ，使用 $event.target和 event.currentTarget 有什么区别？

`$event.currentTarget` 始终指向事件所绑定的元素，而 `$event.target` 指向事件发生时的元素。

# 使用事件修饰符要注意什么？

要注意顺序很重要，用 `@click.prevent.self` 会阻止所有的点击，而 `@click.self.prevent` 只会阻止对元素自身的点击。

# 说说你对 Vue 的表单修饰符.lazy 的理解？

input 标签 v-model 用 lazy 修饰之后，并不会立即监听 input 的 value 的改变，会在 input 失去焦点之后，才会监听 input 的 value 的改变。

# v-once 的使用场景有哪些？

其作用是只渲染元素和组件一次。随后的重新渲染，元素 / 组件及其所有的子节点将被视为静态内容并跳过。故当组件中有大量的静态的内容可以使用这个指令。

# v-cloak 和 v-pre 有什么作用？

`v-cloak`：可以解决在页面渲染时把未编译的 Mustache 标签（`{{value}}`）给显示出来。

```vue
[v-cloak] {
    display: none!important;
}
<div v-cloak>
    {{ message }}
</div>
```

v-pre`：跳过这个元素和它的子元素的编译过程。可以用来显示原始 Mustache 标签。跳过大量没有指令的节点会加快编译。`

```vue
<span v-pre>{{ this will not be compiled }}</span>
```

# 怎么使 css 样式只在当前组件中生效？在 style 上加 scoped 属性需要注意哪些？你知道 style 上加 scoped 属性的原理吗？

```css
<style lang="less" scoped> </style>
```

- 如果在公共组件中使用，修改公共组件的样式需要用 `/deep/`。
- vue 通过在 DOM 结构以及 css 样式上加上唯一的标记 `data-v-xxxxxx`，保证唯一，达到样式私有化，不污染全局的作用。

# Vue 渲染模板时怎么保留模板中的 HTML 注释呢？

- 在组件中将 `comments` 选项设置为 `true`
- `<template comments> ... <template>`

# Vue 中怎么重置 data？

```javascript
Object.assign(this.$data,this.$options.data())
```

# 过滤器中可以用 this 吗？

不可以

# Vue在created和mounted这两个生命周期中请求数据有什么区别呢？

在created中，页面视图未出现，如果请求信息过多，页面会长时间处于白屏状态，DOM节点没出来，无法操作DOM节点。在mounted不会这样，比较好。

# 说说你对keep-alive的理解

keep-alive是一个抽象组件：它自身不会渲染一个DOM元素，也不会出现在父组件链中；使用keep-alive包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。

其有三个参数

- `include`定义缓存白名单，会缓存的组件；
- `exclude`定义缓存黑名单，不会缓存的组件；
- 以上两个参数可以是逗号分隔字符串、正则表达式或一个数组,`include="a,b"`、`:include="/a|b/"`、`:include="['a', 'b']"`；
- 匹配首先检查组件自身的 name 选项，如果 name 选项不可用，则匹配它的局部注册名称 (父组件 components 选项的键值)。匿名组件不能被匹配；
- `max`最多可以缓存多少组件实例。一旦这个数字达到了，在新实例被创建之前，已缓存组件中最久没有被访问的实例会被销毁掉；
- 不会在函数式组件中正常工作，因为它们没有缓存实例；
- 当组件在内被切换，它的activated和deactivated这两个生命周期钩子函数将会被对应执行。

# 使用v-for遍历对象时，是按什么顺序遍历的？如何保证顺序？

按 Object.keys() 的顺序的遍历，转成数组保证顺序。

# key除了在v-for中使用，还有什么作用？

还可以强制替换元素/组件而不是重复使用它。在以下场景可以使用

- 完整地触发组件的生命周期钩子
- 触发过渡

```javascript
<transition>
  <span :key="text">{{ text }}</span>
</transition>
```

当 text 发生改变时，`<span>`会随时被更新，因此会触发过渡。

# 使用key要什么要注意的吗？

- 不要使用对象或数组之类的非基本类型值作为key，请用字符串或数值类型的值；

- 不要使用数组的index作为key值，因为在删除数组某一项，index也会随之变化，导致key变化，渲染会出错。

  例：在渲染`[a,b,c]`用 index 作为 key，那么在删除第二项的时候，index 就会从 0 1 2 变成 0 1（而不是 0 2)，随之第三项的key变成1了，就会误把第三项删除了。

# 说说组件的命名规范

给组件命名有两种方式，一种是使用链式命名my-component，一种是使用大驼峰命名MyComponent，

- 在字符串模板中`<my-component></my-component>` 和 `<MyComponent></MyComponent>`都可以使用，
- 在非字符串模板中最好使用`<MyComponent></MyComponent>`，因为要遵循W3C规范中的自定义组件名

(字母全小写且必须包含一个连字符)，避免和当前以及未来的 HTML 元素相冲突。

# 为什么组件中data必须用函数返回一个对象？

对象为引用类型，当重用组件时，由于数据对象都指向同一个data对象，当在一个组件中修改data时，其他重用的组件中的data会同时被修改；而使用返回对象的函数，由于每次返回的都是一个新对象（Object的实例），引用地址不同，则不会出现这个问题。

# 组件的name选项有什么作用？

- 递归组件时，组件调用自身使用；
- 用`is`特殊特性和`component`内置组件标签时使用；
- `keep-alive`内置组件标签中`include`和`exclude`属性中使用。

# 说下`$attrs`和`$listeners`的使用场景？

`$attrs`: 包含了父作用域中（组件标签）不作为 prop 被识别 (且获取) 的特性绑定 (class 和 style 除外)。 在创建基础组件时候经常使用，可以和组件选项`inheritAttrs:false`和配合使用在组件内部标签上用`v-bind="$attrs"`将非prop特性绑定上去；

`$listeners`: 包含了父作用域中（组件标签）的 (不含`.native`) v-on 事件监听器。 在组件上监听一些特定的事件，比如focus事件时，如果组件的根元素不是表单元素的，则监听不到，那么可以用`v-on="$listeners"`绑定到表单元素标签上解决。

# EventBus注册在全局上时，路由切换时会重复触发事件，如何解决呢？

在有使用`$on`的组件中要在`beforeDestroy`钩子函数中用`$off`销毁。

# Vue组件里写的原生addEventListeners监听事件，要手动去销毁吗？为什么？

要，不然会造成多次绑定和内存泄露。

# Vue组件里的定时器要怎么销毁？

- 如果页面上有很多定时器，可以在`data`选项中创建一个对象`timer`，给每个定时器取个名字一一映射在对象`timer`中，在`beforeDestroy`构造函数中`for(let k in this.timer){clearInterval(k)}`；

- 如果页面只有单个定时器，可以这么做。

  ```javascript
  const timer = setInterval(() =>{}, 500);
  this.$once('hook:beforeDestroy', () => {
     clearInterval(timer);
  })
  ```

# Vue中能监听到数组变化的方法有哪些？为什么这些方法能监听到呢？

`push()`、`pop()`、`shift()`、`unshift()`、`splice()`、`sort()`、`reverse()`，这些方法在Vue中被重新定义了，故可以监听到数组变化；

`filter()`、`concat()`、`slice()`，这些方法会返回一个新数组，也可以监听到数组的变化。

# 在Vue中哪些数组变化无法监听，为什么，怎么解决？

- 利用索引直接设置一个数组项时；
- 修改数组的长度时。
- 第一个情况，利用**已有索引**直接设置一个数组项时`Object.defineProperty()`是**可以**监听到，利用**不存在的索引**直接设置一个数组项时`Object.defineProperty()`是**不可以**监听到，但是官方给出的解释是由于JavaScript的限制，Vue不能检测以上数组的变动，其实根本原因是性能问题，性能代价和获得的用户体验收益不成正比。
- 第二个情况，原因是`Object.defineProperty()`不能监听到数组的`length`属性。

用`this.$set(this.items, indexOfItem, newValue)`或`this.items.splice(indexOfItem, 1, newValue)`来解决第一种情况；

用`this.items.splice(newLength)`来解决第二种情况。

# 在Vue中哪些对象变化无法监听，为什么，怎么解决？

- 对象属性的添加
- 对象属性的删除

因为Vue是通过`Object.defineProperty`来将对象的key转成getter/setter的形式来追踪变化，但getter/setter只能追踪一个数据是否被修改，无法追踪新增属性和删除属性，所以才会导致上面对象变化无法监听。

- 用`this.$set(this.obj,"key","newValue")`来解决第一种情况；
- 用`Object.assign`来解决第二种情况。

# 删除对象用delete和Vue.delete有什么区别？

- delete：只是被删除对象成员变为`' '`或`undefined`，其他元素键值不变；
- Vue.delete：直接删了对象成员，如果对象是响应式的，确保删除能触发更新视图，这个方法主要用于避开 Vue 不能检测到属性被删除的限制。

# `<template></template>`有什么用？

当做一个不可见的包裹元素，减少不必要的DOM元素，整个结构会更加清晰。

# Vue怎么定义全局方法

有三种

- 挂载在Vue的prototype上

  ```javascript
  // base.js
  const install = function (Vue, opts) {
      Vue.prototype.demo = function () {
          console.log('我已经在Vue原型链上')
      }
  }
  export default {
      install
  }
  ```

  

  ```javascript
  //main.js
  //注册全局函数
  import base from 'service/base';
  Vue.use(base);
  ```

- 利用全局混入mixin

- 用`this.$root.$on`绑定方法，用`this.$root.$off`解绑方法，用`this.$root.$emit`全局调用。

  ```javascript
  this.$root.$on('demo',function(){
      console.log('test');
  })
  this.$root.$emit('demo')；
  this.$root.$off('demo')；
  ```

# Vue怎么改变插入模板的分隔符？

用`delimiters`选项,其默认是`["{{", "}}"]`

```javascript
// 将分隔符变成ES6模板字符串的风格
new Vue({
  delimiters: ['${', '}']
})
```

# Vue变量名如果以_、$开头的属性会发生什么问题？怎么访问到它们的值？

以 `_`或 `$` 开头的属性 不会 被 Vue 实例代理，因为它们可能和 Vue 内置的属性、API 方法冲突，你可以使用例如 `vm.$data._property` 的方式访问这些属性。

# 怎么捕获Vue组件的错误信息？

`errorCaptured`是组件内部钩子，当捕获一个来自子孙组件的错误时被调用，接收`error`、`vm`、`info`三个参数，`return false`后可以阻止错误继续向上抛出。

`errorHandler`为全局钩子，使用`Vue.config.errorHandler`配置，接收参数与`errorCaptured`一致，2.6后可捕捉`v-on`与`promise`链的错误，可用于统一错误处理与错误兜底。

# Vue.observable你有了解过吗？说说看

让一个对象可响应。可以作为最小化的跨组件状态存储器。

# Vue项目中如何配置favicon？

- 静态配置 `<link rel="icon" href="<%= BASE_URL %>favicon.ico">`, 其中`<%= BASE_URL %>`等同vue.config.js中`publicPath`的配置;

- 动态配置

  ```
  <link rel="icon" type="image/png" href="">
  1
  ```

  ```
  import browserImg from 'images/kong.png';//为favicon的默认图片
  const imgurl ='后端传回来的favicon.ico的线上地址'
  let link = document.querySelector('link[type="image/png"]');
  if (imgurl) {
      link.setAttribute('href', imgurl);
  } else {
      link.setAttribute('href', browserImg);
  }
  12345678
  ```

# 怎么修改Vue项目打包后生成文件路径？

- 在Vue CLI2中修改config/index.js文件中的`build.assetsPublicPath`的值；
- 在Vue CLI3中配置publicPath的值。

# 怎么解决Vue项目打包后静态资源图片失效的问题？

在项目中一般通过配置alias路径别名的方式解决,下面是Vue CLI3的配置。

```
configureWebpack: {
    resolve: {
        extensions: ['.js', '.vue', '.json'],
        alias: {
            '@': resolve('src'),
            'assets': resolve('src/assets'),
            'css': resolve('src/assets/css'),
            'images': resolve('src/assets/images'),
        }
    },
},
1234567891011
```

# 怎么解决Vue中动态设置img的src不生效的问题？

因为动态添加src被当做静态资源处理了，没有进行编译，所以要加上require。

```
<template>
    <img class="logo" :src="logo" alt="公司logo">
</template>
<script>
export default {
    data() {
        return {
            logo:require("assets/images/logo.png"),
        };
    }
};
</script>
123456789101112
```

# 在Vue项目中如何引入第三方库（比如jQuery）？有哪些方法可以做到？

先在主入口页面 index.html 中用 script 标签引入`<script src="./static/jquery-1.12.4.js"></script>`,如果你的项目中有用ESLint检测，会报`'$' is not defined`，要在文件中加上`/* eslint-disable */`

先在主入口页面 index.html 中用 script 标签引入`<script src="./static/jquery-1.12.4.js"></script>`,然后在webpack 中配置一个 externals，即可在项目中使用。

```javascript
externals: {
    'jquery': 'jQuery'
}
```

先在webpack中配置alias，最后在main.js中用`import $ from 'jquery'`，即可在项目中使用。

```javascript
resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
        '@': resolve('src'),
        'jquery': resolve('static/jquery-1.12.4.js')
    }
}
```

在webpack中新增一个plugins，即可在项目中使用

```javascript
plugins: [
    new webpack.ProvidePlugin({
        $:"jquery",
        jQuery:"jquery",
        "windows.jQuery":"jquery"
    })
]
```

## 说说你对SPA单页面的理解，它的优缺点分别是什么？

是一种只需要将单个页面加载到服务器之中的web应用程序。当浏览器向服务器发出第一个请求时，服务器会返回一个index.html文件，它所需的js，css等会在显示时统一加载，部分页面按需加载。url地址变化时不会向服务器在请求页面，通过路由才实现页面切换。

优点：

- 良好的交互体验，用户不需要重新刷新页面，获取数据也是通过Ajax异步获取，页面显示流畅；
- 良好的前后端工作分离模式。

缺点：

- SEO难度较高，由于所有的内容都在一个页面中动态替换显示，所以在SEO上其有着天然的弱势。
- 首屏加载过慢（初次加载耗时多）

# SPA单页面的实现方式有哪些？

- 在hash模式中，在window上监听hashchange事件（地址栏中hash变化触发）驱动界面变化；
- 在history模式中，在window上监听popstate事件（浏览器的前进或后退按钮的点击触发）驱动界面变化，监听a链接点击事件用history.pushState、history.replaceState方法驱动界面变化；
- 直接在界面用显示隐藏事件驱动界面变化。

# 说说你对Object.defineProperty的理解

- Object.defineProperty(obj,prop,descriptor)方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。

  - obj：要在其上定义属性的对象。
  - prop：要定义或修改的属性的名称。
  - descriptor：将被定义或修改的属性描述符。

- descriptor属性描述符主要有两种形式：数据描述符和存取描述符。

  描述符必须是这两种形式之一；不能同时是两者。

  - 数据描述符和存取描述符共同拥有
    - configurable：特性表示对象的属性是否可以被删除，以及除value和writable特性外的其他特性是否可以被修改。默认为false。
    - enumerable：当该属性的enumerable为true时，该属性才可以在for…in循环和Object.keys()中被枚举。默认为false。
  - 数据描述符
    - value：该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。默认为undefined。
    - writable：当且仅当该属性的writable为true时，value才能被赋值运算符改变。默认为false。
  - 存取描述符
    - get：一个给属性提供 getter的方法，如果没有getter则为undefined。当访问该属性时，该方法会被执行，方法执行时没有参数传入，但是会传入this对象（由于继承关系，这里的this并不一定是定义该属性的对象）。默认为undefined。
    - set：一个给属性提供 setter的方法，如果没有setter则为undefined。当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值。默认为undefined。

- 定义descriptor时，最好先把这些属性都定义清楚，防止被继承和继承时出错。

```javascript
function Archiver() {
    var temperature = null;
    var archive = [];
    Object.defineProperty(this, 'temperature', {
        get: function() {
          console.log('get!');
          return temperature;
        },
        set: function(value) {
          temperature = value;
          archive.push({ val: temperature });
        }
    });
    this.getArchive = function() { return archive; };
}
var arc = new Archiver();
arc.temperature; // 'get!'
arc.temperature = 11;
arc.temperature = 13;
arc.getArchive(); // [{ val: 11 }, { val: 13 }]
```

# 说说你对Proxy的理解

官方定义：proxy对象用于定义基本操作的自定义行为（如属性查找，赋值，枚举，函数调用等）。

通俗来说是在对目标对象的操作之前提供了拦截，对外界的操作进行过滤和修改某些操作的默认行为，可以不直接操作对象本身，而是通过操作对象的代理对象来间接来操作对象。

```javascript
let proxy = new Proxy(target, handler)
```

- `target` 是用 Proxy 包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）;
- `handler` 一个对象，其属性是当执行一个操作时定义代理的行为的函数，也就是自定义的行为。

**`handle`可以为`{}`，但是不能为`null`，否则会报错**

Proxy 目前提供了 13 种可代理操作，比较常用的

- handler.get(target,property,receiver)获取值拦截
- handler.set(target,property,value,receiver)设置值拦截
- handler.has(target,prop)in 操作符拦截

```javascript
let obj = {
	a : 1,
	b : 2
}
let test = new Proxy(obj,{
    get : function (target,property) {
        return property in target ? target[property] : 0
    },
    set : function (target,property,value) {
        target[property] = 6;
    },
    has: function (target,prop){
        if(prop == 'b'){
            target[prop] = 6;
        }
        return prop in target;
    },
})

console.log(test.a);        // 1
console.log(test.c);        // 0

test.a = 3;
console.log(test.a)         // 6

if('b' in test){
    console.log(test)       // Proxy {a: 6, b: 6}
}
```

# Object.defineProperty和Proxy的区别

Object.defineProperty

- 不能监听到数组length属性的变化；
- 不能监听对象的添加；
- 只能劫持对象的属性,因此我们需要对每个对象的每个属性进行遍历。

Proxy

- 可以监听数组length属性的变化；
- 可以监听对象的添加；
- 可代理整个对象，不需要对对象进行遍历，极大提高性能；
- 多达13种的拦截远超Object.defineProperty只有get和set两种拦截。

# Vue的模板语法用的是哪个web模板引擎的吗？说说你对这模板引擎的理解?

采用的是Mustache的web模板引擎[mustache.js](https://github.com/janl/mustache.js/blob/master/mustache.js)

```javascript
<script type="text/javascript" src="./mustache.js"></script>
<script type="text/javascript">
    var data = {
        "company": "Apple",
    }

    var tpl = '<h1>Hello {{company}}</h1>';
    var html = Mustache.render(tpl, data);

    console.log(html);
</script>
```

# 你认为Vue的核心是什么？

Vue.js 的核心是一个允许采用简洁的模板语法来声明式地将数据渲染进 DOM 的系统。

# 说说你对单向数据流和双向数据流的理解

单向数据流是指数据只能从父级向子级传递数据，子级不能改变父级向子级传递的数据。

双向数据流是指数据从父级向子级传递数据，子级可以通过一些手段改变父级向子级传递的数据。

比如用`v-model`、`.sync`来实现双向数据流。

# 什么是虚拟DOM？

虚拟DOM是将状态映射成视图的众多解决方案中的一种，其是通过状态生成一个虚拟节点树，然后使用虚拟节点树进行渲染生成真实DOM，在渲染之前，会使用新生成的虚拟节点树和上一次虚拟节点树进行对比，只渲染不同的部分。

# Vue中如何实现一个虚拟DOM？说说你的思路

首先要构建一个VNode的类，DOM元素上的所有属性在VNode类实例化出来的对象上都存在对应的属性。例如tag表示一个元素节点的名称，text表示一个文本节点的文本，chlidren表示子节点等。将VNode类实例化出来的对象进行分类，例如注释节点、文本节点、元素节点、组件节点、函数式节点、克隆节点。

然后通过编译将模板转成渲染函数render，执行渲染函数render，在其中创建不同类型的VNode类，最后整合就可以得到一个虚拟DOM（vnode）。

最后通过patch将vnode和oldVnode进行比较后，生成真实DOM。

# Vue为什么要求组件模板只能有一个根元素？

当前的virtualDOM差异和diff算法在很大程度上依赖于每个子组件总是只有一个根元素。

# axios是什么？怎样使用它？怎么解决跨域的问题？

axios 是一个基于 promise 的 HTTP 库，先封装在使用。

使用proxyTable配置解决跨域问题。

比如你要调用`http://172.16.13.205:9011/getList`这个接口

先在axios.create()配置baseURL增加标志

```javascript
const service = axios.create({
  baseURL: '/api',
});
service.get(getList, {params:data});
```

然后在config/index.js文件中配置

```javascript
dev:{
    proxyTable: {
        '/api': {
            target: 'http://172.16.13.205:9011', // 设置你调用的接口域名和端口号
            secure: false,
            changeOrigin: true,// 跨域
            pathRewrite: {
                '^/api': '' // 去掉标志
            }
        }
    },
}
```

配置后要重新`npm run dev`

F12中看到请求是`http://localhost:8080/api/getList`，实际上请求是`http://172.16.13.205:9011/getList`。

# 如果想扩展某个现有的Vue组件时，怎么做呢？

- 用mixins混入
- 用extends，比mixins先触发
- 用高阶组件HOC封装

# vue-loader是什么？它有什么作用？

vue-loader是一个webpack的loader，是一个模块转换器，用于把模块原内容按照需求转换成新内容。

它允许你以一种名为单文件组件 (SFCs)的格式撰写 Vue 组件。可以解析和转换 .vue 文件，提取出其中的逻辑代码 script、样式代码 style、以及 HTML 模版 template，再分别把它们交给对应的loader去处理。

# 你有使用过JSX吗？说说你对JSX的理解？

JSX就是Javascript和XML结合的一种格式。React发明了JSX，利用HTML语法来创建虚拟DOM。当遇到`<`，JSX就当HTML解析，遇到`{`就当JavaScript解析。