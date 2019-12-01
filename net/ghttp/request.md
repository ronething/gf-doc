[TOC]

# 请求输入

请求输入依靠 `ghttp.Request` 对象实现，`ghttp.Request`继承了底层的`http.Request`对象，并且包含了会话相关的`Cookie`和`Session`对象(每个请求都有独立的`Cookie`和`Session`对象)。此外，成员对象包含一个与当前请求对应的返回输出对象指针`Response`，用于数据的返回。

相关方法：
https://godoc.org/github.com/gogf/gf/net/ghttp

## 方法说明

可以看到`Request`对象的参数获取方法非常丰富，可以分为以下几类：
1. `Get*`: 常用方法，简化参数获取，`GetRequest*`的别名。
1. `GetQuery*`: 获取`GET`方式传递过来的参数，包括`Query String`及`Body`参数解析。
2. `GetPost*`: 获取`POST`方式传递过来的参数，包括`Form`参数以及`Body`参数解析。
2. `GetForm*`: 获取表单方式传递过来的参数，表单方式提交的参数`Content-Type`往往为`application/x-www-form-urlencoded`, `application/form-data`, `multipart/form-data`, `multipart/mixed`等等。
3. `GetRequest*`: 获取客户端提交的参数，不区分提交方式。
4. `GetBody*`: 获取客户端提交的原始数据，与`HTTP Method`无关，例如客户端提交`JSON/XML`数据格式时可以通过该方法获取原始的提交数据。
5. `GetJson`: 自动将原始请求信息解析为`gjson.Json`对象指针返回，`gjson.Json`对象具体在【[gjson模块](encoding/gjson/index.md)】章节中介绍。
1. `Get*ToStruct`: 将请求参数绑定到指定的`struct`对象上，注意给定的参数为对象指针。
1. `Exit*`: 用于请求流程退出控制，详见本章后续说明；

## 参数类型

获取的参数方法可以对指定键名的数据进行自动类型转换，例如： http://127.0.0.1:8199/?amount=19.66 ，通过`GetQueryString`将会返回`19.66`的字符串类型，`GetQueryFloat32`/`GetQueryFloat64`将会分别返回`float32`和`float64`类型的数值`19.66`。但是，`GetQueryInt`/`GetQueryUint`将会返回`19`（如果参数为`float`类型的字符串，将会按照**向下取整**进行整型转换）。

变量类型的获取方法仅提供了常用类型的直接获取方法，如果有更多参数类型转换的需求，可以使用`Get*Var`参数获取方法，获得`*gvar.Var`变量再进行相应类型转换。例如，假如我们要获取一个`int8`类型的参数，我们可以这样`GetVar("id").Int8()`。

## 参数优先级

我们知道参数提交有多种方式，在`GF`框架下，有以下几种方式：
1. `Router`: 路由参数，来源于路由规则匹配。
1. `Query`: `URL`中的`query string`参数解析，如：`http://127.0.0.1/index?id=1&name=john` 中的`id=1&name=john`。
1. `Form`: 表单提交参数，最常见的提交方式。
1. `Body`: 原始提交内容，从`Body`中获取并解析得到的参数。
1. `Custom`: 自定义参数。

我们考虑一种场景，当不同的提交方式中存在同名的参数名称会怎么样？在`GF`框架下，我们根据不同的获取方法，将会按照不同的优先级进行获取，优先级高的方式提交的参数将会优先覆盖其他方式的同名参数。优先级规则如下：
1. `GetRequset*`以及`Get*`别名方法：`Router < Query < Form < Body < Custom`，也就是说自定义参数的优先级最高，其次是`Body`提交参数，再者是`Form`表单参数，以此类推。例如，`Query`和`Form`中都提交了同样名称的参数`id`，参数值分别为`1`和`2`，那么`Get("id")`将会返回`2`，而`GetQuery("id")`将会返回`1`。
1. `GetQuery*`方法：`Query > Body`，也就是说`query string`的参数将会覆盖`Body`中提交的同名参数。例如，`Query`和`Body`中都提交了同样名称的参数`id`，参数值分别为`1`和`2`，那么`Get("id")`将会返回`2`，而`GetQuery("id")`将会返回`1`。
1. `GetForm*`方法：由于该类型的方法仅用于获取`Form`表单参数，因此没什么优先级的差别。

