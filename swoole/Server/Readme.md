## 导言
Swoole 的Server可以创建一个异步服务器程序，支持TCP，UDP，UNIXSOCKET 3种协议，支持IPV4，IPV6，支持SSL/TLS单向双向证书的隧道加密。

注意： **请勿在使用Server创建之前调用其他异步IO的API，否则将会创建失败。可以在Server启动后OnWorkerStart回调函数中使用**

 > Server 只能作用于php-cli环境中，其他环境会抛出致命错误


## 起步

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

## 运行流程图

![Swoole扩展架构图](https://wiki.swoole.com/static/uploads/swoole.jpg)

## 进程/线程结构图

![Swoole进程/线程结构图](https://wiki.swoole.com/static/image/process.jpg) ![进程/线程结构图2](https://wiki.swoole.com/static/uploads/wiki/201808/03/635680420659.png "进程/线程结构图2")
