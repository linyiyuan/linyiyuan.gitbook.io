## PHP 7.3

#### 1\. 灵活的heredoc 和nowdoc

在php 7.3 之前我们定义一大段的字符串。需要用到heredoc

```
<?php 
    $a = <<<H
    hello world
H;

```

结束标记必须在新行的开头。在php7.3 我们可以就不用受那个限制了

```
<?php
    $a = <<<H
    hello world
    H;

```

### 2\. 函数后面支持尾逗号

```
function fn($a,$b,$c)
{

}

fn(1,2,3,)//最后一个参数后面可以加逗号

```

### 3\. JSON_THROW_ON_ERROR

在php3 之前我们解析json的时候，`json_decode` 、`json_encode` 会返回失败 我们会通过json_last_error 获取错误的信息 。在php7.3 我们可以通过异常来获取

```
#php 7.3 之前

$res = json_decode($jsonString,true);
if(json_last_error() !== JSON_ERROR_NONE) {
    echo json_last_error_msg();
}
# php 7.3

try{
    json_decode("invalid json", null, 512, JSON_THROW_ON_ERROR);

}catch($e){

}

```

### 4\. is_countable 函数

在 PHP 7.2 中，用 count() 获取对象和数组的数量。如果对象不可数，PHP 会抛出警告⚠️ 。所以需要检查对象或者数组是否可数。 PHP 7.3 提供新的函数 is_countable() 来解决这个问题。

该 RFC 提供新的函数 is_countable()，对数组类型或者实现了 `Countable` 接口的实例的变量返回 true 。

之前:

```
if (is_array($foo) || $foo instanceof Countable) {
   // $foo 是可数的
}

```

之后:

```
if (is_countable($foo)) {
   // $foo 是可数的
}

```

### 5\. 新增数组函数 array_key_first(), array_key_last()

```
$array = ['a'=>'1','b'=>'2'];
#php 7.3之前
$firstKey  = key(reset($array));
# php 7.3
$firstKey = array_key_first($array);//a
$lastKey = array_key_last($array);//b

```

### 6.废除并移除大小写不敏感的常量

你可以同时使用大小写敏感和大小写不敏感的常量。但大小写不敏感的常量会在使用中造成一点麻烦。所以，为了解决这个问题，PHP 7.3 废弃了大小写不敏感的常量。

+

原先的情况是：

*   类常量始终为「大小写敏感」。
*   使用 `const` 关键字定义的全局常量始终为「大小写敏感」。注意此处仅仅是常量自身的名称，不包含命名空间名的部分，PHP 的命名空间始终为「大小写不敏感」。
*   使用 `define()` 函数定义的常量默认为「大小写敏感」。
*   使用 `define()` 函数并将第三个参数设为 `true` 定义的常量为「大小写不敏感」。

如今 PHP 7.3 提议废弃并移除以下用法：

*   In PHP 7.3: 废弃使用 `true` 作为 `define()` 的第三个参数。
*   In PHP 7.3: 废弃使用与定义时的大小写不一致的名称，访问大小写不敏感的常量。`true`、`false` 以及 `null` 除外。