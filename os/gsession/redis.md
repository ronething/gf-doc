# Redis存储

文件存储的方式在单节点的场景下非常不错，但是涉及到对应用进行多节点部署的场景下，各个节点的`Session`无法共享，因此需要将`Session`存储单独剥离出来管理，`Redis`服务器是比较常用的一个选择。

`gsession`的`Redis`存储使用`StorageRedis`对象实现，与文件存储比较类似，为了提高执行效率，也是采用了`内存+Redis`的方式。与文件存储唯一不同的是，在每一次请求中如果需要对`Session`进行操作时，将会从`Redis`中拉取一次最新的`Session`数据（而文件存储只会在`Session`不存在时读取一次文件）。


# 使用示例

https://github.com/gogf/gf/blob/master/.example/os/gsession/storage-redis/redis.go

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
		"SessionStorage": gsession.NewStorageRedis(g.Redis()),
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
1. 接着，我们停止程序，并重新启动，再次访问  http://127.0.0.1:8199/get  ，可以看到`Session`变量已经从`Redis`存储中恢复；如果我们手动修改`Redis`中的对应键值数据，页面刷新时也会读取到最新的值；
1. 等待1分钟后，再次访问  http://127.0.0.1:8199/get  可以看到已经无法获取该`Session`，因为该`Session`已经过期；


