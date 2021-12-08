---
title: 【CSS】底部栏动态切换
date: 2021-12-07 18:48:10
tags:
    - CSS
    - H5
cover_picture: /images/css.png

---

![底部栏菜单动态切换](/images/navigation_menu.gif)

![底部栏动态切换](data:image/png;base64,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)



[底部动态栏样式仓库](https://github.com/WiSiW/frontend/tree/master/CSS/BottomNavigationMenu)

# 1.创建页面

![底部栏基础样式](data:image/png;base64,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)

## 1-2.HTML

```html
<div class="menu-box">
  <div class="menu-item active">
  	<i class="menu-icon home"></i>
  	<span>首页</span>
  </div>
  <div class="menu-item">
    <i class="menu-icon discover"></i>
    <span>发现</span>
  </div>
  <div class="menu-item">
    <i class="menu-icon search"></i>
    <span>探索</span>
  </div>
  <div class="menu-item">
    <i class="menu-icon my"></i>
    <span>我的</span>
  </div>
</div>
```

## 1-2.CSS

```less
:root {
  --iconSize: 80px;
  --borderW: 10px;
  --bgClr: #f0f0f0;
  --actClr: cadetblue;
  --actTextClr: #ffffff;
}
.menu {
  &-box {
    width: 600px;
    height: 80px;
    background: var(--bgClr);
    border-radius: 10px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    display: flex;
    justify-content: space-around;
  }
  &-item {
    width: 25%;
    text-align: center;
    position: relative;
    z-index: 9;
    span {
      display: none;
    }
  }
  &-icon {
    width: var(--iconSize);
    height: var(--iconSize);
    background-repeat: no-repeat;
    background-size: 40%;
    background-position: 50%;
    margin: 0 auto;
    display: inline-block;
    &.home {
      background-image: url("./imgs/home.svg");
    }
    &.discover {
      background-image: url("./imgs/discover.svg");
    }
    &.search {
      background-image: url("./imgs/search.svg");
    }
    &.my {
      background-image: url("./imgs/my.svg");
    }
  }
}
```





# 2.添加选中样式



![底部栏选中样式](data:image/png;base64,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)





```less
.menu {
   &-item {
     // ...
     &.active {
       .menu-icon {
         transform: translate(0, -50%);
         animation: active 1s;
       }
       @keyframes active {
         0% {
           transform: translate(0, 0);
         }
         100% {
           transform: translate(0, -50%);
         }
       }
       span {
         display: block;
         color: var(--actTextClr);
         transform: translate(0, -30px);
       }
     }
     &:nth-child(1).active ~ .menu-indicator {
       left: 12.5%;
     }
     &:nth-child(2).active ~ .menu-indicator {
       left: 12.5% + 25%;
     }
     &:nth-child(3).active ~ .menu-indicator {
       left: 12.5% + 2 * 25%;
     }
     &:nth-child(4).active ~ .menu-indicator {
       left: 12.5% + 3 * 25%;
     }
  }
  &-indicator {
    width:var(--iconSize);
    height: var(--iconSize);
    border-radius: 50%;
    box-sizing: border-box;
    border: var(--borderW) solid #ffffff;
    background-color: var(--actClr);
    position: absolute;
    transform: translate(-50%, -50%);
    &:before {
      content: '';
      width: 20px;
      height: 20px;
      background: #f0f0f0;
      border-top-right-radius: 20px;
      display: inline-block;
      position: absolute;
      left: -27px;
      bottom: 10px;
      box-shadow: 0 -10px 0 0 #ffffff;
    }
    &:after {
      content: '';
      width: 20px;
      height: 20px;
      background: #f0f0f0;
      border-top-left-radius: 20px;
      display: inline-block;
      position: absolute;
      right: -27px;
      bottom: 10px;
      box-shadow: 0 -10px 0 0 #ffffff;
    }
  }
}
```



# 3.添加动态切换样式

## 3-1.根据当前选中的标签和之前选中的标签动态切换样式

```javascript
let current = 0
const menus = document.getElementsByClassName('menu-item')
for (let i = 0; i < menus.length; i++) {
  const menu = menus[i]
  menu.addEventListener('click', e => {
    e.preventDefault()
    changeTab(i)
  })
}
function changeTab(index) {
  if (index === current) {
    return
  }
  const menus = document.getElementsByClassName('menu-item')
  if (!menus.length) {
    return
  }
  for (let i = 0; i < menus.length; i++ ) {
    if (i === index) {
      menus[i].classList.add('active')
    }else {
      menus[i].classList.remove('active')
    }
  }
  current = index
}
```



## 3-2.为动态切换添加动画效果

```javascript
function changeTab(index) {
  // ...
  for (let i = 0; i < menus.length; i++ ) {
    if (i === index) {
      // ...
      setKeyframes(`move${index + 1}`, current, index)
      document.getElementsByClassName('menu-indicator')[0].style.animation = `move${index + 1} 1s`
    }else {
      // ...
    }
  }
  // ...
}
function setKeyframes(key_name, current, index) {
  const token = window.WebKitCSSKeyframesRule ? '-webkit-':'';
  const nameRule = getKeyframes(key_name);
  let rules = `
    @${token}keyframes ${key_name}{
      0% {
     		left: ${12.5 + 25 * current}%
      }
      100% {
      	left: ${12.5 + 25 * index}%;
      }
    }
  `
  if(JSON.stringify(nameRule) == '{}'){
    document.styleSheets[0].insertRule(rules,0);
  }else{
    nameRule.styleSheet.deleteRule(nameRule.index)
    nameRule.styleSheet.insertRule(rules,nameRule.index)
  }
}
function getKeyframes(name){
  const animation={}
  const styleSheets=document.styleSheets
  for(let i=0;i<styleSheets.length;i++){
    const item = styleSheets[i];
    for (let j = 0; j < item.cssRules.length; j ++) {
      if(item.cssRules[j] && item.cssRules[j].name && item.cssRules[j].name == name){
        animation.cssRule = item.cssRules[j];
        animation.styleSheet = item;
        animation.index = j;
      }
    }
  }
  return animation;
}
```

![底部栏菜单动态切换](/images/navigation_menu.gif)

## 全部代码

```javascript
let current = 0
const menus = document.getElementsByClassName('menu-item')
for (let i = 0; i < menus.length; i++) {
  const menu = menus[i]
  menu.addEventListener('click', e => {
    e.preventDefault()
    changeTab(i)
  })
}
function changeTab(index) {
  if (index === current) {
    return
  }
  const menus = document.getElementsByClassName('menu-item')
  if (!menus.length) {
    return
  }
  for (let i = 0; i < menus.length; i++ ) {
    if (i === index) {
      menus[i].classList.add('active')
      setKeyframes(`move${index + 1}`, current, index)
      document.getElementsByClassName('menu-indicator')[0].style.animation = `move${index + 1} 1s`
    }else {
      menus[i].classList.remove('active')
    }
  }
  current = index
}
function setKeyframes(key_name, current, index) {
  const token = window.WebKitCSSKeyframesRule ? '-webkit-':'';
  const nameRule = getKeyframes(key_name);
  let rules = `
    @${token}keyframes ${key_name}{
      0% {
      	left: ${12.5 + 25 * current}%
      }
      100% {
      	left: ${12.5 + 25 * index}%;
      }
    }
  `
  if(JSON.stringify(nameRule) == '{}'){
    document.styleSheets[0].insertRule(rules,0);
  }else{
    nameRule.styleSheet.deleteRule(nameRule.index)
    nameRule.styleSheet.insertRule(rules,nameRule.index)
  }
}
function getKeyframes(name){
  const animation={}
  const styleSheets=document.styleSheets
  for(let i=0;i<styleSheets.length;i++){
    const item = styleSheets[i];
    for (let j = 0; j < item.cssRules.length; j ++) {
      if(item.cssRules[j] && item.cssRules[j].name && item.cssRules[j].name == name){
        animation.cssRule = item.cssRules[j];
        animation.styleSheet = item;
        animation.index = j;
      }
    }
  }
  return animation;
}
```



# 源码

## HTML

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>BottomNavigationMenu</title>
    <link rel="stylesheet/less" type="text/css" href="./style.less" />
    <script src="https://cdn.jsdelivr.net/npm/less@4.1.2/dist/less.min.js" ></script>
</head>
<body>
  <div class="menu-box">
    <div class="menu-item active">
      <i class="menu-icon home"></i>
      <span>首页</span>
    </div>
    <div class="menu-item">
      <i class="menu-icon discover"></i>
      <span>发现</span>
    </div>
    <div class="menu-item">
      <i class="menu-icon search"></i>
      <span>探索</span>
    </div>
    <div class="menu-item">
      <i class="menu-icon my"></i>
      <span>我的</span>
    </div>
    <div class="menu-indicator"></div>
  </div>
  <script>
    let current = 0
    const menus = document.getElementsByClassName('menu-item')
    for (let i = 0; i < menus.length; i++) {
      const menu = menus[i]
      menu.addEventListener('click', e => {
        e.preventDefault()
        changeTab(i)
      })
    }
    function changeTab(index) {
      if (index === current) {
        return
      }
      const menus = document.getElementsByClassName('menu-item')
      if (!menus.length) {
        return
      }
      for (let i = 0; i < menus.length; i++ ) {
        if (i === index) {
          menus[i].classList.add('active')
          setKeyframes(`move${index + 1}`, current, index)
          document.getElementsByClassName('menu-indicator')[0].style.animation = `move${index + 1} 1s`
        }else {
          menus[i].classList.remove('active')
        }
      }
      current = index
    }
    function setKeyframes(key_name, current, index) {
      const token = window.WebKitCSSKeyframesRule ? '-webkit-':'';
      const nameRule = getKeyframes(key_name);
      let rules = `
        @${token}keyframes ${key_name}{
          0% {
          	left: ${12.5 + 25 * current}%
          }
          100% {
          	left: ${12.5 + 25 * index}%;
          }
        }
      `
      if(JSON.stringify(nameRule) == '{}'){
        document.styleSheets[0].insertRule(rules,0);
      }else{
        nameRule.styleSheet.deleteRule(nameRule.index)
        nameRule.styleSheet.insertRule(rules,nameRule.index)
      }
    }
    function getKeyframes(name){
      const animation={}
      const styleSheets=document.styleSheets
      for(let i=0;i<styleSheets.length;i++){
        const item = styleSheets[i];
        for (let j = 0; j < item.cssRules.length; j ++) {
          if(item.cssRules[j] && item.cssRules[j].name && item.cssRules[j].name == name){
            animation.cssRule = item.cssRules[j];
            animation.styleSheet = item;
            animation.index = j;
          }
        }
      }
      return animation;
    }
  </script>
</body>
</html>

```



## CSS

**使用less**

```less
:root {
  --iconSize: 80px;
  --borderW: 10px;
  --bgClr: #f0f0f0;
  --actClr: cadetblue;
  --actTextClr: #ffffff;
}
.menu {
  &-box {
    width: 600px;
    height: 80px;
    background: var(--bgClr);
    border-radius: 10px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    display: flex;
    justify-content: space-around;
  }
  &-item {
    width: 25%;
    text-align: center;
    position: relative;
    z-index: 9;
    span {
      display: none;
    }
    &.active {
      .menu-icon {
        transform: translate(0, -50%);
        animation: active 1s;
      }
      @keyframes active {
        0% {
          transform: translate(0, 0);
        }
        100% {
          transform: translate(0, -50%);
        }
      }
      span {
        display: block;
        color: var(--actTextClr);
        transform: translate(0, -30px);
      }
    }
    &:nth-child(1).active ~ .menu-indicator {
      left: 12.5%;
    }
    &:nth-child(2).active ~ .menu-indicator {
      left: 12.5% + 25%;
    }
    &:nth-child(3).active ~ .menu-indicator {
      left: 12.5% + 2 * 25%;
    }
    &:nth-child(4).active ~ .menu-indicator {
      left: 12.5% + 3 * 25%;
    }
  }
  &-icon {
    width: var(--iconSize);
    height: var(--iconSize);
    background-repeat: no-repeat;
    background-size: 40%;
    background-position: 50%;
    margin: 0 auto;
    display: inline-block;
    &.home {
      background-image: url("./imgs/home.svg");
    }
    &.discover {
      background-image: url("./imgs/discover.svg");
    }
    &.search {
      background-image: url("./imgs/search.svg");
    }
    &.my {
      background-image: url("./imgs/my.svg");
    }
  }
  &-indicator {
    width:var(--iconSize);
    height: var(--iconSize);
    border-radius: 50%;
    box-sizing: border-box;
    border: var(--borderW) solid #ffffff;
    background-color: var(--actClr);
    position: absolute;
    transform: translate(-50%, -50%);
    &:before {
      content: '';
      width: 20px;
      height: 20px;
      background: #f0f0f0;
      border-top-right-radius: 20px;
      display: inline-block;
      position: absolute;
      left: -27px;
      bottom: 10px;
      box-shadow: 0 -10px 0 0 #ffffff;
    }
    &:after {
      content: '';
      width: 20px;
      height: 20px;
      background: #f0f0f0;
      border-top-left-radius: 20px;
      display: inline-block;
      position: absolute;
      right: -27px;
      bottom: 10px;
      box-shadow: 0 -10px 0 0 #ffffff;
    }
  }
}
```

