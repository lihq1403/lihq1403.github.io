---
copyright: true
cover: https://media-1256184194.file.myqcloud.com/image/bing/2020-05-07-5eb3d1968cd06.jpg
title: PHP 扩展包开发流程
date: 2020-05-09 17:51 
tags:
  - php
  - tools
categories:
  - 分享
---

# PHP 扩展包开发流程

> 现在程序开发，应该很少人再去造轮子吧，直接github一顿搜，哈哈哈哈。
难免有时候会找不到或者不适合，刚好又碰巧这些代码很通用，那么我们不妨自己开发一个轮子 ( 手动狗头 0-0 )

### composer安装

- [Composer 图文安装教程](https://blog.lihq.xyz/2020/01/03/composer-graphic-installation-tutorial/tps://note.youdao.com/)


### 拓展包的基础结构
虽然说扩展包并没有什么强制的规定一定要如何组织代码，但是我们推荐根据业界约定俗成的结构：
```
helper/
├── .editorconfig      # 编辑器配置文件，比如缩进大小、换行模式等
├── .gitattributes     # git 配置文件，可以设计导出时忽略文件等
├── .gitignore         # git 忽略文件配置列表
├── .php_cs            # PHP-CS-Fixer 配置文件
├── README.md
├── composer.json      # 包定义，很关键
├── phpunit.xml.dist
├── src                # 源码
│   └── .gitkeep
└── tests              # 单元测试
    └── .gitkeep
```

### 包构建工具

我这里使用的是[overtrue/package-builder](https://github.com/overtrue/package-builder)提供的一个包结构生成工具
```
$ composer global require "overtrue/package-builder" --prefer-source
```

##### 基本用法

```
 $ package-builder help
如果命令未找到，那就在composer全局vendor里面啦！这个是我的：/root/.config/composer/vendor/overtrue/package-builder/bin/package-builder
```


### 创建项目

```
$ package-builder build helper
```

### 声明自动加载

> tips：一般自动加载有改动的话，最好重新生成一次composer dump-autoload 或者 composer du 
```
{
    .
    .
    .
    "autoload": {
        "psr-4": {
            "Lihq1403\\Helper\\": "src"
        },
    },
    .
    .
    .
}
```


### 完成代码开发，单元测试

- [lihq1403/helper](https://github.com/lihq1403/helper)

最后的composer.json如下：

```
{
    "name": "lihq1403\/helper",
    "description": "helper",
    "license": "MIT",
    "authors": [
        {
            "name": "lihq1403",
            "email": "lihqing1403@gmail.com"
        }
    ],
    "require": {
        "php": ">=7.0.0",
        "ext-json": "*",
        "larapack/dd": "^1.1",
        "phpoffice/phpspreadsheet": "^1.10",
        "ext-iconv": "*"
    },
    "require-dev": {
        "phpunit/phpunit": "^8.3",
        "mockery/mockery": "^1.2"
    },
    "autoload": {
        "psr-4": {
            "Lihq1403\\Helper\\": "src"
        },
        "files": [
            "src/common.php"
        ]
    }
}

```


### 本地项目测试扩展包

建立测试项目，目录结构如下
```
dir/
├── helper           # 扩展包
└── helper-test      # 测试项目
```

进入测试目录之后
```
# 需要先初始化 composer.json, 一路回车即可
$ composer init  

# 配置包路径，注意，这里 `../helper` 为相对路径，不要弄错了
$ composer config repositories.helper path ../helper    

# 安装扩展包  这里  `dev-master`  中的 dev 指该分支下最新的提交，master 是指定的包中的分支名
$ composer require lihq1403/helper:dev-master
```

> 小细节：本地composer安装，会创建一个软链接 vendor/lihq1403/helper 到包所在目录 ../helper，这样一来，你可以直接在测试项目的 vendor/lihq1403/helper 下修改文件，包里的文件也会跟着变了

建立测试文件 index.php

```
<?php

require __DIR__ . '/vendor/autoload.php';

use Lihq1403\Helper\Interfaces\DateGlobal;


echo DateGlobal::SECONDS_IN_A_DAY;
```

测试一下：


```
$ php index.php
86400
```
达到预期结果，完美！！！

### 发布上线

1. 将本地代码放到github上面 ~~（不会真有人不会提交代码到github吧）~~

![微信截图_20200509173401.png](https://blog-1256184194.file.myqcloud.com/2020/05/09/17a7a4f2afc61.png)

2. 提交到 Packagist

![微信截图_20200509173554.png](https://blog-1256184194.file.myqcloud.com/2020/05/09/c19fd205d7077.png)

3. 启用项目的 Packagist 通知服务

访问你在 Packagist 的个人主页：packagist.org/profile/ ，点击 "Show API Token"，复制 token 备用。

4. 给项目代码库启用 Packagist 通知服务
填写对应的内容：
- Payload URL: https://packagist.org/api/github?username=Packagist 的用户名
- Content type 选择为 application/json
- Secret 填写为您刚刚复制的 token
![微信截图_20200509173816.png](https://blog-1256184194.file.myqcloud.com/2020/05/09/2953629f1341d.png)

### 发布第一个版本

##### 版本号约定

```
版本格式：主版本号。次版本号。修订号，版本号递增规则如下：
主版本号：当你做了不兼容的 API 修改，
次版本号：当你做了向下兼容的功能性新增，
修订号：当你做了向下兼容的问题修正。
先行版本号及版本编译信息可以加到 “主版本号。次版本号。修订号” 的后面，作为延伸。
```

简单介绍就是，如果你现在的最新版本是 1.0.0，下面的动作的区别是：

- 打补丁，修了一些小 bug，没做 API 修改，那么你应该发布 1.0.1，同理以后也是递增第三位。
- 有一天网友在你的基础上提交了新功能，原来的 API 调用方式也没改变，这时候你应该发布 1.1.0 。
- 一段时间以后，你心血来潮重构了你的扩展包，调用方式也发生了变化，也就是说安装了以前版本的是无法直接升级的，这时候你需要发布 2.0.0 了。


##### Create new release

填写版本号、这次发版的标题、以及这次版本变化的内容描述，点击提交。


### 发布成功之后的测试
在新项目里面直接composer require lihq1403/helper，问题不大的话（国内镜像会有延迟），应该能看到安装成功


### 备注

如果这个开发的包，是一些比较私有的话，建议自己搭建一个私有的 Composer 包仓库
- [使用 satis 搭建一个私有的 Composer 包仓库](https://blog.lihq.xyz/2019/12/25/Using-satis-to-build-a-private-composer-package-warehouse/)

### helper源码地址
{% githubCard user:lihq1403 repo:helper width:100% height:200 theme:default %}

![image](https://media-1256184194.file.myqcloud.com/image/bing/2020-05-07-5eb3d1968cd06.jpg)
