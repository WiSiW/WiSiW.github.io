---
title: 【JavaScript】forEach、map、reduce、filter的区别
date: 2021-08-30 11:43:43
tags:
    - JavaScript
cover_picture: /images/javascript.png
---

|  |  |  |
| ------- | ---------------------------- | ------------------------------------------------------------ |
| forEach | 循环数组                     | return只能打断当前循环中的执行的代码，不能打断循环本身；不支持continue 与 break |
| map     | 循环数组，返回新数组         | return会改变返回数组                                         |
| reduce  | 循环数组，依次相加           |                                                              |
| filter  | 循环数组，根据条件返回新数组 |                                                              |
| some    |                              |                                                              |
| every   |                              |                                                              |

