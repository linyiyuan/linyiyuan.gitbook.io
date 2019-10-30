### PHP7.1新特性

#### 1\. 可为空（Nullable）类型

参数以及返回值的类型现在可以通过在类型前加上一个问号使之允许为空。当启用这个特性时，传入的参数或者函数返回的结果要么是给定的类型，要么是null

```
#php5
function($a = null){
  if($a===null) {
    return null;
  }
  return $a;
}
#php7+
function fun() :?string
{
  return null;
}

function fun1(?$a)
{
  var_dump($a);
}
fun1(null);//null
fun1('1');//1

```

#### 2\. void 类型

返回值声明为 void 类型的方法要么干脆省去 return 语句。对于 void来说，**NULL** 不是一个合法的返回值。

```
function fun() :void
{
  echo "hello world";
}

```

#### 3\. 类常量可见性

```
class Something
{
    const PUBLIC_CONST_A = 1;
    public const PUBLIC_CONST_B = 2;
    protected const PROTECTED_CONST = 3;
    private const PRIVATE_CONST = 4;
}

```

#### 4\. iterable 伪类

这可以被用在参数或者返回值类型中，它代表接受数组或者实现了**Traversable**接口的对象.

```
function iterator(iterable $iter)
{
    foreach ($iter as $val) {
        //
    }
}

```

#### 5\. 多异常捕获处理

一个catch语句块现在可以通过管道字符(_|_)来实现多个异常的捕获。 这对于需要同时处理来自不同类的不同异常时很有用

```
try {
    // some code
} catch (FirstException | SecondException $e) {
    // handle first and second exceptions
}

```

#### 6\. list支持键名

```
$data = [
    ["id" => 1, "name" => 'Tom'],
    ["id" => 2, "name" => 'Fred'],
];

// list() style
list("id" => $id1, "name" => $name1) = $data[0];
var_dump($id1);//1

```

#### 7\. 字符串支持负向

```
$a= "hello";
$a[-2];//l

```

#### 8\. 将callback 转闭包

Closure新增了一个静态方法，用于将callable快速地 转为一个Closure 对象。

```
<?php
class Test
{
    public function exposeFunction()
    {
        return Closure::fromCallable([$this, 'privateFunction']);
    }

    private function privateFunction($param)
    {
        var_dump($param);
    }
}

$privFunc = (new Test)->exposeFunction();
$privFunc('some value');

```

#### 9\. http2 服务推送

对http2服务器推送的支持现在已经被加入到 CURL 扩展

### PHP7.1变更

#### 1\. 传递参数过少时将抛出错误

过去我们传递参数过少 会产生warning。php7.1开始会抛出error

#### [](https://xianyunyh.gitbooks.io/php-interview/PHP/php7.html#2-%E7%A7%BB%E9%99%A4%E4%BA%86extmcrypt%E6%8B%93%E5%B1%95)2\. 移除了ext/mcrypt拓展