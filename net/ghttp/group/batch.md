# 批量注册

`gf`框架的分组路由同时也支持批量的路由注册方式。该方式不是特别常用。

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

type Object struct{}

func (o *Object) Show(r *ghttp.Request) {
	r.Response.Writeln("Show")
}

func (o *Object) Delete(r *ghttp.Request) {
	r.Response.Writeln("REST Delete")
}

func Handler(r *ghttp.Request) {
	r.Response.Writeln("Handler")
}

func HookHandler(r *ghttp.Request) {
	r.Response.Writeln("HOOK Handler")
}

func main() {
	s := g.Server()
	obj := new(Object)
	s.Group("/api").Bind([]ghttp.GroupItem{
		{"ALL", "*",          HookHandler, ghttp.HOOK_BEFORE_SERVE},
		{"ALL", "/handler",   Handler},
		{"ALL", "/obj",       obj},
		{"GET", "/obj/show",  obj, "Show"},
		{"REST", "/obj/rest", obj},
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，终端输出的路由表如下：
```
  SERVER  | DOMAIN  | ADDRESS | METHOD |      ROUTE      |        HANDLER        |    MIDDLEWARE      
|---------|---------|---------|--------|-----------------|-----------------------|-------------------|
  default | default | :8199   | ALL    | /api/*          | main.HookHandler      | HOOK_BEFORE_SERVE  
|---------|---------|---------|--------|-----------------|-----------------------|-------------------|
  default | default | :8199   | ALL    | /api/handler    | main.Handler          |                    
|---------|---------|---------|--------|-----------------|-----------------------|-------------------|
  default | default | :8199   | ALL    | /api/obj/delete | main.(*Object).Delete |                    
|---------|---------|---------|--------|-----------------|-----------------------|-------------------|
  default | default | :8199   | DELETE | /api/obj/rest   | main.(*Object).Delete |                    
|---------|---------|---------|--------|-----------------|-----------------------|-------------------|
  default | default | :8199   | ALL    | /api/obj/show   | main.(*Object).Show   |                    
|---------|---------|---------|--------|-----------------|-----------------------|-------------------|
  default | default | :8199   | GET    | /api/obj/show   | main.(*Object).Show   |                    
|---------|---------|---------|--------|-----------------|-----------------------|-------------------|
```














