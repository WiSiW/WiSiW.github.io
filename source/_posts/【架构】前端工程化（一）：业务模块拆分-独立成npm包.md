---
title: '【架构】前端工程化（一）业务模块拆分,独立成npm包'
date: 2021-08-13 11:07:02
tags:
    - 前端工程化
    - 架构
---

# 1.构建业务模块npm包

```
mkdir module_vue
cd module_vue
npm init -y
```



## 文件目录

```
├── packages
│   └── alert
│       ├── src
│       │   └── main.vue
│       └── index.js
└── src
    └── index.js
```

**模块**

```vue
// packages/alert/src/main.vue
<template>
    <div>alert</div>
</template>

<script>
    export default {
        name: "Alert",
        data() {
            return {
            }
        }
    }
</script>

<style scoped>

</style>
```

**导出模块**

```javascript
// packages/alert/index.js
import Alert from './src/main';

Alert.install = function(Vue) {
    Vue.component(Alert.name, Alert);
};

export default Alert;
```

**VUE中注册模块组件**

```javascript
// src/index.js
import Alert from '../packages/alert/index.js';

const components = [
    Alert
];

const install = function(Vue) {
    components.forEach(component => {
        Vue.component(component.name, component);
    });

};

/* istanbul ignore if */
if (typeof window !== 'undefined' && window.Vue) {
    install(window.Vue);
}

export default {
    version: '0.1.0',
    install,
    Alert
};
```



# 2.项目引用本地模块npm包

## 2.1 webpack 4.X 开始，需要安装 webpack-cli 依赖

```
npm install webpack webpack-cli -g
```



## 2.2 如果未安装vue-cli，全局安装

```
npm install --global vue-cli
```



## 2.3 创建项目

```
vue create project
cd project
```



## 2.4 安装本地npm包

```
npm install <package path>/module_vue
```



## 2.5 建立npm本地映射，此操作是为了方便开发

```
npm link module_vue
```

**此操作可以将node_modules中的module_vue链接到本地module_vue，实时更新**
