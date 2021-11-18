---
title: 【架构】前端工程化（三）：git分支管理
date: 2021-11-12 10:51:48
tags:
    - git
    - 前端工程化
    - 分支管理

cover_picture: /images/git.png
---

![](/images/git_branch_manage.png)

**注意：**

1.  负责修复线上紧急Bug的```hotfix```分支在开发完成后必须合并到当前正在开发的```develop```分支，否则会造成下次版本发布后丢失```hotfix```的修复代码
2.  ```release```分支在测试阶段可能会有频繁的修复Bug行为，如果在此过程中同时进行下一个版本的迭代，必须在修复Bug之前将```release```分支合并到```develop```分支，否则会引起下一版本发布后上一版本Bug再次出现



参考来源：

前端工程化：体系设计与实践:http://product.dangdang.com/25204506.html

