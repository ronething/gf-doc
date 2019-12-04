[TOC]

# 中间件设计

`GF`提供了优雅的中间件请求控制方式，该方式也是主流的`WebServer`提供的请求流程控制方式，基于中间件设计可以为`WebServer`提供更灵活强大的插件机制。经典的中间件洋葱模型：

![经典的中间件洋葱模型](/images/middleware.png)

## 中间件定义

中间件的定义和普通HTTP执行方法`HandlerFunc`一样，但是可以在`Request`参数中使用`Middleware`属性对象来控制请求流程。

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

## 中间件注册

中间件的注册有多种方式，参考接口文档： https://godoc.org/github.com/gogf/gf/net/ghttp

### 全局中间件

```go
// 通过Server对象绑定
func (s *Server) BindMiddleware(pattern string, handlers ...HandlerFunc)
func (s *Server) BindMiddlewareDefault(handlers ...HandlerFunc)

// 通过Domain对象绑定
func (d *Domain) BindMiddleware(pattern string, handlers ...HandlerFunc)
func (d *Domain) BindMiddlewareDefault(handlers ...HandlerFunc)
```
全局中间件是可以独立使用的请求拦截方法，通过路由规则的方式进行注册，绑定到`Server`/`Domain`上，由于中间件需要执行请求拦截操作，因此往往是使用"模糊匹配"或者"命名匹配"规则。
其中：
1. `BindMiddleware`方法是将中间件注册到指定的路由规则下，中间件参数可以给定多个。
1. `BindMiddlewareDefault`方法是将中间件注册到`/*`全局路由规则下。


### 分组路由中间件

```go
func (g *RouterGroup) Middleware(handlers ...HandlerFunc) *RouterGroup
```
通过分组路由使用中间件特性是比较常用的方式。分组路由中注册的中间件绑定到当前分组路由中的所有的服务请求上，当服务请求被执行前会调用到其绑定的中间件方法。
分组路由仅有一个`Middleware`的中间件注册方法。


## 中间件执行优先级

### 全局中间件
由于全局中间件也是通过路由规则执行，那么也会存在执行优先级：

1. 首先，由于全局中间件是基于模糊路由匹配，因此**当同一个路由匹配到多个中间件时，会按照路由的深度优先规则执行**，具体请查看路由章节；
1. 其次，**同一个路由规则下，会按照中间件的注册先后顺序执行**，中间件的注册方法也支持同时按照先后顺序注册多个中间件；
1. 最后，为避免优先级混淆和后续管理，建议将所有中间件放到同一个地方进行先后顺序注册来控制执行优先级；

> 这里的建议来参考于`gRPC`的拦截器设计，没有过多的路由控制，仅在一个地方同一个方法统一注册。往往越简单，越容易理解，也便于长期维护。

### 分组路由中间件

分组路由中间件是绑定到分组路由上的服务方法，不存在路由规则匹配，因此只会按照注册的先后顺序执行。参考后续示例。

