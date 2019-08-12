# 一、拉取组件是不是很慢？

## 换源（中国全量镜像）

### 修改 composer 的全局配置文件

打开命令行窗口（windows用户）或控制台（Linux、Mac 用户）并执行如下命令：

```
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

## 镜像原理

一般情况下，安装包的数据（主要是 zip 文件）一般是从 [github.com](http://github.com/) 上下载的，安装包的元数据是从 [packagist.org](http://packagist.org/) 上下载的。

然而，由于众所周知的原因，国外的网站连接速度很慢，并且随时可能被“墙”甚至“不存在”。

“Packagist 中国全量镜像”所做的就是缓存所有安装包和元数据到国内的机房并通过国内的 CDN 进行加速，这样就不必再去向国外的网站发起请求，从而达到加速 composer install 以及 composer update 的过程，并且更加快速、稳定。因此，即使 [packagist.org](http://packagist.org/)、[github.com](http://github.com/) 发生故障（主要是连接速度太慢和被墙），你仍然可以下载、更新安装包。

# 二、安装应该使用哪个命令呢，install, update 还是 require ？

## 举个例子

这个fork过百的composer管理的组件包，他的安装命令有问题。[地址在此](https://github.com/orangehill/iseed#installation)。
![composer包文档错误](https://box.kancloud.cn/2e7d11b6b01812aad769ca43b5a72fe4_1974x1424.png)

由于对composer命令的不准确理解，导致出现加载版本错误的情况。

`composer update` 这个命令在我们现在的逻辑中，可能会对项目造成巨大伤害。

因为`composer update`的逻辑是按照 `composer.json`指定的扩展包版本规则，把所有扩展包更新到最新版本，注意，是 所有扩展包，举个例子，你在项目一开始的时候使用了 monolog，当时的配置信息是

```
"monolog/monolog": "1.*",

```

### 后果

安装的是 monolog 1.1 版本，而一个多月以后的现在，monolog 已经是 1.2 了，运行命令后直接更新到 1.2，这时项目并没有针对 1.2 进行过测试，项目一下子变得很不稳定，情况有时候会比这个更糟糕，尤其是在一个庞大的项目中，你没有对项目写完整覆盖测试的情况，什么东西坏掉了你都不知道。

### 命令定义

```
composer install - 如有 composer.lock 文件，直接安装，否则从 composer.json 安装最新扩展包和依赖；

composer update - 从 composer.json 安装最新扩展包和依赖；

composer update vendor/package - 从 composer.json 或者对应包的配置，并更新到最新；

composer require new/package - 添加安装 new/package, 可以指定版本，如： composer require new/package ~2.5.

```

下来介绍几个日常生产的流程，来方便加深大家的理解。

#### 情景一：开发者引入新组件

```
创建 composer.json，并添加依赖到的扩展包；
运行 composer install，安装扩展包并生成 composer.lock；
提交 composer.lock 到git代码版本控制器中，不要加入忽略。

```

#### 情景二：协作者安装现有组件

```
克隆或者更新项目后，其他开发者引入新的组件，这是我们需要也拉下来。根目录下直接运行 composer install 从 composer.lock 中安装 指定版本 的扩展包以及其依赖；

此流程适用于代码部署。

```

#### 情景三：为项目添加新组件

```
使用 composer require vendor/package 添加扩展包，

```

可以规定版本号，如：

```
composer require "foo/bar:1.0.0"

```

相比较情景一，更推荐require的方式对组件进行添加，升级操作。

# 三、拉取失败后怎么办？

## 分析

每个人是从菜鸡阶段过来的，一开始，我们对 Composer 不熟，用的是官方源，非国内镜像，导致网络不稳定或者很慢很慢导致超时。往往一次不顺之后，再次拉取还是不行。
此时需要检查composer.json是否格式正确，不过一般我们不希望phper自己去修改composer.json文件，因为无论安装，卸载，升级，都可以通过composer命令进行管理。

## 原因

拉取失败很大几率是网速问题导致的。

## 解决

先清理cache

```
 composer clear-cache
 // Aliased to clearcache, clears composer's internal package cache.

```

清理chache之后，再次拉取就可以避免进入失败的死循环，同时建议更换国内源以提高网速和稳定性，如果条件具备可以挂网速稳定的vpn代理科学上网，保证composer顺利工作。

此时我们在项目中使用composer，基本没什么问题了。