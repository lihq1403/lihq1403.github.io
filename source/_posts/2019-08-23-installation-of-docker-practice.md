---
cover: https://image.lihq.xyz/imgs/2019/12/4e2fb69834b5d87f.png
title: Docker实践之安装篇
date: 2019-08-23 19:53
tags:
  - docker
  - tools
categories:
  - 分享
keywords:
description:
---

[TOC]

> 由于重新搞了虚拟机，所以docker也要重新安装了

## 了解docker

> [百度一下，哈哈哈哈](https://www.baidu.com/s?wd=docker)

## 打开官方文档

- [版本说明](https://blog.csdn.net/yk20091201/article/details/80016135)
- [我们选择：Docker Engine](https://docs.docker.com/install/)
- [Centos7 Docker EE](https://docs.docker.com/install/linux/docker-ce/centos/)


## 安装前

> centos7的安装就很简单啦，添加稳定源去yum配置

安装一些必要的系统工具：
```
# yum install -y yum-utils device-mapper-persistent-data lvm2
```
添加软件源信息：
```
# yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
更新 yum 缓存：
```
# yum makecache fast
```

#### 没有yum-config-manager？
```
# yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

#### 之前装过了，有残留？
```
# yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

## 安装中

#### 直接安装最新的
```
# yum install docker-ce docker-ce-cli containerd.io

略...


```

#### 如果不要最新的，要自定义的话，可以先查看相关版本
```
# yum list docker-ce --showduplicates | sort -r

https://download.docker.com/linux/centos/7/x86_64/stable/repodata/dd13ede13493c87b8ed6c08b710ec1fe318a71fc5f37ae3703890fd37ed1dd43-primary.sqlite.bz2: [Errno 12] Timeout on https://download.docker.com/linux/centos/7/x86_64/stable/repodata/dd13ede13493c87b8ed6c08b710ec1fe318a71fc5f37ae3703890fd37ed1dd43-primary.sqlite.bz2: (28, 'Operation too slow. Less than 1000 bytes/sec transferred the last 30 seconds')
Trying other mirror.
 * updates: mirrors.aliyun.com
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror
 * extras: mirrors.aliyun.com
 * epel: mirrors.tuna.tsinghua.edu.cn
docker-ce.x86_64            3:19.03.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.0-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.7-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.6-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64            18.06.3.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.2.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.3.ce-1.el7                    docker-ce-stable
docker-ce.x86_64            17.03.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos             docker-ce-stable
 * base: mirrors.aliyun.com
Available Packages

```

#### 自由选择

```
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

## 安装后

#### 启动，并开机启动
```
# systemctl start docker && systemctl enable docker
```

#### 测试安装结果
```
# docker version
Client: Docker Engine - Community
 Version:           19.03.1
 API version:       1.40
 Go version:        go1.12.5
 Git commit:        74b1e89
 Built:             Thu Jul 25 21:21:07 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.1
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.5
  Git commit:       74b1e89
  Built:            Thu Jul 25 21:19:36 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.6
  GitCommit:        894b81a4b802e4eb2a91d1ce216b8817763c29fb
 runc:
  Version:          1.0.0-rc8
  GitCommit:        425e105d5a03fabd737a126ad93d62a9eeede87f
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683

```

#### 更换仓库源
```
# vim /etc/docker/daemon.json

我用的是七牛云的加速
{
  "registry-mirrors": ["https://reg-mirror.qiniu.com"]
}

# systemctl restart docker
```

#### 跑一个helloword
```
# docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:451ce787d12369c5df2a32c85e5a03d52cbcef6eb3586dd03075f3034f10adcd
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
