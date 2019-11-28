[TOC]

# 中间件设计

`GF`提供了优雅的中间件请求控制方式，该方式也是主流的`WebServer`提供的请求流程控制方式，基于中间件设计可以为`WebServer`提供更灵活强大的插件机制。经典的中间件洋葱模型：

![经典的中间件洋葱模型](/images/middleware.png)

## 中间件定义

中间件的定义和普通HTTP执行方法一样，但是可以在`Request`参数中使用`Middleware`属性对象来控制请求流程。

我们拿一个跨域请求的中间件定义来示例说明一下：
```go
func MiddlewareCORS(r *ghttp.Request) {
	r.Response.CORSDefault()
	r.Middleware.Next()
}
```
可以看到在该中间件中执行完成跨域请求处理的逻辑后，使用`r.Middleware.Next()`方法进一步执行下一个流程；如果这个时候直接退出不调用`r.Middleware.Next()`方法的话，将会退出后续的执行流程（例如可以用于请求的鉴权处理）。

## 中间件类型
中间件的类型分为两种：前置中间件和后置中间件。前置即在路由服务函数调用之前调用，后置即在其后调用。

1. 前置中间件。其定义类似于：
	```go
	func Middleware(r *ghttp.Request) {
		// 中间件处理逻辑
		r.Middleware.Next()
	}
	```
1. 后置中间件。其定义类似于：
	```go
	func Middleware(r *ghttp.Request) {
		r.Middleware.Next()
		// 中间件处理逻辑
	}
	```

## 中间件执行优先级

1. 首先，由于中间件是基于模糊路由匹配，因此当同一个路由匹配到多个中间件时，会按照路由的深度优先规则执行，具体请查看路由章节；
1. 其次，同一个路由规则下，会按照中间件的注册先后顺序执行，中间件的注册方法也支持同时按照先后顺序注册多个中间件；
1. 最后，为避免优先级混淆和后续管理，建议将所有中间件放到同一个地方进行先后顺序注册来控制执行优先级；

> 这里的建议来参考于`gRPC`的拦截器设计，没有过多的路由控制，仅在一个地方同一个方法统一注册。往往越简单，越容易理解，也便于长期维护。

## 中间件与事件回调
中间件（`Middleware`）与事件回调（`HOOK`）是`GF`框架的两大流程控制特性，两者都可用于控制请求流程，并且也都支持绑定特定的路由规则。但两者区别也是非常明显的。
1. 首先，中间件侧重于应用级的流程控制，而事件回调侧重于服务级流程控制；也就是说中间件的作用域仅限于应用，而事件回调的“权限”更强大，属于`Server`级别，并可处理静态文件的请求回调。
1. 再者，中间件设计采用了“洋葱”设计模型；而事件回调采用的是特定事件的钩子触发设计。
1. 此外，中间件无法被任何的退出函数阻止，只要请求匹配到路由服务函数，则对应的中间件必定会执行；事件回调受限于没有灵活的流程控制方式，需要通过`ExitHook`方式来退出后续的`HOOK`执行。
1. 最后，中间件相对来说灵活性更高，也是比较推荐的流程控制方式；而事件回调比较简单，但灵活性较差。

## 使用示例1，允许跨域请求

第一个例子，也是比较常见的功能需求。

我们需要在所有API请求之前增加允许跨域请求的返回`Header`信息，该功能可以通过中间件实现：
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
随后我们可以通过请求 http://127.0.0.1:8199/api.v2/user/list 来查看允许跨域请求的`Header`是否有返回。

## 使用示例2，请求鉴权处理

我们在跨域请求中间件的基础之上加上鉴权中间件。

为了简化示例，在该示例中，当请求带有`token`参数，并且参数值为`123456`时可以通过鉴权，并且允许跨域请求，执行请求方法；否则返回`403 Forbidden`状态码。

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

随后我们可以通过请求 http://127.0.0.1:8199/api.v2/user/list 和 http://127.0.0.1:8199/api.v2/user/list?token=123456 对比来查看效果。


