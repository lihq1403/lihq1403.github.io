---
cover: https://media-1256184194.file.myqcloud.com/image/bing/2018-07-13-5e7ca13d90c9c.jpg
copyright: true
title: RabbitMQ 初体验
date: 2020-08-13 21:39
tags:
  - php
  - tools
categories:
  - 分享
---

# RabbitMQ 初体验

### RabbitMQ介绍
- MQ是 message queue 的简称，是应用程序和应用程序之间通信的方法。

- RabbitMQ是一个由erlang语言编写的、开源的、在AMQP基础上完整的、可复用的企业消息系统。支持多种语言，包括java、Python、ruby、PHP、C/C++等。

- AMQP：advanced message queuing protocol ，一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息并不受客户端/中间件不同产品、不同开发语言等条件的限制。
- 实用优点：应用解耦，流量削峰，异步处理


### RabbitMQ安装
- https://hub.docker.com/_/rabbitmq

自从有了docker，妈妈再也不担心我安装软件啦

为了方便管理docker容器，我们采用compose的方式运行

> tips: management版本是带web管理工具的

#### 1、超级简单版本

###### 一键启动
```
docker run -d --name rabbitmq -e RABBITMQ_DEFAULT_USER=guest -e RABBITMQ_DEFAULT_PASS=guest -p 15672:15672 -p 5672:5672 rabbitmq:management
```

#### 2、看起来好看版本

###### docker-compose.yml
```
version: '2'
services:
  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    hostname: myrabbitmq
    ports:
      - 15672:15672
      - 5672:5672
    environment:
      - RABBITMQ_DEFAULT_USER=guest
      - RABBITMQ_DEFAULT_PASS=guest
    restart: always
    # 里面的/var/lib/rabbitmq/.erlang.cookie 需要600权限
    volumes:
      - ./data:/var/lib/rabbitmq
```

###### 启动

```
docker-compose up -d
```

###### 检查
1. docker ps 查看状态
2. 访问 localhost:15762 查看web管理页面

###### 卸载

```
docker-compose down
```

### 尝试实用
> 官方文档：https://www.rabbitmq.com/getstarted.html


##### 1、[Hello World](https://www.rabbitmq.com/tutorials/tutorial-one-php.html)
> 单发送单接收：简单的发送与接收，没有特别的处理

