[TOC]

# glist

并发安全双向列表。

**使用场景**：

并发安全场景下的链表操作，也可以关闭并发安全性当做普通的链表来使用。

**使用方式**：
```go
import "github.com/gogf/gf/container/glist"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/container/glist

## 性能测试

https://github.com/gogf/gf/blob/master/g/container/glist/glist_z_bench_test.go

```
goos: darwin
goarch: amd64
pkg: github.com/gogf/gf/g/container/glist
Benchmark_PushBack-4             5000000               268 ns/op              56 B/op          2 allocs/op
Benchmark_PushFront-4           10000000               435 ns/op              56 B/op          2 allocs/op
Benchmark_Len-4                 30000000              44.5 ns/op               0 B/op          0 allocs/op
Benchmark_PopFront-4            20000000              71.1 ns/op               0 B/op          0 allocs/op
Benchmark_PopBack-4             30000000              70.1 ns/op               0 B/op          0 allocs/op
PASS
```
