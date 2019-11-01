
# 服务配置

我们使用`go.mod`来管理项目依赖。

## go.mod

https://github.com/gogf/gf-demos/blob/master/go.mod
```
module github.com/gogf/gf-demos

require github.com/gogf/gf latest
```
其中注意`module`名称设置为`github.com/gogf/gf-demos`。



## 配置文件
这里主要配置了数据库的连接信息。

https://github.com/gogf/gf-demos/blob/master/config/config.example.toml
```toml
# 应用系统设置
[setting]
    logpath = "/tmp/log/gf-demos"

# 数据库连接
[database]
    link = "mysql:root:12345678@tcp(127.0.0.1:3306)/test"
```

## 启动设置

https://github.com/gogf/gf-demos/blob/master/boot/boot.go
```go
package boot

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/os/glog"
)

// 用于应用初始化。
func init() {
	v := g.View()
	c := g.Config()
	s := g.Server()

	// 模板引擎配置
	v.AddPath("template")
	v.SetDelimiters("${", "}")

	// glog配置
	logpath := c.GetString("setting.logpath")
	glog.SetPath(logpath)
	glog.SetStdoutPrint(true)

	// Web Server配置
	s.SetServerRoot("public")
	s.SetLogPath(logpath)
	s.SetNameToUriType(ghttp.URI_TYPE_ALLLOWER)
	s.SetPort(8199)
}
```
可以看到，我们的包初始化管理使用了包初始化方法`init`，这样做的好处是可以在`boot`目录中使用不同的`go`文件注册不同的`init`来分别实现不同的初始化配置管理，在业务比较复杂的项目中比较实用。如果仅仅是需要一个初始化方法即可完成配置，那么开发者可以自定义一个初始化方法，在`main`包中直接调用即可。

