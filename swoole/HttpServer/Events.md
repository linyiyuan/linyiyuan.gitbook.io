# Events

注册事件回调函数，与`Server->on`相同，不同之处是：

*   `Http\Server->on`不接受`onConnect`/`onReceive`回调设置
*   `Http\Server->on`额外接受1种新的事件类型`onRequest`
* 
## onRequest事件

```
$http_server->on('request', function(swoole_http_request $request, swoole_http_response $response) {
     $response->end("<h1>hello swoole</h1>");
})

```

在收到一个完整的`Http`请求后，会回调此函数。回调函数共有`2`个参数：

*   `$request`，`Http`请求信息对象，包含了`header/get/post/cookie`等相关信息
*   `$response`，`Http`响应对象，支持`cookie/header/status`等`Http`操作
*   在`onRequest`回调函数返回时底层会销毁`$request`和`$response`对象，如果未执行`$response->end()`操作，底层会自动执行一次`$response->end("")`

> `onRequest`在`1.7.7`或更高版本可用
> `$response/$request`对象传递给其他函数时，不要加`&`引用符号
> `$response/$request`对象传递给其他函数后，引用计数会增加，`onRequest`退出时不会销毁


## \Http\Request 

`Http`请求对象，保存了`Http`客户端请求的相关信息，包括`GET`、`POST`、`COOKIE`、`Header`等。

*   `Request`对象销毁时会自动删除上传的临时文件
*   请勿使用`&`符号引用`$request`对象

1. [Http\Request->$header](https://wiki.swoole.com/wiki/page/332.html)
2. [Http\Request->$server](https://wiki.swoole.com/wiki/page/341.html)
3. [Http\Request->$get](https://wiki.swoole.com/wiki/page/333.html)
4. [Http\Request->$post](https://wiki.swoole.com/wiki/page/334.html)
5. [Http\Request->$cookie](https://wiki.swoole.com/wiki/page/335.html)
6. [Http\Request->$files](https://wiki.swoole.com/wiki/page/428.html)
7. [Http\Request->rawContent](https://wiki.swoole.com/wiki/page/375.html)
8. [Http\Request->getData](https://wiki.swoole.com/wiki/page/876.html)

## \Http\Response
`Http`响应对象，通过调用此对象的方法，实现`Http`响应发送。

*   当`Response`对象销毁时，如果未调用`end`发送`Http`响应，底层会自动执行`end`
*   请勿使用`&`符号引用`$response`对象

1.   [Http\Response->header](https://wiki.swoole.com/wiki/page/336.html)
2.   [Http\Response->cookie](https://wiki.swoole.com/wiki/page/337.html)
3.   [Http\Response->status](https://wiki.swoole.com/wiki/page/338.html)
4.   [Http\Response->gzip](https://wiki.swoole.com/wiki/page/410.html)
5.   [Http\Response->redirect](https://wiki.swoole.com/wiki/page/927.html)
6.   [Http\Response->write](https://wiki.swoole.com/wiki/page/403.html)
7.   [Http\Response->sendfile](https://wiki.swoole.com/wiki/page/540.html)
8.   [Http\Response->end](https://wiki.swoole.com/wiki/page/339.html)
9.   [Http\Response->detach](https://wiki.swoole.com/wiki/page/925.html)
11.   [Http\Response->upgrade](https://wiki.swoole.com/wiki/page/1116.html)
12.   [Http\Response->recv](https://wiki.swoole.com/wiki/page/1117.html)
13.   [Http\Response->push](https://wiki.swoole.com/wiki/page/1118.html)
14.   [Http\Response->close](https://wiki.swoole.com/wiki/page/1530.html)
15.   [Http\Response::create](https://wiki.swoole.com/wiki/page/926.html)


## Option
*   [upload_tmp_dir](https://wiki.swoole.com/wiki/page/p-upload_tmp_dir.html)
*   [http_parse_post](https://wiki.swoole.com/wiki/page/p-http_parse_post.html)
*   [http_parse_cookie](https://wiki.swoole.com/wiki/page/1098.html)
*   [http_compression](https://wiki.swoole.com/wiki/page/p-http_compression.html)
*   [document_root](https://wiki.swoole.com/wiki/page/783.html)
*   [enable_static_handler](https://wiki.swoole.com/wiki/page/p-enable_static_handler.html)
*   [static_handler_locations](https://wiki.swoole.com/wiki/page/p-static_handler_locations.html)
