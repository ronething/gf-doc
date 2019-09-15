[TOC]

# 允许跨域访问

允许接口跨域往往是需要结合[事件回调](net/ghttp/router/hook.md)一起使用，来统一设置某些路由规则下的接口可以跨域访问。同时，针对允许`WebSocket`的跨域请求访问，也是通过该方式来实现。

相关方法：
https://godoc.org/github.com/gogf/gf/net/ghttp#Response
```go
// 默认的CORS配置
func (r *Response) DefaultCORSOptions() CORSOptions
// See https://www.w3.org/TR/cors/ .
// 允许请求跨域访问.
func (r *Response) CORS(options CORSOptions)
// 允许请求跨域访问(使用默认配置).
func (r *Response) CORSDefault()
```

## CORS对象
`CORS`是`W3`互联网标准组织对HTTP跨域请求的标准，在`ghttp`模块中，我们可以通过`CORSOptions`对象来管理对应的跨域请求选项。定义如下：
```go
// See https://www.w3.org/TR/cors/ .
// 服务端允许跨域请求选项
type CORSOptions struct {
    AllowOrigin      string // Access-Control-Allow-Origin
    AllowCredentials string // Access-Control-Allow-Credentials
    ExposeHeaders    string // Access-Control-Expose-Headers
    MaxAge           int    // Access-Control-Max-Age
    AllowMethods     string // Access-Control-Allow-Methods
    AllowHeaders     string // Access-Control-Allow-Headers
}
```
具体参数的介绍请查看W3组织[官网手册](https://www.w3.org/TR/cors/)。

## 默认CORSOptions

当然，为方便跨域设置，在`ghttp`模块中也提供了默认的跨域请求选项，通过`ghttp.Response.DefaultCORSOptions`获取。
```go
// 默认的CORS配置
func (r *Response) DefaultCORSOptions() CORSOptions {
    return CORSOptions {
        AllowOrigin      : gstr.TrimRight(r.request.Referer(), "/"),
        AllowMethods     : HTTP_METHODS,
        AllowCredentials : "true",
        MaxAge           : 3628800,
    }
}
```
大多数情况下，我们在需要允许跨域请求的接口中（一般情况下使用HOOK）可以直接使用`CORSDefault()`允许该接口跨域访问。

## 使用示例
我们来看一个简单的`REST`接口示例：
```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

type Order struct {
}

func (order *Order) Get(r *ghttp.Request) {
	r.Response.Write("GET")
}

func main() {
	s := g.Server()
	s.BindObjectRest("/api.v1/{.struct}", new(Order))
	s.SetPort(8199)
	s.Run()
}
```
接口地址是`http://localhost/api.v1/order`，当然这个接口是不允许跨域的。我们打开一个不同的域名，例如：百度首页(正好用了jQuery，方便调试)，然后按`F12`打开开发者面板，在`console`下执行以下AJAX请求：
```javascript
$.get("http://localhost:8199/api.v1/order", function(result){
    console.log(result)
});
```
结果如下：
![](/images/Selection_154.png)
返回了不允许跨域请求的提示错误，接着我们修改一下服务端接口的测试代码，如下：

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

type Order struct{}

func (order *Order) Get(r *ghttp.Request) {
	r.Response.Write("GET")
}

func main() {
	s := g.Server()
	s.BindHookHandlerByMap("/api.v1/*any", map[string]ghttp.HandlerFunc{
		"BeforeServe": func(r *ghttp.Request) {
			r.Response.CORSDefault()
		},
	})
	s.BindObjectRest("/api.v1/{.struct}", new(Order))
	s.SetPort(8199)
	s.Run()
}
```
我们增加了针对于路由`/api.v1/*any`的绑定事件`BeforeServe`，该事件将会在所有服务执行之前调用，该事件的回调方法中，我们通过调用`CORSDefault`方法使用默认的跨域设置允许跨域请求。该绑定的事件路由规则使用了模糊匹配规则，表示所有`/api.v1`开头的接口地址都允许跨域请求。

返回刚才的百度首页，再次执行请求`AJAX`请求，这次便成功了：
![](/images/Selection_155.png)

当然我们也可以通过`CORSOptions`对象以及`CORS`方法对跨域请求做更多的设置。
