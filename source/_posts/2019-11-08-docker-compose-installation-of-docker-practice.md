---
copyright: true
cover: https://blog-1256184194.file.myqcloud.com/2019/12/23/7809429dc611c.jpg
title: Docker实践之Docker-Compose安装
date: 2019-11-08 15:52
tags:
  - docker
categories:
  - 分享
keywords:
description:
---

# Docker实践之Docker-Compose安装

###  什么是docker-compose

- 是一个用于运行和管理多个容器化应用的工具
- [官方介绍](https://docs.docker.com/compose/)

### 安装

- [官方教程](https://docs.docker.com/compose/install/)

> 我这里的环境是centos7，要安装自己的实际情况进行安装哦，ps：根据网络情况可能会下载的很慢，大家可以扶墙之后去[https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)上面寻找适合自己环境的包直接下载

```
# sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# sudo chmod +x /usr/local/bin/docker-compose
# sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

命令讲解：其实就是去github上面下载一个docker-compose当前主机的发行版本，二进制文件，并赋予可执行权限，环境变量的话，就要实际情况了，可以进行修改路径，或者进行ln -s创建软链接进行快速访问

### 测试可用
```
# docker-compose --version
docker-compose version 1.24.1, build 4667896b
docker-py version: 3.7.3
CPython version: 3.6.8
OpenSSL version: OpenSSL 1.1.0j  20 Nov 2018
```