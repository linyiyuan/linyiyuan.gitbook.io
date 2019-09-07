# 前言
在认识这个强大的神器之前我们先了解几个常见PHP内置函数内存相关常见常数，
1. memory_get_usage(): 返回当前分配给PHP脚本的内存量，单位是字节（byte）
2. memory_get_peak_usage(): 返回内存使用峰值
3. getrusage():返回CUP使用情况

我们在日常开发中可以使用这些内置函数来调试PHP代码性能。但需要注意的是这几个函数只能在linux系统中使用

在认识以上三个函数之后我们来做一个简单的小测, 我们迭代数组一个1-10000的数组
```php
$start_memory = memory_get_usage();

$numberArr = range(1, 1000);
foreach($numberArr as $key) {
  //echo $key;
}
$end_memory = memory_get_usage();

echo '运行该迭代数组所耗内存：'  . ($end_memory - $start_memory) . 'bytes';
```

运行结果：

    运行该迭代数组所耗内存：528440 bytes

折算成kb单位就是大概是528kb, 这样一看并没觉得什么，这时候我们继续增大数组范围，我们将范围扩大成1-1000000

运行结果：

    运行该迭代数组所耗内存：4198480 bytes

大概是4m左右，这时候还是可以遍历得出来，我们继续增大1-1000000

运行结果：

**Fatal error**: Allowed memory size of 134217728 bytes exhausted (tried to allocate 536870920 bytes) in **/data/default/index.php** on line **11**

发现报错了，系统提示内存溢出了 134217728 bytes 大概是134MB, 我们再查看一下PHP配置文件

![](http://images.linyiyuan.top/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190907150407.png
)

果不其然，配置文件限制的内存最大值是128 我们执行该程序的内存使用明显超出限制，这时候我们只能修改此配置文件使其限制内存更大些，但是如果遇到超级大的数据量，或者让你去处理一个几个G的Excel文件时，你会发现修改这个配置文件并不能从根本解决问题，配置内存还需要考虑到服务器的内存使用情况等等，

## 认识Yield
这时候我们就可以使用 `Yield` 这个强大的神器，下面我们使用`Yield` 优化我们上面的代码

```php
$start_memory = memory_get_usage();

function yield_range($start, $end) {
	while( $start <= $end ){
    $start++;
    yield $start;
  }
}

$numberArr = yield_range(1, 10000000);
foreach($numberArr as $key) {
  // echo $key;
}
$end_memory = memory_get_usage();

echo '运行该迭代数组所耗内存：'  . ($end_memory - $start_memory) . ' bytes';
```

运行一下，让你们感受下

    运行该迭代数组所耗内存：320 bytes

我擦，使用内存竟然连1kb都没有，那么，我们来分析一波儿这个神奇的yield_range函数。这个yield关键字到底返回的是什么？我们简单看一下：

```php
function yield_range( $start, $end ){
 while( $start <= $end ){
 $start++;
 yield $start;
 }
}
$rs = yield_range( 1, 100 );

var_dump( $rs );

```


     /*
     object(Generator)#1 (0) {
     }
     */

## Generator

`Yield` 返回的是一个叫做Generator（中文名就是生成器）的object对象，该对象是由generators(生成器)返回，不能通过new实例化，而这个生成器是实现了Iterator接口，该接口提供了一下几个方法：

