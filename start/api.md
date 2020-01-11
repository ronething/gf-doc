[TOC]

# 控制器实现

## 基本介绍

### 结构化约束
控制器的输入与输出使用了结构体定义进行约束，结构化维护输入输出数据结构是推荐的方式。例如：
```go
// 账号唯一性检测请求参数，用于前后端交互参数格式约定
type CheckPassportRequest struct {
	Passport string
}
```
虽然只有一个参数，但也使用了结构体进行定义，我们可以直接查看该结构体便可得知该接口的输入参数是什么，而不用进入代码中去分析，从而极大提高维护效率。

此外，这里使用了"对象继承"：
```go
// 注册请求参数，用于前后端交互参数格式约定
type SignUpRequest struct {
	user.SignUpInput
}
```

### 结构体转换
结构体转换可以使用`GetStruct`或者`Parse`方法，其中`Parse`同时可以执行数据校验。结构体转换方法的参数都可以给定一个结构体的空指针，内部会自动初始化结构体对象，转换失败（例如提交参数不存在）不会执行初始化。例如：
```go
var data *SignUpRequest
// 这里没有使用Parse而是仅用GetStruct获取对象，
// 数据校验交给后续的service层统一处理。
if err := r.GetStruct(&data); err != nil {
    response.JsonExit(r, 1, err.Error())
}
```

### 数据校验

可以通过给结构体绑定`v`的标签进行设定校验规则以及定义的错误提示。例如：
```go
// 登录请求参数，用于前后端交互参数格式约定
type SignInRequest struct {
	Passport string `v:"required#账号不能为空"`
	Password string `v:"required#密码不能为空"`
}
```

### 数据传参

控制器负责接收、转换、校验、处理请求参数后，将所需的参数传递给调用的`service`层一个或者多个包方法，而不是直接将`Request`对象传递给`service`。例如：
```go
// 用户登录接口
func (c *Controller) SignIn(r *ghttp.Request) {
	var data *SignInRequest
	if err := r.Parse(&data); err != nil {
		response.JsonExit(r, 1, err.Error())
	}
	if err := user.SignIn(data.Passport, data.Password, r.Session); err != nil {
		response.JsonExit(r, 1, err.Error())
	} else {
		response.JsonExit(r, 0, "ok")
	}
}
```
需要注意的是，其中的`Session`对象也是通过控制器传递给`service`。

> 在`Go`的HTTP请求流程中，不存在"全局变量"获取请求参数的方式，只有根据`service`的需要按需传递参数。


## 实现代码

https://github.com/gogf/gf-demos/blob/master/app/api/user/user.go

```go
package user

import (
	"github.com/gogf/gf-demos/app/service/user"
	"github.com/gogf/gf-demos/library/response"
	"github.com/gogf/gf/net/ghttp"
)

// 用户API管理对象
type Controller struct{}

// 注册请求参数，用于前后端交互参数格式约定
type SignUpRequest struct {
	user.SignUpInput
}

// 用户注册接口
func (c *Controller) SignUp(r *ghttp.Request) {
	var data *SignUpRequest
	// 这里没有使用Parse而是仅用GetStruct获取对象，
	// 数据校验交给后续的service层统一处理。
	if err := r.GetStruct(&data); err != nil {
		response.JsonExit(r, 1, err.Error())
	}
	if err := user.SignUp(&data.SignUpInput); err != nil {
		response.JsonExit(r, 1, err.Error())
	} else {
		response.JsonExit(r, 0, "ok")
	}
}

// 登录请求参数，用于前后端交互参数格式约定
type SignInRequest struct {
	Passport string `v:"required#账号不能为空"`
	Password string `v:"required#密码不能为空"`
}

// 用户登录接口
func (c *Controller) SignIn(r *ghttp.Request) {
	var data *SignInRequest
	if err := r.Parse(&data); err != nil {
		response.JsonExit(r, 1, err.Error())
	}
	if err := user.SignIn(data.Passport, data.Password, r.Session); err != nil {
		response.JsonExit(r, 1, err.Error())
	} else {
		response.JsonExit(r, 0, "ok")
	}
}

// 判断用户是否已经登录
func (c *Controller) IsSignedIn(r *ghttp.Request) {
	if user.IsSignedIn(r.Session) {
		response.JsonExit(r, 0, "ok")
	} else {
		response.JsonExit(r, 1, "")
	}
}

// 用户注销/退出接口
func (c *Controller) SignOut(r *ghttp.Request) {
	if err := user.SignOut(r.Session); err != nil {
		response.JsonExit(r, 1, "")
	}
	response.JsonExit(r, 0, "ok")
}

// 账号唯一性检测请求参数，用于前后端交互参数格式约定
type CheckPassportRequest struct {
	Passport string
}

// 检测用户账号接口(唯一性校验)
func (c *Controller) CheckPassport(r *ghttp.Request) {
	var data *CheckPassportRequest
	if err := r.Parse(&data); err != nil {
		response.JsonExit(r, 1, err.Error())
	}
	if data.Passport != "" && !user.CheckPassport(data.Passport) {
		response.JsonExit(r, 1, "账号已经存在")
	}
	response.JsonExit(r, 0, "ok")
}

// 账号唯一性检测请求参数，用于前后端交互参数格式约定
type CheckNickNameRequest struct {
	Nickname string
}

// 检测用户昵称接口(唯一性校验)
func (c *Controller) CheckNickName(r *ghttp.Request) {
	var data *CheckNickNameRequest
	if err := r.Parse(&data); err != nil {
		response.JsonExit(r, 1, err.Error())
	}
	if data.Nickname != "" && !user.CheckNickName(data.Nickname) {
		response.JsonExit(r, 1, "昵称已经存在")
	}
	response.JsonExit(r, 0, "ok")
}

// 获取用户详情
func (c *Controller) Profile(r *ghttp.Request) {
	response.JsonExit(r, 0, "", user.GetProfile(r.Session))
}
```