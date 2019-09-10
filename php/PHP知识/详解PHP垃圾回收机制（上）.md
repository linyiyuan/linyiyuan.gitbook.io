## 前言
在学习PHP垃圾回收机制之前先让我们来学习下 `引用计数` 的基本知识，学习 `引用计数` 有助于我们了解垃圾回收机制的原理以及php底层，因为PHP垃圾回收机制就是基于`引用计数`的，在介绍引用计数之前，我曾翻阅过PHP的官方手册，在测试代码的时候发现运行结果与官方手册所写的不符，翻阅相关资料之后发现是官方手册还停留在PH5.x的版本上，而我本地的环境是PHP7.3,所以导致代码结果与官方手册不符，关于为什么不符合，在下文会详细说明，并且会说明引用计数在PHP5.x版本与PHP7.x版本的差异性，当然主要侧重于解释新zval容器中的引用计数机制。


## 引用计数基本知识

首先我们先来了解一下 `引用计数`，引用官方文档的一段话： 每个php变量存在一个叫"zval"的变量容器中。容器除了包含变量的类型和值，还包括两个字节的额外信息。第一个是"is_ref"，是个bool值，用来标识这个变量是否是属于引用集合(reference set)。通过这个字节，php引擎才能把普通变量和引用变量区分开来，由于php允许用户通过使用&来使用自定义引用，zval变量容器中还有一个内部引用计数机制，来优化内存使用。第二个额外字节是"refcount"，用以表示指向这个zval变量容器的变量(也称符号即symbol)个数，这是PHP官方手册的一个讲解，而这个手册只适用于PHP5.x版本，我们用代码来演示一下

```php
$a = 'new string';

xdebug_debug_zval('a');

//打印结果：

a: (refcount=1, is_ref=0)='new string'
```

