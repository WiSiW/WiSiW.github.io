---
title: 【JavaScript】构建单页面应用
date: 2021-12-06 13:13:06
tags:
    - JavaScript
    - single
cover_picture: /images/javascript.png
---

# 页面目录

```
single
  ├── frontend
  |     ├── static
  |     |     ├──js
  |     |     |   └─ index.js
  |     |     └─ views
  |     |         ├─ View.js
  |     |         └─ Setting.js
  |     └─ index.html
  ├── package.json
  └── server.js
```



# 1.使用express构建应用

```javascript
npm init -y
npm i express
```



```javascript
// server.js
const express = require('express')
const path = require('path')
const app = express()

function resolvePath(_path) {
  return path.resolve(__dirname, 'frontend', _path)
}
app.use('/static', express.static(resolvePath('static')))
app.get('/*', (req, res) => {
  res.sendFile(resolvePath('index.html'))
})

app.listen(process.env.PORT || 5001 , () => console.log('serve running...'))
```



![express开启](data:image/png;base64,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)

# 2.创建首页

此时点击链接会跳转相应页面，并刷新当前页

```html
<!-- frontend/index.html -->
<html>
  <body>
    <nav>
      <a href="/" data-link>/</a>
      <a href="/post" data-link>post</a>
      <a href="/setting" data-link>setting</a>
    </nav>
    <script type="module" src="/static/js/index.js"></script>
  </body>
</html>
```



# 3.创建路由

```javascript
// fronted/static/js/index.js
const router = async () => {
  const routes = [
    {path: '/', view: () => console.log('/')},
    {path: '/setting', view: () => console.log('/setting')}
  ]
  // 匹配当前路由。无匹配项则匹配根路由
  const potentialMatches = routes.map(route => {
    return {
      route,
      isMatch: route.path === location.pathname
    }
  })
  let match = potentialMatches.find(potentialMatche => potentialMatche.isMatch)
  if(!match) {
    match = {
      route: routes[0],
      isMatch: true
    }
  }
  console.log(match.route.view())
}
```



# 4.监听路由切换

```javascript
// fronted/static/js/index.js
// 监听历史记录条目的更改
window.addEventListener('popstate', () => {
  console.log("popstate");
  router()
})
// 当初始的HTML文档被完全加载和解析完毕后，监听页面的click事件，当触发clik的元素包含[data-link]属性，阻止冒泡，并对路由进行处理
window.addEventListener("DOMContentLoaded", () => {
  router()
  document.body.addEventListener('click', e => {
    if (e.target.matches('[data-link]')) {
      e.preventDefault()
      // ...
    }
  })
})
```



# 5.切换路由，但DOM不刷新

```javascript
// fronted/static/js/index.js
const navigateTo = url => {
  // 切换路由，但不刷新页面
  history.pushState(null, null, url)
  router()
}
window.addEventListener("DOMContentLoaded", () => {
  router()
  document.body.addEventListener('click', e => {
    if (e.target.matches('[data-link]')) {
      e.preventDefault()
      navigateTo(e.target.href)
    }
  })
})
```



# 6.创建单页面

## 6-1.创建根页面

```javascript
// fronted/static/views/View.js
export default class {
  constructor() {
    this.title = null
  }
  setTitle(title) {
    this.title = title
    document.title = title
  }

  async getHtml() {
    return ``
  }
}
```

## 6-2.创建```/setting```页面

```javascript
// fronted/static/views/Setting.js
import View from './View.js'

export default class extends View {
  constructor() {
    super()
    this.setTitle('Setting')
  }

  async getHtml() {
    return `<h1>Setting</h1>`
  }
}
```



## 6-3.在页面中创建展示页面的元素

```html
<!-- frontend/index.html -->
<html>
  <body>
    <nav>...</nav>
    <div id="app"></div>
    <script type="module" src="/static/js/index.js"></script>
  </body>
</html>
```



# 7.在框架中应用页面

```javascript
// fronted/static/js/index.js
import SettingView from '../views/Setting.js'
import View from '../views/View.js'
const router = async () => {
  const routes = [
    {path: '/', view: View},
    {path: '/setting', view: SettingView}
  ]
  // ...
  const view = new match.route.view()
  // 将返回的当页面嵌入当前页面
  document.querySelector('#app').innerHTML = await view.getHtml()
}
```



# 8.获取地址栏中的参数

主要使用正则获取

## 8-1.路由匹配

```javascript
const pathToRegex = path => new RegExp('^' + path.replace(/\//g, '\\/').replace(/:\w+/g, '(.+)') + '$')

import SettingView from '../views/Setting.js'
import View from '../views/View.js'

const router = async () => {
  const routes = [
    { path: '/', view: View },
    { path: '/setting/:id', view: SettingView }
  ]

  const potentialMatches = routes.map(route => {
    return {
      route,
      result: location.pathname.match(pathToRegex(route.path))
    }
  })
  let match = potentialMatches.find(potentialMatche => potentialMatche.result !== null)
  if(!match) {
    match = {
      route: routes[0],
      result: [location.pathname]
    }
  }
  // ...
}
```



## 8-2.路由传值

```javascript
const getParams = match => {
    const values = match.result.slice(1)
    const keys = Array.from(match.route.path.matchAll(/:(\w+)/g)).map(result => result[1]);
    return Object.fromEntries( keys.map( (key,i) => {
        return [key, values[i]]
    }))

}
const router = async () => {
  // ...
  const view = new match.route.view(getParams(match))
  // ...
}
```



## 8-3.页面接受参数

```javascript
// fronted/static/views/View.js
export default class {
  constructor(params) {
    this.params = params
    this.title = null
  }
  setTitle(title) {
    this.title = title
    document.title = title
  }

  async getHtml() {
    return ``
  }
}
```



```javascript
// fronted/static/views/Setting.js
import View from './View.js'

export default class extends View {
  constructor() {
    super(props)
    this.setTitle('Setting')
  }

  async getHtml() {
    return `<h1>Setting${this.params.id}</h1>`
  }
}
```



# 源码：

https://github.com/WiSiW/frontend/tree/master/single

[Build a Single Page Application with JavaScript (No Frameworks)]:https://www.youtube.com/watch?v=6BozpmSjk-Y
[Adding Client Side URL Params - Build a Single Page Application with JavaScript (No Frameworks)]:https://www.youtube.com/watch?v=OstALBk-jTc

