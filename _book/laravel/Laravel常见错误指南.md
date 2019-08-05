Laravel常见问题总结
=============

1.Whoops, looks like something went wrong.
------------------------------------------

```
这个错误代表服务器出现错 

解决方法：查看日志以及打开.env 修改APP_DEBUG 为true再重新刷新页面
```

2.当使用post提交数据时 报The page has expired due to inactivity. Please refresh and try again.
-------------------------------------------------------------------------------------

```
一般这个问题是由于表单缺少csrf令牌时 报错误 或者是路由选择访问的方式不是为post

解决方法    ： 在表单加上{{ csrf_field() }} 这个 或者在web路由更改相应的路由设置
```

3.Call to undefined function Illuminate\\Encryption\\openssl\_cipher\_iv\_length()
----------------------------------------------------------------------------------

```
一般出现这个问题是由于服务器缺少openssl这个php拓展

解决方法：装上即可
    安装步骤：
        yum -y install openssl-devel   必须安装
        yum -y install openssl-devel   必须安装
        cd /lamp/php-7.0.7/ext/openssl
        mv config0.m4 config.m4                否则报错：找不到config.m4
        /usr/local/php/bin/phpize 
        ./configure --with-openssl --with-php-config=/usr/local/php/bin/php-config 
        make
        make install
```

4. 禁止访问错误
---------

```
一般出现这个问题是由于重写模块没有打开

解决方法： 在httpd.conf 中打开rewrite重写模块 在226行左右将 AllowOverride None
设置为All 然后重启apache即可
    *FollowSymLinks  允许你的网页文件夹下的链接文件链接到首页目录以外的文件
```

5.使用composer安装laravel时出现问题
--------------------------

```
问题1.详解  

    Failed to download laravel/laravel from dist: The zip extension and unzip command are both missing, skipping.
The php.ini used by your command-line PHP is: /usr/local/php/etc/php.ini

c出现这个问题是由于环境中缺少zip跟unzip

解决方法 ：yum install zip unzip php7.0-zip
```

6.当同步更新laravel时发现视图层根本没更新
-------------------------

`这是因为laravel 里面的storage\framework\views缓存问题 将里面东西都删除即可`

7.利用composer装laravel 时报版本错误
---------------------------

```
命令错误

解决方法composer create-project --prefer-dist laravel/laravel=5.5 blog      
```

8.服务器报500错误
-----------

```
这是由于服务器内部错误 一般是代码错误或者Apache错误

解决方法：打开php.ini 的display_error 中的错误报告 如果是Laravel框架则在配置文件打开调试模式
或者是缺少env这个文件 这个问题一般是由于git克隆或者直接复制文件夹问题
```

9.No application encryption key has been specified.
---------------------------------------------------

```
这是因为.env 配置文件中缺少key这个秘钥

解决方法； hp artisan   key:generate
```

10 如果ajax post发起请求 出现419错误
--------------------------

```
由于没有csrf_token令牌的原因

解决方法： 在页面头部加一行<meta name="csrf-token" content="{!! csrf_token() !!}"/> 
          然后在ajax 的请求头里加多一行headers: {'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')},
```