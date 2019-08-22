
[TOC]


# 接口文档

https://godoc.org/github.com/gogf/gf/net/ghttp#Cookie


任何时候都可以通过```*ghttp.Request```对象获取到当前请求对应的Cookie对象，因为Cookie和Session都是和请求会话相关，因此都属于ghttp.Request的成员对象，并对外公开。Cookie对象不需要手动Close，请求流程结束后，HTTP Server会自动关闭掉。

此外，Cookie中封装了两个```SessionId```相关的方法，```Cookie.SessionId()```用于获取当前请求的SessionId，每个请求的SessionId都是唯一的，并且伴随整个请求流程。```Cookie.SetSessionId(session string)```用于自定义设置当前的SessionId，可以实现自定义的Session控制，以及Session跨域支持(特别是多服务器集群环境下)。

在任何时候，我们都可以通过ghttp.Server对象来修改和获取Cookie的过期时间。

# 使用示例

```go
package main

import (
    "github.com/gogf/gf/g"
    "github.com/gogf/gf/g/os/gtime"
    "github.com/gogf/gf/g/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/cookie", func(r *ghttp.Request) {
        datetime := r.Cookie.Get("datetime")
        r.Cookie.Set("datetime", gtime.Datetime())
        r.Response.Write("datetime:", datetime)
    })
    s.SetPort(8199)
    s.Run()
}
```
执行外层的main.go，可以尝试刷新页面 http://127.0.0.1:8199/cookie ，显示的时间在一直变化。


对于控制器对象而言，从基类控制器中继承了很多会话相关的对象指针，可以看做alias，可以直接使用，他们都是指向的同一个对象：
```go
type Controller struct {
    Request  *ghttp.Request  // 请求数据对象
    Response *ghttp.Response // 返回数据对象(r.Response)
    Server   *ghttp.Server   // Web Server对象(r.Server)
    Cookie   *ghttp.Cookie   // COOKIE操作对象(r.Cookie)
    Session  *ghttp.Session  // SESSION操作对象
    View     *View           // 视图对象
}
```

由于对于Web开发者来讲，Cookie都已经是非常熟悉的组件了，相关API也非常简单，这里便不再赘述。

<!--
# cookie、session与localhost

大多数开发语言，或者开发框架，使用```session id```作为HTTP客户端的唯一访问标识，该id一般都是存放在cookie中，伴随着用户浏览器的访问流程同时在请求中传递给服务端。

在本地开发过程中，开发者往往使用```localhost```作为网站域名，但是该域名会对开发者造成一些混淆，主要是对于session和cookie的影响。

在容易发现的问题之中，```localhost```域名有时会影响到session无法保存，其实该问题归根到底是sessionid无法在浏览器客户端保存的问题，也就是cookie无法保存。因为localhost不是一个有效的域名，有效域名至少要包含两个"."符号。不同的浏览器针对于localhost的处理方式不太一样，有的浏览器支持（如Chromium/Firefox），有的不支持（如Chrome/IE），不支持的现象就是虽然服务端返回了cookie，但是浏览器不认可，不做保存。下一次浏览器请求的时候就不会提交该cookie信息，这样cookie便设置失败了。

解决方案是建议开发者不使用localhost这种特殊的地址，改用IP地址或者自定义的本地域名（需要手动修改```hosts```文件），如 ```127.0.0.1```、```www.local.com```、```www.local.test```等等。
-->




