---
description: SpringBoot集成MyBatisPlus
---

# SpringBoot集成MyBatisPlus

> **GitHub：[https://github.com/JoeyBling/SpringBoot_MyBatisPlus](https://github.com/JoeyBling/SpringBoot_MyBatisPlus)**

> **码云：[https://gitee.com/zhousiwei/springboot_mybatisplus](https://gitee.com/zhousiwei/springboot_mybatisplus)**

> **欢迎使用和Star支持，如使用过程中碰到问题，可以提出Issue，我会尽力完善**

### 项目结构
--------------------------------------------
```lua
wstro
├── sql  -- 项目SQL语句
│
├── App -- 项目启动类
│
├── config -- 配置信息
│
├── controller -- 控制器
|    ├── admin -- 后台管理员控制器
│
├── service -- 业务逻辑接口
|    ├── impl -- 业务逻辑接口实现类
│
├── dao -- 数据访问接口
│
├── entity--  数据持久化实体类
│
├── datasources -- 多数据源工具类
│
├── shiro -- Shiro验证框架
│
├── task -- Quartz定时任务
│
├── util -- 工具类
|    ├── FreeMarker -- 自定义FreeMarker标签
│
├── resources
|    ├── mapper -- SQL对应的XML文件
|    ├── templates -- FreeMarker模版
│
├── webapp
|    ├── statics -- 静态资源
|    ├── upload -- 上传文件
|    ├── WEB-INF
|    |    ├── templates -- 页面FreeMarker模版
```

### 技术选型：
--------------------------------------------
- 核心框架：`Spring Boot 1.5.1`
- 安全框架：`Apache Shiro`
- 视图框架：`Spring MVC`
- 持久层框架：`MyBatis`、`MyBatisPlus`
- 缓存技术：`EhCache`、`Redis`
- 定时器：`Quartz`
- 数据库连接池：`Druid`
- 日志管理：`SLF4J`、`Log4j`
- 模版技术：`FreeMarker`
- 页面交互：`BootStrap`、`Layer`等

### 本地部署
--------------------------------------------
- 创建数据库**wstro**，数据库编码为UTF-8
- 执行**sql/wstro.sql**文件，初始化数据
- 修改**application-dev.properties**，更新MySQL账号和密码
- 修改**application-dev.properties**，更改Redis连接信息
- 如果不想要Redis服务,注解掉`RedisConfig.java`的`@Configuration`注解
- Eclipse、IDEA运行App.java，则可启动项目
- 项目访问路径：http://localhost:8080/admin
- 账号密码：**admin/admin**
- 多数据源配置：需要自己实现，参考`DataSourceConfig.java`

### 演示效果图：
--------------------------------------------
![](/assets/Swagger.png "Swagger管理")
![](/assets/menu.png "菜单管理")
![](/assets/admin.png "新建用户")
![](/assets/role.png "新建角色")
![](/assets/menu_new.png "新建菜单")
![](/assets/user_info.png "用户信息")
