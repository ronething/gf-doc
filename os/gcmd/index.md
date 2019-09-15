# gcmd

`gcmd`模块提供了对命令行参数、选项的读取功能，以及对应参数的回调函数绑定功能。

**使用方式**：
```go
import "github.com/gogf/gf/os/gcmd"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/os/gcmd

## 参数/选项读取

**参数**是从索引`0`开始的字符串数组，**选项**是以"`--`"或者"`-`"开始的KV键值对数据。

假如执行命令如下：
```
./binary start daemon --timeout=60 --logpath=/home/www/log
```
1. 获取执行参数start、daemon
```go
// start
gcmd.Value.Get(0)
// daemon
gcmd.Value.Get(1)
```


1. 获取执行选项timeout、logpath
```go
// timeout
gcmd.Option.Get("timeout")
// logpath
gcmd.Option.Get("logpath")
```

执行选项通过给定键名获取，并且执行选项的格式可以是```--键名=键值```(两个"`-`"符号)，也可以是```-键名=键值```(一个"`-`"符号)。


## 回调函数绑定

对于命令行的可执行程序来讲，需要根据执行参选定位到对应的入口函数进行处理，gcmd提供了以下方法来实现：
```go
func AutoRun() error
func BindHandle(cmd string, f func()) error
func RunHandle(cmd string) error
```

1. `BindHandler`绑定执行参数对应的回调函数，执行参数索引为`0`，回调函数参数类型为`func()`。
2. `RunHandler`运行指定执行参数的回调函数；
3. `AutoRun`根据执行`参数[0]`自动运行对应注册的回调函数； 

使用示例：
```go
package main

import (
    "fmt"
    "github.com/gogf/gf/os/gcmd"
)

func help() {
    fmt.Println("This is help.")
}

func test() {
    fmt.Println("This is test.")
}

func main() {
    gcmd.BindHandle("help", help)
    gcmd.BindHandle("test", test)
    gcmd.AutoRun()
}
```
编译成二进制文件后进行执行测试：
```shell
$ go build main.go 
$ ./main test
This is test.
$ ./main help
This is help.
$ ./main 
$ 
```







