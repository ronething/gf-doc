
# 路由层级注册

> `GF`框架的层级路由注册方式灵感来源于`PHP Laravel`框架。

推荐使用路由层级注册方式，注册的路由代码更清晰直观。

`GF`框架的分组路由注册支持更加直观优雅层级的注册方式，以便于开发者更方便地管理路由列表。路由层级注册方式也是推荐的路由注册方式。`Talk is cheap, so` 我们来看一个比较完整的示例，便能很好地说明一切：

```go
package main

import (
	"net/http"

	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/os/glog"
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
		g.GET("/test", func(r *ghttp.Request) {
			r.Response.Write("test")
		})
		g.Group("/order", func(g *ghttp.RouterGroup) {
			g.GET("/list", func(r *ghttp.Request) {
				r.Response.Write("list")
			})
			g.PUT("/update", func(r *ghttp.Request) {
				r.Response.Write("update")
			})
		})
		g.Group("/user", func(g *ghttp.RouterGroup) {
			g.GET("/info", func(r *ghttp.Request) {
				r.Response.Write("info")
			})
			g.POST("/edit", func(r *ghttp.Request) {
				r.Response.Write("edit")
			})
			g.DELETE("/drop", func(r *ghttp.Request) {
				r.Response.Write("drop")
			})
		})
		g.Group("/hook", func(g *ghttp.RouterGroup) {
			g.Hook("/*", ghttp.HOOK_BEFORE_SERVE, func(r *ghttp.Request) {
				r.Response.Write("hook any")
			})
			g.Hook("/:name", ghttp.HOOK_BEFORE_SERVE, func(r *ghttp.Request) {
				r.Response.Write("hook name")
			})
		})
	})
	s.SetPort(8199)
	s.Run()
}
```

执行后，注册的路由列表如下：

```
  SERVER  | ADDRESS | DOMAIN  | METHOD | P |        ROUTE         |       HANDLER       |    MIDDLEWARE      
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
  default |  :8199  | default | ALL    | 1 | /*                   | main.MiddlewareLog  | MIDDLEWARE         
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
  default |  :8199  | default | ALL    | 3 | /api.v2/*            | main.MiddlewareAuth | MIDDLEWARE         
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
  default |  :8199  | default | ALL    | 2 | /api.v2/*            | main.MiddlewareCORS | MIDDLEWARE         
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
  default |  :8199  | default | ALL    | 3 | /api.v2/hook/*       | main.main.func2.4.1 | HOOK_BEFORE_SERVE  
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
  default |  :8199  | default | ALL    | 3 | /api.v2/hook/:name   | main.main.func2.4.2 | HOOK_BEFORE_SERVE  
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
  default |  :8199  | default | GET    | 3 | /api.v2/order/list   | main.main.func2.2.1 |                    
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
  default |  :8199  | default | PUT    | 3 | /api.v2/order/update | main.main.func2.2.2 |                    
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
  default |  :8199  | default | GET    | 2 | /api.v2/test         | main.main.func2.1   |                    
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
  default |  :8199  | default | DELETE | 3 | /api.v2/user/drop    | main.main.func2.3.3 |                    
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
  default |  :8199  | default | POST   | 3 | /api.v2/user/edit    | main.main.func2.3.2 |                    
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
  default |  :8199  | default | GET    | 3 | /api.v2/user/info    | main.main.func2.3.1 |                    
|---------|---------|---------|--------|---|----------------------|---------------------|-------------------|
```







