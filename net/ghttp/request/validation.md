


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
	s.Group("/", func(group *ghttp.RouterGroup) {
		group.ALL("/user", func(r *ghttp.Request) {
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

