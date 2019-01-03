---
layout:     post
title:      "redis消息订阅"
subtitle:   "如果你需要一个简单发布/订阅 机制的消息队列"
date:       2019-01-03 14:00:00
author:     "憧憬"
header-img: "img/in-post/post-linux-pd/redis.png"
header-mask: 0.3
catalog:    true
tags:
  - redis
  - php
---

### Laravel实现redis发布 订阅消息

如果说我们需要一个比较简单的这种机制,我们可以采用redis这个轻量级的订阅机制,我们可以参考redis的 Publish/Subscribe 机制,得到比较好的问题解决方案
当然,如果是项目比较复杂,可以考虑使用Kafka， RabbitMQ之类的消息队列组件

首先简单介绍关于redis这个机制相关的几个命令

```
PSUBSCRIBE pattern [pattern ...] 
订阅一个或多个符合给定模式的频道。

PUBSUB subcommand [argument [argument ...]] 
查看订阅与发布系统状态。

PUBLISH channel message 
将信息发送到指定的频道。

PUNSUBSCRIBE [pattern [pattern ...]] 
退订所有给定模式的频道。

SUBSCRIBE channel [channel ...] 
订阅给定的一个或多个频道的信息。

UNSUBSCRIBE [channel [channel ...]] 
指退订给定的频道。
```

我们是使用Laravel来实现这个

```
composer require predis/predis  安装redis组件

使用Laravel创建发送消息文件及接收消息文件
php artisan make:command PublishMsg --command=Pub:Msg
php artisan make:command SubscribeMsg --command=Sub:Msg

```

在App\Console\Commands\SubscribeMsg.php中 handle订阅redis队列消息

```
		// 启用redis订阅功能   持续监听redis-msg队列是否有消息   如果要消息就会到回调里面被echo
       Redis::subscribe(['redis-msg'],function ($message){
            echo $message;
        });

```

在App\Console\Commands\PublishMsg.php中 handle发送redis队列消息

```
try{
					// 队列名称      消息
        Redis::publish('redis-msg','this a test hahhhhhhhhhhhh');
    }catch (\Exception $e){
        echo "发送失败";
}
```

```
php artisan Sub:Msg  开启消息订阅
php artisan Pub:Msg  开启消息发布
```

在实际中需要传输数据时,一般会将其序列化为字符串或以json XML等格式进行发送