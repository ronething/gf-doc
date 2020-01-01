
# 上下文变量

请求流程往往会在上下文中共享一些自定义设置的变量，例如在请求开始之前通过中间件或者`HOOK`设置一些变量（或者改变`Request`对象的一些设置），随后在路由服务方法中可以获取该变量并相应对一些处理。这种需求比较常见，在标准库中往往使用`context`上下文包来做处理，但相对较繁琐（应用场景有限）。我们推荐使用`GF`框架的`Param`自定义变量来处理。

使用示例：

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

// 前置中间件1
func MiddlewareBefore1(r *ghttp.Request) {
	r.SetParam("name", "GoFrame")
	r.Response.Writeln("set name")
	r.Middleware.Next()
}

// 前置中间件2
func MiddlewareBefore2(r *ghttp.Request) {
	r.SetParam("site", "https://goframe.org")
	r.Response.Writeln("set site")
	r.Middleware.Next()
}

func main() {
	s := g.Server()
	s.Group("/", func(group *ghttp.RouterGroup) {
		group.Middleware(MiddlewareBefore1, MiddlewareBefore2)
		group.ALL("/", func(r *ghttp.Request) {
			r.Response.Writefln(
				"%s: %s",
				r.GetParamVar("name").String(),
				r.GetParamVar("site").String(),
			)
		})
	})
	s.SetPort(8199)
	s.Run()
}
```
可以看到，我们可以通过`SetParam`和`GetParam`来设置和获取自定义的变量，该变量生命周期仅限于当前请求流程。

执行后，访问 [http://127.0.0.1:8199/ ，页面输出内容为：
```
set name
set site
GoFrame: https://goframe.org
```












