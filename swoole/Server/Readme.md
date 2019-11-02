# 导言
Swoole 的Server可以创建一个异步服务器程序，支持TCP，UDP，UNIXSOCKET 3种协议，支持IPV4，IPV6，支持SSL/TLS单向双向证书的隧道加密。

注意： **请勿在使用Server创建之前调用其他异步IO的API，否则将会创建失败。可以在Server启动后OnWorkerStart回调函数中使用**

 > Server 只能作用于php-cli环境中，其他环境会抛出致命错误

# 起步

### 构建Server对象
第一步先构建一个Server对象

```php
$server = new Swoole\Server('0.0.0.0', 9501, SWOOLE_BASE, SWOOLE_SOCK_TCP);
```

### 设置运行时的参数

```php
$server->set([
   'worker_num'  => 4,    //进程数
   'daemonize' => true,   //是否在后台运行
   'backlog' => 128       //Listen队列长度
]);
```

###  注册事件回调函数

```php
  $serv->on('Connect', 'my_onConnect');
  $serv->on('Receive', 'my_onReceive');
  $serv->on('Close', 'my_onClose');
```

### 启动服务

```php
  $server->start();
```

### 属性列表

```php
    $serv->manager_pid;  //管理进程的PID，通过向管理进程发送SIGUSR1信号可实现柔性重启
    $serv->master_pid;  //主进程的PID，通过向主进程发送SIGTERM信号可安全关闭服务器
    $serv->connections; //当前服务器的客户端连接，可使用foreach遍历所有连接
```


# 生命周期

开发`Server`程序与普通`LAMP`的`Web`编程有本质区别。在传统的`Web`编程中，`PHP`程序员只需要关注`request`到达，`request`结束即可。而在`Server`程序中程序员可以操控更大范围，变量/对象可以有四种生存周期。

> 变量、对象、资源、`require/include`的文件等下面统称为对象

## 程序全局期

在`Server->start`之前就创建好的对象，我们称之为程序全局生命周期。这些变量在程序启动后就会一直存在，直到整个程序结束运行才会销毁。

有一些服务器程序可能会连续运行数月甚至数年才会关闭/重启，那么程序全局期的对象在这段时间持续驻留在内存中的。程序全局对象所占用的内存是`Worker`进程间共享的，不会额外占用内存。

这部分内存会在写时分离（`COW`），在`Worker`进程内对这些对象进行写操作时，会自动从共享内存中分离，变为**进程全局**对象。

> 程序全局期`include`/`require`的代码，必须在整个程序`shutdown`时才会释放，`reload`无效

## 进程全局期

`Server`启动后会创建多个进程，每个`Worker`子进程处理的请求数超过`max_request`配置后，就会自动销毁。`Worker`进程启动后创建的对象（`onWorkerStart`中创建的对象），在这个子进程存活周期之内，是常驻内存的。`onConnect/onReceive/onClose` 中都可以去访问它。

> 进程全局对象所占用的内存是在当前子进程内存堆的，并非共享内存。对此对象的修改仅在当前`Worker`进程中有效
> 进程期include/require的文件，在`reload`后就会重新加载

## 会话期

会话期是在`onConnect`后创建，或者在第一次`onReceive`时创建，`onClose`时销毁。一个客户端连接进入后，创建的对象会常驻内存，直到此客户端离开才会销毁。

在`LAMP`中，一个客户端浏览器访问多次网站，就可以理解为会话期。但传统`PHP`程序，并不能感知到。只有单次访问时使用`session_start`，访问`$_SESSION`全局变量才能得到会话期的一些信息。

`Server`中会话期的对象直接是常驻内存，不需要`session_start`之类操作。可以直接访问对象，并执行对象的方法。

## 请求期

请求期就是指一个完整的请求发来，也就是`onReceive`收到请求开始处理，直到返回结果发送`response`。这个周期所创建的对象，会在请求完成后销毁。

请求期对象与普通`PHP`程序中的对象就是一样的。请求到来时创建，请求结束后销毁。

## 运行流程图

![Swoole扩展架构图](https://wiki.swoole.com/static/uploads/swoole.jpg)

## 进程/线程结构图

![Swoole进程/线程结构图](https://wiki.swoole.com/static/image/process.jpg) ![进程/线程结构图2](https://wiki.swoole.com/static/uploads/wiki/201808/03/635680420659.png "进程/线程结构图2")

