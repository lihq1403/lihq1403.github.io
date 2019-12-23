---
cover: https://blog-1256184194.file.myqcloud.com/2019/12/23/db11f1e28564d.jpg
title: PHP 常用过滤器
date: 2019-08-28 16:16
tags:
 - php
categories:
 - 分享
keywords:
description:
---

# PHP 常用过滤器

> 平时用到的框架，基本都实现了表单验证，探究一下他们的代码，就会发现，使用了大量的过滤器和正则。果然PHP是世界上最好的语言！！！哈哈哈



### filter_has_var

#### 笨拙的方法

```php
if(isset($_GET["name"])
```

#### 最好的语言提供的方法

```php
filter_has_var(INPUT_GET, "name") 
```

######  返回值：

成功时返回 **TRUE**， 或者在失败时返回 **FALSE**。

> 第一个参数可以填入的值有，看英文应该懂是啥意思吧ヽ(ー_ー)ノ

| type         |
| :----------- |
| INPUT_GET    |
| INPUT_POST   |
| INPUT_COOKIE |
| INPUT_SERVER |
| INPUT_ENV    |

### **filter_var**

#### 笨拙的方法

```php
疯狂正则，疯狂字符串拆解
```

#### 优秀的语言提供的方法

```php
 filter_var("hhh@qq.com", FILTER_VALIDATE_EMAIL);
```

###### 返回值：

如果OK 会返回原值，如果不OK 则返回false

###### 验证过滤

| FILTER                  | DESC                                                         |
| ----------------------- | ------------------------------------------------------------ |
| FILTER_VALIDATE_BOOLEAN | 当第一个参数是”1″, “true”, “on” and “yes” 这些字符串时会返回true .否则为false |
| FILTER_VALIDATE_EMAIL   | 验证是否邮箱地址                                             |
| FILTER_VALIDATE_FLOAT   | 浮点                                                         |
| FILTER_VALIDATE_INT     | 整形                                                         |
| FILTER_VALIDATE_IP      | IP地址                                                       |
| FILTER_VALIDATE_MAC     | 物理MAC地址                                                  |
| FILTER_VALIDATE_REGEXP  | 正则表达式                                                   |
| FILTER_VALIDATE_URL     | URL网址                                                      |

###### 净化过滤，常用的几个

| FILTER                       | DESC                 | EXEMPLE                                                      |
| ---------------------------- | -------------------- | ------------------------------------------------------------ |
| FILTER_SANITIZE_NUMBER_INT   | 过滤掉非数字型的内容 | ```filter_var('test123', FILTER_SANITIZE_NUMBER_INT)``` 返回"123" |
| FILTER_SANITIZE_MAGIC_QUOTES |       对字符串执行 addslashes() 函数               | ```filter_var("test'456", FILTER_SANITIZE_MAGIC_QUOTES)``` 返回"test\'456" |
| FILTER_SANITIZE_STRING       |    过滤器去除或编码不需要的字符                  |   ```filter_var("<script>alert('xss')</script>", FILTER_SANITIZE_STRING)``` 返回"alert(&#39;xss&#39;)" |



如果想要了解更多的话，查看官方文档是最好的选择

[https://www.php.net/manual/zh/function.filter-var.php](https://www.php.net/manual/zh/function.filter-var.php)

### 多动手才是最好的学习途径，加油

自己动手，丰衣足食 示例代码：[https://github.com/lihq1403/gadget/blob/master/filter-function/filter.php](https://github.com/lihq1403/gadget/blob/master/filter-function/filter.php)