[TOC]


# 包名约定

根据官方[《Effective Go》](https://golang.google.cn/doc/effective_go.html#package-names)建议，包名尽量采用言简意赅的名称(`short, concise, evocative`)。并且推荐通过不同的`import`路径来区分相同包名的包引入。


# 控制器实现

控制器主要用于接口的路由注册以及对于用户输入参数的处理。

我们这里使用执行对象注册方式。

https://github.com/gogf/gf-demos/blob/master/app/api/user/user.go
```go
package user

import (
	"github.com/gogf/gf-demos/app/service/user"
	"github.com/gogf/gf-demos/library/response"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/util/gvalid"
)

// 用户API管理对象
type Controller struct{}

// 用户注册接口
func (c *Controller) SignUp(r *ghttp.Request) {
	if err := user.SignUp(r.GetPostMapStrStr()); err != nil {
		response.Json(r, 1, err.Error())
	} else {
		response.Json(r, 0, "ok")
	}
}

// 用户登录接口
func (c *Controller) SignIn(r *ghttp.Request) {
	data := r.GetPostMapStrStr()
	rules := map[string]string{
		"passport": "required",
		"password": "required",
	}
	msgs := map[string]interface{}{
		"passport": "账号不能为空",
		"password": "密码不能为空",
	}
	if e := gvalid.CheckMap(data, rules, msgs); e != nil {
		response.Json(r, 1, e.String())
	}
	if err := user.SignIn(data["passport"], data["password"], r.Session); err != nil {
		response.Json(r, 1, err.Error())
	} else {
		response.Json(r, 0, "ok")
	}
}

// 判断用户是否已经登录
func (c *Controller) IsSignedIn(r *ghttp.Request) {
	if user.IsSignedIn(r.Session) {
		response.Json(r, 0, "ok")
	} else {
		response.Json(r, 1, "")
	}
}

// 用户注销/退出接口
func (c *Controller) SignOut(r *ghttp.Request) {
	user.SignOut(r.Session)
	response.Json(r, 0, "ok")
}

// 检测用户账号接口(唯一性校验)
func (c *Controller) CheckPassport(r *ghttp.Request) {
	passport := r.GetString("passport")
	if e := gvalid.Check(passport, "required", "请输入账号"); e != nil {
		response.Json(r, 1, e.String())
	}
	if user.CheckPassport(passport) {
		response.Json(r, 0, "ok")
	}
	response.Json(r, 1, "账号已经存在")
}

// 检测用户昵称接口(唯一性校验)
func (c *Controller) CheckNickName(r *ghttp.Request) {
	nickname := r.Get("nickname")
	if e := gvalid.Check(nickname, "required", "请输入昵称"); e != nil {
		response.Json(r, 1, e.String())
	}
	if user.CheckNickName(r.GetString("nickname")) {
		response.Json(r, 0, "ok")
	}
	response.Json(r, 1, "昵称已经存在")
}
```

# 逻辑封装实现

我们这里没有使用到数据模型，仅使用了 逻辑封装层+`gdb` 来操作数据库。

## 用户逻辑

主要用于用户接口的业务逻辑封装。

https://github.com/gogf/gf-demos/blob/master/app/service/user/user.go
```go
package user

import (
	"database/sql"
	"errors"
	"fmt"
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/os/gtime"
	"github.com/gogf/gf/util/gvalid"
)

const (
	USER_SESSION_MARK = "user_info"
)

var (
	// 表对象
	table = g.DB().Table("user").Safe()
)

// 用户注册
func SignUp(data g.MapStrStr) error {
	// 数据校验
	rules := []string{
		"passport @required|length:6,16#账号不能为空|账号长度应当在:min到:max之间",
		"password2@required|length:6,16#请输入确认密码|密码长度应当在:min到:max之间",
		"password @required|length:6,16|same:password2#密码不能为空|密码长度应当在:min到:max之间|两次密码输入不相等",
	}
	if e := gvalid.CheckMap(data, rules); e != nil {
		return errors.New(e.String())
	}
	if _, ok := data["nickname"]; !ok {
		data["nickname"] = data["passport"]
	}
	// 唯一性数据检查
	if !CheckPassport(data["passport"]) {
		return errors.New(fmt.Sprintf("账号 %s 已经存在", data["passport"]))
	}
	if !CheckNickName(data["nickname"]) {
		return errors.New(fmt.Sprintf("昵称 %s 已经存在", data["nickname"]))
	}
	// 记录账号创建/注册时间
	if _, ok := data["create_time"]; !ok {
		data["create_time"] = gtime.Now().String()
	}
	if _, err := table.Filter().Data(data).Save(); err != nil {
		return err
	}
	return nil
}

// 判断用户是否已经登录
func IsSignedIn(session *ghttp.Session) bool {
	return session.Contains(USER_SESSION_MARK)
}

// 用户登录，成功返回用户信息，否则返回nil; passport应当会md5值字符串
func SignIn(passport, password string, session *ghttp.Session) error {
	record, err := table.Where("passport=? and password=?", passport, password).One()
	if err != nil && err != sql.ErrNoRows {
		return err
	}
	if record == nil {
		return errors.New("账号或密码错误")
	}
	session.Set(USER_SESSION_MARK, record)
	return nil
}

// 用户注销
func SignOut(session *ghttp.Session) {
	session.Remove(USER_SESSION_MARK)
}

// 检查账号是否符合规范(目前仅检查唯一性),存在返回false,否则true
func CheckPassport(passport string) bool {
	if i, err := table.Where("passport", passport).Count(); err != nil && err != sql.ErrNoRows {
		return false
	} else {
		return i == 0
	}
}

// 检查昵称是否符合规范(目前仅检查唯一性),存在返回false,否则true
func CheckNickName(nickname string) bool {
	if i, err := table.Where("nickname", nickname).Count(); err != nil && err != sql.ErrNoRows {
		return false
	} else {
		return i == 0
	}
}
```

## 工具类包

主要用于返回JSON数据格式的统一。

https://github.com/gogf/gf-demos/blob/master/library/response/response.go
```go
package response

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
)

// 标准返回结果数据结构封装。
// 返回固定数据结构的JSON:
// err:  错误码(0:成功, 1:失败, >1:错误码);
// msg:  请求结果信息;
// data: 请求结果,根据不同接口返回结果的数据结构不同;
func Json(r *ghttp.Request, err int, msg string, data ...interface{}) {
	responseData := interface{}(nil)
	if len(data) > 0 {
		responseData = data[0]
	}
	r.Response.WriteJson(g.Map{
		"err":  err,
		"msg":  msg,
		"data": responseData,
	})
	r.Exit()
}
```
