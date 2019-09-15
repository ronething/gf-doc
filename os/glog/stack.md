# Trace特性

错误日志信息支持`trace`特性，可以通过`Notice*/Warning*/Error*/Critical*/Panic*/Fatal*`错误方法触发，也可以通过`GetStack/PrintStack`获取/打印。错误信息的`trace`信息对于调试错误来说相当有用。

## 示例1，通过错误方法触发
```go
package main

import "github.com/gogf/gf/os/glog"

func Test() {
	glog.Error("This is error!")
}

func main() {
	Test()
}
```
打印出的结果如下：
```html
2019-07-12 22:19:23.421 [ERRO] This is error! 
Stack:
1. main.Test
   /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_error.go:6
2. main.main
   /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_error.go:10
```

## 示例2，通过Stack方法打印

```go
package main

import (
	"fmt"

	"github.com/gogf/gf/os/glog"
)

func main() {

	glog.PrintStack()
	glog.New().PrintStack()

	fmt.Println(glog.GetStack())
	fmt.Println(glog.New().GetStack())
}
```
执行后，输出结果为：
```html
2019-07-12 22:20:28.070 Stack:
1. main.main
   /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_stack.go:11

2019-07-12 22:20:28.070 Stack:
1. main.main
   /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_stack.go:12

1. main.main
   /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_stack.go:14

1. main.main
   /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_stack.go:15
```

## 示例3，打印gerror.Error

`glog`日志模块支持对标准错误以及`gerror`错误的堆栈打印支持。

```go
package main

import (
	"errors"
	"github.com/gogf/gf/errors/gerror"
	"github.com/gogf/gf/os/glog"
)

func MakeError() error {
	return errors.New("connection closed with normal error")
}

func MakeGError() error {
	return gerror.New("connection closed with gerror")
}

func TestGError() {
	err1 := MakeError()
	err2 := MakeGError()
	glog.Error(err1)
	glog.Error(err2)
}

func main() {
	TestGError()
}
```
执行后，终端输出：
```html
2019-07-12 22:23:11.467 [ERRO] connection closed with normal error 
Stack:
1. main.TestGError
   /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_gerror.go:20
2. main.main
   /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_gerror.go:25

2019-07-12 22:23:11.467 [ERRO] connection closed with gerror
1. connection closed with gerror
    1). main.MakeGError
        /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_gerror.go:14
    2). main.TestGError
        /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_gerror.go:19
    3). main.main
        /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_gerror.go:25
Stack:
1. main.TestGError
   /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_gerror.go:21
2. main.main
   /home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/glog/glog_gerror.go:25
```




