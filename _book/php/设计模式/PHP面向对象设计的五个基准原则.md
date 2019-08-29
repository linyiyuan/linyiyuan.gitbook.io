**S.O.L.I.D** 是 **首个 5 个面向对象设计**(**OOD**)** 准则的首字母缩写** ，这些准则是由 Robert C. Martin 提出的, 他更为人所熟知的名字是 [Uncle Bob](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Robert_Cecil_Martin)。

这些准则使得开发出易扩展、可维护的软件变得更容易。也使得代码更精简、易于重构。同样也是敏捷开发和自适应软件开发的一部分。

**备注**: _这不是一篇简单的介绍 "欢迎来到 _**S.O.L.I.D" 的文章，这篇文章想要阐明 S.O.L.I.D**_ 是什么。

## S.O.L.I.D 意思是：

扩展出来的首字母缩略词看起来可能很复杂，实际上它们很容易理解。

*   **S** - 单一功能原则
*   **O** - 开闭原则
*   **L** - 里氏替换原则
*   **I** - 接口隔离原则
*   **D** - 依赖反转原则

接下来让我们看看每个原则，来了解为什么 S.O.L.I.D 可以帮助我们成为更好的开发人员。

## 单一职责原则

缩写是 **S.R.P** ，该原则内容是:

> 一个类有且只能有一个因素使其改变，意思是一个类只应该有单一职责．

例如，假设我们有一些图形，并且想要计算这些图形的总面积．是的，这很简单对不对？

```php
class Circle {
    public $radius;

    public function __construct($radius) {
        $this->radius = $radius;
    }
}

class Square {
    public $length;

    public function __construct($length) {
        $this->length = $length;
    }
}
```

首先，我们创建图形类，该类的构造方法初始化必要的参数．接下来，创建**AreaCalculator** 类，然后编写计算指定图形总面积的逻辑代码．

```php
class AreaCalculator {

    protected $shapes;

    public function __construct($shapes = array()) {
        $this->shapes = $shapes;
    }

    public function sum() {
        // logic to sum the areas
    }

    public function output() {
        return implode('', array(
            "",
                "Sum of the areas of provided shapes: ",
                $this->sum(),
            ""
        ));
    }
}
```

**AreaCalculator** 使用方法，我们只需简单的实例化这个类，并且传递一个图形数组，在页面底部展示输出内容．

```php
$shapes = array(
    new Circle(2),
    new Square(5),
    new Square(6)
);

$areas = new AreaCalculator($shapes);

echo $areas->output();
```

输出方法的问题在于，**AreaCalculator** 处理了数据输出逻辑．因此，假如用户希望将数据以 json 或者其他格式输出呢？

所有逻辑都由 **AreaCalculator** 类处理，这恰恰违反了单一职责原则(SRP); **AreaCalculator** 类应该只负责计算图形的总面积，它不应该关心用户是想要json还是HTML格式数据。

因此，要解决这个问题，可以创建一个 **SumCalculatorOutputter** 类，并使用它来处理所需的显示逻辑，以处理所有图形的总面积该如何显示。

**SumCalculatorOutputter** 类的工作方式如下：

```php
$shapes = array(
    new Circle(2),
    new Square(5),
    new Square(6)
);

$areas = new AreaCalculator($shapes);
$output = new SumCalculatorOutputter($areas);

echo $output->JSON();
echo $output->HAML();
echo $output->HTML();
echo $output->JADE();
```

现在，无论你想向用户输出什么格式数据，都由 **SumCalculatorOutputter** 类处理。

## 开闭原则

> 对象和实体应该对扩展开放，但是对修改关闭．

简单的说就是，一个类应该不用修改其自身就能很容易扩展其功能．让我们看一下 **AreaCalculator** 类，特别是 **sum** 方法．

```php
public function sum() {
    foreach($this->shapes as $shape) {
        if(is_a($shape, 'Square')) {
            $area[] = pow($shape->length, 2);
        } else if(is_a($shape, 'Circle')) {
            $area[] = pi() * pow($shape->radius, 2);
        }
    }

    return array_sum($area);
}
```

如果我们想用 **sum** 方法能计算更多图形的面积，我们就不得不添加更多的 **if/else blocks** ，然而这违背了开闭原则．

