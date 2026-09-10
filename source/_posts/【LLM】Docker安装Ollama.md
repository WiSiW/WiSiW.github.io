---
title: 【LLM】Docker安装Ollama
date: 2026-01-16 09:06:47
tags:
  - Docker
  - LLM
  - Ollama
---

## 一、Docker安装Ollama

### 1.1、官方文档

安装平台哪里都不如官方文档专业，所以看官方文档吧

官方文档地址：[https://github.com/ollama/ollama/blob/main/docs/docker.md](https://link.zhihu.com/?target=https%3A//github.com/ollama/ollama/blob/main/docs/docker.md)

### 1.2、环境信息

由于我使用的是 [Virtualbox](https://zhida.zhihu.com/search?content_id=257285662&content_type=Article&match_order=1&q=Virtualbox&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3Njg2NTAxMDgsInEiOiJWaXJ0dWFsYm94IiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MjU3Mjg1NjYyLCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.Q__-_DzCb26fYmPcvUTBTWCn0SYwMevPs4kAKznrCSc&zhida_source=entity) 虚拟机，安装的 [Centos7](https://zhida.zhihu.com/search?content_id=257285662&content_type=Article&match_order=1&q=Centos7&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3Njg2NTAxMDgsInEiOiJDZW50b3M3IiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MjU3Mjg1NjYyLCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.z4behRXMbpWe-KQ6-9x1yLRYSh3TSdII1LWUwOkWKxQ&zhida_source=entity)

我查了文档，这个虚拟机无法使用 Nvidia GPU

所以暂且使用 CPU 安装吧

如果后续性能比较差，再使用 GPU

### 1.3、Docker安装

### 1.3.1、检查 Docker 状态

安装之前查看docker 是否启动

```bash
systemctl status docker
```



![img](https://pic1.zhimg.com/v2-d09f3ea7100e981e669f69f97e8c21d8_1440w.jpg)



如果没启动，执行启动命令

```bash
systemctl start docker
```

### 1.3.2、安装 Ollama

执行安装命令 （CPU Only）

```bash
docker run -d -v /qjp/software/ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

参数说明：

### 1. `docker run`

- **作用**：启动一个新的容器实例。
- **说明**：Docker 的核心命令，用于从镜像创建并启动容器。

### 2. `-d`（`--detach`）

- **作用**：以 **后台模式** 运行容器。
- **说明**：容器启动后会与当前终端分离，终端不会被阻塞，可以继续执行其他命令。
- **典型场景**：适用于需要长期运行的服务（如 Web 服务器、数据库等）。

### 3. `-v /qjp/software/ollama:/root/.ollama`（`--volume`）

- **作用**：挂载 **数据卷**，实现宿主机与容器之间的目录映射。
- **参数结构**：`宿主机目录:容器内目录`
- **说明**：
- 将宿主机的 `/qjp/software/ollama` 目录挂载到容器内的 `/root/.ollama` 目录。
- 容器内的 `/root/.ollama` 通常是 Ollama 的配置和数据存储目录，挂载后数据会持久化到宿主机，避免容器删除后数据丢失。
- **典型场景**：用于保存模型文件、配置文件或日志等持久化数据。

### 4. `-p 11434:11434`（`--publish`）

- **作用**：映射 **容器端口** 到 **宿主机端口**。
- **参数结构**：`宿主机端口:容器内端口`
- **说明**：
- 将宿主机的 `11434` 端口映射到容器的 `11434` 端口。
- 外部通过宿主机的 `11434` 端口访问容器内的服务。
- **典型场景**：Ollama 的 API 或 Web 服务可能默认监听 `11434` 端口，通过此映射允许外部访问。

### 5. `--name ollama`

- **作用**：为容器指定一个 **自定义名称**（这里是 `ollama`）。
- **说明**：
- 如果不指定名称，Docker 会自动生成一个随机名称（如 `angry_curie`）。
- 自定义名称便于后续通过 `docker start/stop/rm` 等命令管理容器。
- **典型场景**：简化容器管理，避免依赖容器 ID。

### 6. `ollama/ollama`

- **作用**：指定要使用的 **Docker 镜像**。
- **说明**：
- 如果本地不存在该镜像，Docker 会从 Docker Hub 自动拉取。
- 未指定标签时，默认使用 `latest` 标签。
- 镜像的完整格式为 `镜像名:标签`（如 `ollama/ollama:0.1`）。
- **典型场景**：使用官方或自定义镜像启动服务。

### 1.3.3、 Ollama 安装结果

### 1.3.3.1、执行结果

如下图所示，则安装成功



![img](https://pica.zhimg.com/v2-5f69801c7f978c75738643a8fffa777c_1440w.jpg)



Docker 的ID是

```bash
e92591edf0bc53602e594c956dba19431d3d7de342af5ffb3b15b57b13280036
```

### 1.3.3.2、运行验证

页面访问：[http://xxx.xxx.xx.xx:11434/](https://link.zhihu.com/?target=http%3A//xxx.xxx.xx.xx%3A11434/)

显示 如下图，则 Ollama 安装成功，并正在运行



![img](https://pic4.zhimg.com/v2-739395340e9a8caf4c1200192171ebfd_1440w.jpg)



## 二、Ollama部署大模型

### 2.1、 部署codellama:7b

大模型可根据自己的需求去官网上去找

官网地址：[https://ollama.com/search](https://link.zhihu.com/?target=https%3A//ollama.com/search)

### 2.1.1、进入 Docker 容器

```bash
docker exec -it ollama bash
```

### 2.1.2、安装大模型

执行运行大模型，没有的话，会自动安装

```bash
ollama run codellama:7b
```

如下图所示则安装成功，且进入了大模型问答环境，可以对话提问



![img](https://picx.zhimg.com/v2-7261fa28e3e987ce48eab6ab9cd487fd_1440w.jpg)



### 2.1.3、退出 Docker 容器

```bash
CTRL + D
```

### 2.2、查看移除大模型

### 2.2.1、查看大模型

```bash
ollama list
```

### 2.2.2、移除大模型

```bash
ollama rm codellama:7b
```



转载自

[权先生的技术空间](https://www.zhihu.com/people/pengpenhhh)

[Docker安装Ollama及使用Ollama部署大模型](https://zhuanlan.zhihu.com/p/1902057589019251082)
