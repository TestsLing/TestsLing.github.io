---
layout:     post
title:      "Swoole基础学习笔记"
subtitle:   "如果你想学习swoole 请先阅读这篇文章"
date:       2018-12-04 14:00:00
author:     "憧憬"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
catalog:    true
tags:
  - swoole
  - php
---

# php异步扩展 Swoole

> 环境docker容器 php7.2  

### 简单了解swoole

![](/img/in-post/post-linux-pd/process.jpg)

### 安装Swoole扩展

```
cd swoole-src-swoole-1.7.6-stable/
phpize
./configure
sudo make
sudo make install

然后去配置php.ini的扩展 extension=swoole.so
现在一般都使用软连接的配置方式 放在model目录一般是
php -m 查看安装的模块  出现swoole即安装成功
然后去官网找个示例  运行一下  测试是否能够正常使用
```



### Worker进程

> 当启动一个Swoole应用时，一共会创建2 + n + m个进程，其中n为
> Worker进程数，m为TaskWorker进程数，2为一个Master进程和一个Manager进程

其中，Master进程为主进程，该进程会创建Manager进程、Reactor线程等工作进/线程。

Worker进程作为Swoole的工作进程，所有的业务逻辑代码均在此进程上运行。当Reactor线
程接收到来自客户端的数据后，会将数据打包通过管道发送给某个Worker进程

当一个Worker进程被成功创建后，会调用 onWorkerStart 回调，随后进入事件循环等待数
据。当通过回调函数接收到数据后，开始处理数据。如果处理数据过程中出现严重错误导致
进程退出，或者Worker进程处理的总请求数达到指定上限，则Worker进程调
用 onWorkerStop 回调并结束进程。

### Task Worker

>Task Worker是Swoole中一种特殊的工作进程，该进程的作用是处理一些耗时较长的任务，以
>达到释放Worker进程的目的。Worker进程可以通过 swoole_server 对象的task方法投递一个
>任务到Task Worker进程
>
>Worker进程通过Unix Sock管道将数据发送给Task Worker，这样Worker进程就可以继续处理
>新的逻辑，无需等待耗时任务的执行。需要注意的是，由于Task Worker是独立进程，因此无
>法直接在两个进程之间共享全局变量，需要使用Redis、MySQL或者swoole_table来实现进程
>间共享数据。

### Timer定时器

> 是一个毫秒级的定时器 不依赖于单独的定时进程 
>
> Swoole扩展使
> 用最小堆存储定时器，减少定时器的检索次数，提高了运行效率。

1. tick定时器是一个永久定时器，使用tick方法创建的定时器会一直运行，每隔指定的毫秒数之
   后执行一次callback函数。在创建定时器的时候，可以通过tick函数的第三个参数，传递一些
   自定义参数到callback回调函数中。另外，也可以使用PHP的闭包（use关键字）实现传参。
   具体实例如下：

```
$str = "Say ";
$timer_id = swoole_timer_tick( 1000 , function($timer_id , $params) use ($str) {
echo $str . $params; // 输出“Say Hello”
} , "Hello" );

tick函数会返回定时器的id  当我们不需要某个定时器的时候 可以根据这个id 调
用 swoole_timer_clear 函数删除定时器
需要注意的是，创建的定时器是不能跨进程的，因
此，在一个Worker进程中创建的定时器，也只能在这个Worker进程中删除，这一点一定要注
意（使用 $worker_id 变量来区分Worker进程）；
```

2. after定时器是一个临时定时器。使用after方法创建的定时器仅在指定毫秒数之后执行一次
   callback函数，执行完成后定时器就会删除。after定时器的回调函数不接受任何参数，可以通
   过闭包方式传递参数，也可以通过类成员变量的方式传递。具体实例如下：

```
class Test
{
private $str = "Say Hello";
public function onAfter()
{
echo $this->str; // 输出”Say Hello“
}
}
$test = new Test();
swoole_timer_after(1000, array($test, "onAfter"); // 成员变量
swoole_timer_after(2000, function() use($test){ // 闭包
$test->onAfter(); // 输出”Say Hello“
});
```

### 多端口监听

> 在一些场景中 我们可能要监听host下的不同端口。 
>
> Swoole提供了addlistener函数用于给服务器添加一个需要监听的host及
> port，并指定对应的Socket类型（TCP，UDP，Unix Socket以及对应的IPv4和IPv6版本）

```
$serv = new swoole_server("192.168.1.1", 9501); // 监听外网的9501端口
$serv->addlistener("127.0.0.1", 9502 , SWOOLE_TCP); // 监听本地的9502端口
$serv->start(); // addlistener必须在start前调用
```

此时，swoole_server就会同时监听两个host下的两个端口。这里要注意的是，来自两个端口
的数据会在同一个 onReceive 回调函数中获取到，这时就要用到swoole的另一个成员函数
connection_info，通过这个函数获取到fd的from_port，就可以判定消息的类型。

```
$info = $serv->connection_info($fd, $from_id);
//来自9502的内网管理端口
if($info['from_port'] == 9502) {
$serv->send($fd, "welcome admin\n");
}
//来自外网
else {
$serv->send($fd, 'Swoole: '.$data);
}
```



