---
cover: https://blog-1256184194.file.myqcloud.com/2019/12/23/98ca616a8cf05.jpg
title: 压测工具 - ab
date: 2019-10-19 11:22
tags:
  - tools
categories:
  - 分享
keywords:
description:
---

# 压测工具 - ab

## 实验环境
- Centos7.5

## 基本概念

1. QPS：每秒钟请求或查询数量，在互联网领域指每秒响应的请求数（指HTTP请求）
2. 吞吐量：单位时间内处理的请求数量（通常由QPS和并发数决定）
3. 响应时间：从请求发出到收到响应花费时间
4. PV：综合浏览量（Page View），即页面浏览量或者点击量，一个访客在24小时内访问的页面数量。同一个人浏览你的网站同一个页面，只记作一次PV
5. UV：独立访客（UniQue Visitor）,即一定时间范围内相同访客多次访问网站，只能计算为1个独立访客
6. 带宽：计算带宽大小需关注两个指标，峰值流量和页面的平均大小
7. 日网站带宽=PV/统计时间（秒）*平均页面大小（KB）*8
8. 峰值一般是平均值的倍数
9. QPS不等于并发并发连接数。QPS是每秒HTTP请求数量，并发连接数是系统同时处理的请求数量
10. 二八定律（80%的访问量集中在20%的时间）：（总PV数*80%）/（6小时秒速*20%）=峰值每秒请求数（QPS）
11. 压力测试：能承受最大的并发数和最大承受的QPS值

##### 注意事项：
1. 测试机器与被测机器分开
2. 不要对线上服务做压力测试
3. 观察测试工具所在机器，以及被测试的前端机的CPU、内存、网络等都不超过最高限度的75%

### QPS指标
1. QPS达到50，可以称之为小型网站，一般服务器都可以应付 
2. QPS达到100；瓶颈：MySQL查询达到瓶颈；优化方案：数据库缓存层，数据库负载均衡
3. QPS达到800；瓶颈：带宽速度达到瓶颈；优化方案：CDN加速，负载均衡
4. QPS达到1000；瓶颈：缓存服务器的带宽达到瓶颈；优化方案：静态HTML缓存
5. QPS达到2000；瓶颈：文件系统访问锁成为灾难；优化方案：做业务分离，分布式存储

## 安装
```
# yum install httpd-tools
```

## 检验是否成功
```
# ab -V
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
```



## 基本使用

### 检测接口最大qps

模拟并发请求10次，总请求100次
```
# ab -n 100 -c 10 http://xxx

Requests per second: 101.15[#/sec](mean)
```


## 工具能力发掘

[ab工具](http://httpd.apache.org/docs/2.0/programs/ab.html)

[more](https://www.baidu.com/s?wd=ab%E5%B7%A5%E5%85%B7)