[TOC]

# 中间件设计

`GF`提供了优雅的中间件请求控制方式，该方式也是主流的`WebServer`提供的请求流程控制方式，基于中间件设计可以为`WebServer`提供更灵活强大的插件机制。

网上找个了个图，经典的中间件洋葱模型：

![经典的中间件洋葱模型](/images/middleware.png)

## 中间件定义

中间件的定义和普通HTTP执行方法一样，但是可以在`Requrst`参数中使用`Middleware`属性对象来控制请求流程。

我们拿一个跨域请求的中间件定义来示例说明一下：
```go
func MiddlewareCORS(r *ghttp.Request) {
	r.Response.CORSDefault()
	r.Middleware.Next()
}
```
可以看到在该中间件中执行完成跨域请求处理的逻辑后，使用`r.Middleware.Next()`方法进一步执行下一个流程；如果这个时候直接退出不调用`r.Middleware.Next()`方法的话，将会退出流程执行（例如可以用于请求的鉴权处理）。

## 中间件与事件回调

中间件（`Middleware`）与事件回调（`HOOK`）是`GF`框架的两大流程控制特性，两者都可用于控制请求流程，并且也都支持绑定特定的路由规则才会生效。但是两者区别也是非常明显的。
1. 首先，中间件侧重于应用层的流程控制，而事件回调侧重于服务层的流程控制；也就是说中间件的作用域仅限于应用层作用域，而事件回调的“权限”更强大，属于`WebServer`级别的流程控制。
1. 其次，中间件仅在路由匹配到服务方法时才会生效，无法独立使用；而事件回调可以抛开服务方法单独使用。
1. 此外，中间件设计采用了“洋葱”设计模型；而事件回调采用的是特定事件的钩子触发设计。
1. 最后，中间件相对来说灵活性更高，也是比较推荐的流程控制方式；而事件回调比较简单，但局限性相对更大。

## 使用示例1，允许跨域请求

我们需要在所有API请求之前增加跨域请求的返回Header信息，该功能可以通过中间件实现：
```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func MiddlewareCORS(r *ghttp.Request) {
	r.Response.CORSDefault()
	r.Middleware.Next()
}

func main() {
	s := g.Server()
	s.Group("/api.v2", func(g *ghttp.RouterGroup) {
		g.Middleware(MiddlewareCORS)
		g.ALL("/user/list", func(r *ghttp.Request) {
			r.Response.Write("list")
		})
	})
	s.SetPort(8199)
	s.Run()
}
```

## 使用示例2，请求鉴权处理

我们在跨域请求中间件的基础之上加上鉴权中间件。为了简化示例，在该示例中，当请求带有`token`参数，并且参数值为`123456`时可以通过鉴权，并且允许跨域请求，执行请求方法；否则返回`403`状态码。

```go
package main

import (
	"net/http"

	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func MiddlewareAuth(r *ghttp.Request) {
	token := r.Get("token")
	if token == "123456" {
		r.Middleware.Next()
	} else {
		r.Response.WriteStatus(http.StatusForbidden)
	}
}

func MiddlewareCORS(r *ghttp.Request) {
	r.Response.CORSDefault()
	r.Middleware.Next()
}

func main() {
	s := g.Server()
	s.Group("/api.v2", func(g *ghttp.RouterGroup) {
		g.Middleware(MiddlewareAuth, MiddlewareCORS)
		g.ALL("/user/list", func(r *ghttp.Request) {
			r.Response.Write("list")
		})
	})
	s.SetPort(8199)
	s.Run()
}
```

## 使用示例3，自定义日志处理

我们来更进一步完善一下以上示例，当请求成功后，我们将请求日志包括状态码输出到终端。

需要注意的是，由于中间件只有在路由匹配到服务方法时才会生效，因此中间件无法处理到例如`404`类型的错误日志。

```go
package main

import (
	"net/http"

	"github.com/gogf/gf/os/glog"

	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

func MiddlewareAuth(r *ghttp.Request) {
	token := r.Get("token")
	if token == "123456" {
		r.Middleware.Next()
	} else {
		r.Response.WriteStatus(http.StatusForbidden)
	}
}

func MiddlewareCORS(r *ghttp.Request) {
	r.Response.CORSDefault()
	r.Middleware.Next()
}

func MiddlewareLog(r *ghttp.Request) {
	r.Middleware.Next()
	glog.Println(r.Response.Status, r.URL.Path)
}

func main() {
	s := g.Server()
	s.Group("/", func(g *ghttp.RouterGroup) {
		g.Middleware(MiddlewareLog)
	})
	s.Group("/api.v2", func(g *ghttp.RouterGroup) {
		g.Middleware(MiddlewareAuth, MiddlewareCORS)
		g.ALL("/user/list", func(r *ghttp.Request) {
			panic("custom error")
		})
	})
	s.SetPort(8199)
	s.Run()
}
```

可以看到，我们注册了一个全局的日志处理中间件，而鉴权和跨域中间件是注册到`/api.v2`路由下。



