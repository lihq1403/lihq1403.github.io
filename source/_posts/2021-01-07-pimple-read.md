---
cover: https://blog-1256184194.file.myqcloud.com/2021/01/07/5788b2d7b136d.jpg
copyright: true
title: Pimple 浅析
date: 2021-01-07 22:57
tags:
  - php
  - composer
categories:
  - 分享
---

# Pimple 浅析

> 最近项目里面要封装一些composer包，在查询资料的过程中，发现了牛逼的项目，强啊！[Github](https://github.com/silexphp/Pimple)

## Pimple
官方的介绍就一句话：Pimple - 一个简单的 PHP 依赖注入容器；哈哈哈，果然简介也很简单，接下来看看源码，先从简单的1.x看起

### ArrayAccess
看pimple的时候，就不得不提到一个常用的接口[ArrayAccess](https://www.php.net/manual/zh/class.arrayaccess.php)，提供像访问数组一样访问对象的能力的接口。

举个例子
```
<?php

class ArrayAccessObj implements ArrayAccess
{
    private $container = [];

    public function __construct()
    {
        $this->container = [
            'one' => 1,
            'two' => 2,
            'three' => 3,
        ];
    }

    /**
     * 检查一个偏移位置是否存在.
     * @param mixed $offset
     * @return bool
     */
    public function offsetExists($offset)
    {
        return isset($this->container[$offset]);
    }

    /**
     * 获取一个偏移位置的值
     * @param mixed $offset
     * @return mixed
     */
    public function offsetGet($offset)
    {
        return $this->container[$offset] ?? null;
    }

    /**
     * 设置一个偏移位置的值
     * @param mixed $offset
     * @param mixed $value
     */
    public function offsetSet($offset, $value)
    {
        $this->container[$offset] = $value;
    }

    /**
     * 复位一个偏移位置的值
     * @param mixed $offset
     */
    public function offsetUnset($offset)
    {
        unset($this->container[$offset]);
    }
}

$foo = new ArrayAccessObj();

var_dump($foo['one']); // int(1)
unset($foo['one']);
var_dump($foo['one']); // NULL
$foo['four'] = 4;
var_dump($foo['four']); // int(4)
```

那么你会获得结果：

```
int(1)
NULL
int(4)
```

简单的总结一下啦：
- 类实现了ArrayAccess接口，那么这个类的对象就可以使用$foo['xxx']这种结构了。
- $foo['xxx'] 对应调用offsetGet方法。
- $foo['xxx'] = 'yyy' 对应调用offsetSet方法。
- isset($foo['xxx']) 对应调用offsetExists方法。
- unset($foo['xxx']) 对应调用offsetUnset方法。

比如我将offsetGet改写成

```
public function offsetGet($offset)
{
    echo '进入offsetGet方法，参数为:' . $offset . PHP_EOL;
    return $this->container[$offset] ?? null;
}
```
那么结果将会变成

```
进入offsetGet方法，参数为:one
int(1)
进入offsetGet方法，参数为:one
NULL
进入offsetGet方法，参数为:four
int(4)
```
果然，ArrayAccess并没有限制你的想象，只要在定义的几个实现中完成你自己的天马行空，哈哈哈

### 闭包和匿名函数
都开始php8了，大家应该都知道这[是啥](https://www.cnblogs.com/liluxiang/p/9499298.html)了吧

```
<?php
$closure = function ($name) {
    return 'Hello ' . $name;
};
echo $closure('Lihq');//Hello Lihq
var_dump(method_exists($closure, '__invoke'));//true
```

## 1.x

> Pimple是一个用于PHP 5.3的小型依赖项注入容器，它仅由一个文件和一个类（约80行代码）组成。

通过ArrayAccess、闭包、匿名函数的结合，创造了一个艺术品。[源码很简单的](https://github.com/silexphp/Pimple/tree/1.1)

### 定义参数

存入与读取，仅仅用到了ArrayAccess
```
<?php

require __DIR__ . '/vendor/autoload.php';

$container = new Pimple();
$container['bar'] = 'foo';
var_dump($container['bar']); // string(3) "foo"
```

### 定义服务

使用匿名函数定义服务类
```
<?php

require __DIR__ . '/vendor/autoload.php';

class Service
{
    public function bar()
    {
        return 'bar';
    }
}

$container = new Pimple();
$container['service'] = function () {
    return new Service();
};

/** @var Service $service */
$service = $container['service'];
echo $service->bar(); // bar
```

### 定义共享服务

这样实例过的服务类就不会再次实例了
```
<?php

require __DIR__ . '/vendor/autoload.php';

class Service
{
    public function __construct()
    {
        echo 'share仅调用一次实例' . PHP_EOL;
    }

    public function bar()
    {
        return 'bar';
    }
}

$container = new Pimple();

$container['service'] = $container->share(function () {
    return new Service();
});

/** @var Service $service */
$service = $container['service']; // share仅调用一次实例
echo $service->bar() . PHP_EOL; // bar

/** @var Service $service */
$service = $container['service']; // 再次获取不会重新new
echo $service->bar() . PHP_EOL; // bar
```

### 保护参数

原汁原味的匿名函数保护，没有被保护的会直接执行
```
<?php

require __DIR__ . '/vendor/autoload.php';

$container = new Pimple();

$container['bar'] = function () {
    return 'bar';
};

$container['foo'] = $container->protect(function () {
    return 'foo';
});

var_dump($container['bar']); // string(3) "bar"
var_dump($container['foo']); // object(Closure)#4 (0) {}
var_dump($container['foo']()); // string(3) "foo"
```

### 扩展服务

可以在原服务的基础上，进行功能修改
```
<?php

require __DIR__ . '/vendor/autoload.php';

$container = new Pimple();
$container['service'] = function () {
    return 'service';
};

$container['service'] = $container->extend('service', function ($service) {
    return $service . ' extend';
});

var_dump($container['service']); // string(14) "service1 extend"

```

### 原始访问

```
<?php

require __DIR__ . '/vendor/autoload.php';

class Service
{
    public function bar()
    {
        return 'bar';
    }
}

$container = new Pimple();

$container['service'] = $container->share(function () {
    return new Service();
});

var_dump($container['service']); // object(Service)#5 (0) {}

var_dump($container->raw('service')); // object(Closure)#4 (2) {}
```

## 3.x

> 为啥跳过了2.x，其实版本是有2的，但是我看官方的分支已经只有1和master了，是不是代表2不怎么维护了，嘻嘻嘻，直接看3吧，毕竟项目里面用到的是3啦，下面来讲讲有区别的地方吧

### PSR Container

在安装3.x的时候，发现它引入了"psr/container": "^1.0"，并且"php": ">=7.2.5"，666，果然要与时并进，使用当下最好的规范与语言。其中，PSR Container即大名鼎鼎的[PSR 11](https://www.php-fig.org/psr/psr-11/)

##### PSR 是 php-fig 提供的标准化建议，虽然不是官方组织，但是得到广泛认可。PSR-11 提供了容器接口。它包含 ContainerInterface 和 两个异常接口，并提供使用建议。

### SplObjectStorage

```
public function __construct(array $values = [])
{
    $this->factories = new \SplObjectStorage();
    $this->protected = new \SplObjectStorage();

    foreach ($values as $key => $value) {
        $this->offsetSet($key, $value);
    }
}
```
可以看到，在源码里，factories和protected都使用了[SplObjectStorage](https://www.php.net/manual/zh/class.splobjectstorage.php)

即同时继承了Countable , Iterator , Serializable , ArrayAccess接口，拥有很多方法
```
SplObjectStorage implements Countable , Iterator , Serializable , ArrayAccess {
    // 从另一个存储添加所有对象
    public addAll ( SplObjectStorage $storage ) : void
    
    // 在存储中添加一个对象
    public attach ( object $object , mixed $data = null ) : void
    
    // 检查存储是否包含特定对象
    public contains ( object $object ) : bool
    
    // 返回存储中的对象数
    public count ( ) : int
    
    // 返回当前的存储条目
    public current ( ) : object
    
    // 从存储中删除对象
    public detach ( object $object ) : void
    
    // 计算所包含对象的唯一标识符
    public getHash ( object $object ) : string
    
    // 返回与当前迭代器条目关联的数据
    public getInfo ( ) : mixed
    
    // 返回当前迭代器所在的索引
    public key ( ) : int
    
    // 移至下一个条目
    public next ( ) : void
    
    // 检查存储中是否存在对象
    public offsetExists ( object $object ) : bool
    
    // 返回与对象关联的数据
    public offsetGet ( object $object ) : mixed
    
    // 将数据关联到存储中的对象
    public offsetSet ( object $object , mixed $data = null ) : void
    
    // 从存储中删除一个对象
    public offsetUnset ( object $object ) : void
    
    // 从当前存储中删除另一个存储中包含的对象
    public removeAll ( SplObjectStorage $storage ) : void
    
    // 从当前存储中删除除另一个存储中包含的对象以外的所有对象
    public removeAllExcept ( SplObjectStorage $storage ) : void
    
    // 将迭代器后退到第一个存储元素
    public rewind ( ) : void
    
    // 序列化存储
    public serialize ( ) : string
    
    // 设置与当前迭代器条目关联的数据
    public setInfo ( mixed $data ) : void
    
    // 从其字符串表示形式反序列化存储
    public unserialize ( string $serialized ) : void
    
    // 返回当前迭代器条目是否有效
    public valid ( ) : bool
}
```

### 工厂factory
现在的默认情况，每次获得实例都会返回相同的实例，使用factory将每次返回新的实例，跟1.x是反过来的

```
<?php

use Pimple\Container;

require __DIR__ . '/vendor/autoload.php';

class Service
{
    public function __construct()
    {
        echo 'share仅调用一次实例' . PHP_EOL;
    }

    public function bar()
    {
        return 'bar';
    }
}

$container = new Container();

$container['service'] = function () {
    return new Service();
};

/** @var Service $service */
$service = $container['service']; // share仅调用一次实例
echo $service->bar() . PHP_EOL; // bar

/** @var Service $service */
$service = $container['service']; // 不出现实例信息
echo $service->bar() . PHP_EOL; // bar
```

### 注册容器

通过接口定义，注册服务，方便解耦
```
use Pimple\Container;

class FooProvider implements Pimple\ServiceProviderInterface
{
    public function register(Container $pimple)
    {
        // register some services and parameters
        // on $pimple
    }
}
```

```
$pimple->register(new FooProvider());
```

### 使用方式
哈哈哈，我自己的代码太low了，这里介绍一个用pimple写的开源代码
- [easywechat](https://github.com/overtrue/wechat)
- [dingtalk](https://github.com/mingyoung/dingtalk)

了解了一些基础概念[ArrayAccess、闭包和匿名函数、SplObjectStorage、服务、容器、注册]之后，应该是很好理解这些代码的

## 个人学习代码

- https://github.com/lihq1403/gadget/tree/master/composer/pimple

