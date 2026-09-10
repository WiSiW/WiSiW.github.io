---
title: 【JavaScript】CommonJS与ES6的区别
date: 2021-10-26 17:22:08
tags:
    - JavaScript
    - CommonJS
    - ES6
    - module
---
[Differences between ES modules and CommonJS](https://nodejs.org/docs/latest/api/esm.html#esm_differences_between_es_modules_and_commonjs)

# Differences between ES modules and CommonJS

## No require, exports, or module.exports

In most cases, the ES module import can be used to load CommonJS modules.

If needed, a require function can be constructed within an ES module using module.createRequire().

## No __filename or __dirname

These CommonJS variables are not available in ES modules.

__filename and __dirname use cases can be replicated via import.meta.url.

## No Native Module Loading

Native modules are not currently supported with ES module imports.

They can instead be loaded with module.createRequire() or process.dlopen.

## No require.resolve

Relative resolution can be handled via new URL('./local', import.meta.url).

For a complete require.resolve replacement, there is a flagged experimental import.meta.resolve API.

Alternatively module.createRequire() can be used.

## No NODE_PATH

NODE_PATH is not part of resolving import specifiers. Please use symlinks if this behavior is desired.

## No require.extensions

require.extensions is not used by import. The expectation is that loader hooks can provide this workflow in the future.

## No require.cache

require.cache is not used by import as the ES module loader has its own separate cache.

# 导入导出

功能 | CommonJS | ES6
 -- | -- | --
导入 | ```exports | module.export``` | ```export | export  default```
导出 | ```require``` | ```import```

```export```指向```module.exports```
## 补充
### CommonJS输出的是一个<font color="red">值的<u>拷贝</u></font>，ES6输出的是<font color="red">值的<u>引用</u></font>

CommonJS模块作为一个对象被导出，会加载一个新的对象，直接修改变量中的参数，会造成对象中的参数变化
如果通过解构获取模块中的变量，然后通过模块中暴露的方法修改模块内部的变量，不会造成当前模块创建的变量的变化

```javascript
// common.js
exports.name = 'aaa'

exports.changeName = (name) => {
  this.name = name
}
```

```javascript
// index.js
const common = require('./common')

console.log(common.name); // aaa

common.changeName('bbb');

console.log(common.name); // bbb
```

```javascript
// index.js
const {name, changeName} = require('./common')

console.log(name); // aaa

changeName('bbb');

console.log(name); // aaa
```

ES6的运行机制与CommonJS不一样，遇到模块加载命令import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。换句话说，ES6的import 有点像 Unix 系统的“符号连接”，原始值变了，import加载的值也会跟着变。因此，ES6模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

```javascript
// es.js
export const name = "aaa"
export const changeName = name => {
  console.log(this);
  this.name = name
}

export default {
  name,
  changeName
}
```

```html
<body>
  <script type="module" src="./common.js"></script>
  <script type="module">
    import common from './common.js'
    console.log(common.name); // aaa
    common.changeName("bbb"); // Uncaught TypeError: Cannot set properties of undefined (setting 'name')
  </script>
</body>
```

ES6支持再导出

```javascript
export { es } from './es.js'

export default {};
```

动态导入

```javascript
import('es.js').then(es => {
    console.log(es); // Module {Symbol(Symbol.toStringTag): 'Module'}
    console.log(es.default); // {name: 'aaa', changeName: ƒ}
})
```

# 加载

功能| CommonJS | ES6
-- | -- | --
加载 | 运行时 | 编译时
详情 | CommonJS 模块就是对象；即在输入时是先加载整个模块，生成一个对象，然后再从这个对象上面读取方法 | ES6 模块不是对象，而是通过 export 命令显式指定输出的代码，import时采用静态命令的形式。即在import时可以指定加载某个输出值，而不是加载整个模块

# 文件命名

Node.js 会将.cjs文件视为 CommonJS 模块，将.mjs文件视为 ECMAScript 模块。它会将.js文件视为项目的默认模块系统（除非package.json说的是 CommonJS "type": "module",）。
