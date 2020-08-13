---
copyright: true
cover: https://blog-1256184194.file.myqcloud.com/2019/12/23/d0111cb9fbc25.jpg
title: Docker 常用命令
date: 2019-10-11 09:35
tags:
  - docker
categories:
  - 分享
keywords:
description:
---

# Docker 常用命令

## 查看

```
docker images           # 列出所有镜像(images)
docker ps               # 列出正在运行的容器(containers)
docker ps -a            # 列出所有的容器
docker pull centos      # 下载centos镜像
docker top <container>  # 查看容器内部运行程序
```

## 容器
```
docker stop <container>                  # 停止一个正在运行的容器，<container>可以是容器ID或名称
docker start <container>                 # 启动一个已经停止的容器
docker restart <container>               # 重启容器
docker rm <container>                    # 删除容器

docker run -i -t -p :80 LAMP /bin/bash   # 运行容器并做http端口转发
docker exec -it <container> /bin/bash    # 进入ubuntu类容器的bash
docker exec -it <container> /bin/sh      # 进入alpine类容器的sh

docker rm `docker ps -a -q`              # 删除所有已经停止的容器
docker kill $(docker ps -a -q)           # 杀死所有正在运行的容器，$()功能同``
```

## 提交/导出
```
docker build --rm=true -t hjue/lamp .    # 建立映像文件。–rm 选项是告诉Docker，在构建完成后删除临时的Container，Dockerfile的每一行指令都会创建一个临时的Container，一般这些临时生成的Container是不需要的
docker commit 3a09b2588478 mynewimage    # 提交你的变更，并且把容器保存成镜像，命名为mynewimage，3a09b2588478为容器的ID

docker save mynewimage | bzip2 -9 -c> /home/save.tar.bz2  # 把 mynewimage 镜像保存成 tar 文件
bzip2 -d -c < /home/save.tar.bz2 | docker load            # 加载 mynewimage 镜像

docker export <CONTAINER ID> > /home/export.tar           # 导出Image
cat /home/export.tar | sudo docker import - mynewimage    # 导入Image镜像
```

## 镜像
```
docker run -i -t centos /bin/bash          # 运行centos镜像
docker run -d -p 80:80 hjue/centos-lamp    # 运行centos-lamp镜像

docker rmi [image-id]                      # 删除镜像
docker rmi $(docker images -q)             # 删除所有镜像
```

## 帮助
```
docker run --help
```

## 批量操作
```
docker stop $(docker ps -a -q)                                         # 停止所有
docker kill $(docker ps -a -q)                                         # 杀死所有正在运行的容器
docker rm $(docker ps -a -q)                                           # 删除所有已经停止的容器
docker rmi $(docker images -q -f dangling=true)                        # 删除所有未打 dangling 标签的镜像
docker rmi $(docker images -q)                                         # 删除所有镜像
docker rmi --force $(docker images | grep doss-api | awk '{print $3}') # 强制删除镜像名称中包含“doss-api”的镜像
```