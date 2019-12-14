---
cover: https://image.lihq.xyz/imgs/2019/12/83cd646b0527f634.jpg
title: Docker实践之应用篇 - MailHog
date: 2019-10-12 10:18
tags:
  - docker
  - tools
categories:
  - 分享
keywords:
description:
---
# Docker实践之应用篇 - MailHog


## MailHog 介绍

![](https://image.lihq.xyz/imgs/2019/10/901b69f9be24d19d.png)

> 引用[mailhog/MailHog](https://github.com/mailhog/MailHog)的介绍：Web and API based SMTP testing

即 本地开发测试的邮件服务，它提供了一个 Web 界面，可以检查应用发送的邮件，运行 MailHog 最简单的方法是用 Docker

## 运行应用
```
docker run --name mailhog -p 1025:1025 -p 8025:8025 -d mailhog/mailhog
```
其中1025 是发邮件用的端口，8025 是 Web 界面用的端口

运行之后看到下面的结果就代表成功了，是不是很简单 -.-!
![](https://image.lihq.xyz/imgs/2019/10/1e16102c65066934.png)

## 查看web页面
因为我是虚拟机环境，配置了虚拟网络，所以访问地址是http://192.168.56.10:8025

访问ip地址要根据自己的环境来切换哦，别照搬

![](https://image.lihq.xyz/imgs/2019/10/924ab46c3c5d3ba8.png)

## 测试一下

我这里使用了[mailhog/mhsendmail](https://github.com/mailhog/mhsendmail/)模拟了邮件发送

安装也是很简单的
```
yum install go
go get github.com/mailhog/mhsendmail
ln  ~/go/bin/mhsendmail /usr/bin/mhsendmail
mhsendmail -h
```
测试发送
```
mhsendmail --smtp-addr="127.0.0.1:1025" test@mailhog.local <<EOF
From: App <app@mailhog.local>
To: Test <test@mailhog.local>
Subject: Test message

Some content!
EOF
```

![](https://image.lihq.xyz/imgs/2019/10/211367dfa26d7766.png)

![](https://image.lihq.xyz/imgs/2019/10/f10267917917547b.png)

可以通过web页面看到我们已经发送成功了

## 项目中

配置应用的 SMTP 邮件功能的时候，邮件服务主机填写 localhost，端口号是 1025。这样应用发送的邮件都会被 mailhog 接收到，你在它提供的 Web 界面可以检查邮件内容。

如在laravel中时，.env配置如下
```
MAIL_DRIVER=smtp
MAIL_HOST=127.0.0.1
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

## 停止服务
```
docker stop mailhog
```