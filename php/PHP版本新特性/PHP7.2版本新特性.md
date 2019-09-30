# PHP7.2版本新特性


## PHP7.2

### PHP7.2新特性

#### 1\. 增加新的类型object

```
function test(object $obj) : object
{
    return new SplQueue();
}

test(new StdClass());

```

#### 2\. 通过名称加载扩展

扩展文件不再需要通过文件加载 (Unix下以_.so_为文件扩展名，在Windows下以 _.dll_ 为文件扩展名) 进行指定。可以在php.ini配置文件进行启用

```
; ini file
extension=php-ast
zend_extension=opcache

```

#### 3.允许重写抽象方法

当一个抽象类继承于另外一个抽象类的时候，继承后的抽象类可以重写被继承的抽象类的抽象方法。

```
<?php

abstract class A
{
    abstract function test(string $s);
}
abstract class B extends A
{
    // overridden - still maintaining contravariance for parameters and covariance for return
    abstract function test($s) : int;
}

```

#### 4\. 使用Argon2算法生成密码散列

Argon2 已经被加入到密码散列（password hashing） API (这些函数以 _password__ 开头), 以下是暴露出来的常量

#### 5\. 新增 PDO 字符串扩展类型

当你准备支持多语言字符集，PDO的字符串类型已经扩展支持国际化的字符集。以下是扩展的常量：

*   **PDO::PARAM_STR_NATL**
*   **PDO::PARAM_STR_CHAR**
*   **PDO::ATTR_DEFAULT_STR_PARAM**

```
$db->quote('über', PDO::PARAM_STR | PDO::PARAM_STR_NATL);

```

#### 6\. 命名分组命名空间支持尾部逗号

```
use Foo\Bar\{
    Foo,
    Bar,
    Baz,
};

```

### PHP7.2 变更

#### 1\. number_format 返回值

```
var_dump(number_format(-0.01)); // now outputs string(1) "0" instead of string(2) "-0"

```

#### 2\. get_class()不再允许null。

```
var_dump(get_class(null))// warning

```

#### 4\. count 作用在不是 Countable Types 将发生warning

```
count(1), // integers are not countable

```

#### 5\. 不带引号的字符串

在之前不带引号的字符串是不存在的全局常量，转化成他们自身的字符串。现在将会产生waring。

```
var_dump(HEELLO);

```

#### 6\. __autoload 被废弃

__autoload方法已被废弃

#### 7\. each 被废弃

使用此函数遍历时，比普通的 _foreach_ 更慢， 并且给新语法的变化带来实现问题。因此它被废弃了。

#### 8\. is_object、gettype修正

is_object 作用在**__PHP_Incomplete_Class** 将反正 true

gettype作用在闭包在将正确返回resource

#### 9\. Convert Numeric Keys in Object/Array Casts

把数组转对象的时候，可以访问到整型键的值。

```
// array to object
$arr = [0 => 1];
$obj = (object)$arr;
var_dump(
    $obj,
    $obj->{'0'}, // now accessible
    $obj->{0} // now accessible
);
```
