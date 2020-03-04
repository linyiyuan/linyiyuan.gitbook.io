# 了解PHP_FPM运行机制


## CGI

web server（比如说[nginx](https://www.baidu.com/s?wd=nginx&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)）只是内容的分发者。比如，如果请求`/index.html`，那么web server会去文件系统中找到这个文件，发送给浏览器，这里分发的是静态数据。好了，如果现在请求的是`/index.php`，根据配置文件，nginx知道这个不是静态文件，需要去找PHP解析器来处理，那么他会把这个请求简单处理后交给PHP解析器。Nginx会传哪些数据给PHP解析器呢？url要有吧，查询字符串也得有吧，POST数据也要有，HTTP header不能少吧，好的，CGI就是规定要传哪些数据、以什么样的格式传递给后方处理这个请求的协议。仔细想想，你在PHP代码中使用的用户从哪里来的。
当web server收到`/index.php`这个请求后，会启动对应的CGI程序，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以规定CGI规定的格式返回处理后的结果，退出进程。web server再把结果返回给浏览器。

## Fastcgi

提高性能，那么CGI程序的性能问题[在哪](https://www.baidu.com/s?wd=%E5%9C%A8%E5%93%AA&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)呢？"PHP解析器会解析php.ini文件，初始化执行环境"，就是这里了。标准的CGI对每个请求都会执行这些步骤（不闲累啊！启动进程很累的说！），所以处理每个时间的时间会比较长。这明显不合理嘛！那么Fastcgi是怎么做的呢？首先，Fastcgi会先启一个master，解析配置文件，初始化执行环境，然后再启动多个worker。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是fastcgi的对进程的管理。

## PHP-FPM

大家都知道，PHP的解释器是php-cgi。php-cgi只是个CGI程序，他自己本身只能解析请求，返回结果，不会进程管理（皇上，臣妾真的做不到啊！）所以就出现了一些能够调度php-cgi进程的程序，比如说由lighthttpd分离出来的spawn-fcgi。好了PHP-FPM也是这么个东东，在长时间的发展后，逐渐得到了大家的认可（要知道，前几年大家可是抱怨PHP-FPM稳定性太差的），也越来越流行。


## 总结归纳

`CGI` 是webserver和web应用程序交流时的一组接口规范。 而 `PHP-CGI` 是一个进程， 用于实现`CGI` 这种协议，大师他是单进程的，一个进程处理一个请求，处理结束后就自我销毁
我们请求web服务器，web服务器需要解析PHP文件就会临时启动一个`PHP-CGI`解释器，通过`CGI`协议转发要运行的内容，当`PHP-CGI`脚本结束后将结果返回给web服务器，然后这个`PHP-CGI`解释器进程就会自我销毁，假如有10000个请求就会有10000个CGI解释器进程 这种效率极低。

`FAST-CGI` 可以理解是 `CGI`升级版 也是一种协议，用于web服务器（nginx，Apache）和处理程序间进行通信，是一种应用层通信协议。但是与`CGI` 不同的是，`FAST-CGI`每次处理完请求后，不会自动销毁这个进程，而是保留这个进程，使这个进程可以一次处理多个请求。这样每次就不用重新fork一个进程了，从而避免了每个请求进程创建和终止的开销，大大提高了效率。同样，`PHP-FPM`  实现了 `FAST-CGI`这种协议 它直接管理多个`PHP-CGI`进程/线程。也就是说，`PHP-FPM`是`PHP-CGI`的进程管理器，因此它算是`FAST-CGI`协议的实现。在一定程度上讲，php-fpm与php的关系，和tomcat对java的关系是类似的.

需注意的时，当请求量过大时，进程有时候并不能处理得很快，会一直排队，这时候就造成了`PHP-FPM` 进程CPU过高，甚至导致网站502 504，但是我们可以根据自身服务器的配置，适当对进程进行增加，详情请看[PHP-FPM配置详解](php/PHP优化配置/PHP-FPM配置详解.md) 

## php-fpm工作原理

php-fpm的管理对象是php-cgi进程

![](blob:http://vue-admin.linyiyuan.top/23fa88ca-1eae-430b-b4ee-7afe3b33b4a7)

### 1.master进程

php-fpm是一种多进程的模型，由一个master进程和若干worker进程组成（具体数量需要看php-fpm.conf的配置），master不会处理请求，而是fork出worker子进程去接受和处理请求

master进程的主要作用就是管理worker进程，负责fork或者kill掉子进程。在启动时根据配置文件会预先启动一定数量的php-fpm进程。当请求比较多worker处理不过来时，master会fork新的worker进程处理。如果空闲的进程较多时，就会kill掉一些worker进程，避免占用浪费系统资源。

### 2.worker进程

worker进程的主要工作是处理请求,每个worker进程都会accept请求，接受成功后会解析fastcgi，然后执行脚本，完成后关闭请求，继续等待新的连接。
