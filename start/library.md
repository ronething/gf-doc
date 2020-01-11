
## 基础类库

主要固定返回数据格式及数据结构。

其中`JsonExit`与`Json`的区别在于，`JsonExit`调用时会输出`JSON`数据后直接退出当前的路由方法；而`Json`在执行输出后会继续执行后续的路由方法逻辑。

https://github.com/gogf/gf-demos/blob/master/library/response/response.go
```go
package response

import (
	"github.com/gogf/gf/net/ghttp"
)

// 数据返回通用JSON数据结构
type JsonResponse struct {
	Code    int         `json:"code"`    // 错误码((0:成功, 1:失败, >1:错误码))
	Message string      `json:"message"` // 提示信息
	Data    interface{} `json:"data"`    // 返回数据(业务接口定义具体数据结构)
}

// 标准返回结果数据结构封装。
func Json(r *ghttp.Request, code int, message string, data ...interface{}) {
	responseData := interface{}(nil)
	if len(data) > 0 {
		responseData = data[0]
	}
	r.Response.WriteJson(JsonResponse{
		Code:    code,
		Message: message,
		Data:    responseData,
	})
}

// 返回JSON数据并退出当前HTTP执行函数。
func JsonExit(r *ghttp.Request, err int, msg string, data ...interface{}) {
	Json(r, err, msg, data...)
	r.Exit()
}
```
