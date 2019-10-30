# PHP捕获异常和错误


在`PHP`大致有三种类型的可捕获的异常/错误

1.  `Error`：`PHP`内核抛出错误的专用类型, 如类不存在, 函数不存在, 函数参数错误, 都会抛出此类型的错误, `PHP`代码中不应该使用`Error类`来作为异常抛出
2.  `Exception`：应用开发者应该使用的异常基类
3.  `ErrorException`：此异常基类专门负责将`PHP`的`Warning`/`Notice`等信息通过`set_error_handler`转换成异常, PHP未来的规划必然是将所有的`Warning`/`Notice`转为异常, 以便于`PHP`程序能够更好更可控地处理各种错误

> 以上所有类都实现了`Throwable`接口, 也就是说, 通过`try {} catch(Throwable $e) {}` 即可捕获所有可抛出的异常/错误

实例：

```php
try {

    要执行的代码.......

} catch (\Error $e){
    return ['status'=>'Error','error_msg'=>$e->getMessage()];
} catch (\Exception $e) {
    return ['status'=>'Exception','error_msg'=>$e->getMessage()];
} catch (\Throwable $e) {
    return ['status'=>'Throwable','error_msg'=>$e->getMessage()];
}

Exception
->getMessage 异常消息内容
->getPrevious 返回异常链中的前一个异常
->getCode 获取异常代码
->gitFile 获取发生异常的程序文件名称
->getLine 发生异常的代码行号
->getTrace 获取异常追踪信息
->__toString 将异常对象转换为字符串

Fatal 致命错误，脚本终止运行
Parse 编译时解析错误，语法错误，脚本终止运行
Warning 警告错误 脚本不终止运行
Notice 通知错误，脚本不终止运行
```


#  不可捕获的致命错误和异常

`PHP`错误的一个重要级别, 如异常/错误未捕获时, 内存不足时, 或是一些编译期错误(继承的类不存在), 将会以`E_ERROR`级别抛出一个`Fatal Error`, 是在程序发生不可回溯的错误时才会触发的, `PHP`程序无法捕获这样级别的一种错误, 只能通过`register_shutdown_function`在后续进行一些处理操作。

## register_shutdown_function

**register_shutdown_function()** 表示 PHP 在**程序结束**时触发某个函数行为。

如果仅仅是打印 log 日志，我们在php.ini文件中 log_error 开启，即可写入日志。
此注册函数是为了其他行为的方便操作，比如出现错误后程序自动抓取程序错误发送给管理员邮件或者短信功能。

程序结束有四种情况：

1. php代码执行过程中发生错误
2. php代码顺利执行成功
3. php代码运行超时
4. 页面被用户强制停止

以下情况不会执行回调函数：
1. 程序有语法错误；
2. 调用register_shutdown_function函数调用之前发生了致命错误；
3. 调用register_shutdown_function()函数之前有exit()函数调用，register_shutdown_function()函数将不能执行；

实例：
```php

register_shutdown_function(function() {
	 if ($error = error_get_last()) {
	        //必须是一个绝对路径
            $filename = '/var/log/php/error.php';
            $message = '时间 : ' . date('Y-m-d H:i:s') . PHP_EOL;
            $message .= '文件 : ' . $error['file'] . PHP_EOL;
            $message .= '行数 : ' . $error['line'] . PHP_EOL;
            $message .= '错误 : ' . $error['message'] . PHP_EOL;
            $message .= '类型 : ' . $error['type'] . PHP_EOL . PHP_EOL;
            file_put_contents($filename, $message, FILE_APPEND);

            var_dump($message);
        }
});

```
