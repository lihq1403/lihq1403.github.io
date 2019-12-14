---
cover: https://image.lihq.xyz/imgs/2019/12/61b92e6001bbc582.jpg
title: 【BUG修复日记】之支付回调websocket busy
date: 2019-08-30 09:59
tags: 
 - php
 - fix bug
 - workerman
categories:
 - PHP
keywords:
description:
---

# 【BUG修复日记】之支付回调websocket busy

### 问题描述
项目中，用到了第四方聚合扫码支付，支付成功会有回调。回调支付成功是服务端的数据整理（这段实现没问题），但是前台页面需要弹出支付成功，并关闭二维码，一开始想了好几个方案：
1. 展示二维码的同时，前端轮询查询状态接口，直到成功就提示支付成功
2. 展示二维码的同时，使用websocket进行长连接，支付成功，服务端主动推送成功结果

> 作为技术爱好者，当然是选择第二种啦（感觉第一种没啥技术含量，而且会造成服务端接受太多无用请求了）

说干就干：
用的框架是tp5.1，可以用workeman来实现websocket
```
composer require topthink/think-worker=2.0.* // 5.1仅支持2.0，别搞错了哦
```
有关workeman的介绍可以看官方文档：[Workerman，高性能socket服务框架](http://doc.workerman.net/)

安装完成之后，会生成几个文件，我们需要关系的就只是config/worker_server.php

我这里只是做了简单的长连接，主要修改了onMessage回调（记得修改protocol，host，port，count等等配置）
```
    'onMessage'      => function ($connection, $data) {
        // 客户端回应服务端的心跳
        if ($data == 'pong') {
            $connection->send('pong');
            return ;
        }

        // 客户端传递的是json数据
        $message_data = json_decode($data, true);
        if(!$message_data) {
            $connection->send(error('error require data'));
            return ;
        }

        switch ($message_data['type']) {
            // 客户端检查是否扫码支付完成
            case 'qrPayQuery':
                if (empty($message_data['orderId'])) {
                    $return = error('empty orderId');
                    break;
                }
                // 查询订单是否已经完成支付
                while (1) {
                    $status = Cache::connect(config('***'))->get('***'.$message_data['orderId']);
                    if ($status === false) {
                        $return = error('no found orderId');
                        break ;
                    }
                    if ($status == 'success') {
                        // 直到状态是成功
                        $return = success('success');
                        break ;
                    }
                    sleep(mt_rand(1,2));
                }
                break;
            default:
                $return = error('error type');
        }

        $connection->send($return);
    }
```
在支付成功的回调中，我创建了一个订单号值的缓存，置为success。有请求过来的时候，就不断轮询缓存值，直到成功标志位被检查到才停止，主动推送。

将这一套方案提交到测试服的时候，前端接入的时候一切正常，当时就没太在意里面的问题。果然，测试介入的时候，问题就暴露出来了，测试反馈：前端页面没有提示支付成功也没有关闭二维码，支付记录是有值的。

然后我去测试websocket链接的时候，发现连不上，服务端看状态
```
pid     memory  listening                 worker_name        connections send_fail timers  total_request qps    status
29270   N/A      websocket://0.0.0.0:17730 name               N/A          N/A         N/A       N/A            N/A      [busy]
```
哇塞，服务挂掉了。。。万能的重启，果然恢复了，但是测试用了一段时间，又挂掉了。

初步猜测：
1. 进程死循环了

### 问题解决

那么开始解决问题，查了一些资料，并[调试busy进程](http://doc.workerman.net/debug/busy-process.html)

果然问题和我猜测的一样，死循环了，因为测试打开支付页面，并不支付，就导致订单号一直存在，回调没有给到结果，就会出现不断查询不支付的缓存，导致服务进程内存爆掉了。

> 出现了问题，就要想一想解决办法，为了防止服务端有死循环，那么就要抛弃掉轮询查询，而是改为回调的时候，通知服务端，再由服务端通知客户端

```
graph LR
第四方回调-->服务端
服务端-->指定客户端
```
当时想了很久：
1. 使用[GatewayWorker](http://doc2.workerman.net/)提供用户标识定向推送结果
2. WorkerMan实现指定客户端推送

> 第一种方案，有点大材小用了，毕竟是用来专业做消息推送的

果然千辛万苦（哈哈哈，其实只是学习的时候，知道有这个方案。），找到了官方的一个方案 [WorkerMan中如何向某个特定客户端发送数据](http://doc.workerman.net/faq/send-data-to-client.html)
```
说明：
以上例子可以针对uid推送，虽然是单进程，但是支持个10W在线是没问题的。
注意这个例子只能单进程，也就是$worker->count 必须是1
```
看上去，本项目应该也没有10W在线那么夸张，就这样做了

那么开始动手，修改我的onMessage回调
```
    'onMessage'      => function ($connection, $data) {

        // 客户端回应服务端的心跳
        if ($data == 'pong') {
            $connection->send('pong');
            return ;
        }

        // 客户端传递的是json数据
        $message_data = json_decode($data, true);
        if(!$message_data) {
            $connection->send(error('error require data'));
            return ;
        }

        switch ($message_data['type']) {
            // 客户端检查是否扫码支付完成
            case 'qrPayQuery':
                if (empty($message_data['orderId'])) {
                    $return = error('empty orderId');
                    break;
                }
                global $worker;

                // 把orderId当做uid
                $connection->uid = $message_data['orderId'];
                $worker->uidConnections[$connection->uid] = $connection;
                $return = success('login success, your orderId is ' . $connection->uid, [], 201);

                break;
            // 服务端通知客户端
            case 'qrPayNotify':
                if (empty($message_data['orderId'])) {
                    $return = error('empty orderId');
                    break;
                }
                // 发送给订单
                sendMessageByUid($message_data['orderId'], success('success'));
                $return = success('success');
                break;
            default:
                $return = error('error type');
        }

        $connection->send($return);
    }
    
// 针对uid推送数据
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}
```
在回调的地方，进行websocket请求发送qrPayNotify
```
// 用到了一个客户端
composer require textalk/websocket
```
```
try {
    $client = new Client($notify_websocket);
    $payload = [
        'type' => 'qrPayNotify',
        'orderId' => $params['orderId']
    ];
    $client->send(json_encode($payload));
    $message = $client->receive();
} catch (\Exception $e){
    // websocket发送失败，但是不抛出异常
    $message = $e->getMessage();
}
```

经过测试，通过了

### TIPS
使用到这种，最好做一下参数加密或者认证之类的，我这里只是提供了原理。服务进程记得加上[开机启动](http://doc.workerman.net/faq/start-with-system.html)

### 结尾
目前看来，项目运行正常，没啥问题。后续有问题的话，肯定是上强大的GatewayWorker啦