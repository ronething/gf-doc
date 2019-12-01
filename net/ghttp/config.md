
[TOC]

编辑中。。。


`GF`框架的`Web Server`配置管理非常方便，支持多种配置方式以及若干配置方法。

# 配置管理对象
https://godoc.org/github.com/gogf/gf/net/ghttp#ServerConfig



## 配置管理方法
方法列表： https://godoc.org/github.com/gogf/gf/net/ghttp#Server

简要说明：
1. 可以通过`SetConfig`及`SetConfigWithMap`来设置。
1. 也可以使用`Server`对象的`Set*/Enable*`方法进行特定配置的设置。
1. 主要注意的是，配置项在`Server`执行`Start`之后便不能再修改，以便产生并发安全问题。

## `SetConfigWithMap`方法

我们可以使用`SetConfigWithMap`方法通过`Key-Value`键值对来设置/修改`Server`的特定配置，其余的配置使用默认配置即可。其中`Key`的名称即是`ServerConfig`这个`struct`中的属性名称，并且不区分大小写，单词间也支持使用`-`/`_`/`空格`符号连接，具体可参考【[gconv.Struct转换规则](util/gconv/struct.md)】章节。


# 常用配置介绍

## 关闭路由表打印

在WebServer启动的时候默认会打印出当前注册的所有路由信息(包括HOOK注册)，这对于开发者来说非常有用。如果不想启动时打印路由表信息，可以通过以下方式关闭：
```go
g.Server().SetDumpRouteMap(false)
```
此外，我们也可以通过以下方式获取路由表信息(不自动打印)，随后我们可以自定义处理：
```go
routes := g.Server().GetRouteMap()
```








