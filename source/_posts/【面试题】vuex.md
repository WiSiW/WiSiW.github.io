---
title: 【面试题】vuex
date: 2021-02-21 22:04:33
tags:
	- vuex
cover_picture:/images/vue.png
---

# vuex是什么？怎么使用？哪种功能场景使用它？

是专门为vue开发的状态管理模式

采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一直能够可预测的方式发生变化

解决多个组件共享状态或来自不同组件的行为需要变更同一状态：音乐播放、登录状态、购物车

# vuex 的 store 特性是什么

1. vuex 就是一个仓库，仓库里放了很多对象。其中 state 就是数据源存放地，对应于一般 vue 对象里面的 data
2. state 里面存放的数据是响应式的，vue 组件从 store 读取数据，若是 store 中的数据发生改变，依赖这相数据的组件也会发生更新
3. 它通过 mapState 把全局的 state 和 getters 映射到当前组件的 computed 计算属性

# Vuex中状态是对象时，使用时要注意什么？

因为对象是引用类型，复制后改变属性还是会影响原始数据，这样会改变state里面的状态，是不允许，所以先用深度克隆复制对象，再修改。

# 怎么在组件中批量使用Vuex的state状态？

使用mapState辅助函数, 利用对象展开运算符将state混入computed对象中

```javascript
import {mapState} from 'vuex'
export default{
    computed:{
        ...mapState(['price','number'])
    }
}
```

# Vuex中要从state派生一些状态出来，且多个组件使用它，该怎么做？

使用getter属性，相当Vue中的计算属性computed，只有原状态改变派生状态才会改变。

getter接收两个参数，第一个是state，第二个是getters(可以用来访问其他getter)。

```javascript
const store = new Vuex.Store({
    state: {
        price: 10,
        number: 10,
        discount: 0.7,
    },
    getters: {
        total: state => {
            return state.price * state.number
        },
        discountTotal: (state, getters) => {
            return state.discount * getters.total
        }
    },
});
```

然后在组件中可以用计算属性computed通过`this.$store.getters.total`这样来访问这些派生转态。

```javascript
computed: {
    total() {
        return this.$store.getters.total
    },
    discountTotal() {
        return this.$store.getters.discountTotal
    }
}
```

# 怎么通过getter来实现在组件内可以通过特定条件来获取state的状态？

通过让getter返回一个函数，来实现给getter传参。然后通过参数来进行判断从而获取state中满足要求的状态。

```javascript
const store = new Vuex.Store({
    state: {
        todos: [
            { id: 1, text: '...', done: true },
            { id: 2, text: '...', done: false }
        ]
    },
    getters: {
        getTodoById: (state) => (id) =>{
            return state.todos.find(todo => todo.id === id)
        }
    },
});
```

然后在组件中可以用计算属性computed通过`this.$store.getters.getTodoById(2)`这样来访问这些派生转态。

```javascript
computed: {
    getTodoById() {
        return this.$store.getters.getTodoById
    },
}
mounted(){
    console.log(this.getTodoById(2).done)//false
}
```

# vuex有哪几种属性

State , Getter , Mutation , Action , Module (就是mapAction)

**state**：vuex的基本数据，用来存储变量

**geeter**：

​		(1) getter 可以对 state 进行计算操作，它就是 store 的计算属性

　　(2) 虽然在组件内也可以做计算属性，但是 getters 可以在多给件之间复用

　　(3) 如果一个状态只在一个组件内使用，是可以不用 getters

**mutation**：提交更新数据的方法，必须是同步的(如果需要异步使用action)。每个mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数，提交载荷作为第二个参数。

**action**：和mutation的功能大致相同，不同之处在于 ==》

	1. Action 提交的是 mutation，而不是直接变更状态。 
	2. Action 可以包含任意异步操作。

**modules**：模块化vuex，可以让每一个模块拥有自己的state、mutation、action、getters,使得结构非常清晰，方便管理。

## Vue.js中ajax请求代码应该写在组件的methods中还是vuex的actions中？

- 单独组件使用，放在methods中
- 多组件复用，放在action中，包装成promise返回，在调用处用`async await`返回

# **不用 vuex 会带来什么问题**

- 可维护性会下降，你要修改数据，你得维护3个地方
- 可读性下降，因为一个组件里的数据，你根本就看不出来是从哪里来的
- 增加耦合，大量的上传派发，会让耦合性大大的增加，本来Vue用Component就是为了减少耦合，现在这么用，和组件化的初衷相背

