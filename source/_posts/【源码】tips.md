---
title: 【源码】tips
date: 2021-11-08 09:43:08
tags:
    - webpack
cover_picture: /images/babel.jpeg
---



# babel解析

```javascript
class Person {
  constructor () {
    this.name = "haimeimei"
  }
  getName() {
    return this.name
  }
}
```

**结果**

```javascript
function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

function _defineProperties(target, props) {
  for (var i = 0; i < props.length; i++) {
    var descriptor = props[i];
    descriptor.enumerable = descriptor.enumerable || false;
    descriptor.configurable = true;
    if ("value" in descriptor) descriptor.writable = true;
    Object.defineProperty(target, descriptor.key, descriptor);
  }
}

function _createClass(Constructor, protoProps, staticProps) {
  if (protoProps) _defineProperties(Constructor.prototype, protoProps);
  if (staticProps) _defineProperties(Constructor, staticProps);
  return Constructor;
}

var Person = /*#__PURE__*/function () {
  "use strict";

  function Person() {
    _classCallCheck(this, Person);

    this.name = "haimeimei";
  }

  _createClass(Person, [{
    key: "getName",
    value: function getName() {
      return this.name;
    }
  }]);

  return Person;
}();
```



**```/*#__PURE__*/```terser的行内注释，标识当前函数是一个纯函数，可以被安全的删除**

### Annotations

Annotations in Terser are a way to tell it to treat a certain function call differently. The following annotations are available:

-   `/*@__INLINE__*/` - forces a function to be inlined somewhere.
-   `/*@__NOINLINE__*/` - Makes sure the called function is not inlined into the call site.
-   `/*@__PURE__*/` - Marks a function call as pure. That means, it can safely be dropped.

You can use either a `@` sign at the start, or a `#`.

Here are some examples on how to use them:

```javascript
/*@__INLINE__*/
function_always_inlined_here()

/*#__NOINLINE__*/
function_cant_be_inlined_into_here()

const x = /*#__PURE__*/i_am_dropped_if_x_is_not_used()
```



**terser源码https://github.com/terser/terser**
