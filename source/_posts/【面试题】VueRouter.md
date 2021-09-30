---
title: 【面试题】VueRouter
date: 2021-02-20 10:37:28
tags:
	- Vue
	- VueRouter
---



# Vue-Router基本应用

### 怎么重定向页面

第一种方法：

```javascript
const router = new VueRouter({
    routes: [
        { path: '/a', redirect: '/b' }
    ]
})
```

第二种方法:

```javascript
const router = new VueRouter({
    routes: [
        { path: '/a', redirect: { name: 'foo' }}
    ]
})
```

第三种方法：

```javascript
const router = new VueRouter({
    routes: [
        { 
            path: '/a', 
            redirect: to =>{
                const { hash, params, query } = to
                if (query.to === 'foo') {
                    return { path: '/foo', query: null }
                }else{
                   return '/b' 
                }
            }
            
        }
    ]
})
```

# 怎么配置404页面？

```javascript
const router = new VueRouter({
    routes: [
        {
            path: '*', redirect: {path: '/'}
        }
    ]
})
```


# 切换路由时，需要保存草稿的功能，怎么实现呢？

```javascript
<keep-alive :include="include">
    <router-view></router-view>
 </keep-alive>
```

# 路由有几种模式？说说它们的区别？

- **hash**: 兼容所有浏览器，包括不支持 HTML5 History Api 的浏览器，例`http://www.abc.com/#/index`，hash值为#/index， hash的改变会触发hashchange事件，通过监听hashchange事件来完成操作实现前端路由。hash值变化不会让浏览器向服务器请求。
- **history**: 兼容能支持 HTML5 History Api 的浏览器，依赖HTML5 History API来实现前端路由。没有`#`，路由地址跟正常的url一样，但是初次访问或者刷新都会向服务器请求，如果没有请求到对应的资源就会返回404，所以路由地址匹配不到任何静态资源，则应该返回同一个index.html 页面，需要在nginx中配置。
- **abstract**: 支持所有 JavaScript 运行环境，如 Node.js 服务器端。如果发现没有浏览器的 API，路由会自动强制进入这个模式。

# 讲一下完整的导航守卫流程？

导航被触发。

在失活的组件里调用离开守卫`beforeRouteLeave(to,from,next)`。

调用全局的`beforeEach( (to,from,next) =>{} )`守卫。

在重用的组件里调用 `beforeRouteUpdate(to,from,next)` 守卫。

在路由配置里调用`beforeEnter(to,from,next)`路由独享的守卫。

解析异步路由组件。

在被激活的组件里调用`beforeRouteEnter(to,from,next)`。

在所有组件内守卫和异步路由组件被解析之后调用全局的`beforeResolve( (to,from,next) =>{} )`解析守卫。

导航被确认。

调用全局的`afterEach( (to,from) =>{} )`钩子。

触发 DOM 更新。

用创建好的实例调用beforeRouteEnter守卫中传给 next 的回调函数

```javascript
beforeRouteEnter(to, from, next) {
    next(vm => {
        //通过vm访问组件实例
    })
},
```

# 导航守卫的三个参数的含义？

to：即将要进入的目标 路由对象。

from：当前导航正要离开的路由对象。

next：函数，必须调用，不然路由跳转不过去。

- `next()`：进入下一个路由。
- `next(false)`：中断当前的导航。
- `next('/')`或`next({ path: '/' })` : 跳转到其他路由，当前导航被中断，进行新的一个导航。

# 全局导航守卫有哪些？怎么使用？

- `router.beforeEach`：全局前置守卫。
- `router.beforeResolve`：全局解析守卫。
- `router.afterEach`：全局后置钩子。

# 在afterEach钩子中可以使用`next()`吗？

不可以，不接受`next`的参数。

# 什么是路由独享的守卫，怎么使用？

是`beforeEnter`守卫

```javascript
const router = new VueRouter({
    routes: [
        {
            path: '/foo',
            component: Foo,
            beforeEnter: (to, from, next) => {
            // ...
            }
        }
    ]
})
```



# 说说你对router-link的了解

`<router-link>`是Vue-Router的内置组件，在具有路由功能的应用中作为声明式的导航使用。

`<router-link>`有8个props，其作用是：

- to

  ：必填，表示目标路由的链接。当被点击后，内部会立刻把to的值传到 router.push()，所以这个值可以是一个字符串或者是描述目标位置的对象。

  ```javascript
  <router-link to="home">Home</router-link>
  <router-link :to="'home'">Home</router-link>
  <router-link :to="{ path: 'home' }">Home</router-link>
  <router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
  <router-link :to="{ path: 'user', query: { userId: 123 }}">User</router-link>
  ```

  注意path存在时params不起作用，只能用query

