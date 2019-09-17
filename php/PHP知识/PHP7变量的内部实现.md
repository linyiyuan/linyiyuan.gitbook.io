# 导言
在学习PHP垃圾回收机制过程，跌跌撞撞，网上大多数文章都是基于PHP5.x环境下，就连官方文档也一样，在学习PHP垃圾回收过程中，由于PHP7.0采用全新的Zval结构，所以针对网上大多数文章都所给的参考都比较参差不齐，在总结了网上相关文章后总结出这篇关于PHP变量的内存管理文章，文章主要讲解PHP7变量的内存管理，关于变量的内部实现可以参考《PHP内核解析 -- 变量的内部实现》  这篇文章有助于理解PHP7新的Zval结构，变量内存管理，垃圾回收机制

<!--less-->


# 1. Zval

在任何一个语言中都存在着变量，变量是一个语言实现的基础, 而变量有两个组成部分：变量名、变量值 而 `Zval` 就是PHP语言中变量基础结构, 变量有两个组成部分：变量名、变量值，PHP中可以将其对应为：zval、zend_value，这两个概念一定要区分开，PHP中变量的内存是通过引用计数进行管理的，而且PHP7中引用计数是在zend_value而不是zval上，变量之间的传递、赋值通常也是针对zend_value，下面我先简单了解下Zval的基础结构

## 1.1 Zval基础结构
```c
//zend_types.h
typedef struct _zval_struct     zval;

typedef union _zend_value {
    zend_long         lval;    //int整形
    double            dval;    //浮点型
    zend_refcounted  *counted;
    zend_string      *str;     //string字符串
    zend_array       *arr;     //array数组
    zend_object      *obj;     //object对象
    zend_resource    *res;     //resource资源类型
    zend_reference   *ref;     //引用类型，通过&$var_name定义的
    zend_ast_ref     *ast;     //下面几个都是内核使用的value
    zval             *zv;
    void             *ptr;
    zend_class_entry *ce;
    zend_function    *func;
    struct {
        uint32_t w1;
        uint32_t w2;
    } ww;
} zend_value;

struct _zval_struct {
    zend_value        value; //变量实际的value
    union {
        struct {
            ZEND_ENDIAN_LOHI_4( //这个是为了兼容大小字节序，小字节序就是下面的顺序，大字节序则下面4个顺序翻转
                zend_uchar    type,         //变量类型
                zend_uchar    type_flags,  //类型掩码，不同的类型会有不同的几种属性，内存管理会用到
                zend_uchar    const_flags,
                zend_uchar    reserved)     //call info，zend执行流程会用到
        } v;
        uint32_t type_info; //上面4个值的组合值，可以直接根据type_info取到4个对应位置的值
    } u1;
    union {
        uint32_t     var_flags;
        uint32_t     next;                 //哈希表中解决哈希冲突时用到
        uint32_t     cache_slot;           /* literal cache slot */
        uint32_t     lineno;               /* line number (for ast nodes) */
        uint32_t     num_args;             /* arguments number for EX(This) */
        uint32_t     fe_pos;               /* foreach position */
        uint32_t     fe_iter_idx;          /* foreach iterator index */
    } u2; //一些辅助值
};
```

对于该结构的详细描述来自于《PHP内核解析》，`zval`结构比较简单，内嵌一个union类型的`zend_value`保存具体变量类型的值或指针，`zval`中还有两个union：`u1`、`u2`  我们重点理解 `zend_value`  从`zend_value`可以看出，除`long`、`double`类型直接存储值外，其它类型都为指针，指向各自的结构。

## 1.2 引用类型

在本文章我们重点讲解 `引用` 这个特殊类型，而其他类型如：`字符串` `数组`等均不作详细讲解，引用《PHP内核解析》的一段话  “引用是PHP中比较特殊的一种类型，它实际是指向另外一个PHP变量，对它的修改会直接改动实际指向的zval，可以简单的理解为C中的指针，在PHP中通过`&`操作符产生一个引用变量，也就是说不管以前的类型是什么，`&`首先会创建一个`zend_reference`结构，其内嵌了一个zval，这个zval的value指向原来zval的value(如果是布尔、整形、浮点则直接复制原来的值)，然后将原zval的类型修改为IS_REFERENCE，原zval的value指向新创建的`zend_reference`结构”
```php
struct _zend_reference {
    zend_refcounted_h gc;
    zval              val;
};
```

其中 `zend_reference` 内存拥有一个自己的value 还有一个zend_fefcounted_h(引用计数器), 而我们的垃圾回收机制也是基于此，现在让我们来看一个官方例子：
```php
  $a = "time:" . time();      //$a    -> zend_string_1(refcount=1)
  $b = &$a;                   //$a,$b -> zend_reference_1(refcount=2) ->     zend_string_1(refcount=1)
```

