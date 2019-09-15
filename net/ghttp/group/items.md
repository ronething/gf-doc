[TOC]



# 基本使用

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

type Object struct {}

func (o *Object) Show(r *ghttp.Request) {
	r.Response.Writeln("Object Show")
}

func (o *Object) Delete(r *ghttp.Request) {
	r.Response.Writeln("Object REST Delete")
}

func Handler(r *ghttp.Request) {
	r.Response.Writeln("Handler")
}

func HookHandler(r *ghttp.Request) {
	r.Response.Writeln("Hook Handler")
}

func main() {
	s     := g.Server()
	obj   := new(Object)
	group := s.Group("/api")
	group.ALL ("*",            HookHandler, ghttp.HOOK_BEFORE_SERVE)
	group.ALL ("/handler",     Handler)
	group.ALL ("/obj",         obj)
	group.GET ("/obj/showit",  obj, "Show")
	group.REST("/obj/rest",    obj)
	s.SetPort(8199)
	s.Run()
}
```
执行后，注册的路由表如下：
```
  SERVER  | ADDRESS | DOMAIN  | METHOD | P |      ROUTE       |         HANDLER         |    HOOK      
|---------|---------|---------|--------|---|------------------|-------------------------|-------------|
  default | :8199   | default | ALL    | 2 | /api/*           | main.HookHandler        | BeforeServe  
|---------|---------|---------|--------|---|------------------|-------------------------|-------------|
  default | :8199   | default | GET    | 3 | /api/obj/showit  | main.(*Object).Show     |              
|---------|---------|---------|--------|---|------------------|-------------------------|-------------|
  default | :8199   | default | ALL    | 3 | /api/obj/delete  | main.(*Object).Delete   |              
|---------|---------|---------|--------|---|------------------|-------------------------|-------------|
  default | :8199   | default | DELETE | 3 | /api/obj/rest    | main.(*Object).Delete   |              
|---------|---------|---------|--------|---|------------------|-------------------------|-------------|
  default | :8199   | default | ALL    | 3 | /api/obj/show    | main.(*Object).Show     |              
|---------|---------|---------|--------|---|------------------|-------------------------|-------------|
  default | :8199   | default | ALL    | 2 | /api/handler     | main.Handler            |              
|---------|---------|---------|--------|---|------------------|-------------------------|-------------|
```

# 批量注册

`gf`框架的分组路由同时也支持批量的路由注册方式。

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

type Object struct {}

func (o *Object) Show(r *ghttp.Request) {
	r.Response.Writeln("Object Show")
}

func (o *Object) Delete(r *ghttp.Request) {
	r.Response.Writeln("Object REST Delete")
}

func Handler(r *ghttp.Request) {
	r.Response.Writeln("Handler")
}

func HookHandler(r *ghttp.Request) {
	r.Response.Writeln("Hook Handler")
}

func main() {
	s   := g.Server()
	obj := new(Object)
	s.Group("/api").Bind([]ghttp.GroupItem{
		{"ALL",  "*",            HookHandler, ghttp.HOOK_BEFORE_SERVE},
		{"ALL",  "/handler",     Handler},
		{"ALL",  "/obj",         obj},
		{"GET",  "/obj/showit",  obj, "Show"},
		{"REST", "/obj/rest",    obj},
	})
	s.SetPort(8199)
	s.Run()
}
```
执行后，注册的路由表和前一个示例一致。














