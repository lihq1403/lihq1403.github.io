---
copyright: false
cover: https://blog-1256184194.file.myqcloud.com/2020/01/03/adf9516baec2e.jpg
title: 【转载】Composer 图文安装教程
date: 2020-01-03 13:58 
tags:
  - php
  - tools
categories:
  - 转载
---

# Composer 图文安装教程

# 原文地址：
- [傻瓜都会的 Composer 图文安装教程
](https://learnku.com/articles/38982)
- [Composer 国内全量镜像大全
](https://learnku.com/articles/30258)


>Composer 不是一个包管理器，不同于python的pi,nodejs的npm，它是 PHP 用来管理依赖（dependency）关系的工具。你可以在自己的项目中声明所依赖的外部工具库（libraries），Composer 会帮你安装这些依赖的库文件。
为什么要是用composer呢，或者它有哪些好处呢？它使代码模块化，提高代码的复用性，另外还提供自动加载等。

### 安装
运行 Composer 需要 PHP 5.3.2+ 以上版本，composer 支持windows、linux等多平台。

#### linux上安装
1、执行php -v 查看PHP版本

![l1HWQA](https://cdn.learnku.com/uploads/images/201912/31/21543/sbqZsYqcx3.png!large)

2、执行以下命令进行全局安装
```php
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

![l1H6iD](https://cdn.learnku.com/uploads/images/201912/31/21543/frQArxJQWr.png!large)

>`composer.phar` 是 Composer 的二进制文件。这是一个 PHAR 包（PHP 的归档），这是 PHP 的归档格式可以帮助用户在命令行中执行一些操作

3、查看是否安装成功
```php
//三选一
composer
composer -V
composer --version
```
![l1HHJg](https://cdn.learnku.com/uploads/images/201912/31/21543/fbcukm1BYX.png!large)

4、全局修改`composer`镜像
```php
//设置中国全量镜像，推荐使用阿里云镜像
composer config -g repo.packagist composer https://packagist.phpcomposer.com 
//查看配置是否成功
composer config -gl
```

![l1OY01](https://cdn.learnku.com/uploads/images/201912/31/21543/tjo0TxuitJ.png!large)

5、当某些不可抗因素导致原镜像不能正常使用的时候，切换composer镜像
```php
//1、切换源
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/ 
//2、清除所有 package 缓存（此步奏选泽性操作）
composer clear-cache
//3、查看配置是否成功
composer config -gl
```
![l1XUDs](https://cdn.learnku.com/uploads/images/201912/31/21543/MKpKL7mgHw.png!large)

6、如果需要解除镜像并恢复到 packagist 官方源
```php
composer config -g --unset repos.packagist
```

![l1vFSO](https://cdn.learnku.com/uploads/images/201912/31/21543/39imVJCkDE.png!large)

7、composer升级 
```php
composer self-update
```

![l1Hxe0](https://cdn.learnku.com/uploads/images/201912/31/21543/ccu84I4Z0d.png!large)

中国全量镜像站内说如果版本更新不成功需要重新下载包。
![l1HOQs](https://cdn.learnku.com/uploads/images/201912/31/21543/6ZQ9eJYcuk.png!large)

**注意：一般正常擦做到上述第4步就可以**

#### windos上安装composer

windows上安装composer比较方便，在[composer中文网](https://docs.phpcomposer.com/00-intro.html)直接下载exe文件进行一路确定安装即可，剩余的操作与linux上的操作无差别。

![l1zdQU](https://cdn.learnku.com/uploads/images/201912/31/21543/DiiwNaEMme.png!large)

# Composer 国内全量镜像大全

### 镜像使用
```
$ composer config -g repo.packagist composer  镜像地址
$ composer clearcache
$ composer update || install
```
说明：若项目之前已通过其他源安装，可以删除 composer.lock 以及 vendor 目录，重新生成。

### 关闭镜像
```
$ composer config -g --unset repos.packagist
```

### 阿里云 Composer 全量镜像
镜像地址：[https://mirrors.aliyun.com/composer/](https://mirrors.aliyun.com/composer/)
官方地址：[https://mirrors.aliyun.com/composer/index.html](https://mirrors.aliyun.com/composer/index.html)
说明：终于接上大厂水管了，还没来得急测，先更新，估计阿里云做的也不会差。

### 腾讯云 Composer 全量镜像
镜像地址：[https://mirrors.cloud.tencent.com/composer/](https://mirrors.cloud.tencent.com/composer/)
官方地址：[https://mirrors.cloud.tencent.com/composer](https://mirrors.cloud.tencent.com/composer)
说明：若您使用腾讯云服务器，可以将源的域名从 mirrors.cloud.tencent.com 改为 mirrors.tencentyun.com，使用内网流量不占用公网流量，是不是非常良心。

### 华为 Composer 全量镜像
镜像地址：[https://mirrors.huaweicloud.com/repository/php/](https://mirrors.huaweicloud.com/repository/php/)
官方地址：[https://mirrors.huaweicloud.com/](https://mirrors.huaweicloud.com/)
说明：华为 composer 镜像目前还不够完善，composer i 时会出现一些 bug ，而且同步速度也比较慢，好像并非是全量的。
 
### Packagist / Composer 中国全量镜像
镜像地址：[https://packagist.phpcomposer.com](https://packagist.phpcomposer.com)
官方地址：[https://pkg.phpcomposer.com/](https://pkg.phpcomposer.com/)
说明：Packagist 中国全量镜像是从 2014 年 9 月上线的，在安装和同步方面都比较完善，也一直是公益运营，但不知道目前这个镜像是否还是可用状态。
 
### Composer / Packagist 中国全量镜像
镜像地址：[https://php.cnpkg.org](https://php.cnpkg.org)
官方地址：[https://php.cnpkg.org/](https://php.cnpkg.org/)
说明：此 composer 镜像由安畅网络赞助，目前支持元数据、下载包全量代理，还是不错的，推荐使用。

### Packagist / JP
镜像地址：[https://packagist.jp](https://packagist.jp)
官方地址：[https://packagist.jp](https://packagist.jp)
说明：这是日本开发者搭建的 composer 镜像，早上测了一下，感觉速度还不错。

### Packagist Mirror
镜像地址：[https://packagist.mirrors.sjtug.sjtu.edu.cn](https://packagist.mirrors.sjtug.sjtu.edu.cn)
官方地址：[https://mirrors.sjtug.sjtu.edu.cn/packagist/](https://mirrors.sjtug.sjtu.edu.cn/packagist/)
说明：上海交通大学提供的 composer 镜像，稳定、快速、现代的镜像服务，推荐使用。

### Laravel China Composer 全量镜像
镜像地址：[https://packagist.laravel-china.org](https://packagist.laravel-china.org)
官方地址：[https://learnku.com/laravel](https://learnku.com/laravel)
说明：这个就不多了，国内 PHP 开发者使用量最多的 composer 镜像，同步速度快、稳定，推荐使用。