首先我们声明了一个变量$a并赋值 “time() . time()”, 此时内存分配给了$a一个`zval_struct`容器 并且容器value指向 `zend_string` 此时该`zend_string` 的refcount(引用次数) 为1，然后我们继续声明了$b 变量 并引用赋值$a, 此时$b 也拥有了属于她的`zval_struct` 容器，由于是引用传递，所以`&`首先会创建一个`zend_reference`结构 然后该结构的`zend_reference.value`指向$a 所指向的`zend_string` , 紧接着$a 的 `zval_struct.value` 的数据类型会被修改为 `zend_refrence`并将 $a  `zval_struct.value` 指向刚刚申请并初始化后的 `zend_reference` 最后为新变量申请 `zval_struct` 结构，将他的 `value` 指向刚刚创建的 `zend_reference`， 此时：$a 和 $b 都拥有一个 `zval_struct` 结构体，并且他们的 `zval_struct.value` 都指向了同一个 `zend_reference` 结构，所以该结构的引用计数器的值为 2。

    题外话：zend_reference 又指向了一个整形或浮点型的 value，如果指向的 value 类型是 zend_string，那么该 value 引用计数器的值为 1。而 xdebug 出来的 refcount 显示的是 zend_reference 的计数器值（即 2）。

最终的结果如图：

