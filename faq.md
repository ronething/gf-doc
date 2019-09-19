[TOC]

# Web服务开发

## 1. 路由注册中: 路由`/`与`/*`的区别

- `/`仅匹配URI为`/`时的路由，这是一个`精准匹配规则`；
- `/*`匹配所有的路由，是`模糊匹配规则`，并且在优先级的控制下，该路由的优先级最低，在其他路由都无法匹配的情况下，最后会使用`/*`路由；

## 2. 在`init`包初始化方法中执行路由注册，但是却访问不到注册的路由
我按照文档上面的示例进行执行对象注册，但是运行后访问注册的路由（ http://127.0.0.1/object ）提醒我`404`，好着急啊，我在线等。
```go
package demo

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

type Object struct {}

func init() {
    g.Server().BindObject("/object", new(Object))
}

func (o *Object) Index(r *ghttp.Request) {
    r.Response.Write("object index")
}

func (o *Object) Show(r *ghttp.Request) {
    r.Response.Write("object show")
}
```

回答：

1. 如何判断路由是否执行成功？在程序运行后终端会打印出当前注册的路由表信息，检查是否注册成功。
1. 该问题是没有在`main`包中引入`demo`包造成的，引入方式:
    ```go
    import _ "PATH/TO/YOUR/PROJECT/PACKAGE"
    ```
    文档链接: 【[路由注册-基本介绍](net/ghttp/router/index.md)】



# 数据库ORM

参考 【[数据库ORM-FAQ常见问题](database/gdb/faq.md)】 章节。以下为问题索引。

## 1. 数据库查询结果转`json`没有了数值
## 2. 表字段类型为`datetime`，参数为`time.Time`类型，写入后时区不对






