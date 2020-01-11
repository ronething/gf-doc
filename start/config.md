
# 服务配置

我们推荐使用`go.mod`来管理项目依赖。

## go.mod

https://github.com/gogf/gf-demos/blob/master/go.mod
```
module github.com/gogf/gf-demos

require github.com/gogf/gf latest

go 1.12
```
其中注意`module`名称设置为`github.com/gogf/gf-demos`。这里我们只需要依赖`GF`框架即可。其中的`go 1.12`表示运行该项目所需的最低`Go`版本，这里也可以不设置。`Goland`会自动帮我们设置为当前使用的`Go`版本。

## 配置文件
`GF`框架的核心组件均实现了便捷的文件配置管理方式，包括`Server`、日志组件、数据库ORM、模板引擎等等，非常强大便捷。具体的配置项可以查看后续对应的章节介绍。

https://github.com/gogf/gf-demos/blob/master/config/config.example.toml
```toml
# HTTP Server配置
[server]
	Address        = ":8199"
	ServerRoot     = "public"
	ServerAgent    = "gf-demos"
	LogPath        = "/tmp/log/gf-demos/server"
	NameToUriType  = 2
	RouteOverWrite = true

# 全局日志配置
[logger]
    Path   = "/tmp/log/gf-demos"
    Level  = "all"
    Stdout = true

# 模板引擎配置
[viewer]
    Path        = "template"
    DefaultFile = "index.html"
    Delimiters  =  ["${", "}"]

# 数据库连接
[database]
    link  = "mysql:root:12345678@tcp(127.0.0.1:3306)/test"
    debug = true
    # 数据库日志对象配置
    [database.logger]
        Path   = "/tmp/log/gf-demos/sql"
        Level  = "all"
        Stdout = true
```



## 启动设置

在`boot`包中执行代码层级的初始化，比如一些组件模块的设置。

https://github.com/gogf/gf-demos/blob/master/boot/boot.go
```go
package boot

// 用于应用初始化。
func init() {
	// 添加代码层级的启动配置
}
```
可以看到，我们的包初始化管理使用了包初始化方法`init`，这样做的好处是可以在`boot`目录中使用不同的`go`文件注册不同的`init`来分别实现不同的初始化配置管理，在业务比较复杂的项目中比较实用。

