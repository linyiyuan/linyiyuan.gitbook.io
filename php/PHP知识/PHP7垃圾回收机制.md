# 导言
在小编的《深入理解PHP7变量的内部实现》文章中，我们大概理解了变量的一个结构组成，并且也学习了`引用计数`相关知识点和变量的回收，这篇文章我们将会详细带大家理解关于PHP7的垃圾回收机制.


# 1. 垃圾的产生
我们PHP的变量都是基于`引用计数`这个机制来进行一个变量回收，当变量赋值、传递时并不会直接硬拷贝，而是增加value的引用数，unset、return等释放变量时再减掉引用数，减掉后如果发现refcount变为0则直接释放value，这是变量的基本gc过程，但是还存在着一种情况是这种机制无法解决的，由这种情况产生的垃圾无法被回收导致内存始终得不到释放。这种情况就是循环引用，我们先来看一个官方例子：

```php
$a = [1];

$a[] = &$a;


unset($a);
```

这是一个数据循环引用的例子， 首先我们申明一个变量`$a` 并赋值一个数组`[1]` ，紧接着我们给`$a`数组元素赋值，而这个值又是引用自己, 即变量 a 变成了自己引用自己.

![](http://images.linyiyuan.top/gc_1.png)

此时`zend_reference`的refcount(引用次数) 为 2, 然后我们现在unset($a)

![](http://images.linyiyuan.top/gc_2.png)

从图中可以看到 `unset($a)` 之后 `zend_reference` 结构体的引用计数减 1，但是仍然大于 0，此时是无法通过正常的gc机制回收的，但是$a已经已经没有任何外部引用了，所以这种变量就是垃圾，垃圾回收器要处理的就是这种情况，对此不处理的话，就可能会造成内存泄露。这里就需要垃圾收集器将这部分收集到缓冲区，之后进行回收处理。
这里明确两个准则：

> > 1.  如果一个变量value的refcount减少到0， 那么此value可以被释放掉，不属于垃圾

> > 1.  如果一个变量value的refcount减少之后大于0，那么此zval还不能被释放，此zval可能成为一个垃圾

针对第一个情况GC不会处理，只有第二种情况GC才会将变量收集起来。另外变量是否加入垃圾检查buffer并不是根据zval的类型判断的，而是与前面介绍的是否用到引用计数一样通过`zval.u1.type_flag`记录的，只有包含`IS_TYPE_COLLECTABLE`的变量才会被GC收集。

目前垃圾只会出现在array、object两种类型中，数组的情况上面已经介绍了，object的情况则是成员属性引用对象本身导致的，其它类型不会出现这种变量中的成员引用变量自身的情况，所以垃圾回收只会处理这两种类型的变量。


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

# 2. 回收过程

如果当变量的`refcount`减少后大于0，PHP并不会立即进行对这个变量进行垃圾鉴定，而是放入一个缓冲buffer中，等这个buffer满了以后(10000个值)再统一进行处理，加入buffer的是变量zend_value的`zend_refcounted_h`:

zend_refcounted_h 结构如下：

```c

typedef struct _zend_refcounted_h {
    uint32_t         refcount; // 记录 zend_value 的引用数
    union {
        struct {
            zend_uchar    type,  // zend_value的类型, 与zval.u1.type一致
            zend_uchar    flags, 
            uint16_t      gc_info // GC信息，记录在 gc 池中的位置和颜色，垃圾回收的过程会用到
        } v;
        uint32_t type_info;
    } u;
} zend_refcounted_h;

```

一个变量只能加入一次buffer，为了防止重复加入，变量加入后会把`zend_refcounted_h.gc_info`置为`GC_PURPLE`，即标为紫色，下次refcount减少时如果发现已经加入过了则不再重复插入。垃圾缓存区是一个双向链表，等到缓存区满了以后则启动垃圾检查过程：遍历缓存区，再对当前变量的所有成员进行遍历，然后把成员的refcount减1(如果成员还包含子成员则也进行递归遍历，其实就是深度优先的遍历)，最后再检查当前变量的引用，如果减为了0则为垃圾。这个算法的原理很简单，垃圾是由于成员引用自身导致的，那么就对所有的成员减一遍引用，结果如果发现变量本身refcount变为了0则就表明其引用全部来自自身成员。具体的过程如下：

(1) 从buffer链表的roots开始遍历，把当前value标为灰色(zend_refcounted_h.gc_info置为GC_GREY)，然后对当前value的成员进行深度优先遍历，把成员value的refcount减1，并且也标为灰色；

(2) 重复遍历buffer链表，检查当前value引用是否为0，为0则表示确实是垃圾，把它标为白色(GC_WHITE)，如果不为0则排除了引用全部来自自身成员的可能，表示还有外部的引用，并不是垃圾，这时候因为步骤(1)对成员进行了refcount减1操作，需要再还原回去，对所有成员进行深度遍历，把成员refcount加1，同时标为黑色；

(3) 再次遍历buffer链表，将非GC_WHITE的节点从roots链表中删除，最终roots链表中全部为真正的垃圾，最后将这些垃圾清除。


以上就是我对PHP垃圾回收机制的一些理解总结，大部分内容都是来《PHP内核解析》，关于垃圾收集的内部实现 如果有兴趣想了解的话可以去阅读《PHP内核解析》，因为里面基本都是基于C语言的一些算法，本文章只是大概总结PHP垃圾回收机制的一些知识点，底层的一些算法实现不给予总结。


# 参考资料
1. [PHP内核解析之变量的内部实现](https://github.com/pangudashu/php7-internal/blob/master/2/zval.md)
2. [PHP内核解析之垃圾回收](https://github.com/pangudashu/php7-internal/blob/master/5/gc.md)
3. [深入理解 PHP7 中全新的 zval 容器和引用计数机制](https://juejin.im/post/5bbf50e86fb9a05ce02a9c19)
4. [PHP内核解析之变量的内部实现](https://github.com/pangudashu/php7-internal/blob/master/2/zval.md)
5. [浅析 PHP7 的垃圾回收机制](https://learnku.com/articles/33451)
6. [PHP垃圾回收机制](https://segmentfault.com/q/1010000000624583)