1. abstract public [current](https://www.php.net/manual/zh/iterator.current.php) ( void ) : [mixed](https://www.php.net/manual/zh/language.pseudo-types.php#language.types.mixed)
2. abstract public [key](https://www.php.net/manual/zh/iterator.key.php) ( void ) : scalar
3. abstract public [next](https://www.php.net/manual/zh/iterator.next.php) ( void ) : void
4. abstract public [rewind](https://www.php.net/manual/zh/iterator.rewind.php) ( void ) : void
5. abstract public [valid](https://www.php.net/manual/zh/iterator.valid.php) ( void ) : bool

而Generator又包含一下几个方法：

*   [Generator::current](https://www.php.net/manual/zh/generator.current.php) — 返回当前产生的值
*   [Generator::key](https://www.php.net/manual/zh/generator.key.php) — 返回当前产生的键
*   [Generator::next](https://www.php.net/manual/zh/generator.next.php) — 生成器继续执行
*   [Generator::rewind](https://www.php.net/manual/zh/generator.rewind.php) — 重置迭代器
*   [Generator::send](https://www.php.net/manual/zh/generator.send.php) — 向生成器中传入一个值
*   [Generator::throw](https://www.php.net/manual/zh/generator.throw.php) — 向生成器中抛入一个异常
*   [Generator::valid](https://www.php.net/manual/zh/generator.valid.php) — 检查迭代器是否被关闭
*   [Generator::__wakeup](https://www.php.net/manual/zh/generator.wakeup.php) — 序列化回调


所以，既然实现了Iterator接口（也正是因为如此，这个东西可以使用foreach进行迭代，明白了吧？），所以可以有如下代码：

```php
<?php
function yield_range( $start, $end ){
 while( $start <= $end ){
 yield $start;
 $start++;
 }
}
$generator = yield_range( 1, 10 );

// valid() current() next() 都是Iterator接口中的方法

while( $generator->valid() ){
 echo $generator->current().PHP_EOL;
 $generator->next();
}
```


    1 2 3 4 5 6 7 8 9 10

  重点来了：这个yield_range函数似乎能够记住它上一次运行到哪儿了，上一次运行的结果是什么，然后紧接着在下一次运行的时候继续从上次终止的地方继续开始。这不是普通的PHP函数可以做得到的！

我们知道，操作系统在调度进程的时候，会触发一个叫做“进程上下文切换”的概念。比如CPU从进程A调度给进程B了，那么当再次从进程B调度给进程A的时候，当初进程A运行到哪儿了、临时的数据结果是什么都是需要被还原的，不然，一切都要从头，那就要出大问题了。而，这个yield关键字，似乎在用户态（非系统内核级）就可以实现这个概念.

接下来我们来认识一个Generator对象的一个方法 --send 
  
```php
<?php
function yield_range( $start, $end ){
   while( $start <= $end ){
   $ret = yield $start;
   $start++;
   echo "yield receive : ".$ret.PHP_EOL;
   }
}
$generator = yield_range( 1, 10 );
$generator->send( $generator->current() * 10 );
```

    //执行结果
    yield receive : 10

send方法可以修改yield的返回值 , 我们继续修改代码

```php
<?php
function yield_range( $start, $end ){
  while( $start <= $end ){
    $ret = yield $start;
    $start++;
    echo "yield receive : ".$ret . PHP_EOL;
  }
}
$generator = yield_range( 1, 10 );
foreach( $generator as $item ){
  $generator->send( $generator->current() * 10 );
}
```

结果发现

![](http://images.linyiyuan.top/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190907154456.png)

这是PHP存在的一个Bug, 我们需要注意的是我们在foreach 去使用生成器的send方法，以下是bug的链接，有兴趣的可以去看下

[https://bugs.php.net/bug.php?id=76104](https://bugs.php.net/bug.php?id=76104) [https://stackoverflow.com/questions/37817315/how-does-generatorsend-work](https://stackoverflow.com/questions/37817315/how-does-generatorsend-work)

## 引用文章

1. [PHP中的yield（上)](https://github.com/elarity/advanced-php/blob/master/17.%20PHP%E4%B8%AD%E7%9A%84yield%EF%BC%88%E4%B8%8A%EF%BC%89.md)
2. [生成器类](https://www.php.net/manual/zh/class.generator.php)
3. [Iterator（迭代器）接口](https://www.php.net/manual/zh/class.iterator.php)
4. [PHP内置函数memory_get_usage()获取内存使用和getrusage()返回CUP使用情况](https://blog.csdn.net/h330531987/article/details/78993933)


