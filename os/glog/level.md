# 日志级别

日志级别用于管理日志的输出，我们可以通过设定特定的日志级别来关闭/开启特定的日志内容。通过`SetLevel`方法可以设置日志级别，`glog`支持以下几种日志级别常量设定：
```go
LEVEL_ALL  
LEVEL_DEBU 
LEVEL_INFO
LEVEL_NOTI
LEVEL_WARN
LEVEL_ERRO
LEVEL_CRIT
```
我们可以通过`位操作`组合使用这几种级别，例如其中`LEVEL_ALL`等价于`LEVEL_DEBU | LEVEL_INFO | LEVEL_NOTI | LEVEL_WARN | LEVEL_ERRO | LEVEL_CRIT`。例如我们可以通过`LEVEL_ALL & ^LEVEL_DEBU & ^LEVEL_INFO & ^LEVEL_NOTI`来过滤掉`LEVEL_DEBU/LEVEL_INFO/LEVEL_NOTI`日志内容。

> 当然`glog`模块还有其他的一些级别，如`CRIT`和`FATA`，但是这两个级别是非常严重的错误，无法由开发者自定义屏蔽，产生严重错误的时候。将会产生一些额外的系统动作，如`panic`/`exit`。


简单示例：

```go
package main

import (
    "github.com/gogf/gf/os/glog"
)

// 设置日志等级，过滤掉Info日志信息
func main() {
    l := glog.New()
    l.Info("info1")
    l.SetLevel(glog.LEVEL_ALL^glog.LEVEL_INFO)
    l.Info("info2")
}
```
执行后，输出结果为：
```html
2018-10-10 14:38:48.687 [INFO] info1
```

> 我们也可以通过gf框架的`g.SetLogLevel`/`g.SetDebug`模块来管理日志级别、调试信息。对应的日志等级如下：
```
LOG_LEVEL_ALL
LOG_LEVEL_DEBU
LOG_LEVEL_INFO
LOG_LEVEL_NOTI
LOG_LEVEL_WARN
LOG_LEVEL_ERRO
LOG_LEVEL_CRIT
```