## 中间件与事件回调
中间件（`Middleware`）与事件回调（`HOOK`）是`GF`框架的两大流程控制特性，两者都可用于控制请求流程，并且也都支持绑定特定的路由规则。但两者区别也是非常明显的。
1. 首先，中间件侧重于应用级的流程控制，而事件回调侧重于服务级流程控制；也就是说中间件的作用域仅限于应用，而事件回调的“权限”更强大，属于`Server`级别，并可处理静态文件的请求回调。
1. 其次，中间件设计采用了“洋葱”设计模型；而事件回调采用的是特定事件的钩子触发设计。
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
	s.Group("/api.v2", func(group *ghttp.RouterGroup) {
		group.Middleware(MiddlewareCORS)
		group.ALL("/user/list", func(r *ghttp.Request) {
			r.Response.Writeln("list")
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，终端打印出路由表信息如下：
```
  SERVER  | ADDRESS | DOMAIN  | METHOD | P |       ROUTE       |      HANDLER      |     MIDDLEWARE       
|---------|---------|---------|--------|---|-------------------|-------------------|---------------------|
  default |  :8199  | default |  ALL   | 3 | /api.v2/user/list | main.main.func1.1 | main.MiddlewareCORS  
|---------|---------|---------|--------|---|-------------------|-------------------|---------------------|
```
可以看到我们的服务方法绑定了一个中间件`main.MiddlewareCORS`，其中`main`为包名，`MiddlewareCORS`为方法名。
随后我们可以通过请求 http://127.0.0.1:8199/api.v2/user/list 来查看允许跨域请求的`Header`信息是否有返回。

![中间件使用示例1，允许跨域请求](/images/middleware-example-1.png)


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
		r.Response.Writeln("auth")
		r.Middleware.Next()
	} else {
		r.Response.WriteStatus(http.StatusForbidden)
	}
}

func MiddlewareCORS(r *ghttp.Request) {
	r.Response.Writeln("cors")
	r.Response.CORSDefault()
	r.Middleware.Next()
}

func main() {
	s := g.Server()
	s.Group("/api.v2", func(group *ghttp.RouterGroup) {
		group.Middleware(MiddlewareAuth, MiddlewareCORS)
		group.ALL("/user/list", func(r *ghttp.Request) {
			r.Response.Writeln("list")
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，终端打印出路由表信息如下：
```
  SERVER  | ADDRESS | DOMAIN  | METHOD | P |       ROUTE       |      HANDLER      |               MIDDLEWARE                 
|---------|---------|---------|--------|---|-------------------|-------------------|-----------------------------------------|
  default |  :8199  | default |  ALL   | 3 | /api.v2/user/list | main.main.func1.1 | main.MiddlewareAuth,main.MiddlewareCORS  
|---------|---------|---------|--------|---|-------------------|-------------------|-----------------------------------------|
```
可以看到，我们的服务方法绑定了两个中间件，按照先后顺序执行：`main.MiddlewareAuth,main.MiddlewareCORS`。
随后我们可以通过请求 http://127.0.0.1:8199/api.v2/user/list 和 http://127.0.0.1:8199/api.v2/user/list?token=123456 对比来查看效果。

![使用示例2，请求鉴权处理](/images/middleware-example-2-1.png)

![使用示例2，请求鉴权处理](/images/middleware-example-2-2.png)

## 使用示例3，鉴权例外处理

使用分组路由中间件可以很方便地添加鉴权例外，因为只有当前分组路由下注册的服务方法才会绑定并执行鉴权中间件，否则并不会执行到鉴权中间件。

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
	s.Group("/admin", func(group *ghttp.RouterGroup) {
		group.ALL("/login", func(r *ghttp.Request) {
			r.Response.Writeln("login")
		})
		group.Group("/", func(group *ghttp.RouterGroup) {
			group.Middleware(MiddlewareAuth)
			group.ALL("/dashboard", func(r *ghttp.Request) {
				r.Response.Writeln("dashboard")
			})
		})

	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，终端打印出路由表信息如下：
```
  SERVER  | ADDRESS | DOMAIN  | METHOD | P |      ROUTE       |       HANDLER       |     MIDDLEWARE       
|---------|---------|---------|--------|---|------------------|---------------------|---------------------|
  default |  :8199  | default |  ALL   | 2 | /admin/dashboard | main.main.func1.2.1 | main.MiddlewareAuth  
|---------|---------|---------|--------|---|------------------|---------------------|---------------------|
  default |  :8199  | default |  ALL   | 2 | /admin/login     | main.main.func1.1   |                      
|---------|---------|---------|--------|---|------------------|---------------------|---------------------|
```
可以看到，只有`/admin/dashboard`路由的服务方法绑定了鉴权中间件`main.MiddlewareAuth`，而`/admin/login`路由的服务方法并没有添加鉴权处理。
随后我们访问以下URL查看效果：
1. http://127.0.0.1:8199/admin/login
1. http://127.0.0.1:8199/admin/dashboard
1. http://127.0.0.1:8199/admin/dashboard?token=123456

![使用示例3，鉴权例外处理](/images/middleware-example-3-1.png)

![使用示例3，鉴权例外处理](/images/middleware-example-3-2.png)

![使用示例3，鉴权例外处理](/images/middleware-example-3-3.png)

## 使用示例4，统一的错误处理
基于中间件，我们可以在服务函数执行完成后做一些后置判断的工作，特别是统一数据格式返回、结果处理、错误判断等等。这种需求我们可以使用后置的中间件类型来实现。我们使用一个简单的例子，用来演示如何使用中间件对所有的接口请求做后置判断处理，作为一个抛砖引玉作用。
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

func MiddlewareErrorHandler(r *ghttp.Request) {
	r.Middleware.Next()
	if r.Response.Status >= http.StatusInternalServerError {
		r.Response.ClearBuffer()
		r.Response.Writeln("哎哟我去，服务器居然开小差了，请稍后再试吧！")
	}
}

func main() {
	s := g.Server()
	s.Group("/api.v2", func(group *ghttp.RouterGroup) {
		group.Middleware(MiddlewareAuth, MiddlewareCORS, MiddlewareErrorHandler)
		group.ALL("/user/list", func(r *ghttp.Request) {
			panic("db error: sql is xxxxxxx")
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，终端打印出路由表信息如下：
```
  SERVER  | ADDRESS | DOMAIN  | METHOD | P |       ROUTE       |      HANDLER      |                             MIDDLEWARE                               
|---------|---------|---------|--------|---|-------------------|-------------------|---------------------------------------------------------------------|
  default |  :8199  | default |  ALL   | 3 | /api.v2/user/list | main.main.func1.1 | main.MiddlewareAuth,main.MiddlewareCORS,main.MiddlewareErrorHandler  
|---------|---------|---------|--------|---|-------------------|-------------------|---------------------------------------------------------------------|
```
在该示例中，我们在后置中间件中判断有无系统错误，如果有则返回固定的提示信息，而不是把敏感的报错信息展示给用户。当然，在真实的项目场景中，往往还有是需要解析返回缓冲区的数据，例如`JSON`数据，根据当前的执行结果进行封装返回固定的数据格式等等。

执行该示例后，访问 http://127.0.0.1:8199/api.v2/user/list?token=123456 查看效果。

![使用示例4，统一的错误处理](/images/middleware-example-4-1.png)

## 使用示例5，自定义日志处理

我们来更进一步完善一下以上示例，当请求成功后，我们将请求日志包括状态码输出到终端。这里我们必须得使用"全局中间件"了，这样可以拦截处理到所有的服务请求，甚至`404`请求。

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

func MiddlewareLog(r *ghttp.Request) {
	r.Middleware.Next()
	g.Log().Println(r.Response.Status, r.URL.Path, r.GetError().Error())
}

func main() {
	s := g.Server()
	s.SetConfigWithMap(g.Map{
		"AccessLogEnabled": false,
		"ErrorLogEnabled":  false,
	})
	s.BindMiddlewareDefault(MiddlewareLog)
	s.Group("/api.v2", func(group *ghttp.RouterGroup) {
		group.Middleware(MiddlewareAuth, MiddlewareCORS)
		group.ALL("/user/list", func(r *ghttp.Request) {
			panic("啊！我出错了！")
		})
	})
	s.SetPort(8199)
	s.Run()
}
```

![使用示例5，自定义日志处理](/images/middleware-example-5-1.png)

![使用示例5，自定义日志处理](/images/middleware-example-5-2.png)

可以看到，我们注册了一个全局的日志处理中间件，而鉴权和跨域中间件是注册到`/api.v2`路由下。

执行后，我们可以通过请求 http://127.0.0.1:8199/api.v2/user/list 和 http://127.0.0.1:8199/api.v2/user/list?token=123456 对比来查看效果，并查看终端的日志输出情况。

此外，有一个比较有意思的细节。我们这里的服务方法中使用了`panic`抛出一个异常错误。但是可以看到，该异常并没有中断后续中间件的执行。并且在终端也可以看到详细的错误堆栈信息。也就是说，只要请求匹配路由服务方法时，注册的中间件必定会执行。


