# 内存存储

内存存储比较简单，性能也很高效，但没有持久化存储`Session`数据，因此应用程序重启之后便会丢失`Session`数据，可用于特定的业务场景中。`gsession`的`内存`存储使用`StorageMemory`对象实现，

# 使用示例

https://github.com/gogf/gf/blob/master/.example/os/gsession/storage-memory/memory.go

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/os/gsession"
	"github.com/gogf/gf/os/gtime"
	"time"
)

func main() {
	s := g.Server()
	s.SetConfigWithMap(g.Map{
		"SessionMaxAge":  time.Minute,
		"SessionStorage": gsession.NewStorageMemory(),
	})
	s.BindHandler("/set", func(r *ghttp.Request) {
		r.Session.Set("time", gtime.Second())
		r.Response.Write("ok")
	})
	s.BindHandler("/get", func(r *ghttp.Request) {
		r.Response.Write(r.Session.Map())
	})
	s.BindHandler("/del", func(r *ghttp.Request) {
		r.Session.Clear()
		r.Response.Write("ok")
	})
	s.SetPort(8199)
	s.Run()
}
```
在该实例中，为了方便观察过期失效，我们将`Session`的过期时间设置为`1分钟`。执行后，
1. 首先，访问  http://127.0.0.1:8199/set  设置一个`Session`变量；
1. 随后，访问  http://127.0.0.1:8199/get  可以看到该`Session`变量已经设置并成功获取；
1. 接着，我们停止程序，并重新启动，再次访问  http://127.0.0.1:8199/get  ，可以看到`Session`变量已经没有了；