![python-one](https://www.rabbitmq.com/img/tutorials/python-one.png)

代码示例

###### send.php
```
<?php

require_once __DIR__ . '/../vendor/autoload.php';

// 连接RabbitMQ
$connection = new \PhpAmqpLib\Connection\AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
// 创建一个通道
$channel = $connection->channel();
// 声明一个队列
$channel->queue_declare('hello', false, false, false, false);

$msg = new \PhpAmqpLib\Message\AMQPMessage('Hello World');
$channel->basic_publish($msg, '', 'hello');

echo " [x] Sent 'Hello World!'\n";

$channel->close();
$connection->close();
```
###### receive.php

```
<?php

require_once __DIR__ . '/../vendor/autoload.php';

// 连接RabbitMQ
$connection = new \PhpAmqpLib\Connection\AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
// 创建一个通道
$channel = $connection->channel();
// 声明一个队列
$channel->queue_declare('hello', false, false, false, false);

echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";

$callback = function ($msg) {
    echo " [x] Received ", $msg->body, "\n";
};

$channel->basic_consume('hello', '', false, true, false, false, $callback);

while (count($channel->callbacks)) {
    $channel->wait();
}
```

发送测试：
```
$ php HelloWord/send.php
 [x] Sent 'Hello World!'
```
接受测试：

```
$ php HelloWord/receive.php
 [*] Waiting for messages. To exit press CTRL+C
 [x] Received Hello World
```

##### 2、Work queues
> 单发送多接收：一个发送端，多个接收端，如分布式的任务派发

![python-two](https://www.rabbitmq.com/img/tutorials/python-two.png)

###### new_task.php

```
<?php

require_once __DIR__ . '/../vendor/autoload.php';

use PhpAmqpLib\Message\AMQPMessage;

$connection = new \PhpAmqpLib\Connection\AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->queue_declare('task_queue', false, true, false, false);

$data = implode(' ', array_slice($argv, 1));
if (empty($data)) {
    $data = "Hello World!";
}
$msg = new AMQPMessage(
    $data,
    [
        'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
    ]
);

$channel->basic_publish($msg, '', 'task_queue');

echo '[x] Sent ', $data, "\n";

$channel->close();
$connection->close();
```

###### worker.php

```
<?php

require_once __DIR__ . '/../vendor/autoload.php';

// 连接RabbitMQ
$connection = new \PhpAmqpLib\Connection\AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
// 创建一个通道
$channel = $connection->channel();
// 声明一个队列
$channel->queue_declare('task_queue', false, true, false, false);

echo " [*] Waiting for messages. To exit press CTRL+C\n";

$callback = function ($msg) {
    echo ' [x] Received ', $msg->body, "\n";
    // 假装耗时任务，一个.代表1秒
    sleep(substr_count($msg->body, '.'));
    echo " [x] Done\n";
    // 消费者发送回一个确认
    $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
};

$channel->basic_qos(null, 1, null);
$channel->basic_consume('task_queue', '', false, false, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

开启两个worker：
```
$ php WorkQueues/worker.php
 [*] Waiting for messages. To exit press CTRL+C
```
```
$ php WorkQueues/worker.php
 [*] Waiting for messages. To exit press CTRL+C
```
发送任务：

```
$ php WorkQueues/new_task.php "第一个任务耗时5秒....."
 [x] Sent 第一个任务耗时5秒.....
$ php WorkQueues/new_task.php "第二个任务耗时5秒....."
 [x] Sent 第二个任务耗时5秒.....
```
这时候第1个worker开始工作，第2个任务进来之后，就循环顺延到下一个worker
```
$ php WorkQueues/worker.php
 [*] Waiting for messages. To exit press CTRL+C
 [x] Received 第一个任务耗时5秒.....
 [x] Done
```
```
$ php WorkQueues/worker.php
 [*] Waiting for messages. To exit press CTRL+C
 [x] Received 第二个任务耗时5秒.....
 [x] Done
```

##### 3、Publish/Subscribe
> 发布/订阅模式：发送端发送广播消息，多个接收端接收

![python-three](https://www.rabbitmq.com/img/tutorials/python-three.png)

###### receive_logs.php

```
<?php

require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->exchange_declare('logs', 'fanout', false, false, false);

list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

$channel->queue_bind($queue_name, 'logs');

echo " [*] Waiting for logs. To exit press CTRL+C\n";

$callback = function ($msg) {
    echo ' [x] ', $msg->body, "\n";
};

$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

###### emit_log.php

```
<?php

require_once __DIR__ . '/../vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->exchange_declare('logs', 'fanout', false, false, false);

$data = implode(' ', array_slice($argv, 1));
if (empty($data)) {
    $data = "info: Hello World!";
}
$msg = new AMQPMessage($data);

$channel->basic_publish($msg, 'logs');

echo ' [x] Sent ', $data, "\n";

$channel->close();
$connection->close();
```

开启两个消费者：
```
$ php PublishSubscribe/receive_logs1.php
 [*] Waiting for logs. To exit press CTRL+C
```
```
$ php PublishSubscribe/receive_logs2.php
 [*] Waiting for logs. To exit press CTRL+C
```
发送任务：

```
$ php PublishSubscribe/emit_log.php "创建日志"
 [x] Sent 创建日志
```
这时候生产者讲任务推给了交换机，由交换机将数据发送给与之绑定的队列
```
$ php PublishSubscribe/receive_logs1.php
 [*] Waiting for logs. To exit press CTRL+C
 [x] 创建日志
```
```
$ php PublishSubscribe/receive_logs2.php
 [*] Waiting for logs. To exit press CTRL+C
 [x] 创建日志
```

简单解释：可以将消息发送给不同类型的消费者。做到发布一次，消费多个

- [ ] **todo 下面的介绍，留住后面慢慢消化一下**

##### 4、Routing
> 按路线发送接收：发送端按routing key发送消息，不同的接收端按不同的routing key接收消息。

![python-four](https://www.rabbitmq.com/img/tutorials/python-four.png)

##### 5、Topics

> topic类型的Exchange在匹配规则上进行了扩展，它与direct类型的Exchage相似，也是将消息路由到binding key与routing key相匹配的Queue中

![python-five](https://www.rabbitmq.com/img/tutorials/python-five.png)

##### 6、RPC

![python-six](https://www.rabbitmq.com/img/tutorials/python-six.png)

> 实际的应用场景中，我们很可能需要一些同步处理，需要同步等待服务端将我的消息处理完成后再进行下一步处理。这相当于RPC（Remote Procedure Call，远程过程调用）。

##### 7、Publisher Confirms

> 发布者确认 是实现可靠发布的RabbitMQ扩展。在通道上启用发布者确认后，代理将异步确认客户端发布的消息，这意味着它们已在服务器端处理。

### 示例代码
https://github.com/lihq1403/gadget/tree/master/RabbitMQ


### 延时队列实现

Rabbitmq默认没有支持延迟队列，查阅了一些资料发现，是可以通过两种方式实现
1. TTL和死信队列
2. [rabbitmq_delayed_message_exchange](rabbitmq/rabbitmq-delayed-message-exchange) 插件


##### TTL和死信队列实现方式
> 参考文章：https://blog.csdn.net/u011069013/article/details/107079470/

###### send.php
```
<?php

<?php

require_once __DIR__ . '/../vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Wire\AMQPTable;
use PhpAmqpLib\Message\AMQPMessage;

// 连接RabbitMQ
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
// 创建一个通道
$channel = $connection->channel();

$exchange_name = 'test_exchange';
$queue_name    = 'test_queue';

// 定义默认的交换器
$channel->exchange_declare($exchange_name, 'topic', false, true, false);
// 定义延迟交换器
$channel->exchange_declare('delayed_exchange', 'topic', false, true, false);

//定义延迟队列
$channel->queue_declare('delayed_queue', false, true, false, false, false, new AMQPTable(array(
    "x-dead-letter-exchange"    => "delayed_exchange",
    "x-dead-letter-routing-key" => "delayed_exchange",
    "x-message-ttl"             => 5000, //5秒延迟
)));
//绑定延迟队列到默认队列上
$channel->queue_bind('delayed_queue', $exchange_name);

// 声明一个队列
$channel->queue_declare($queue_name, false, true, false, false, false);
//绑定正常消费队列到延迟交换器上
$channel->queue_bind($queue_name, 'delayed_exchange', 'delayed_exchange');

$nowTime = date('H:i:s');
$msg = new AMQPMessage('Hello World 发送时间：'. $nowTime, [
    'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT
]);
$channel->basic_publish($msg, $exchange_name);

echo " [x] Sent 'Hello World!'\n";

$channel->close();
$connection->close();
```
###### receive.php

```
<?php

require_once __DIR__ . '/../vendor/autoload.php';

// 连接RabbitMQ
$connection = new \PhpAmqpLib\Connection\AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
// 创建一个通道
$channel = $connection->channel();

$queue_name = 'test_queue';

// 声明一个队列
$channel->queue_declare($queue_name, false, true, false, false, false);

echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";

$callback = function ($msg) {
    echo " [x] Received ", $msg->body, ' 接受时间：', date('H:i:s'), "\n";
};

$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while (count($channel->callbacks)) {
    $channel->wait();
}
```

发送测试：
```
$ php HelloWord/send.php
 [x] Sent 'Hello World!'
```
接受测试：

```
$ php HelloWord/receive.php
 [*] Waiting for messages. To exit press CTRL+C
 [x] Received Hello World 发送时间：21:19:06 接受时间：21:19:11
```