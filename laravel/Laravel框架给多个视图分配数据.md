步骤如下 以下假设需要将数据共享给Home/index以及Home/user视图

a. 创建服务提供者 //这条命令会在app/Providers目录创建ShareUserDataProvider.php

```bash 
    php artisan make:provider ShareUserDataProvider

```

b. 将上一步创建好的服务提供者，添加到配置文件中 在config.php/app.php配置文件中，

```php 
 'providers' => [
        // 其他服务提供器
     App\Providers\ShareUserDataProvider::class,
 ],
```

c. 在ShareUserDataProvider类文件中的boot方法

```php 
    use Illuminate\Support\Facades\View;

     //Home/index,Home/user都是视图名
     View::composer(
          ['Home/index', 'Home/user'], 'App\Http\ViewComposers\ProfileComposer'
     );
```

d. 创建App\\Http\\ViewComposers\\ProfileComposer d-1. 在app/Http目录下创建一个ViewComposers目录

```bash 
    mkdir  ViewComposers`d-2. 在ViewComposers目录下创建ProfileComposer文件

```

```php 
   touch ProfileComposer.php
    <?php

          namespace App\Http\ViewComposers;

          use Illuminate\View\View;
          use App\Http\Controllers\Api\Index;

          class ProfileComposer
          {
              protected $user;
              public function __construct(Index $index)
              {
                  $this->user = $index;
              }

              /**
              * 绑定数据到视图.
              *
              * @param View $view
              * @return void
              */
              public function compose(View $view)
              {
                  ////在视图中使用{{$count}}拿到aa
                  $view->with('count', 'aa');

                  ////拿到UserApi类的test()方法的返回数据，并且分配到模板
                  $view->with('userdata', $this->user->test() );
              }
          }
```

//视图被渲染前，Composer 类的 compose 方法被调用，同时 Illuminate\\View\\View 实例被注入该方法，从而可以使用其 with 方法来绑定数据到视图。