### 多端口混合协议接听

> 上示例

```
$port1 = $server->listen("127.0.0.1", 9501, SWOOLE_SOCK_TCP);   // 多协议
$port2 = $server->listen("127.0.0.1", 9502, SWOOLE_SOCK_UDP);
$port3 = $server->listen("127.0.0.1", 9503, SWOOLE_SOCK_TCP | SWOOLE_SSL);

$port1->set( // 开启固定包头协议
'open_length_check' => true,
'package_length_type' => 'N',
'package_length_offset' => 0,
'package_max_length' => 800000,
);
$port3->set( // 开启EOF协议并设置SSL文件
'open_eof_split' => true,
'package_eof' => "\r\n",
'ssl_cert_file' => 'ssl.cert',
'ssl_key_file' => 'ssl.key',
);

$port1->on('receive', function ($serv, $fd, $from_id, $data) {
$serv->send($fd, 'Swoole: '.$data);
$serv->close($fd);
});
$port3->on('receive', function ($serv, $fd, $from_id, $data) {
echo "Hello {$fd} : {$data}\n";
});

```



注意事项

+ 未设置协议处理选项的监听端口，默认使用无协议模式
+ 未设置回调函数的监听端口，使用$server对象的回调函数
+ 监听端口返回的对象类型为swoole_server_port
+ 不同监听端口的回调函数，仍然是相同的Worker进程空间内执行
+ 主服务器是WebSocket或Http协议，新监听的TCP端口默认会继承主Server的协议设置。
+ 必须单独调用 set 方法设置新的协议才会启用新协议

### 配置详解

```
$serv = new swoole_server("0.0.0.0", 9501);
$serv->set(array(
'worker_num' => 8,             		开启的work进程数量
'max_request' => 10000,			   每个worker进程在处理完max_request个请求后就会自动重启。设置该值
的主要目的是为了防止worker进程处理大量请求后可能引起的内存溢出。
'max_conn' => 100000,              设置此参数后，当服务器已有的连接数达到该值时，新的连接会被拒绝。
'dispatch_mode' => 2,
'debug_mode'=> 1，
'daemonize' => false, 设置程序进入后台作为守护进程运行 长时间运行的服务器端程序必须启用此项。如果不启用守护进程，当ssh终端退出后，程序将被终止运行,所有输出会被丢弃
));

ipc_mode
描述：设置进程间的通信方式。
说明：共有三种通信方式，参数如下：
1 => 使用unix socket通信
2 => 使用消息队列通信
3 => 使用消息队列通信，并设置为争抢模式
示例：
'ipc_mode' => 1

dispatch_mode
描述：指定数据包分发策略。
说明：共有三种模式，参数如下：
1 => 轮循模式，收到会轮循分配给每一个worker进程
2 => 固定模式，根据连接的文件描述符分配worker。这样可以保证同一个连接发来
的数据只会被同一个worker处理
3 => 抢占模式，主进程会根据Worker的忙闲状态选择投递，只会投递给处于闲置状
态的Worker
示例：
'dispatch_mode' => 2

task_worker_num
描述：服务器开启的task进程数。
说明：设置此参数后，服务器会开启异步task功能。此时可以使用task方法投递异步任务。
设置此参数后，必须要给swoole_server设置onTask/onFinish两个回调函数，否则启动服
务器会报错。
示例：
'task_worker_num' => 8
```



### 函数只做部分解释

```
sned  指定发送的fd
说明：
data，发送的数据，最大不得超过2M。send操作具有原子性，多个进程同时调用send向
同一个连接发送数据，不会发生数据混杂。
如果要发送超过2M的数据，建议将数据写入临时文件，然后通过sendfile接口直接发送文
件
UDP协议，send会直接在worker进程内发送数据包
向UDP客户端发送数据，必须要传入from_id
发送成功会返回true，如果连接已被关闭或发送失败会返回false
swoole-1.6以上版本不需要from_id
UDP协议使用fd保存客户端IP，from_id保存from_fd和port
UDP协议如果是在onReceive后立即向客户端发送数据，可以不传from_id
附录*函数列表
55
样例:
$serv->send( $fd , "Hello World");

$udp_client = $serv->connection_info($fd, $from_id);  $from_id 来自哪个Reactor

task
说明： 指定投递给哪个task进程，默认随机投递
此功能用于将慢速的任务异步地去执行，调用task函数会立即返回，Worker进程可以继续处
理新的请求。
函数会返回一个 $task_id 数字，表示此任务的ID
任务完成后，可以通过return(低于swoole-1.7.2版本使用finish函数)将结果通过onFinish回调
返回给Worker进程。
发送的data必须为字符串，如果是数组或对象，请在业务代码中进行serialize处理，并
在onTask/onFinish中进行unserialize。
data可以为二进制数据，最大长度为8K(超过8K可以使用临时文件共享)。字符串可以使用
gzip进行压缩。
使用task必须为Server设置onTask和onFinish回调,并指定task_worker_num配置参数。
样例:
附录*函数列表
59
$task_id = $serv->task("some data");


```





