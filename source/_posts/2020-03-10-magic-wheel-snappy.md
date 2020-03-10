---
copyright: false
cover: https://blog-1256184194.file.myqcloud.com/2020/03/10/e92f1132824c7.jpg
title: 神奇的轮子之 - snappy
date: 2020-03-10 17:31 
tags:
  - php
  - tools
categories:
  - 分享
---

# 神奇的轮子之 - snappy


### 起因
- 因为项目中，需要将多张图片和文字进行拼接生成图片，一开始用GD库，果然难用，哈哈哈
- 经过不断的努力搜索，发现了神器[wkhtmltoimage](https://wkhtmltopdf.org/)
- 原来可以先写好静态html，直接进行转换就好了
- 但是这个是一个shell脚本，php调用的话，已经有人写好了轮子 [knplabs/knp-snappy](https://github.com/KnpLabs/snappy)【pdf和image统统不在话下】


### 安装

```
composer require knplabs/knp-snappy
```

选择安装脚本，直接引用官方说明
## wkhtmltopdf binary as composer dependencies

If you want to download wkhtmltopdf and wkhtmltoimage with composer you add to `composer.json`:

```bash
$ composer require h4cc/wkhtmltopdf-i386 0.12.x
$ composer require h4cc/wkhtmltoimage-i386 0.12.x
```

or this if you are in 64 bit based system:

```bash
$ composer require h4cc/wkhtmltopdf-amd64 0.12.x
$ composer require h4cc/wkhtmltoimage-amd64 0.12.x
```

And then you can use it

我这里选择了

```
composer require h4cc/wkhtmltoimage-amd64
```


### 使用



```
<?php

require __DIR__ .'/vendor/autoload.php';

// 首先实例类，根据系统不同，选择不一样的脚本程序

$imageSnappy = new \Knp\Snappy\Image(__DIR__ . '/vendor/h4cc/wkhtmltoimage-amd64/bin/wkhtmltoimage-amd64');

// 如果有中文的话，需要添加中文字体
$ttf = __DIR__ . '/ttf/sc.ttf';
// 需要合成的图片地址，base64也可以的，其实跟这个也无关，主要是html能静态展示，那么合成的就是什么样的
$image_url = 'https://www.php.net/images/logos/php-logo.svg';
$name = 'wkhtmltoimage';

// html代码，随意搞了搞，真实使用的话，需要用心调整样式哦~
$htmlTemplate = <<<EOF
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>海报</title>
</head>
<style>
    @font-face {
        font-family: myFirstFont;
        src: url('{$ttf}');
    }

    body {
        font-family: myFirstFont;
    }
</style>
<body>
<img src="{$image_url}">
<br>
{$name}
</body>
</html>
EOF;

$output = __DIR__ . '/demo/'.time().'.jpg';
$imageSnappy->generateFromHtml($htmlTemplate, $output);

echo 'ok';
```

