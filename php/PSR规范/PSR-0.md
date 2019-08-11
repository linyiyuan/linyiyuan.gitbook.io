# PSR-0

RS-0规范是他们出的第1套规范，主要是制定了一些自动加载标准（Autoloading Standard)。
目前官网已经废弃了这一规范，以psr-4作为替代。如下文

> Deprecated - As of 2014-10-21 PSR-0 has been marked as deprecated. PSR-4 is now recommended as an alternative.

大概意思：

> 不推荐使用 - 在2014年10月21日PSR-0已被标记为过时。PSR-4现在推荐作为替代。

PSR-0强制性要求几点：

```
    一个完全合格的namespace和class必须符合这样的结构：“\< Vendor Name>(< Namespace>)*< Class Name>”

    每个namespace必须有一个顶层的namespace（"Vendor Name"提供者名字）

    每个namespace可以有多个子namespace

    当从文件系统中加载时，每个namespace的分隔符(/)要转换成 DIRECTORY_SEPARATOR(操作系统路径分隔符)

    在类名中，每个下划线(_)符号要转换成DIRECTORY_SEPARATOR(操作系统路径分隔符)。在namespace中，下划线(_)符号是没有（特殊）意义的

    当从文件系统中载入时，合格的namespace和class一定是以 .php 结尾的

    verdor name,namespaces,class名可以由大小写字母组合而成（大小写敏感的）
```