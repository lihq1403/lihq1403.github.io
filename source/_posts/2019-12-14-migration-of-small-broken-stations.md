---
cover: https://blog-1256184194.file.myqcloud.com/2019/12/23/48e6101ba63f8.jpg
title: 小破站迁移辛酸史
date: 2019-12-14 21:53
tags:
  - web
categories:
  - 分享
keywords:
description:
---

# 小破站迁移辛酸史

> 还不是想体验一把漂亮的主题~~（是因为穷）~~，哈哈哈

## 初始方案

##### 用到的工具
- Typecho
- Vps


一开始这样也确实不错，但是太平常了-.-，```PHP```+```MYSQL```+```NGINX```一把梭

## 现在的方案

##### 用到的工具
- Hexo + Butterfly
- Travis CI
- GitHub Pages + Vps

其实现在这样也是内容多多，但是也学到了很多东西呀，主要是用到了自动部署方案，以后只需要专注于写作就ok了

## 踩坑记录

1. 一开始我是用 ```GitHub Pages``` 和 ```Coding Pages``` 来部署双节点，用了一天之后，CodingPage就突然发问不了了，Bu和Go了一下，发现CodingPage就突然发问不了了好像确实不咋地，服务不稳定啊，那没办法了，还是部署一套在我的小水管上面吧
2. 我的[.travis.yml](https://github.com/lihq1403/lihq1403.github.io/blob/hexo/.travis.yml)文件给大家参考一下，主要是推送到```github```和```coding```上面，一开始是这方案，后面coding也就没有用了，但是也没删，就放着当备份呗
3. Vps上面的代码同步采用了WebHook来进行同步
4. 使用 Git Subtree 管理主题版本，这样的话，以后切换主题妈妈就再也不用担心了
5. 在域名服务商那边记得填写两个域名解析哦，一个是去```GitHub Pages```，一个是去自己的Vps
6. ```GitHub Pages```自定义域名记得在source下面放CNAME文件，不然好像过一会自定义域名就会掉
7. ```hexo d```了之后，代码确实有push到master上面，但是里面没有内容，没办法，我就只能在[.travis.yml](https://github.com/lihq1403/lihq1403.github.io/blob/hexo/.travis.yml)里面用上了手动推送public目录
8. 文章里面有很多部署细节我都没说，因为这些[G](https://www.google.com/)一下或者[B](https://www.baidu.com/)一下都可以获取的到哦，主要是提供一个思路就好啦


## 附上我的双节点

> 果然很完美，我的小水管挺住

![](https://blog-1256184194.file.myqcloud.com/2019/12/23/c949b4c8c0542.png)

## 博客

> 确实漂亮，没的说

![](https://blog-1256184194.file.myqcloud.com/2019/12/23/ec8fa1b1f6ee4.png)