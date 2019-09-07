# 前言
在上一章中我们介绍了`Yield`的概念以及使用方法，在这一章节我们继续深入了解`Yield`的原理以及应用场景

# 深入解析

在上一节中我们使用了memory_get_usage 来检测使用`Yield`时的内存使用情况，可以发现使用`Yield`优化的代码其内存使用量非常少，这也引发了一个问题，为什么使用`Yield`时其内存使用少，我们可以使用以下代码对其进行测试

首先我们先简单实现一个简单的函数

```php
function createRange($number){
    $data = [];
    for($i=0;$i<$number;$i++){
        $data[] = time();
    }
    return $data;
}
```

这是一个非常常见的PHP函数，我们在处理一些数组的时候经常会使用。这里的代码也非常简单：

1.  我们创建一个函数。
2.  函数内包含一个`for`循环，我们循环的把当前时间放到`$data`里面
3.  `for`循环执行完毕，把`$data`返回出去。

下面没完，我们继续。我们再写一个函数，把这个函数的返回值循环打印出来：

```
$result = createRange(10); // 这里调用上面我们创建的函数
foreach($result as $value){
    sleep(1);//这里停顿1秒，我们后续有用
    echo $value.'<br />';
}
```

运行结果：
![](http://images.linyiyuan.top/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190907161352.png)

从图中可以发现，程序运行了10秒， 其遍历的数组全部一致，从结果上看并没有问题，现在我们使用`Yield`生成器对 createRange() 函数进行修改：

```php
function createRange($number){
    for($i=0;$i<$number;$i++){
        yield time();
    }
}
```

运行结果：
![](http://images.linyiyuan.top/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190907161926.png)

从结果上我们可以发现，两次输出值不一样，使用了`Yield`的结果上每次输出的时间戳都多了1秒 这里的间隔一秒其实就是`sleep(1)`造成的后果。但是为什么第一次没有间隔？那是因为：

*   未使用生成器时：`createRange`函数内的`for`循环结果被很快放到`$data`中，并且立即返回。所以，`foreach`循环的是一个固定的数组。
*   使用生成器时：`createRange`的值不是一次性快速生成，而是依赖于`foreach`循环。`foreach`循环一次，`for`执行一次。


我们来还原一下代码执行过程。

1.  首先调用`createRange`函数，传入参数`10`，但是`for`值执行了一次然后停止了，并且告诉`foreach`第一次循环可以用的值。
2.  `foreach`开始对`$result`循环，进来首先`sleep(1)`，然后开始使用`for`给的一个值执行输出。
3.  `foreach`准备第二次循环，开始第二次循环之前，它向`for`循环又请求了一次。
4.  `for`循环于是又执行了一次，将生成的时间戳告诉`foreach`.
5.  `foreach`拿到第二个值，并且输出。由于`foreach`中`sleep(1)`，所以，`for`循环延迟了1秒生成当前时间

所以，整个代码执行中，始终只有一个记录值参与循环，内存中也只有一条信息。

无论开始传入的`$number`有多大，由于并不会立即生成所有结果集，所以内存始终是一条循环的值。

# 概念理解
到这里，你应该已经大概理解什么是生成器了。下面我们来说下生成器原理。

首先明确一个概念：**生成器yield关键字不是返回值，他的专业术语叫产出值，只是生成一个值**

那么代码中`foreach`循环的是什么？其实是PHP在使用生成器的时候，会返回一个`Generator`类的对象。`foreach`可以对该对象进行迭代，每一次迭代，PHP会通过`Generator`实例计算出下一次需要迭代的值。这样`foreach`就知道下一次需要迭代的值了。

而且，在运行中`for`循环执行后，会立即停止。等待`foreach`下次循环时候再次和`for`索要下次的值的时候，`for`循环才会再执行一次，然后立即再次停止。直到不满足条件不执行结束。

# 应用场景
从上一章节中我们可以了解到，使用`Yield` 可以大大减少PHP使用的内存大小，PHP开发的时候很多时候都要读取大文件，比如CSV文件，txt文件，或者一些日志文件，如果文件大小没那么大的情况下我们可以直接一次性将所有内容读取到内存中计算，但是如果文件过大，比如5个G的文件，这时候我们使用直接一次性把所有的内容读取到内存中计算不太现实，所以我们可以使用`Yield`来帮助我们处理这些大型文件

简单看个例子：读取txt文件：

![图片描述](https://segmentfault.com/img/bVZT02?w=339&h=256 "图片描述")

我们创建一个text文本文档，并在其中输入几行文字，示范读取。

```
<?php
header("content-type:text/html;charset=utf-8");
function readTxt()
{
    # code...
    $handle = fopen("./test.txt", 'rb');

    while (feof($handle)===false) {
        # code...
        yield fgets($handle);
    }

    fclose($handle);
}

foreach (readTxt() as $key => $value) {
    # code...
    echo $value.'<br />';
}
```

![图片描述](https://segmentfault.com/img/bVZT1d?w=343&h=343 "图片描述")

通过上图的输出结果我们可以看出代码完全正常。

但是，背后的代码执行规则却一点儿也不一样。使用生成器读取文件，第一次读取了第一行，第二次读取了第二行，以此类推，**每次被加载到内存中的文字只有一行**，大大的减小了内存的使用。

这样，即使读取上G的文本也不用担心，完全可以像读取很小文件一样编写代码。


# 引用文章
1. [PHP中被忽略的性能优化利器：生成器](https://segmentfault.com/a/1190000012334856)
2. [PHP YIELD使读取大文件](https://www.jianshu.com/p/42aea4302460)