# vuex一个例子方法

# vuex中如何异步修改状态

```javascript
this.$store.dispatch('mutations方法名',值)

const actions ={
	asyncIncre({commit,dispatch},payload){
		setTimeout(()=>{
			commit('minus',payload)
		})
	}
}
```



# vuex中actions和mutations的区别

Mutation 更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数

```text
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})
```

Action Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。 .

```text
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```

# 怎么在组件中批量使用Vuex的getter属性

使用mapGetters辅助函数, 利用对象展开运算符将getter混入computed 对象中

```javascript
import {mapGetters} from 'vuex'
export default{
    computed:{
        ...mapGetters(['total','discountTotal'])
    }
}
```

# 怎么在组件中批量给Vuex的getter属性取别名并使用

使用mapGetters辅助函数, 利用对象展开运算符将getter混入computed 对象中

```javascript
import {mapGetters} from 'vuex'
export default{
    computed:{
        ...mapGetters({
            myTotal:'total',
            myDiscountTotal:'discountTotal',
        })
    }
}
```

### 在Vuex中使用mutation要注意什么。

mutation 必须是同步函数

# Vuex中action和mutation有什么区别？

action 提交的是 mutation，而不是直接变更状态。mutation可以直接变更状态。

action 可以包含任意异步操作。mutation只能是同步操作。

提交方式不同，action 是用`this.$store.dispatch('ACTION_NAME',data)`来提交。mutation是用`this.$store.commit('SET_NUMBER',10)`来提交。

接收参数不同，mutation第一个参数是state，而action第一个参数是context，其包含了

```js
{
    state,      // 等同于 `store.state`，若在模块中则为局部状态
    rootState,  // 等同于 `store.state`，只存在于模块中
    commit,     // 等同于 `store.commit`
    dispatch,   // 等同于 `store.dispatch`
    getters,    // 等同于 `store.getters`
    rootGetters // 等同于 `store.getters`，只存在于模块中
}
```

# Vuex中action和mutation有什么相同点？

第二参数都可以接收外部提交时传来的参数。 `this.$store.dispatch('ACTION_NAME',data)`和`this.$store.commit('SET_NUMBER',10)`

# 在组件中多次提交同一个action，怎么写使用更方便。

使用mapActions辅助函数,在组件中这么使用

```js
methods:{
    ...mapActions({
        setNumber:'SET_NUMBER',
    })
}
```

然后调用`this.setNumber(10)`相当调用`this.$store.dispatch('SET_NUMBER',10)`

# Vuex中action通常是异步的，那么如何知道action什么时候结束呢？

在action函数中返回Promise，然后再提交时候用then处理

```js
actions:{
    SET_NUMBER_A({commit},data){
        return new Promise((resolve,reject) =>{
            setTimeout(() =>{
                commit('SET_NUMBER',10);
                resolve();
            },2000)
        })
    }
}
this.$store.dispatch('SET_NUMBER_A').then(() => {
  // ...
})
```

# Vuex中有两个action，分别是actionA和actionB，其内都是异步操作，在actionB要提交actionA，需在actionA处理结束再处理其它操作，怎么实现？

利用ES6的`async`和`await`来实现。

```js
actions:{
    async actionA({commit}){
        //...
    },
    async actionB({dispatch}){
        await dispatch ('actionA')//等待actionA完成
        // ... 
    }
}
```

# 有用过Vuex模块吗，为什么要使用，怎么使用。

有，因为使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。所以将 store 分割成模块（module）。每个模块拥有自己的 state、mutations、actions、getters，甚至是嵌套子模块，从上至下进行同样方式的分割。

在module文件新建moduleA.js和moduleB.js文件。在文件中写入

```js
const state={
    //...
}
const getters={
    //...
}
const mutations={
    //...
}
const actions={
    //...
}
export default{
    state,
    getters,
    mutations,
    actions
}
```

然后再index.js引入模块

```js
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);
import moduleA from './module/moduleA'
import moduleB from './module/moduleB'
const store = new Vuex.Store({
    modules:{
        moduleA,
        moduleB
    }
})
export default store
```

# 在模块中，getter和mutation接收的第一个参数state，是全局的还是模块的？

第一个参数state是模块的state，也就是局部的state。

# 在模块中，getter和mutation和action中怎么访问全局的state和getter？

在getter中可以通过第三个参数rootState访问到全局的state,可以通过第四个参数rootGetters访问到全局的getter。

