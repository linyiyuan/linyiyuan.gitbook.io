# 导言

Http\Server 是Server的子类，继承父类Server，所以它也同样的继承了父类的一些属性跟方法，需要注意的是Http\Server对Http的支持并不完整，建议仅作为应用服务器。并在前端  增加Nginx代理，配合Nginx一起使用,
`Http\Server`支持同步和异步2种模式。

## 同步模式

这种模式等同于`nginx+php-fpm/apache`，它需要设置大量`Worker`进程来完成并发请求处理。`Worker`进程内可以使用同步阻塞`IO`，编程方式与普通`PHP` `Web`程序完全一致。

与`php-fpm/apache`不同的是，客户端连接并不会独占进程，服务器依然可以应对大量并发连接。

## 异步模式

这种模式下整个服务器是异步非阻塞的，服务器可以应对大规模的并发连接和并发请求。但编程方式需要完全使用异步`API`，如`MySQL`、`redis`、`http_client`、`file_get_contents`、`sleep`等阻塞`IO`操作必须切换为异步的方式，如异步`Client`，`Event`，`Timer`等`API`。

# 性能

通过使用`apache bench`工具进行压力测试，在`Inter Core-I5 4`核 + `8G`内存的普通PC机器上，`Http\Server`可以达到近`11`万`QPS`。远远超过`php-fpm`，`Golang`、`Node.js`自带`Http`服务器。性能几乎接近与`Nginx`的静态文件处理。

```
ab -c 200 -n 200000 -k http://127.0.0.1:9501/
```

# Nginx+Swoole配置

```
server {
    root /data/wwwroot/;
    server_name local.swoole.com;

    location / {
        proxy_http_version 1.1;
        proxy_set_header Connection "keep-alive";
        proxy_set_header X-Real-IP $remote_addr;
        if (!-e $request_filename) {
             proxy_pass http://127.0.0.1:9501;
        }
    }
}

```

> 通过读取`$request->header['x-real-ip']`来获取客户端的真实`IP`

# 常见问题

## 1. CURL发送POST请求服务器端超时

`CURL`在发送较大的`POST`请求时会先发一个`100-continue`的请求，如果收到服务器的回应才会发送实际的`POST`数据。而`swoole_http_server`不支持`100-continue`，就会导致CURL请求超时。

解决办法是关闭`CURL`的`100-continue`

```
// 创建一个新cURL资源
$ch = curl_init();
// 设置URL和相应的选项
curl_setopt($ch, CURLOPT_URL, "http://127.0.0.1:9501");
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_POST, 1); //设置为POST方式
curl_setopt($ch, CURLOPT_HTTPHEADER, array('Expect:'));
curl_setopt($ch, CURLOPT_POSTFIELDS, array('test' => str_repeat('a', 800000)));//POST数据
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
```


如果客户端是其他语言编写的，无法修改客户端去掉100-continue，那么还有2个方案可以解决此问题。

*   使用Nginx做前端代理，由Nginx处理100-Continue
*   重新编译Swoole启用100-Continue的支持，需要手工修改swoole_config.h，找到SW_HTTP_100_CONTINUE，去掉注释，执行make clean && make install

> 启用100-CONTINUE后会额外消耗服务器的CPU资源

## 2. 使用Chrome访问服务器会产生2次请求

这是因为`Chrome`浏览器会自动请求一次`favicon.ico`，所以服务器会收到`2`个`Request`，通过打印`$req->server['request_uri']` 能看到请求的`URL`路径。

## 屏蔽 favicon.ico

```
$uri = $req->server['request_uri'];
if ($uri == '/favicon.ico') {
    $res->status(404);
    $res->end();
}

```

> 上述示例代码中，$req 为 Http\Request 对象，$res 为 Http\Response 对象

## 3.GET/POST请求的最大尺寸

### GET请求最大8192

GET请求只有一个Http头，swoole底层使用固定大小的内存缓存区8K，并且不可修改。如果请求不是正确的Http请求，将会出现错误。底层会抛出以下错误：

```
WARN swReactorThread_onReceive_http_request: http header is too long.

```

### POST/文件上传

最大尺寸受到 `package_max_length` 配置项限制，默认为2M，可以调用`swoole_server->set`传入新的值修改尺寸。swoole底层是全内存的，因此如果设置过大可能会导致大量并发请求将服务器资源耗尽。 计算方法：`最大内存占用` = `最大并发请求数` * `package_max_length`