## JSON参数获取

这也是新手问得比较多问题。

从通用性及效率上考虑，服务端不会自动解析客户端提交的`JSON`参数，因此服务端无法通过例如`GetPost*`方法获得`JSON`数据中的特定字段参数。服务端开发者可以调用`GetJson`方法获得`JSON`对象（注意判断解析错误`error`）。也可以通过`GetBody*`方法获得`Body`数据后自行解析。

获得的`JSON`对象其实是一个`gjson.Json`类型，针对于`JSON`参数的检索和操作非常方便，具体请参考【[gjson模块](encoding/gjson/index.md)】章节。

## 请求参数解析
`ghttp.Request`对象支持智能的参数类型解析（不区分请求提交方式及请求提交类型），以下为提交参数示例以及服务端对应解析的变量类型：

Parameter | Variable
---|---
`v=m&v=n` | `map[v:n]`
`v1=m&v2=n` | `map[v1:m v2:n]`
`v[]=m&v[]=n` | `map[v:[m n]]`
`v[a][]=m&v[a][]=n` | `map[v:map[a:[m n]]]`
`v[a]=m&v[b]=n` | `map[v:map[a:m b:n]]`
`v[a][a]=m&v[a][b]=n` | `map[v:map[a:map[a:m b:n]]]`
`v=m&v[a]=n` | `error`



## `Request.URL`与`Request.Router`

`Request.Router`是当前匹配到的路由对象，包含路由注册信息，一般来说开发者不会用得到。该对象在请求没有匹配到注册的服务函数时，可能为`nil`。
`Request.URL`是底层请求的URL对象（继承自标准库`http.Request`），包含请求的URL地址信息，特别是`Request.URL.Path`表示请求的URI地址，该对象必定有值。

因此，假如在服务函数中使用的话，`Request.Router`是有值的，因为只有匹配到了路由才会调用服务回调方法。但是在HOOK事件回调函数中，该对象可能为`nil`（表示没有匹配到服务回调函数路由）。特别是在使用事件回调对请求接口鉴权的时候，应当使用`Request.URL`对象获取请求的URL信息，而不是`Request.Router`。

此外，根据中间件的特性，因此在中间件处理函数中`Request.Router`是必定有值的。

## `Exit`, `ExitAll`与`ExitHook`

1. `Exit`: 仅退出当前执行的逻辑方法，如: 当前`HOOK`方法、服务方法，不退出后续的逻辑处理，可用于替代`return`；
1. `ExitAll`: 强行退出当前执行流程，当前执行方法的后续逻辑以及后续所有的逻辑方法将不再执行，常用于权限控制；
1. `ExitHook`: 当路由匹配到多个`HOOK`方法时，默认是按照路由匹配优先级顺序执行`HOOK`方法。当在`HOOK`方法中调用`ExitHook`方法后，后续的`HOOK`方法将不会被继续执行，作用类似`HOOK`方法覆盖；
1. 这三个退出函数仅在服务函数和`HOOK`事件回调函数中有效，无法控制中间件的执行流程；


# 使用示例

## 示例1，请求数据校验

