[TOC]

# 复杂参数
`ghttp.Request`对象支持智能的参数类型解析（不区分请求提交方式及请求提交类型），以下为提交参数示例以及服务端对应解析的变量类型：

Parameter             | Variable
---                   |---
`k=m&k=n`             | `map[k:n]`
`k1=m&k2=n`           | `map[k1:m k2:n]`
`k[]=m&k[]=n`         | `map[k:[m n]]`
`k[a][]=m&k[a][]=n`   | `map[k:map[a:[m n]]]`
`k[a]=m&k[b]=n`       | `map[k:map[a:m b:n]]`
`k[a][a]=m&k[a][b]=n` | `map[k:map[a:map[a:m b:n]]]`
`k=m&k[a]=n`          | `error`

# 同名参数

同名参数提交格式形如：`k=v1&k=v2` ，后续的变量值将会覆盖前面的变量值。

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func main() {
	s := g.Server()
	s.BindHandler("/", func(r *ghttp.Request) {
		r.Response.Write(r.Get("name"))
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，我们访问 http://127.0.0.1:8199/?name=john&name=smith 后，将会得到返回值 `smith` 。

> 需要注意的是，在标准库`net/http`处理中，提交的同名参数将会被转换为字符串数组。


# 数组参数


数组参数提交格式形如：`k[]=v1&k[]=v2` 。

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func main() {
	s := g.Server()
	s.BindHandler("/", func(r *ghttp.Request) {
		r.Response.Write(r.Get("array"))
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，我们访问 http://127.0.0.1:8199/?array[]=john&array[]=smith 后，将会得到返回值 `["john","smith"]` 。

# `Map`参数

`Map`参数提交格式形如：`k[a]=m&k[b]=n`，并且支持多级`Map`，例如：`k[a][a]=m&k[a][b]=n`。

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func main() {
	s := g.Server()
	s.BindHandler("/", func(r *ghttp.Request) {
		r.Response.Write(r.Get("map"))
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，我们访问 http://127.0.0.1:8199/?map[id]=1&map[name]=john 后，将会得到返回值 `{"id":"1","name":"john"}` 。

我们再试试多级`Map`，手动访问以下地址

http://127.0.0.1:8199/?map[user1][id]=1&map[user1][name]=john&map[user2][id]=2&map[user2][name]=smith 

将会得到返回值 `{"user1":{"id":"1","name":"john"},"user2":{"id":"2","name":"smith"}}` 。