## 使用示例3，鉴权例外处理
中间件是绑定到特定的路由规则下才会生效，并且该路由规则往往是模糊匹配规则。假如我们在鉴权的中间件中需要添加例外的路由该怎么办？有两个思路供参考：
1. 在路由设计规划的时候，将不需要鉴权的路由规则注册到鉴权中间件路由规则之外；
1. 在鉴权中间件中添加对特定路由的例外，或者通过闭包的方式来封装中间件添加鉴权例外；

以下我们通过第2种方式来演示以下如何添加例外：
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

func main() {
	s := g.Server()
	s.Group("/admin", func(g *ghttp.RouterGroup) {
		g.MiddlewarePattern("/*action", func(r *ghttp.Request) {
			if action := r.GetRouterString("action"); action != "" {
				switch action {
				case "login":
					r.Middleware.Next()
					return
				}
			}
			MiddlewareAuth(r)
		})
		g.ALL("/login", func(r *ghttp.Request) {
			r.Response.Write("login")
		})
		g.ALL("/dashboard", func(r *ghttp.Request) {
			r.Response.Write("dashboard")
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
其中，我们通过注册`/admin/*action`的路由的方式，目的是为了获取URL中的操作路径（`action`），便于更好地判断路由规则。当然也可以直接判断完整的路由规则`/admin/login`来判断例外（对比`r.URL.Path`属性值）。执行后，注册的路由列表如下：
```
  SERVER  | ADDRESS | DOMAIN  | METHOD | P |      ROUTE       |      HANDLER      | MIDDLEWARE  
|---------|---------|---------|--------|---|------------------|-------------------|------------|
  default |  :8199  | default | ALL    | 2 | /admin/*action   | main.main.func1.1 | MIDDLEWARE  
|---------|---------|---------|--------|---|------------------|-------------------|------------|
  default |  :8199  | default | ALL    | 2 | /admin/dashboard | main.main.func1.3 |             
|---------|---------|---------|--------|---|------------------|-------------------|------------|
  default |  :8199  | default | ALL    | 2 | /admin/login     | main.main.func1.2 |             
|---------|---------|---------|--------|---|------------------|-------------------|------------|
```
随后我们访问以下URL查看效果：
1. http://127.0.0.1:8199/admin/login
1. http://127.0.0.1:8199/admin/dashboard
1. http://127.0.0.1:8199/admin/dashboard?token=123456

## 使用示例4，统一的错误处理
基于中间件，我们可以在服务函数执行完成后做一些后置判断的工作，特别是统一数据格式返回、结果处理、错误判断等等。这种需求我们可以使用后置中间件类型来实现。我们使用一个简单的例子，用来演示如何使用中间件对所有的接口请求做后置判断处理，作为一个抛砖引玉作用。
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

func MiddlewareError(r *ghttp.Request) {
	r.Middleware.Next()
	if r.Response.Status >= http.StatusInternalServerError {
		r.Response.ClearBuffer()
		r.Response.Write("Internal error occurred, please try again later.")
	}
}

func main() {
	s := g.Server()
	s.Group("/api.v2", func(g *ghttp.RouterGroup) {
		g.Middleware(MiddlewareAuth, MiddlewareCORS, MiddlewareError)
		g.ALL("/user/list", func(r *ghttp.Request) {
			panic("db error: sql is xxxxxxx")
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
在该示例中，我们在后置中间件中判断有无系统错误，如果有则返回固定的提示信息，而不是把敏感的报错信息展示给用户。当然，在真实的项目场景中，往往还有是需要解析返回缓冲区的数据，例如JSON数据，根据当前的执行结果进行封装返回固定的数据格式等等。

执行该示例后，访问 https://127.0.0.1:8199/api.v2/user/list?token=123456 查看效果。



## 使用示例5，自定义日志处理

我们来更进一步完善一下以上示例，当请求成功后，我们将请求日志包括状态码输出到终端。

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

执行后，我们可以通过请求 http://127.0.0.1:8199/api.v2/user/list 和 http://127.0.0.1:8199/api.v2/user/list?token=123456 对比来查看效果，并查看终端的日志输出情况。

此外，有一个比较有意思的细节。我们这里的服务方法中使用了`panic`抛出一个异常错误。但是可以看到，该异常并没有中断后续中间件的执行。并且在终端也可以看到详细的错误堆栈信息。也就是说，只要请求匹配路由服务方法时，注册的中间件必定会执行。


