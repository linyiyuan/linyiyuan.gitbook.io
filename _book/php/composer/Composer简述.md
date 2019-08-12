# 简述

Composer 不是一个包管理器。是的，它涉及 "packages" 和 "libraries"，但它在每个项目的基础上进行管理，在你项目的某个目录中（例如 vendor）进行安装。默认情况下它不会在全局安装任何东西。因此，这仅仅是一个依赖管理。

这种想法并不新鲜，Composer 受到了 `node's npm` 和 `ruby's bundler`的强烈启发。而当时 PHP 下并没有类似的工具。

Composer 将这样为你解决问题：

*   你有一个项目依赖于若干个库。

*   其中一些库依赖于其他库。

*   你声明你所依赖的东西。

*   Composer 会找出哪个版本的包需要安装，并安装它们（将它们下载到你的项目中）。

# 声明依赖关系

比方说，你正在创建一个项目，你需要一个库来做日志记录。你决定使用 monolog。为了将它添加到你的项目中，你所需要做的就是创建一个 composer.json 文件，其中描述了项目的依赖关系。

```
{
    "require": {
        "monolog/monolog": "1.2.*"
    }
}

```

我们只要指出我们的项目需要一些 monolog/monolog 的包，从 1.2 开始的任何版本。

# 系统要求

运行 Composer 需要 PHP 5.3.2+ 以上版本。一些敏感的 PHP 设置和编译标志也是必须的，但对于任何不兼容项安装程序都会抛出警告。

我们将从包的来源直接安装，而不是简单的下载 zip 文件，你需要 git 、 svn 或者 hg ，这取决于你载入的包所使用的版本管理系统。

Composer 是多平台的，我们努力使它在 Windows 、 Linux 以及 OSX 平台上运行的同样出色。

# 安装

全局安装【推荐】

## linux

你可以将此文件放在任何地方。如果你把它放在系统的 PATH 目录中，你就能在全局访问它。 在类Unix系统中，你甚至可以在使用时不加 php 前缀。

你可以执行这些命令让 composer 在你的系统中进行全局调用：

```
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer

注意： 如果上诉命令因为权限执行失败， 请使用 sudo 再次尝试运行 mv 那行命令。

```

现在只需要运行 composer 命令就可以使用 Composer 而不需要输入 php composer.phar。

## OSX

Composer 是 homebrew-php 项目的一部分。

```
brew update
brew tap josegonzalez/homebrew-php
brew tap homebrew/versions
brew install php55-intl
brew install josegonzalez/php/composer

```

## Windows

下载并且运行 Composer-Setup.exe，它将安装最新版本的 Composer ，并设置好系统的环境变量，因此你可以在任何目录下直接使用 composer 命令。

# 使用

继续 上面的例子，composer.json文件已经申明了依赖关系，此时运行：

```
composer install

```

这里将下载 monolog 到 vendor/monolog/monolog 目录。

# 自动加载

除了库的下载，Composer 还准备了一个自动加载文件，它可以加载 Composer 下载的库中所有的类文件。使用它，你只需要将下面这行代码添加到你项目的引导文件中：

```
require 'vendor/autoload.php';

```

通常你会从所使用框架的单入口文件`index.php`很快找到这行代码，现在我们就可以使用 monolog 了！
