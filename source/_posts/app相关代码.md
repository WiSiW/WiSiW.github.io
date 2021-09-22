---
title: app相关代码
date: 2021-09-13 11:14:23
tags:
    - app
---



**跳转到应用商店:**

```
(https)|(itms-apps)://itunes.apple.com/app/id{appID}
```

**跳转到撰写评价：**

```
(https)|(itms-apps)://itunes.apple.com/app/id{appID}?action=write-review
```


**跳转到查看评价：**

```
(https)|(itms-apps)://itunes.apple.com/app/viewContentsUserReviews?id={appID}
```

示例：

```javascript
window.location.href = 'itms-apps://itunes.apple.com/app/id414478124?action=write-review'
```



