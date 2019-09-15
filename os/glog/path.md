# 日志目录

默认情况下，`glog`将会输出日志内容到标准输出，我们可以通过`SetPath`方法设置日志输出的目录路径，这样日志内容将会写入到日志文件中，并且由于其内部使用了`gfpool`文件指针池，文件写入的效率相当优秀。简单示例：

```go
package main

import (
    "github.com/gogf/gf/os/glog"
    "github.com/gogf/gf/os/gfile"
    "github.com/gogf/gf/frame/g"
)

// 设置日志输出路径
func main() {
    path := "/tmp/glog"
    glog.SetPath(path)
    glog.Println("日志内容")
    list, err := gfile.ScanDir(path, "*")
    g.Dump(err)
    g.Dump(list)
}
```
执行后，输出内容为：
```html
2018-10-10 14:03:46.904 日志内容
null
[
	"/tmp/glog/2018-10-10.log"
]
```
当通过`SetPath`设置日志的输出目录，如果目录不存在时，将会递归创建该目录路径。可以看到，执行`Println`之后，在`/tmp`下创建了日志目录`glog`，并且在其下生成了日志文件。同时，我们也可以看见日志内容不仅输出到了文件，默认情况下也输出到了终端，我们可以通过`SetStdoutPrint(false)`方法来关闭终端的日志输出，这样日志内容仅会输出到日志文件中。


