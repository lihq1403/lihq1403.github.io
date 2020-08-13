---
copyright: true
cover: https://blog-1256184194.file.myqcloud.com/2019/12/25/7e3c3530e1a49.jpg
title: 使用 satis 搭建一个私有的 Composer 包仓库
date: 2019-12-25 23:53
tags:
  - tools
categories:
  - 分享
keywords:
description:
---

# 使用 satis 搭建一个私有的 Composer 包仓库

#### 搭建缘由
因为国内使用composer的时候，基本上都是用的镜像源，在开发包的时候，完成之后需要进行安装，就会出现一个同步时间差问题，等不及了，就可以先用自己的私有库顶住。当然，有些不公开的包就更需要一个私有的composer仓库了

说动手就动手，看了[官方文档](https://docs.phpcomposer.com/05-repositories.html#Hosting-your-own)，提供了好几种自建方法，我这里只是选择了[satis](https://docs.phpcomposer.com/articles/handling-private-packages-with-satis.html)，因为比较简单，哈哈哈

## 源码安装

### 安装
```shell
composer create-project composer/satis --stability=dev --keep-vcs
```
安装完成，会出现一个satis目录

### 配置
satis的配置是通过satis.json进行的，我们在当前目录新建一个satis.json
```json
{
  "name": "Lihq Private Composer Repository", 
  "homepage": "http://packagist.test",
  "repositories": [
    { "type": "git", "url": "git@github.com:lihq1403/think-rbac.git" }
  ],
  "require":{
       "lihq1403/think-rbac":"*"
   },
  "archive": {
        "directory": "dist",
        "format": "tar",
        "prefix-url": "http://packagist.test"
    }
}
```
###### 参数说明
- name : 仓库的名字，自己定义
- homepage ：仓库的主页地址
- repositories ：需要获取包的路径
- requre ： 指定获取哪些包（require-all:true 代表获取所有包）

###### archive ：缓存文件
> 我们并不希望每次都clone，其实我们也可以缓存在我们的仓库中，这样每次update的时候就只用下载了

- directory: 必需要的，表示生成的压缩包存放的目录，会在我们build时的目录中
- format: 压缩包格式, zip（默认） tar
- prefix-url: 下载链接的前缀的Url,默认会从homepage中取
- skip-dev: 默认为假，是否跳过开发分支
- absolute-directory: 绝对目录
- whitelist: 白名单，只下载哪些
- blacklist: 黑名单，不下载哪些
- checksum: 可选，是否验证sha1

### 生成
```shell
php bin/satis build satis.json public/
```
执行命令，会生成包的缓存与web应用

### web访问

配置nginx指向刚刚生成的public就好了

![微信截图_20191225232725.png](https://blog-1256184194.file.myqcloud.com/2019/12/25/a229bd967e245.png)

然后打开上面配置的homepage，看到如下就代表成功啦
![微信截图_20191225232930.png](https://blog-1256184194.file.myqcloud.com/2019/12/25/a336af9832fbf.png)

### 使用
我们只需要在项目中，添加本源即可
```json
{
  "repositories": [{
    "type": "composer",
    "url": "http://packagist.test"
  }]
}
```

然后就可以开开心心的```composer update```了

## docker安装

### 镜像拉取
```shell
docker pull composer/satis
```

> ps 吐槽。pull的也太慢了吧，以后再尝试了，todo


## satis.json详细说明
```json
{
  "name": "MY SATIS",//项目名称
  "homepage": "http://satis.example.work",//项目地址
  "repositories": [//指定获取包的仓库地址
    {
      "type": "vcs",
      "url": "git@gitlab.example.cn:xx/test.git"
    },
    {
      "type": "vcs",
      "url": "git@gitlab.example.cn:xx/weibo.git"
    },
   {
// 不再拉取packagist.org上的包，可节省大量时间
     "packagist.org": false
    },
  ],
  "require": {//指定可以获取包的版本，存在"require-all": true设置时将从仓库获取所有相关的依赖包，所以一般不设置。使用"require-all": true时就不能再从packagist.org上拉取包了，不然satis将会把所有packagist.org上的包都拉取下来
    "xx/test": "*",//尝试从以上两个配置好的仓库与管理员的全局配置中的仓库中拉取
    "laravel/laravel": "5.6.21",//由于以上两个仓库中没有该项目，所以将从管理员的全局配置文件中使用的仓库地址中拉取
    "xx/weibo": "1.5.0|1.0.3"//"*"为全部版本，"1.5.0|1.0.3"为1.5.0与1.0.3两个版本
  },
  "archive": {//镜像缓存设置，该设置会缓存require配置项中各个仓库的代码
    "directory": "dist",//缓存目录名称
    "format": "tar",//缓存格式
    "prefix-url": "http://satis.example.work",//下载的前缀不要写成http://satis.example.work/不然链接将会出问题
    "skip-dev": false//是否跳过开发版本，一般为true，但我为了方便测试就选择了false
// 其他配置项：absolute-directory: 绝对目录
// whitelist: 白名单，只下载哪些
// blacklist: 黑名单，不下载哪些
// checksum: 可选，是否验证sha1
  },
  "config": {//拉取代码时的配置
    "secure-http":false,//由于公司部分项目的存放地址可能使用http链接，所以此处设置为false
    "github-oauth": {//部分放在github中的项目需要提前配置好token避免拉取时再输入
        "github.com": "83dxxxxxxxxxxx8888888800062"
    },
    "bitbucket-oauth": {},
    "gitlab-oauth": {},
    "gitlab-token": {
        "gitlab.example.cn":"zxxxxxxxxxxxQjZT"//公司gitlab的token配置
    },
    "http-basic": {}
  }
}
```