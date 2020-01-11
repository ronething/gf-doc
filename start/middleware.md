
# 中间件使用

https://github.com/gogf/gf-demos/tree/master/app/service/middleware

## 跨域处理

允许跨域请求。

```go
package middleware

import "github.com/gogf/gf/net/ghttp"

// 允许接口跨域请求
func CORS(r *ghttp.Request) {
	r.Response.CORSDefault()
	r.Middleware.Next()
}
```

## 鉴权处理

只有在用户登录后才可通过。

```go
package middleware

import (
	"github.com/gogf/gf-demos/app/service/user"
	"github.com/gogf/gf/net/ghttp"
	"net/http"
)

// 鉴权中间件，只有登录成功之后才能通过
func Auth(r *ghttp.Request) {
	if user.IsSignedIn(r.Session) {
		r.Middleware.Next()
	} else {
		r.Response.WriteStatus(http.StatusForbidden)
	}
}
```









