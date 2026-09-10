---
title: 【JavaScript】技巧
date: 2021-07-05 10:01:45
tags:
    - 技巧
    - JavaScript
cover_picture: /images/javascript.png
---

# 引入JavaScript脚本

```html
<script>
  export default {
    name: 'hello-world',
    mounted() {
      alert('hello-world')
    }
  }
</script>
```
以上的```script```组件内部是一个模块，因此可以导出。其中包含组件的名称```name```和声明周期```mounted```

在```import```时，因为是在HTML文件中，因此无法```import```
**```import```语句获取的是模块标识符。大部分情况下，它是包含代码的文件的URL，在内部模块的情况下，是没有这样的标识符的。

但是可以使用数据URI（将JavaScript文件内容转换为URL，然后使用Base64对所有内容进行编码）

**例如：**

```javascript
  export default true
```

转换为如下的数据URI

```
data:application/javascript;base64,ZXhwb3J0IGRlZmF1bHQgdHJ1ZTs=
```

此时可以像引用普通文件一样引用

```javascript
import test from 'data:application/javascript;base64,ZXhwb3J0IGRlZmF1bHQgdHJ1ZTs='

console.log(test) // true
```

此方法最大的缺点是，数据URI会随着文件的增大而增长，以合理的方式将二进制数据放入数据URI中也非常困难

这时就需要使用对象URI，

# 1.多个条件判断的if语句，使用includes

**在数组中存储需要判断的值，使用数组的includes方法**

```javascript
if( x === 'abc' || x === 'def' || x === 'ghi') {
   
}
if(['abc', 'def', 'ghi'].includes(x)) {
  
}
```

# 2.if...else使用三元表达式

**if...else的逻辑判断较为简单的时候，使用三元判断**