让这个 **sum** 方法变得更好的方式是将计算每个形状面积的代码逻辑移出 sum 方法，将其放进各个形状类中：

```php
class Square {
    public $length;

    public function __construct($length) {
        $this->length = $length;
    }

    public function area() {
        return pow($this->length, 2);
    }
}
```

相同的操作应该被用来处理 **Circle** 类, 在类中添加一个 **area** 方法。 现在，计算任何形状面积之和应该像下边这样简单：

```php
public function sum() {
    foreach($this->shapes as $shape) {
        $area[] = $shape->area();
    }

    return array_sum($area);
}
```

接下来我们可以创建另一个形状类并在计算总和时传递它而不破坏我们的代码。 然而现在又出现了另一个问题，我们怎么能知道传入 **AreaCalculator** 的对象实际上是一个形状，或者形状对象中有一个 **area** 方法？

接口编码是实践 **S.O.L.I.D** 的一部分，例如下面的例子中我们创建一个接口类，每个形状类都会实现这个接口类：

```php
interface ShapeInterface {
    public function area();
}

class Circle implements ShapeInterface {
    public $radius;

    public function __construct($radius) {
        $this->radius = $radius;
    }

    public function area() {
        return pi() * pow($this->radius, 2);
    }
}
```

在我们的 **AreaCalculator** 的 sum 方法中，我们可以检查提供的形状类的实例是否是 **ShapeInterface** 的实现，否则我们就抛出一个异常：

```php
public function sum() {
    foreach($this->shapes as $shape) {
        if(is_a($shape, 'ShapeInterface')) {
            $area[] = $shape->area();
            continue;
        }

        throw new AreaCalculatorInvalidShapeException;
    }

    return array_sum($area);
}
```

## 里氏替换原则

> 如果对每一个类型为 T1的对象 o1，都有类型为 T2 的对象o2，使得以 T1定义的所有程序 P 在所有的对象 o1 都代换成 o2 时，程序 P 的行为没有发生变化，那么类型 T2 是类型 T1 的子类型。

这句定义的意思是说：每个子类或者衍生类可以毫无问题地替代基类/父类。

依然使用 **AreaCalculator** 类, 假设我们有一个 **VolumeCalculator** 类，这个类继承了**AreaCalculator** 类：

```php
class VolumeCalculator extends AreaCalculator {
    public function construct($shapes = array()) {
        parent::construct($shapes);
    }

    public function sum() {
        // logic to calculate the volumes and then return and array of output
        return array($summedData);
    }
}
```

**SumCalculatorOutputter** 类:

```php
class SumCalculatorOutputter {
    protected $calculator;

    public function __construct(AreaCalculator $calculator) {
        $this->calculator = $calculator;
    }

    public function JSON() {
        $data = array(
            'sum' => $this->calculator->sum();
        );

        return json_encode($data);
    }

    public function HTML() {
        return implode('', array(
            '',
                'Sum of the areas of provided shapes: ',
                $this->calculator->sum(),
            ''
        ));
    }
}
```

如果我们运行像这样一个例子：

```php
$areas = new AreaCalculator($shapes);
$volumes = new AreaCalculator($solidShapes);

$output = new SumCalculatorOutputter($areas);
$output2 = new SumCalculatorOutputter($volumes);
```

程序不会出问题， 但当我们使用**$output2** 对象调用 **HTML** 方法时 ，我们接收到一个 **E_NOTICE** 错误，提示我们 数组被当做字符串使用的错误。

为了修复这个问题，只需：

```php
public function sum() {
    // logic to calculate the volumes and then return and array of output
    return $summedData;
}
```

而不是让**VolumeCalculator** 类的 sum 方法返回数组。

`$summedData` 是一个浮点数、双精度浮点数或者整型。

## 接口隔离原则

> 使用方（client）不应该依赖强制实现不使用的接口，或不应该依赖不使用的方法。

继续使用上面的 `shapes` 例子，已知拥有一个实心块，如果我们需要计算形状的体积，我们可以在 **ShapeInterface** 中添加一个方法：

```php
interface ShapeInterface {
    public function area();
    public function volume();
}
```

