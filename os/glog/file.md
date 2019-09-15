# 日志文件

默认情况下，日志文件名称以当前时间日期命名，格式为`YYYY-MM-DD.log`，我们可以使用`SetFile`方法来设置文件名称的格式，并且文件名称格式支持【[`gtime`时间格式](os/gtime/index.md)】。简单示例：

```go
package main

import (
    "github.com/gogf/gf/os/glog"
    "github.com/gogf/gf/os/gfile"
    "github.com/gogf/gf/frame/g"
)

// 设置日志等级
func main() {
    path := "/tmp/glog"
    glog.SetPath(path)
    glog.SetStdoutPrint(false)
    // 使用默认文件名称格式
    glog.Println("标准文件名称格式，使用当前时间时期")
    // 通过SetFile设置文件名称格式
    glog.SetFile("stdout.log")
    glog.Println("设置日志输出文件名称格式为同一个文件")
    // 链式操作设置文件名称格式
    glog.File("stderr.log").Println("支持链式操作")
    glog.File("error-{Ymd}.log").Println("文件名称支持带gtime日期格式")
    glog.File("access-{Ymd}.log").Println("文件名称支持带gtime日期格式")

    list, err := gfile.ScanDir(path, "*")
    g.Dump(err)
    g.Dump(list)
}
```
执行后，输出结果为：
```html
null
[
	"/tmp/glog/2018-10-10.log",
	"/tmp/glog/access-20181010.log",
	"/tmp/glog/error-20181010.log",
	"/tmp/glog/stderr.log",
	"/tmp/glog/stdout.log"
]
```
可以看到，文件名称格式中如果需要使用`gtime`时间格式，格式内容需要使用`{xxx}`包含起来。该示例中也使用到了`链式操作`的特性，具体请看后续说明。