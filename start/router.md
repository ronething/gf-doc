# 路由注册

https://github.com/gogf/gf-demos/blob/master/router/router.go
```go
package router

import (
    "github.com/gogf/gf-demos/app/api/user"
    "github.com/gogf/gf/frame/g"
)

func init() {
    // 用户模块 路由注册 - 使用执行对象注册方式
    g.Server().BindObject("/user", new(user.Controller))
}
```

这里使用执行对象注册方式。

可以看到，我们的路由注册管理也使用了包初始化方法`init`，这样做的好处是可以在`router`目录中使用不同的`go`文件注册不同的`init`来分别实现不同的路由注册。当项目的路由比较多的时候，可以采用不同的`go`文件管理不同的路由，这在团队协作的项目中也比较方便。