### 请求参数绑定+数据校验示例
https://github.com/gogf/gf/blob/master/.example/net/ghttp/server/request/request_validation.go
```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/util/gvalid"
)

type User struct {
	Uid   int    `gvalid:"uid@min:1"`
	Name  string `params:"username"  gvalid:"username @required|length:6,30"`
	Pass1 string `params:"password1" gvalid:"password1@required|password3"`
	Pass2 string `params:"password2" gvalid:"password2@required|password3|same:password1#||两次密码不一致，请重新输入"`
}

func main() {
	s := g.Server()
	s.Group("/", func(rg *ghttp.RouterGroup) {
		rg.ALL("/user", func(r *ghttp.Request) {
			user := new(User)
			if err := r.GetToStruct(user); err != nil {
				r.Response.WriteJsonExit(g.Map{
					"message": err,
					"errcode": 1,
				})
			}
			if err := gvalid.CheckStruct(user, nil); err != nil {
				r.Response.WriteJsonExit(g.Map{
					"message": err.Maps(),
					"errcode": 1,
				})
			}
			r.Response.WriteJsonExit(g.Map{
				"message": "ok",
				"errcode": 0,
			})
		})
	})
	s.SetPort(8199)
	s.Run()
}
```

简要说明，
1. 这里使用了`r.GetToStruct(user)`方法将请求参数绑定到指定的`user`对象上（不区分提交方式），注意这里的`user`是`User`的结构体实例化指针；在`struct tag`中，使用`params`标签来指定参数名称与结构体属性名称对应关系；
1. 这里通过`GetToStruct`方法将参数赋值给`struct`对象，给定的参数为`struct`对象的指针，以便于内部自动赋值。同时，该系列方法也支持自动对结构体对象进行自动初始化，但是务必注意此时参数类型为`**User`（这样`GetToStruct`方法内部才能对`*User`进行初始化），如下：
    ```go
    user := (*User)(nil)
    r.GetToStruct(&user)
    ```
1. 通过`gvalid`标签为`gavlid`数据校验包特定的校验规则标签，这里的`gvalid`标签也可以使用`valid`或者`v`，详细请具体参考【[数据校验](util/gvalid/index.md)】章节；其中，密码字段的校验规则为`password3`，表示: `密码格式为任意6-18位的可见字符，必须包含大小写字母、数字和特殊字符`。
1. 当校验返回结果非`nil`时，表示校验不通过，这里使用`r.Response.WriteJson`方法返回`json`结果。

执行后，为方便测试，我们这里使用`curl`工具来测试下（当然也可以通过浏览器访问对应的`URL`来测试）：
```shell
$ curl "http://127.0.0.1:8199/user"
{"errcode":1,"message":{"password1":{"password3":"密码格式不合法，密码格式为任意6-18位的可见字符，必须包含大小写字母、数字和特殊字符","required":"字段不能为空"},"password2":{"password3":"密码格式不合法，密码格式为任意6-18位的可见字符，必须包含大小写字母、数字和特殊字符","required":"字段不能为空"},"uid":{"min":"字段最小值为1"},"username":{"length":"字段长度为6到30个字符","required":"字段不能为空"}}}

$ curl "http://127.0.0.1:8199/user?uid=1&name=john&password1=123&password2=456"
{"errcode":1,"message":{"password1":{"password3":"密码格式不合法，密码格式为任意6-18位的可见字符，必须包含大小写字母、数字和特殊字符"},"password2":{"password3":"密码格式不合法，密码格式为任意6-18位的可见字符，必须包含大小写字母、数字和特殊字符","same":"两次密码不一致，请重新输入"},"username":{"length":"字段长度为6到30个字符"}}}

$ curl "http://127.0.0.1:8199/user?uid=1&name=john&password1=Abc@123&password2=Abc@123"
{"errcode":1,"message":{"username":{"length":"字段长度为6到30个字符"}}}

$ curl "http://127.0.0.1:8199/user?uid=1&name=johng-cn&password1=John@123&password2=John@123"
{"errcode":0,"message":"ok"}
```

> 注意使用`curl`测试时在字符串参数中不能带有特殊含义的字符如`$`, `#`等。

## 示例2，流程共享变量
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
执行后，访问 [http://127.0.0.1:8199/](http://127.0.0.1:8199/) 后，页面输出内容为：
```
set name
set site
GoFrame: https://goframe.org
```


