---
title: 【代码片段】Vue组件化技巧
date: 2021-10-18 10:44:30
tags:
    - Vue
    - 组件化
---

# 使用**插槽prop**实现组件的事件封装
**子组件**
```javascript
Vue.component('Child', {
    template: `<div>
        <slot :handle='handle'></slot>
    </div>`,
    methods: {
        handle() {
            console.log(1)
        }
    }
})
```
**父组件**
```javascript
var parent = new Vue({
    name: 'Parent',
    el: '#app',
    template: `<child v-slot='{ handle }'>
        <div @click='handle'>子组件</div>
    </child>`
})
```

# 完整代码
```html
<!DOCTYPE html>
<script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.9/vue.common.dev.js"></script>
<body>
    <div id="app"></div>
</body>
<script>
Vue.component('Child', {
    template: `<div>
        <slot :handle='handle'></slot>
    </div>`,
    methods: {
        handle() {
            console.log(1)
        }
    }
})
var parent = new Vue({
    name: 'Parent',
    el: '#app',
    template: `<child v-slot='{ handle }'>
        <div @click='handle'>子组件</div>
    </child>`
})
</script>
</html>
```