任何形状创建的时候必须实现 **volume** 方法，但是【平面】是没有体积的，实现这个接口会强制的让【平面】类去实现一个自己用不到的方法。

**ISP** 原则不允许这么去做，所以我们应该创建另外一个拥有 `volume` 方法的`SolidShapeInterface` 接口去代替这种方式，这样类似立方体的实心体就可以实现这个接口了：

```php
interface ShapeInterface {
    public function area();
}

interface SolidShapeInterface {
    public function volume();
}

class Cuboid implements ShapeInterface, SolidShapeInterface {
    public function area() {
        //计算长方体的表面积
    }

    public function volume() {
        // 计算长方体的体积
    }
}
```

这是一个更好的方式，但是要注意提示类型时不要仅仅提示一个 **ShapeInterface** 或 **SolidShapeInterface**。 你能创建其它的接口，比如 **ManageShapeInterface** ,并在平面和立方体的类上实现它，这样你能很容易的看到有一个用于管理形状的api。例：

```php
interface ManageShapeInterface {
    public function calculate();
}

class Square implements ShapeInterface, ManageShapeInterface {
    public function area() { /Do stuff here/ }

    public function calculate() {
        return $this->area();
    }
}

class Cuboid implements ShapeInterface, SolidShapeInterface, ManageShapeInterface {
    public function area() { /Do stuff here/ }
    public function volume() { /Do stuff here/ }

    public function calculate() {
        return $this->area() + $this->volume();
    }
}
```

现在在 **AreaCalculator** 类中，我们可以很容易地用 **calculate**替换对**area** 方法的调用，并检查对象是否是 **ManageShapeInterface** 的实例，而不是 **ShapeInterface** 。

## 依赖倒置原则

最后，但绝不是最不重要的：

> 实体必须依赖抽象而不是具体的实现．即高等级模块不应该依赖低等级模块，他们都应该依赖抽象．

这也许听起来让人头大，但是它很容易理解．这个原则能够很好的解耦，举个例子似乎是解释这个原则最好的方法：

```php
class PasswordReminder {
    private $dbConnection;

    public function __construct(MySQLConnection $dbConnection) {
        $this->dbConnection = $dbConnection;
    }
}
```

首先 **MySQLConnection** 是低等级模块，然而　**PasswordReminder** 是高等级模块，但是根据 S.O.L.I.D. 中 **D** 的解释：_依赖于抽象而不依赖与实现_， 上面的代码段违背了这一原则，因为 **PasswordReminder** 类被强制依赖于 **MySQLConnection** 类．

稍后，如果你希望修改数据库驱动，你也不得不修改 **PasswordReminder** 类，因此就违背了 **Open-close principle**．

此 **PasswordReminder** 类不应该关注你的应用使用了什么数据库，为了进一步解决这个问题，我们「面向接口写代码」，由于高等级和低等级模块都应该依赖于抽象，我们可以创建一个接口：

```php
interface DBConnectionInterface {
    public function connect();
}
```

这个接口有一个连接数据库的方法，**MySQLConnection** 类实现该接口，在 **PasswordReminder** 的构造方法中不要直接将类型约束设置为 **MySQLConnection** 类，而是设置为接口类，这样无论你的应用使用什么类型的数据库，**PasswordReminder** 类都能毫无问题地连接数据库，且不违背 **开闭原则**．

```php
class MySQLConnection implements DBConnectionInterface {
    public function connect() {
        return "Database connection";
    }
}

class PasswordReminder {
    private $dbConnection;

    public function __construct(DBConnectionInterface $dbConnection) {
        $this->dbConnection = $dbConnection;
    }
}
```

从上面一小段代码，你现在能看出高等级和低等级模块都依赖于抽象了。

## 总结

说实话，**S.O.L.I.D** 一开始似乎很难掌握，但只要不断地使用和遵守其原则，它将成为你的一部分，使你的代码易被扩展、修改，测试，即使重构也不容易出现问题。

文章转自：[https://learnku.com/php/t/28922](https://link.zhihu.com/?target=https%3A//learnku.com/php/t/28922) 
更多文章：[https://learnku.com/php/c/translations](https://link.zhihu.com/?target=https%3A//learnku.com/php/c/translations)



