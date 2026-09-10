---
title: '【Javascript】滚动吸附'
date: 2023-09-04 14:08:00
tags:
    - 滚动吸附
---

# 什么是滚动吸附

# 实现原理

1.使用```mousewheel```监听页面滚动
2.使用```scrollIntoView```将元素滚动至视区
3.使用防抖函数进行优化

**页面样式**
```html
<html>
  <head>
    <style>
      * {
        margin: 0;
        padding: 0;
      }
      body {
        width: 100%;
        height: 100vh;
        overflow-y: auto;
      }
      body .content {
        width: 100%;
        height: 100vh;
      }
    </style>
  </head>
  <body>
    <div class="content" style="background: red"></div>
    <div class="content" style="background: yellow"></div>
    <div class="content" style="background: blue"></div>
  </body>
</html>
```

## 1.使用```mousewheel```监听页面滚动

根据```e.wheelDelta```的正负来判断滚动的方向

```javascript
document.body.addEventListener(
  "mousewheel",
  (e) => {
    if (e.wheelDelta < 0) {
      // down
    }
    if (e.wheelDelta > 0) {
      // up
    }
  }
);
```

## 2.使用```scrollIntoView```将元素滚动至视区

```javascript
document.body.addEventListener(
  "mousewheel",
  (e) => {
    const els = document.querySelectorAll(".content");
    if (e.wheelDelta < 0) {
      // down
      scroll(els[0])
    }
    if (e.wheelDelta > 0) {
      // up
      scroll(els[1])
    }
  }
);
function scroll(el) {
  if (el) {
    el.scrollIntoView({ behavior: "smooth", block: "center" });
  }
}
```

这样能实现两个```class="content"```元素之间的滚动吸附，但这不具备通用性，当前页面只有两个元素，可以根据滚动方向来判断将那个元素滚动到当前视区。但如果增加元素数量，就很难判断当前要滚动哪个元素，有的人可能想着增加一个全局idx，通过上下滚动来修改idx进行判断当前应该滚动哪个元素，但因为第一步的滚动监听是持续的，因此会造成idx的不断修改，因此进入第三步，引入防抖函数.

## 3.使用防抖函数进行优化

```javascript
let idx = 0;
document.body.addEventListener(
  "mousewheel",
  debounce((e) => {
    const els = document.querySelectorAll(".content");
    if (e.wheelDelta < 0) {
      // down
      idx++;
      if (idx > els.length - 1) idx = els.length - 1;
    }
    if (e.wheelDelta > 0) {
      // up
      idx--;
      if (idx < 0) idx = 0;
    }
    scroll(els[idx]);
  }, 500)
);
function debounce(fn, delay = 500) {
  let timer = null;
  return function () {
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(this, arguments);
      timer = null;
    }, delay);
  };
}
```

# 代码实现

```html
<!DOCTYPE html>
<html lang="zh">
  <head>
    <style>
      * {
        margin: 0;
        padding: 0;
      }
      body {
        width: 100%;
        height: 100vh;
        overflow-y: auto;
      }
      body .content {
        width: 100%;
        height: 100vh;
      }
    </style>
  </head>
  <body>
    <div class="content" style="background: red"></div>
    <div class="content" style="background: yellow"></div>
    <div class="content" style="background: blue"></div>
    <script>
      let idx = 0;
      document.body.addEventListener(
        "mousewheel",
        debounce((e) => {
          const els = document.querySelectorAll(".content");
          if (e.wheelDelta < 0) {
            // down
            idx++;
            if (idx > els.length - 1) idx = els.length - 1;
          }
          if (e.wheelDelta > 0) {
            // up
            idx--;
            if (idx < 0) idx = 0;
          }
          scroll(els[idx]);
        }, 500)
      );
      function scroll(el) {
        if (el) {
          el.scrollIntoView({ behavior: "smooth", block: "center" });
        }
      }
      function debounce(fn, delay = 500) {
        let timer = null;
        return function () {
          if (timer) {
            clearTimeout(timer);
          }
          timer = setTimeout(() => {
            fn.apply(this, arguments);
            timer = null;
          }, delay);
        };
      }
    </script>
  </body>
</html>
```
