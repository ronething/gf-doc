# gchan

优雅的`Channel`操作。原生的`channel`操作在`channel`关闭后再向`channel`发送数据，或者多次关闭`channel`会引发`panic`，`gchan`模块使得`channel`的操作变得优雅。

**使用场景**：

所有使用`chan`的场景都可使用`gchan`替代。

**使用方式**：
```go
import "github.com/gogf/gf/container/gchan"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/container/gchan


`gchan`与原生`channel`的性能测试：
```html
john@johnstation:~/Workspace/Go/GOPATH/src/github.com/gogf/gf/container/gchan$ go test *.go -bench=".*"
goos: linux
goarch: amd64
BenchmarkGchanPushAndPop-8    	20000000	        71.9 ns/op
BenchmarkChannelPushAndPop-8   	50000000	        39.3 ns/op
PASS
ok  	command-line-arguments	3.663s
```
