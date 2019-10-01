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

https://github.com/gogf/gf/blob/master/container/glist/glist_z_bench_test.go

```
goos: darwin
goarch: amd64
pkg: github.com/gogf/gf/container/glist
Benchmark_PushBack-4             5000000               268 ns/op              56 B/op          2 allocs/op
Benchmark_PushFront-4           10000000               435 ns/op              56 B/op          2 allocs/op
Benchmark_Len-4                 30000000              44.5 ns/op               0 B/op          0 allocs/op
Benchmark_PopFront-4            20000000              71.1 ns/op               0 B/op          0 allocs/op
Benchmark_PopBack-4             30000000              70.1 ns/op               0 B/op          0 allocs/op
PASS
```
## 使用示例

### 示例1，基本使用
```go
package main

import (
	"fmt"
	"github.com/gogf/gf/container/glist"
)

func main() {
	l := glist.New()
	// Push
	l.PushBack(1)
	l.PushBack(2)
	e0 := l.PushFront(0)
	// Insert
	l.InsertBefore(e0, -1)
	l.InsertAfter(e0, "a")
	fmt.Println(l)
	// Pop
	fmt.Println(l.PopFront())
	fmt.Println(l.PopBack())
	fmt.Println(l)
}
```
执行后，输出结果：
```

```

### 示例2，JSON序列化/反序列
`glist`容器实现了标准库`json`数据格式的序列化/反序列化接口。
1. `Marshal`
    ```go
    package main

    import (
        "encoding/json"
        "fmt"
        "github.com/gogf/gf/container/glist"
        "github.com/gogf/gf/frame/g"
    )

    func main() {
        type Student struct {
            Id     int
            Name   string
            Scores *glist.List
        }
        s := Student{
            Id:     1,
            Name:   "john",
            Scores: glist.NewFrom(g.Slice{100, 99, 98}),
        }
        b, _ := json.Marshal(s)
        fmt.Println(string(b))
    }
    ```
    执行后，输出结果：
    ```
    {"Id":1,"Name":"john","Scores":[100,99,98]}
    ```
1. `Unmarshal`
    ```go
    package main

    import (
        "encoding/json"
        "fmt"
        "github.com/gogf/gf/container/glist"
    )

    func main() {
        b := []byte(`{"Id":1,"Name":"john","Scores":[100,99,98]}`)
        type Student struct {
            Id     int
            Name   string
            Scores *glist.List
        }
        s := Student{}
        json.Unmarshal(b, &s)
        fmt.Println(s)
    }
    ```
    执行后，输出结果：
    ```
    {1 john [100,99,98]}
    ```