注意：你需要安装xdebug拓展才可以执行以上代码，使用debug_zval_dump也可以打印出，但是debug_zval_dump打印的结果与xdebug_debug_zval 打印的结果存在差异，具体说明在本文章中不作解释，具体可查看：[PHP xdebug_debug_zval debug_zval_dump 使用](https://blog.csdn.net/pzqingchong/article/details/78004840)

从结果得知，refcount = 1代表有一个变量名指向这个zval容器 is_ref = 0 代表没有存在引用传递，下面我们增加代码，并且分别使用PHP5.6的环境与PHP7.3的环境执行

```php
$a = 'new string';

$b = $a;

xdebug_debug_zval('a');

//PHP5.6执行结果：
a: (refcount=2, is_ref=0)='new string'

//PHP7.3执行结果：
a: (refcount=1, is_ref=0)='new string'
```

从结果发现，PHP5.6的refcount发生了变化，数值增加了1，而PHP7.3的结果却没发生过变化，当我们增加变量引用时同样也不会发生变化 这是因为在PHP5.x版本中同一个变量容器被变量 a 和变量 b关联.当没必要时，php不会去复制已生成的变量容器。变量容器在”refcount“变成0时就被销毁，而在PHP7.x版本中，zval容器结构发生了更新。

## Zval容器

PHP5.x版本的Zval容器：

```c
struct _zval_struct {
	union {
		long lval;
		double dval;
		struct {
			char *val;
			int len;
		} str;
		HashTable *ht;
		zend_object_value obj;
		zend_ast *ast;
	} value;
	zend_uint refcount__gc;
	zend_uchar type;
	zend_uchar is_ref__gc;
};
```

PHP7.x版本的Zval容器：
```c
struct _zval_struct {
	union {
		zend_long         lval;             /* long value */
		double            dval;             /* double value */
		zend_refcounted  *counted;
		zend_string      *str;
		zend_array       *arr;
		zend_object      *obj;
		zend_resource    *res;
		zend_reference   *ref;
		zend_ast_ref     *ast;
		zval             *zv;
		void             *ptr;
		zend_class_entry *ce;
		zend_function    *func;
		struct {
			uint32_t w1;
			uint32_t w2;
		} ww;
	} value;
    union {
        struct {
            ZEND_ENDIAN_LOHI_4(
                zend_uchar    type,         /* active type */
                zend_uchar    type_flags,
                zend_uchar    const_flags,
                zend_uchar    reserved)     /* call info for EX(This) */
        } v;
        uint32_t type_info;
    } u1;
    union {
        uint32_t     var_flags;
        uint32_t     next;                 /* hash collision chain */
        uint32_t     cache_slot;           /* literal cache slot */
        uint32_t     lineno;               /* line number (for ast nodes) */
        uint32_t     num_args;             /* arguments number for EX(This) */
        uint32_t     fe_pos;               /* foreach position */
        uint32_t     fe_iter_idx;          /* foreach iterator index */
    } u2;
};
```

一个全新的容器结构，对于该结构的详细描述可以参考鸟哥的文章，链接在 [引用文章](#引用文章)中会放出来，对比PHP5.x，有几个重要的更新点：

1.  PHP7 中的变量分为**变量名**和**变量值**两部分，分别对应 `zval_struct` 和在其中声明的 `value`*  
 
说明：这点并不难理解，每个新建的变量都有与其相对应的zval_struct容器，而变量值则存储于容器中的value

![](http://images.linyiyuan.top/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190910092340.png)

2. `zval_struct.value` 中的 `zend_long` 、`double` 都是**简单数据类型**，能够直接储存具体的值，而其他复杂数据类型储存一个指向其他数据结构的**指针**

说明:  `zend_long` 、`double`  即我们在 PHP 中的**整形**与**浮点型**, 对于 `zval` 在 `value` 字段中能保存下的值，就不会在对他们进行引用计数，**而是在拷贝的时候直接赋值**

3. PHP7 中，引用计数器储存在 `value` 中而不是 `zval_struct`
4. **NULL**、**布尔型**都属于**没有值**的数据类型（其中布尔型通过 `IS_FALSE` 和 `IS_TRUE` 两个常量来标记），自然也就没有引用计数
5. **引用**（REFERENCE）变为了一种数据结构而不再只是一个标记位了，它的结构如下：

```c
struct _zend_reference {
    zend_refcounted_h gc;
    zval              val;
}
```
6. `zend_reference` 作为 `zval_struct` 中包含的一种 `value` 类型，也拥有自己的 `val` 值，这个值是指向一个 `zval_struct.value` 的。他们都拥有自己的**引用计数器**。

    引用计数器用来记录当前有多少 `zval` 指向同一个 `zend_value`。

以上六点就是对zval容器的更新总结，而针对第六点，我们使用代码进行讲解：请看如下代码：

```php
$a = 'foo'; 

$b = &$a;

$c = $a;

xdebug_debug_zval('a');
xdebug_debug_zval('b');
xdebug_debug_zval('c');
```

$a 与 $b 各拥有一个 `zval_struct` 容器，并且其中的 `value` 都指向同一个 `zend_reference` 结构，`zend_reference` 内嵌一个 `val` 结构， 指向同一个 `zend_string`，**字符串的内容**就储存在其中。

而 $c 也拥有一个 `zval_struct`，而它的 value 在初始化的时候可以直接指向上面提到的 `zend_string` ，这样在拷贝时就不会产生复制。
![](http://images.linyiyuan.top/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190910094943.png)

然后我们运行代码：
![](http://images.linyiyuan.top/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190910095938.png)

从图中不难发现，$a跟$b 的`refcount` 为2，$c为1，我们根据第五点以及第六点可以知道 $a跟$b都指向同一个 `zend_reference` 所以他们的refcount为2  而`zend_reference` 的结构包含gc 其gc 就是` _zend_refcounted_h `的结构体 主要作用是引用计数以及标记变量的类别

```c
typedef struct _zend_refcounted_h {
	uint32_t         refcount;			/* 引用计数 */
	union {
		struct {
			ZEND_ENDIAN_LOHI_3(
				zend_uchar    type,     
				zend_uchar    flags,    /* 字符串的类型 */
				uint16_t      gc_info   /* 垃圾回收信息 */
			)
		} v;
		uint32_t type_info;
	} u;
} zend_refcounted_h;
```

现在让我们再来看一个例子：
```php
$a = 233;
$b= &a;

xdebug_debug_zval('a');
```

这段代码相对于上面唯一不同点就是变量值为整数型，我们根据上面第二点可以得出，当`Value`为简单数据类型时 `zval_struct.value`能够直接储存具体的值，而按值拷贝时，会开辟一个新的 `zval_struct` 以同样的方式将值储存到相同数据类型的 value 中，所以 refcount 的值一直都会为 0。 其中整型就属于简单数据类型，那么我们猜测的结果应该是：
```php
//猜测结果
a: (refcount=0, is_ref=1)=123
b: (refcount=0, is_ref=1)=123
```

然后实际结果：
```php
//实际结果
a: (refcount=2, is_ref=1)=123
b: (refcount=2, is_ref=1)=123
```

这是因为当使用 `&` 操作符进行引用拷贝时，情况就不一样了：
1.  PHP 为 `&` 操作符操作的变量申请一个 `zend_reference` 结构
2.  将 `zend_reference.value` 指向原来的 `zval_struct.value`
3.  `zval_struct.value` 的数据类型会被修改为 `zend_refrence`
4.  将 `zval_struct.value` 指向刚刚申请并初始化后的 `zend_reference`
5.  为新变量申请 `zval_struct` 结构，将他的 `value` 指向刚刚创建的 `zend_reference`

此时：$var_int_1 和 $var_int_2 都拥有一个 `zval_struct` 结构体，并且他们的 `zval_struct.value` 都指向了同一个 `zend_reference` 结构，所以该结构的引用计数器的值为 2。

还有一个需要注意的点，就是数组的引用计数也有一些特殊的地方，这牵扯到 PHP7 中的另一个概念，叫做 `immutable array`（不可变数组）**不可变数组**是 `opcache` 扩展优化出的一种数组类型，简单的说，所有多次编译结果恒定不变的数组，都会被优化为**不可变数组**，下面是一个反例：
```php
  $array = [1, 2, time()];
```

PHP 在编译阶段无法得知 `time()` 函数的返回值，所以此处的 $array 是**可变数组**。

**不可变数组**是**不使用引用计数**的，而不可变数组会使用一个**伪计数值** 2。


## 引用文章
1. [深入理解 PHP7 中全新的 zval 容器和引用计数机制](https://juejin.im/post/5bbf50e86fb9a05ce02a9c19)
2. [深入理解PHP7之zval](https://github.com/laruence/php7-internal/blob/master/zval.md)
3. [confusion about php-7 refcount](https://stackoverflow.com/questions/34764119/confusion-about-php-7-refcount)
4. [PHP引用变量机制(PHP如何处理变量)](https://www.jianshu.com/p/53fcf6128dcd)
5. [跟厂长学PHP7内核（八）：深入理解字符串的实现](https://blog.csdn.net/enoch612/article/details/82806820)
6. [引用计数基本知识](http://docs.php.net/manual/zh/features.gc.refcounting-basics.php)