![](http://images.linyiyuan.top/zend_ref.png)

注意：引用只能通过`&`产生，无法通过赋值传递，比如：

```php
$a = "time:" . time();      //$a    -> zend_string_1(refcount=1)
$b = &$a;                   //$a,$b -> zend_reference_1(refcount=2) -> zend_string_1(refcount=1)
$c = $b;                    //$a,$b -> zend_reference_1(refcount=2) -> zend_string_1(refcount=2)
                            //$c    ->                                 ---
```

`$b = &$a`这时候`$a`、`$b`的类型是引用，但是`$c = $b`并不会直接将`$b`赋值给`$c`，而是把`$b`实际指向的zval赋值给`$c`，如果想要`$c`也是一个引用则需要这么操作：

```php
$a = "time:" . time();      //$a       -> zend_string_1(refcount=1)
$b = &$a;                   //$a,$b    -> zend_reference_1(refcount=2) -> zend_string_1(refcount=1)
$c = &$b;/*或$c = &$a*/     //$a,$b,$c -> zend_reference_1(refcount=3) -> zend_string_1(refcount=1) 
```

这个也表示PHP中的 **引用只可能有一层** ，**不会出现一个引用指向另外一个引用的情况** ，也就是没有C语言中`指针的指针`的概念。

# 2.内存管理
在理解了上面内容后我们对引用计数大概有了一个认识，接下来我们来详细理解PHP7变量的销毁和分配，这些都是基于 **引用计数+写时复制**， PHP变量的管理正是基于这两点实现的。

## 2.1 引用计数
引用计数是指在value中增加一个字段`refcount`记录指向当前value的数量，变量复制、函数传参时并不直接硬拷贝一份value数据;而是将`refcount++`，变量销毁时将`refcount--`，等到`refcount`减为0时表示已经没有变量引用这个value，将它销毁即可, 下面我们来看一个官方例子：

    硬拷贝这种方式是可行的，而且内存管理也很简单，但是，硬拷贝带来的一个问题是效率低，比如我们定义了一个变量然后赋值给另外一个变量，可能后面都只是只读操作，假如硬拷贝的话就会有多余的一份数据.

```php
$a = "time:" . time();   //$a       ->  zend_string_1(refcount=1)
$b = $a;                 //$a,$b    ->  zend_string_1(refcount=2)
$c = $b;                 //$a,$b,$c ->  zend_string_1(refcount=3)

unset($b);               //$b = IS_UNDEF  $a,$c ->  zend_string_1(refcount=2)
```

首先我们先定义一个变量$a并赋值, 这里为什么要赋值“time:” .time() 待会会详细讲解，这设计到引用计数的几种特殊类型，这时候我们的$a有了一个属于它的`zval_struct` 容器，而她的value 则指向一个`zend_value`, 而变量值则处于这个`zend_value`中，接着声明了了$b，并赋值$a,这时候$b同样生成一个属于它的`zval_struct`容器，而它的value 也同意指向`zend_value`,此时该`zend_value`的`refcount`(引用计数)为2, $c同理， 现在我们来看下引用计数所处的结构，引用计数的信息位于给具体value结构的gc中：

```c
typedef struct _zend_refcounted_h {
    uint32_t         refcount;          /* reference counter 32-bit */
    union {
        struct {
            ZEND_ENDIAN_LOHI_3(
                zend_uchar    type,
                zend_uchar    flags,    /* used for strings & objects */
                uint16_t      gc_info)  /* keeps GC root number (or 0) and color */
        } v;
        uint32_t type_info;
    } u;
} zend_refcounted_h;
```


现在我们来讲讲关于几种特殊情况下不会使用到引用计数，我们从上面的结构可以看出并不是所有的数据类型都会用到引用计数，以下几种类型都不会使用引用计数：
1. IS_LONG
2. IS_DOUBLE
3. IS_TRUE
4. IS_FALSE
5. IS_NULL

首先**NULL**、**布尔型**都属于**没有值**的数据类型（其中布尔型通过 `IS_FALSE` 和 `IS_TRUE` 两个常量来标记），自然也就没有引用计数， 而 **IS_LONG**, **IS_DOUBLE** 这两种类型是`zval` 在 `value` 字段中能保存下的值，就不会在对他们进行引用计数，**而是在拷贝的时候直接赋值** 即我们在 PHP 中的**整形**与**浮点型**。所以有下面几种情况:

```php
$str_integer = 123;    ->zend_string_1(refcount=0,val="123")

$str_float = 123.123;  ->zend_string_1(refcount=0,val="123.123")

$str_true = true;      ->zend_string_1(refcount=0,val="true")

$str_false = false;    ->zend_string_1(refcount=0,val="false")

$str_null = null;      ->zend_string_1(refcount=0,val="null")
```

除了以上五种五种特殊类型，我们再来看一个官方例子：
```php
$a = "hi~";

$b = $a;
```

不同于上面最开始的例子，我们对$a 赋值一个简单的字符串   然后$b赋值$a, 此时我们肯定以为$a,$b指向的`zend_value` refcount = 2, 但是官方给出的答案这个是错的，gdb调试发现上面例子zend_string的引用计数为0。这是为什么呢？实际上：

    $a,$b -> zend_string_1(refcount=0,val="hi~")

当然这是官方的说法，我在本地使用PHP7.3的环境运行，使用`xdebug_debug_zval` 调试，发现结果并不是跟官方一致：

```php
echo phpversion() . PHP_EOL;

$a = "hi~";

$b = $a;

xdebug_debug_zval('a');


//结果：
7.3.0-2+ubuntu18.04.1+deb.sury.org+1
a: (refcount=1, is_ref=0)='hi~'
```

然后不信邪的继续使用PHP7.2环境运行，结果：

```php
7.2.19-0ubuntu0.18.04.2
a: (refcount=0, is_ref=0)='123'
```

发现PHP7.2与PHP7.3的结果完全不一致，我怀疑是PHP7.3更新时更新了一些东西导致的，目前在官方文档并未找到任何相关信息，我们还是具体以官方例子为准，在得出结果后作者会对该问题进行补充

我们继续接着上面的例子，事实上并不是所有的PHP变量都会用到引用计数，标量：true/false/double/long/null是硬拷贝自然不需要这种机制，但是除了这几个还有两个特殊的类型也不会用到：interned string(内部字符串，就是上面提到的字符串flag：IS_STR_INTERNED)、immutable array，它们的type是`IS_STRING`、`IS_ARRAY`，与普通string、array类型相同：

*   **interned string：** 内部字符串，这是种什么类型？我们在PHP中写的所有字符都可以认为是这种类型，比如function name、class name、variable name、静态字符串等等，我们这样定义:`$a = "hi~";`后面的字符串内容是唯一不变的，这些字符串等同于C语言中定义在静态变量区的字符串：`char *a = "hi~";`，这些字符串的生命周期为request期间，request完成后会统一销毁释放，自然也就无需在运行期间通过引用计数管理内存。

1.  `interned string` 内部字符串（函数名、类名、变量名、静态字符串）：

```php
 $str = '233';    // 静态字符串
复制代码
```

2.  普通字符串：

```php
 $str = '233' . time(); 
```

*   **immutable array：**  **不可变数组**是 `opcache` 扩展优化出的一种数组类型，简单的说，所有多次编译结果恒定不变的数组，都会被优化为**不可变数组**

1. 可变数组

```php
$array = [1, 2, time()];
```

2.  不可变数组：

```php
 $str = [1,2];
```


*注意* :  **不可变数组**和我们上面讲到的**内部字符串**一样，都是**不使用引用计数**的，但是不同点是，内部字符串的计数值恒为 0，而不可变数组会使用一个**伪计数值** 2。

## 2.2 写时复制

上一小节介绍了引用计数，多个变量可能指向同一个value，然后通过refcount统计引用数，这时候如果其中一个变量试图更改value的内容则会重新拷贝一份value修改，同时断开旧的指向，写时复制的机制在计算机系统中有非常广的应用，它只有在必要的时候(写)才会发生硬拷贝，可以很好的提高效率，下面从示例看下：

```php
$a = array(1,2);
$b = &$a;
$c = $a;

//发生分离
$b[] = 3;
```


![](http://images.linyiyuan.top/zval_sep.png)

不是所有类型都可以copy的，比如对象、资源，事实上只有string、array两种支持，与引用计数相同，也是通过`zval.u1.type_flag`标识value是否可复制的：

```c
#define IS_TYPE_COPYABLE         (1<<4)
```

```c
|     type       |  copyable  |
+----------------+------------+
|simple types    |            |
|string          |      Y     |
|interned string |            |
|array           |      Y     |
|immutable array |            |
|object          |            |
|resource        |            |
|reference       |            |
```

**copyable** 的意思是当value发生duplication时是否需要或者能够copy，这个具体有两种情形下会发生：

*   a.从 **literal变量区** 复制到 **局部变量区** ，比如：`$a = [];`实际会有两个数组，而`$a = "hi~";//interned string`则只有一个string
*   b.局部变量区分离时(写时复制)：如改变变量内容时引用计数大于1则需要分离，`$a = [];$b = $a; $b[] = 1;`这里会分离，类型是array所以可以复制，如果是对象：`$a = new user;$b = $a;$a->name = "dd";`这种情况是不会复制object的，$a、$b指向的对象还是同一个

具体literal、局部变量区变量的初始化、赋值后面编译、执行两篇文章会具体分析，这里知道变量有个`copyable`的属性就行了。

## 2.3 变量回收

PHP变量的回收主要有两种：主动销毁、自动销毁。主动销毁指的就是 **unset** ，而自动销毁就是PHP的自动管理机制，在return时减掉局部变量的refcount，即使没有显式的return，PHP也会自动给加上这个操作，另外一个就是写时复制时会断开原来value的指向，这时候也会检查断开后旧value的refcount。

## 2.4 垃圾回收

PHP变量的回收是根据refcount实现的，当unset、return时会将变量的引用计数减掉，如果refcount减到0则直接释放value，这是变量的简单gc过程，PHP变量一般情况下都可以被回收，但是实际上也会出现gc无法回收导致内存泄漏的bug，我们这里举几个例子进行讲解，具体得一个**垃圾回收机制** 我将会在下篇文章具体讲解 先看下一个例子：
```php
$a = [1];

$a[] = &$a;

unset($a);
```

`unset($a)`之前引用关系：

![](http://images.linyiyuan.top/gc_1.png)

`unset($a)`之后：

![](http://images.linyiyuan.top/gc_2.png)

可以看到，`unset($a)`之后由于数组中有子元素指向`$a`，所以`refcount > 0`，无法通过简单的gc机制回收，这种变量就是垃圾，垃圾回收器要处理的就是这种情况，目前垃圾只会出现在array、object两种类型中，所以只会针对这两种情况作特殊处理：当销毁一个变量时，如果发现减掉refcount后仍然大于0，且类型是IS_ARRAY、IS_OBJECT则将此value放入gc可能垃圾双向链表中，等这个链表达到一定数量(10000)后启动检查程序将所有变量检查一遍，如果确定是垃圾则销毁释放。


标识变量是否需要回收也是通过`u1.type_flag`区分的：

```c
#define IS_TYPE_COLLECTABLE
```

```c
|     type       | collectable |
+----------------+-------------+
|simple types    |             |
|string          |             |
|interned string |             |
|array           |      Y      |
|immutable array |             |
|object          |      Y      |
|resource        |             |
|reference       |             |
```

以上就是对PHP7变量的内部实现的总结，其中大部分内容都来自《PHP内核解析》，本人只不过对其进行了总结整合，因为网上大部分文章其实都是停留在PHP5.x的环境，容易误导，所以我在学习过程中将自己所查阅的资料总结起来，方便大家对其的理解。

# 参考资料
1. [PHP内核解析之变量的内部实现](https://github.com/pangudashu/php7-internal/blob/master/2/zval.md)
2. [深入理解PHP7之zval](https://github.com/laruence/php7-internal/blob/master/zval.md)
3. [confusion about php-7 refcount](https://stackoverflow.com/questions/34764119/confusion-about-php-7-refcount)
4. [PHP引用变量机制(PHP如何处理变量)](https://www.jianshu.com/p/53fcf6128dcd)
5. [跟厂长学PHP7内核（八）：深入理解字符串的实现](https://blog.csdn.net/enoch612/article/details/82806820)
6. [引用计数基本知识](http://docs.php.net/manual/zh/features.gc.refcounting-basics.php)
7. [深入理解 PHP7 中全新的 zval 容器和引用计数机制](https://juejin.im/post/5bbf50e86fb9a05ce02a9c19)


