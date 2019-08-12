## 前言

  本文章会详细介绍php-fpm配置文件的各个配置值，以及对应关系，目前大多数PHP应用采用nginx + php-fpm 的方式去运作，所以了解php-fpm的配置对优化服务器会有很大的帮助


## 配置详解
   该配置为 Ubuntu 18.04 下 通过apt-get install 安装的php7.3-fpm生成的配置文件

1. 运行php-fpm的用户以及用户组配置
```bash
user = www-data		--运行php-fpm的用户
group = www-data	--运行php-fpm的用户组
```

2. 监听php-fpm的方式
```bash
listen = /run/php/php7.3-fpm.sock  --fpm监听的sock文件
listen = 127.0.0.1:9000			   --fpm监听的端口
```

监听php-fpm的方式有两种 第一次种是Unix domain socket模式 另外一种是TCP模式, TCP是使用TCP端口连接127.0.0.1:9000，Socket是使用unix domain socket连接套接字/dev/shm/php-cgi.sock，在服务器压力不大的情况下，tcp和socket差别不大，但在压力比较满的时候，用套接字方式，效果确实比较好。

从原理上来说，unix socket方式肯定要比tcp的方式快而且消耗资源少，因为socket之间在nginx和php-fpm的进程之间通信，而tcp需要经过本地回环驱动，还要申请临时端口和tcp相关资源。

当然还是从原理上来说，unix socket会显得不是那么稳定，当并发连接数爆发时，会产生大量的长时缓存，在没有面向连接协议支撑的情况下，大数据包很有可能就直接出错并不会返回异常。而TCP这样的面向连接的协议，多少可以保证通信的正确性和完整性。其实如果nginx做要做负载均衡的话，根本也不要考虑unix socket的方式了，只能采用TCP的方式。

3. backlog数
```bash
  listen.backlog = 1024  ---1表示无限制，由操作系统决定，此行注释掉就行。
```

4. 监听的用户以及用户组
```bash
listen.owner = www-data
listen.group = www-data
```

5. 允许监听的客户端 ip
```
listen.allowed_clients = 127.0.0.1
```







