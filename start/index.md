[TOC]

我们使用`gf`框架来创建一个简单的`API`服务项目，该项目实现以下几个示例接口：
1. 用户注册
1. 用户登录
1. 用户注销
1. 登录状态判断
1. 账号/昵称唯一性校验

# 源码仓库

该示例项目的源代码仓库位于: https://github.com/gogf/gf-demos 

各位可以通过[《开始运行》](start/buildrun.md)章节末尾示例的`curl`命令行方式进行测试，也可以通过`/docfile/postman`目录的`postman`配置进行测试。

# 项目结构
推荐的Go业务型项目目录结构如下：
```
/
├── app
│   ├── api
│   ├── model
│   └── service
├── boot
├── config
├── docfile
├── i18n
├── library
├── public
├── router
├── template
├── vendor
├── go.mod
└── main.go
```
|目录/文件名称   | 说明 | 描述
|---|---|---
|`app`           | 业务逻辑层 | 所有的业务逻辑存放目录。
| - `api`        | 业务接口   | 接收/解析用户输入参数的入口/接口层。
| - `model`      | 数据模型   | 数据管理层，仅用于操作管理数据，如数据库操作。
| - `service`    | 逻辑封装   | 业务逻辑封装层，实现特定的业务需求，可供不同的包调用。
|`boot`          | 初始化包   | 用于项目初始化参数设置。
|`config`        | 配置管理   | 所有的配置文件存放目录。
|`docfile`       | 项目文档   | DOC项目文档，如: 设计文档、脚本文件等等。
|`i18n`          | I18N国际化 | I18N国际化配置文件目录。
|`library`       | 公共库包   | 公共的功能封装包，往往不包含业务需求实现。
|`public`        | 静态目录   | 仅有该目录下的文件才能对外提供静态服务访问。
|`router`        | 路由注册   | 用于路由统一的注册管理。
|`template`      | 模板文件   | MVC模板文件存放的目录。
|`vendor`        | 第三方包   | 第三方依赖包存放目录(可选, 未来会被淘汰)。
|`go.mod`        | 依赖管理   | 使用`Go Module`包管理的依赖描述文件。
|`main.go`       | 入口文件   | 程序入口文件。

> 注意：如果需要提供静态服务，那么所有静态文件都需要存放到`public`目录下，仅有该目录下的静态文件才能被外部直接访问。不推荐将程序运行目录加入到静态服务中。

# 数据库设计
我们创建一个简单的用户表。

https://github.com/gogf/gf-demos/blob/master/docfile/sql/create.sql
```sql
CREATE TABLE `user` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `passport` varchar(45) NOT NULL COMMENT '账号',
  `password` varchar(45) NOT NULL COMMENT '密码',
  `nickname` varchar(45) NOT NULL COMMENT '昵称',
  `create_time` timestamp NOT NULL COMMENT '创建时间/注册时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
为简化示例项目的接口实现复杂度，这里的`password`没有做任何加密处理，明文存放密码数据。