- `replace`：默认值为false，若设置的话，当点击时，会调用`router.replace()`而不是`router.push()`，于是导航后不会留下 history 记录。

- `append`：设置 append 属性后，则在当前 (相对) 路径前添加基路径。

- `tag`：让`<router-link>`渲染成`tag`设置的标签，如`tag:'li`,渲染结果为`<li>foo</li>`。

- `active-class`：默认值为`router-link-active`,设置链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 linkActiveClass 来全局配置。

- `exact-active-class`：默认值为`router-link-exact-active`,设置链接被精确匹配的时候应该激活的 class。默认值可以通过路由构造函数选项 linkExactActiveClass 进行全局配置的。

- exact：是否精确匹配，默认为false。

  ```javascript
  <!-- 这个链接只会在地址为 / 的时候被激活 -->
  <router-link to="/" exact></router-link>
  ```

- `event`：声明可以用来触发导航的事件。可以是一个字符串或是一个包含字符串的数组，默认是`click`。

# 怎么在组件中监听路由参数的变化？

有两种方法可以监听路由参数的变化，**但是只能用在包含`<router-view />`的组件内。**

- 第一种

  ```javascript
  watch: {
      '$route'(to, from) {
          //这里监听
      },
  },
  ```

- 第二种

  ```javascript
  beforeRouteUpdate (to, from, next) {
      //这里监听
  },
  ```

# 切换路由后，新页面要滚动到顶部或保持原先的滚动位置怎么做呢？

滚动顶部

```javascript
const router = new Router({
    mode: 'history',
    base: process.env.BASE_URL,
    routes,
    scrollBehavior(to, from, savedPosition) {
        if (savedPosition) {
            return savedPosition;
        } else {
            return { x: 0, y: 0 };
        }
    }
});
```

# 路由组件和路由为什么解耦，怎么解耦？

因为在组件中使用 $route 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的 URL 上使用，限制了其灵活性，所有要解耦。

- 耦合如以下代码所示。Home组件只有在

  ```javascript
  http://localhost:8036/home/123
  ```

  URL上才能使用。

  ```javascript
  const Home = {
      template: '<div>User {{ $route.params.id }}</div>'
  }
  const router = new VueRouter({
      routes: [
          { path: '/home/:id', component: Home }
      ]
  })
  ```

- 使用 props 来解耦

  - props为true，`route.params`将会被设置为组件属性。
  - props为对象，则按原样设置为组件属性。
  - props为函数，`http://localhost:8036/home?id=123`,会把123传给组件Home的props的id。

  ```javascript
  const Home = {
      props: ['id'],
      template: '<div>User {{ id }}</div>'
  }
  const router = new VueRouter({
      routes: [
          { path: '/home/:id', component: Home, props: true},
          // 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
          {
              path: '/home/:id',
              components: { default: Home, sidebar: Sidebar },
              props: { default: true, sidebar: false }
          }
          { path: '/home', component: Home, props: {id:123} },
          { path: '/home', component: Home, props: (route) => ({ id: route.query.id }) },
      ]
  })
  ```

# route和router有什么区别

- route是“路由信息对象”，包括path，params，hash，query，fullPath，matched，name等路由信息参数。 
- router是“路由实例对象”，包括了路由的跳转方法，钩子函数等。

# 路由之间是怎么跳转的？有哪些方式？

- 声明式 通过使用内置组件`<router-link :to="/home">`来跳转
- 编程式 通过调用router实例的push方法`router.push({ path: '/home' })`或replace方法`router.replace({ path: '/home' })`

# 怎么实现路由懒加载呢

```javascript
function load(component) {
    //return resolve => require([`views/${component}`], resolve);
    return () => import(`views/${component}`);
}

const routes = [
    {
        path: '/home',
        name: 'home',
        component: load('home'),
        meta: {
            title: '首页'
        },
    },
]
```



# 怎样动态加载路由

使用Router的实例方法addRoutes来实现动态加载路由，一般用来实现菜单权限。

使用时要注意，静态路由文件中不能有404路由，而要通过addRoutes一起动态添加进去。

```javascript
const routes = [
    {
        path: '/overview',
        name: 'overview',
        component: () => import('@/views/account/overview/index'),
        meta: {
            title: '账户概览',
            pid: 869,
            nid: 877
        },
    },
    {
        path: '*',
        redirect: {
            path: '/'
        }
    }
]
vm.$router.options.routes.push(...routes);
vm.$router.addRoutes(routes);
```



# active-class是哪个组件的属性

`<router-link/>`组件的属性，设置链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 linkActiveClass 来全局配置

# Vue路由怎么跳转打开新窗口

```javascript
const obj = {
    path: xxx,//路由地址
    query: {
       mid: data.id//可以带参数
    }
};
const {href} = this.$router.resolve(obj);
window.open(href, '_blank');
```

