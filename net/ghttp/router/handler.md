
[TOC]

# 回调函数注册

回调函数注册方式是最简单且最灵活的的路由注册方式，注册的服务可以是一个实例化对象的方法地址，也可以是一个包方法地址。服务需要的数据可以通过`模块内部变量形式`或者`对象内部变量形式`进行管理，开发者可根据实际情况进行灵活控制。

我们可以直接通过`BindHandler`方法完成回调函数的注册，在框架的开发手册中很多地方都使用了回调函数注册的方式来做演示，因为这种注册方式比较简单。

# 示例1，包方法注册
```go
package main

import (
	"github.com/gogf/gf/container/gtype"
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

var (
	total = gtype.NewInt()
)

func Total(r *ghttp.Request) {
	r.Response.Write("total:", total.Add(1))
}

func main() {
	s := g.Server()
	s.BindHandler("/total", Total)
	s.SetPort(8199)
	s.Run()
}
```
在该示例中，我们通过包方法的形式来注册路由。该方法返回总共访问的次数，由于涉及到并发安全，因此`total`变量使用了`gtype.Int`并发安全类型。执行后，当我们不停访问 http://127.0.0.1:8199/total 时，可以看到返回的数值不停递增。


# 示例2，对象方法注册
```go
package main

import (
	"github.com/gogf/gf/container/gtype"
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

type Controller struct {
	total *gtype.Int
}

func (c *Controller) Total(r *ghttp.Request) {
	r.Response.Write("total:", c.total.Add(1))
}

func main() {
	s := g.Server()
	c := &Controller{
		total: gtype.NewInt(),
	}
	s.BindHandler("/total", c.Total)
	s.SetPort(8199)
	s.Run()
}
```
该示例与示例1的实现的效果一致，但我们使用了对象来封装业务逻辑所需的变量，使用回调函数注册来灵活注册对应的对象方法。










