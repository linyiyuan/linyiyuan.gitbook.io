# PHP7.0版本新特性

#### 1. 组合比较符 (<=>)

组合比较符号用于比较两个表达式。当$a小于、等于或大于$b时它分别返回-1、0或1，比较规则延续常规比较规则。对象不能进行比较

```
var_dump('PHP' <=> 'Node'); // int(1)
var_dump(123 <=> 456); // int(-1)
var_dump(['a', 'b'] <=> ['a', 'b']); // int(0)
```

#### 2. null合并运算符

由于日常使用中存在大量同时使用三元表达式和isset操作。使用null合并运算符可以简化操作

```
# php7以前
if(isset($_GET['a'])) {
  $a = $_GET['a'];
}
# php7以前
$a = isset($_GET['a']) ? $_GET['a'] : 'none';

#PHP 7
$a = $_GET['a'] ?? 'none';
```

#### 3. 变量类型声明

变量类型声明有两种模式。一种是强制的，和严格的。允许使用下列类型参数**int**、**string**、**float**、**bool**

同时不能再使用int、string、float、bool作为类的名字了

```
function sumOfInts(int ...$ints)
{
    return array_sum($ints);
}
var_dump(sumOfInts(2, '3', 4.1)); // int(9)
# 严格模式
declare(strict_types=1);

function add(int $x, int $y)
{
    return $x + $y;
}
var_dump(add('2', 3)); // Fatal error: Argument 1 passed to add() must be of the type integer

```

#### 5. 返回值类型声明

增加了返回类型声明，类似参数类型。这样更方便的控制函数的返回值.在函数定义的后面加上:类型名即可

```
function fun(int $a): array
{
  return $a;
}
fun(3);//Fatal error
```

#### 6. 匿名类

php7允许new class {} 创建一个匿名的对象。

```
//php7以前
class Logger
{
    public function log($msg)
    {
        echo $msg;
    }
}

$util->setLogger(new Logger());

// php7+
$util->setLogger(new class {
    public function log($msg)
    {
        echo $msg;
    }
});
```

#### 7. Unicode codepoint 转译语法

这接受一个以16进制形式的 Unicode codepoint，并打印出一个双引号或heredoc包围的 UTF-8 编码格式的字符串。 可以接受任何有效的 codepoint，并且开头的 0 是可以省略的

```
echo "\u{aa}";// ª
echo "\u{0000aa}";// ª
echo "\u{9999}";// 香
```

#### 8. Closure::call

闭包绑定 简短干练的暂时绑定一个方法到对象上闭包并调用它。

```
class A {private $x = 1;}

// PHP 7 之前版本的代码
$getXCB = function() {return $this->x;};
$getX = $getXCB->bindTo(new A, 'A'); // 中间层闭包
echo $getX();

// PHP 7+ 及更高版本的代码
$getX = function() {return $this->x;};
echo $getX->call(new A);
```

#### 9. 带过滤的unserialize

提供更安全的方式解包不可靠的数据。它通过白名单的方式来防止潜在的代码注入

```
// 将所有的对象都转换为 __PHP_Incomplete_Class 对象
$data = unserialize($foo, ["allowed_classes" => false]);

// 将除 MyClass 和 MyClass2 之外的所有对象都转换为 __PHP_Incomplete_Class 对象
$data = unserialize($foo, ["allowed_classes" => ["MyClass", "MyClass2"]);

// 默认情况下所有的类都是可接受的，等同于省略第二个参数
$data = unserialize($foo, ["allowed_classes" => true]);
```

#### 10. IntlChar类

这个类自身定义了许多静态方法用于操作多字符集的 unicode 字符。需要安装intl拓展

```

printf('%x', IntlChar::CODEPOINT_MAX);
echo IntlChar::charName('@');
var_dump(IntlChar::ispunct('!'));
```

#### 11. 预期

它使得在生产环境中启用断言为零成本，并且提供当断言失败时抛出特定异常的能力。以后可以使用这个这个进行断言测试

```
ini_set('assert.exception', 1);

class CustomError extends AssertionError {}

assert(false, new CustomError('Some error message'));
```

#### 12. 命名空间按组导入

从同一个命名空间下导入的类、函数、常量支持按组一次导入

```
#php7以前
use app\model\A;
use app\model\B;
#php7+
use app\model{A,B}
```

#### 13.生成器支持返回表达式

它允许在生成器函数中通过使用 _return_ 语法来返回一个表达式 （但是不允许返回引用值）， 可以通过调用 _Generator::getReturn()_ 方法来获取生成器的返回值， 但是这个方法只能在生成器完成产生工作以后调用一次。

```
$gen = (function() {
    yield 1;
    yield 2;

    return 3;
})();

foreach ($gen as $val) {
    echo $val, PHP_EOL;
}
# output
//1
//2

echo $gen->getReturn(), PHP_EOL;
# output

//3
```

#### 14.生成器委派

现在，只需在最外层生成其中使用yield from，就可以把一个生成器自动委派给其他的生成器

```
function gen()
{
    yield 1;
    yield 2;

    yield from gen2();
}

function gen2()
{
    yield 3;
    yield 4;
}

foreach (gen() as $val)
{
    echo $val, PHP_EOL;
}
```

#### 15.整数除法函数intdiv

```
var_dump(intdiv(10,3)) //3
```

#### 16.会话选项设置

session_start() 可以加入一个数组覆盖php.ini的配置

```
session_start([
    'cache_limiter' => 'private',
    'read_and_close' => true,
]);
```

#### 18. 随机数、随机字符函数

```
string random_bytes(int length);
int random_int(int min, int max);
```

#### 19. define 支持定义数组

```
#php7+
define('ALLOWED_IMAGE_EXTENSIONS', ['jpg', 'jpeg', 'gif', 'png']);
```
