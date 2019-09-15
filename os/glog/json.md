# Json日志格式

`glog`对日志分析工具非常友好，支持输出`JSON`格式的日志内容，以便于后期对日志内容进行解析分析。想要支持`JSON`数据格式的日志输出非常简单，给打印方法提供`map`/`struct`类型参数即可。

使用示例：

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/os/glog"
)

func main() {
	glog.Debug(g.Map{"uid" : 100, "name" : "john"})
	type User struct {
		Uid  int    `json:"uid"`
		Name string `json:"name"`
	}
	glog.Debug(User{100, "john"})
}
```

执行后，终端输出结果：
```html
2019-06-02 15:28:52.653 [DEBU] {"name":"john","uid":100}
2019-06-02 15:28:52.653 [DEBU] {"uid":100,"name":"john"}
```

