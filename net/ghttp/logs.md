
[TOC]

`gf`框架提供了完善的Web Server日志管理功能，包括`access log`以及`error log`，并可自定义日志输出回调方法，开发者可灵活使用。

# 相关配置
在Web Server中的日志相关配置选项如下：
```go
// 日志配置
LogPath           string                // 存放日志的目录路径(默认为空，表示不写文件)
LogHandler        LogHandler            // 自定义日志处理回调方法(默认为空)
LogStdPrint       bool                  // 是否打印日志到终端(默认开启)
ErrorLogEnabled   bool                  // 是否开启error log(默认开启)
AccessLogEnabled  bool                  // 是否开启access log(默认关闭)
```
其中需要特别说明的是，默认情况下，日志不会输出到文件中，而是直接打印到终端。并且，默认情况下的请求日志终端输出是关闭的，仅有错误日志默认开启。

# 相关方法

```go
// 设置日志目录，只有在设置了日志目录的情况下才会输出日志到日志文件中。
// 日志文件路径格式为：
// 1. 请求日志: access/YYYY-MM-DD.log
// 2. 错误日志: error/YYYY-MM-DD.log
func (s *Server)SetLogPath(path string)
// 设置日志内容是否输出到终端，默认情况下只有错误日志才会自动输出到终端。
// 如果需要输出请求日志到终端，默认情况下使用SetAccessLogEnabled方法开启请求日志特性即可。
func (s *Server)SetLogStdPrint(enabled bool)
// 设置是否开启access log日志功能
func (s *Server)SetAccessLogEnabled(enabled bool)
// 设置是否开启error log日志功能
func (s *Server)SetErrorLogEnabled(enabled bool)

// 获取日志写入的回调函数
func (s *Server)GetLogHandler() LogHandler
// 获取日志目录
func (s *Server)GetLogPath() string
// access log日志功能是否开启
func (s *Server)IsAccessLogEnabled() bool
// error log日志功能是否开启
func (s *Server)IsErrorLogEnabled() bool
```

# 请求日志

我们来看一个简单的例子：
```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := g.GetServer()
    s.BindHandler("/log/access", func(r *ghttp.Request){
        r.Response.Writeln("请在运行终端查看日志输出")
    })
    s.SetAccessLogEnabled(true)
    s.SetPort(8199)
    s.Run()
}
```

我们在任何地方(包括运行时)可以调用`SetAccessLogEnabled(true)`开启`access log`的记录功能(并可通过`SetAccessLogEnabled(false)`随时动态关闭日志记录功能)，默认情况下，日志内容将会输出到终端界面。如以上示例程序执行后，访问 http://127.0.0.1:8199/log/access ，日志内容将会输出到终端上，如下：
```shell
2018-04-20 18:11:57.344 200 "GET http 127.0.0.1:8199 /log/access HTTP/1.1" 0.120, 127.0.0.1, "", "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/53.0.2785.143 Chrome/53.0.2785.143 Safari/537.36"
```
日志格式: `访问时间(精确到毫秒)` `HTTP状态码` "`请求方式` `请求前缀` `请求地址` `请求协议`" `执行时间(毫秒)` `客户端IP` `来源URL` `UserAgent`。其中，`请求前缀`为`http`或者`https`，`请求协议`往往为`HTTP/1.0`或者`HTTP/1.1`。

> 注意，日志中记录的`执行时间`单位为`毫秒`，绝大多数情况下看到的时间几乎都是`0.xxx`毫秒时间，也就是说执行时间都是纳秒级不到1毫秒。

# 错误日志

Web Server运行产生的任何错误信息(默认开启，输出到终端)，都将会被记录到`error log`中，以下是一个手动触发异常的示例：

```go
package main

import (
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := ghttp.GetServer()
    s.BindHandler("/log/error", func(r *ghttp.Request){
        panic("异常信息")
    })
    s.SetErrorLogEnabled(true)
    s.SetPort(8199)
    s.Run()
}
```

运行并访问后，终端输出的错误日志信息如下：
```shell
2018-04-20 18:31:03.484 [ERRO] "GET http 127.0.0.1:8199 /log/error HTTP/1.1" 0.098, 127.0.0.1, "", "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/53.0.2785.143 Chrome/53.0.2785.143 Safari/537.36", 异常信息
Trace:
1. /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/net/ghttp/log.go:10
2. /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/net/ghttp/http_server_handler.go:83
3. /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/net/ghttp/http_server_handler.go:52
4. /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/net/ghttp/http_server_handler.go:25
5. /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/net/ghttp/http_server.go:137
```
错误信息会打印出对应错误产生的```caller backtrace```，以便于错误定位以及开发者分析问题原因。

> WebServer产生的任何`panic`错误都将会被自动捕获到错误日志中，因此对于业务端程序来讲，无论是在控制器中、业务封装层、数据模型中，如果产生了错误想要直接退出业务请求处理，直接`panic`即可。特别是对于绝大多数`golang`风格的方法`error`返回值判断，处理将会变得异常的简单。如果是业务程序自身定义的方法`error`甚至可不用返回，可以根据业务情况选择直接`panic`。

# 输出日志到文件

我们可以通过```SetLogPath```方法来设置日志目录，这样我们的日志信息将会保存到文件中，示例如下：

```go
package main

import (
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := ghttp.GetServer()
    s.BindHandler("/log/path", func(r *ghttp.Request){
        r.Response.Writeln("请到/tmp/gf.log目录查看日志")
    })
    s.SetLogPath("/tmp/gf.log")
    s.SetAccessLogEnabled(true)
    s.SetErrorLogEnabled(true)
    s.SetPort(8199)
    s.Run()
}
```
运行并访问后，日志将会被保存到```/tmp/gf.log/access```目录中，其中```access```目录为请求日志，此外错误日志将会写入到```error```目录中。手动在终端查看目录结构如下：
```shell
john@johnstation:/tmp$ tree gf.log
gf.log/
├── access
│   └── 2018-04-20.log
└── error
    └── 2018-04-20.log
```




# 自定义日志处理

`gf`支持自定义的日志处理特性，便于开发者灵活控制日志输出，使用```SetLogHandler```来实现日志回调方法注册。

以下是自定义日志处理的简单示例：
```go
package main

import (
    "fmt"
    "net/http"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := ghttp.GetServer()
    s.BindHandler("/log/handler", func(r *ghttp.Request){
        r.Response.WriteStatus(http.StatusNotFound, "文件找不到了")
    })
    s.SetAccessLogEnabled(true)
    s.SetErrorLogEnabled(true)
    s.SetLogHandler(func(r *ghttp.Request, error ...interface{}) {
        if len(error) > 0 {
            // 如果是错误日志
            fmt.Println("错误产生了：", error[0])
        }
        // 这里是请求日志
        fmt.Println("请求处理完成，请求地址:", r.URL.String(), "请求结果:", r.Response.Status)
    })
    s.SetPort(8199)
    s.Run()
}
```

运行并请求后，终端输出内容如下：
```shell
请求处理完成，请求地址: /log/handler 请求结果: 404
```