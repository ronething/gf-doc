[TOC]

# `JSON/XML`支持

从`GF v1.11`版本开始，`Request`对象提供了对客户端提交的`JSON/XML`数据格式的原生支持，为开发者提供了更便捷的数据接收，以进一步提高开发效率。

代码示例：

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/util/gvalid"
)

type RegisterReq struct {
	Name  string `p:"username"  v:"required|length:6,30#请输入账号|账号长度为:min到:max位"`
	Pass  string `p:"password1" v:"required|length:6,30#请输入密码|密码长度不够"`
	Pass2 string `p:"password2" v:"required|length:6,30|same:password1#请确认密码|两次密码不一致"`
}

type RegisterRes struct {
	Code  int         `json:"code"`
	Error string      `json:"error"`
	Data  interface{} `json:"data"`
}

func main() {
	s := g.Server()
	s.BindHandler("/register", func(r *ghttp.Request) {
		var req *RegisterReq
		if err := r.Parse(&req); err != nil {
			// Validation error.
			if v, ok := err.(*gvalid.Error); ok {
				r.Response.WriteJsonExit(RegisterRes{
					Code:  1,
					Error: v.FirstString(),
				})
			}
			// Other error.
			r.Response.WriteJsonExit(RegisterRes{
				Code:  1,
				Error: err.Error(),
			})
		}
		// ...
		r.Response.WriteJsonExit(RegisterRes{
			Data: req,
		})
	})
	s.SetPort(8199)
	s.Run()
}
```

## `JSON`

我们通过`curl`工具来提交`JSON`数据来测试：
```
$ curl -d '{"username":"johngcn","password1":"123456","password2":"123456"}' "http://127.0.0.1:8199/register"
{"code":0,"error":"","data":{"Name":"johngcn","Pass":"123456","Pass2":"123456"}}

$ curl -d '{"username":"johngcn","password1":"123456","password2":"123456"}' "http://127.0.0.1:8199/register"
{"code":1,"error":"两次密码不一致","data":null}
```

可以看到，我们提交的`JSON`内容也被`Parse`方法智能地转换为了结构体对象。

## `XML`

我们通过`curl`工具来提交`XML`数据来测试：
```
$ curl -d '<?xml version="1.0" encoding="UTF-8"?><doc><username>johngcn</username><password1>123456</password1><password2>123456</password2></doc>' "http://127.0.0.1:8199/register"
{"code":0,"error":"","data":{"Name":"johngcn","Pass":"123456","Pass2":"123456"}}

$ curl -d '<?xml version="1.0" encoding="UTF-8"?><doc><username>johngcn</username><password1>123456</password1><password2>12345</password2></doc>' "http://127.0.0.1:8199/register"
{"code":1,"error":"两次密码不一致","data":null}
```
可以看到，我们提交的`XML`内容也被`Parse`方法智能地转换为了结构体对象。