在mutation中不可以访问全局的satat和getter，只能访问到局部的state。

在action中第一个参数context中的`context.rootState`访问到全局的state，`context.rootGetters`访问到全局的getter。

# 在组件中怎么访问Vuex模块中的getter和state,怎么提交mutation和action？

直接通过`this.$store.getters`和`this.$store.state`来访问模块中的getter和state。

直接通过`this.$store.commit('mutationA',data)`提交模块中的mutation。

直接通过`this.$store.dispatch('actionA,data')`提交模块中的action。

# 用过Vuex模块的命名空间吗？为什么使用，怎么使用。

默认情况下，模块内部的action、mutation和getter是注册在全局命名空间，如果多个模块中action、mutation的命名是一样的，那么提交mutation、action时，将会触发所有模块中命名相同的mutation、action。

这样有太多的耦合，如果要使你的模块具有更高的封装度和复用性，你可以通过添加`namespaced: true` 的方式使其成为带命名空间的模块。

```js
export default{
    namespaced: true,
    state,
    getters,
    mutations,
    actions
}
```

# 怎么在带命名空间的模块内提交全局的mutation和action？

将 { root: true } 作为第三参数传给 dispatch 或 commit 即可。

```js
this.$store.dispatch('actionA', null, { root: true })
this.$store.commit('mutationA', null, { root: true })
```

#怎么在带命名空间的模块内注册全局的action？

```js
actions: {
    actionA: {
        root: true,
        handler (context, data) { ... }
    }
}
```

# 怎么使用mapState，mapGetters，mapActions和mapMutations这些函数来绑定带命名空间的模块？

首先使用`createNamespacedHelpers`创建基于某个命名空间辅助函数

```js
import { createNamespacedHelpers } from 'vuex';
const { mapState, mapActions } = createNamespacedHelpers('moduleA');
export default {
    computed: {
        // 在 `module/moduleA` 中查找
        ...mapState({
            a: state => state.a,
            b: state => state.b
        })
    },
    methods: {
        // 在 `module/moduleA` 中查找
        ...mapActions([
            'actionA',
            'actionB'
        ])
    }
}
```

# Vuex插件有用过吗？怎么用简单介绍一下？

Vuex插件就是一个函数，它接收 store 作为唯一参数。在Vuex.Store构造器选项plugins引入。 在store/plugin.js文件中写入

```js
export default function createPlugin(param){
    return store =>{
        //...
    }
}
```

然后在store/index.js文件中写入

```js
import createPlugin from './plugin.js'
const myPlugin = createPlugin()
const store = new Vuex.Store({
  // ...
  plugins: [myPlugin]
})
```

# 在Vuex插件中怎么监听组件中提交mutation和action？

- 用Vuex.Store的实例方法`subscribe`监听组件中提交mutation
- 用Vuex.Store的实例方法`subscribeAction`监听组件中提交action 在store/plugin.js文件中写入

```javascript
export default function createPlugin(param) {
    return store => {
        store.subscribe((mutation, state) => {
            console.log(mutation.type)//是那个mutation
            console.log(mutation.payload)
            console.log(state)
        })
        // store.subscribeAction((action, state) => {
        //     console.log(action.type)//是那个action
        //     console.log(action.payload)//提交action的参数
        // })
        store.subscribeAction({
            before: (action, state) => {//提交action之前
                console.log(`before action ${action.type}`)
            },
            after: (action, state) => {//提交action之后
                console.log(`after action ${action.type}`)
            }
        })
    }
}
```

然后在store/index.js文件中写入

```javascript
import createPlugin from './plugin.js'
const myPlugin = createPlugin()
const store = new Vuex.Store({
  // ...
  plugins: [myPlugin]
})
```

# 在v-model上怎么用Vuex中state的值？

需要通过computed计算属性来转换。

```javascript
<input v-model="message">
// ...
computed: {
    message: {
        get () {
            return this.$store.state.message
        },
        set (value) {
            this.$store.commit('updateMessage', value)
        }
    }
}
```

# Vuex的严格模式是什么,有什么作用,怎么开启？

在严格模式下，无论何时发生了状态变更且不是由 mutation函数引起的，将会抛出错误。这能保证所有的状态变更都能被调试工具跟踪到。

在Vuex.Store 构造器选项中开启,如下

```javascript
const store = new Vuex.Store({
    strict:true,
})
```

# 如何解决vuex持久化问题
