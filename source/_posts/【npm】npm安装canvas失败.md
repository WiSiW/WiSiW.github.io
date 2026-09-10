---
title: '【npm】npm安装canvas失败'
date: 2024-03-07 00:16:36
tags:
---

canvas的安装过程
npm下载在仓库中的canvas包
执行canvas的package.json中的install命令(node-pre-gyp install --fallback-to-build)
node-pre-gyp下载canvas已编译好的 macOS、Linux 和 Windows 的二进制文件，如果预构建的二进制文件不存在或不可用，则回退到从源代码构建
node-gyp编译为当前平台可用的node模块
为什么安装慢，还容易失败
从安装过程可以发现, 步骤3和步骤4是核心问题区, canvas的二进制文件托管在https://github.com/Automattic/node-canvas/releases/download/ ，下载服务由https://objects.githubusercontent.com 提供，如果你的网络运营商给力，你将会大概率成功，否则你将会得到read ECONNRESET的提示。而使用源码构建非常繁琐，对系统os存在诸多限制，开发环境和运行环境的系统os不一致也会大大增加我们的工作量且导致一些不可预料且难以捉摸的问题。

如何解决canvas安装过程中的慢、失败问题
既然github相关的服务在国内速度不理想，我们自然可以使用国内镜像的方式加速。那我们如何对canvas的二进制文件指定下载镜像呢? 我们需要给npm提供config参数--{module_name}_binary_host_mirror可以通过镜像下载二进制文件

```shell
npm install canvas --canvas_binary_host_mirror=https://registry.npmmirror.com/-/binary/canvas
```

当然我们不希望在命令行中添加参数，因为它不便于维护。我们更希望能在配置文件中固化，我们可以在.npmrc(npm的配置文件)中设置一个或多个模块的二进制文件下载镜像地址，一个好的长期维护的.npmrc文件可以让我们的任何项目安装过程更加轻松愉悦。

```
registry=https://registry.npmmirror.com

disturl=https://registry.npmmirror.com/-/binary/node/
# node-sass预编译二进制文件下载地址
sass_binary_site=https://registry.npmmirror.com/-/binary/node-sass
# sharp预编译共享库, 截止2022-09-20 sharp@0.31.0的预编译共享库并未同步到镜像, 入安装失败可切换到sharp@0.30.7使用
sharp_libvips_binary_host=https://registry.npmmirror.com/-/binary/sharp-libvips
python_mirror=https://registry.npmmirror.com/-/binary/python/
electron_mirror=https://registry.npmmirror.com/-/binary/electron/
electron_builder_binaries_mirror=https://registry.npmmirror.com/-/binary/electron-builder-binaries/
# 无特殊配置参考{pkg-name}_binary_host_mirror={mirror}
canvas_binary_host_mirror=https://registry.npmmirror.com/-/binary/canvas
node_sqlite3_binary_host_mirror=https://registry.npmmirror.com/-/binary/sqlite3
better_sqlite3_binary_host_mirror=https://registry.npmmirror.com/-/binary/better-sqlite3
```