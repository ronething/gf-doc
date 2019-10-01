[TOC]

# garray

数组容器，提供普通数组，及排序数组，支持数据项唯一性矫正，支持可选的并发安全参数控制。

**使用场景**：

数组操作。

**使用方式**：
```go
import "github.com/gogf/gf/container/garray"
```

**接口文档**：

https://godoc.org/github.com/gogf/gf/container/garray


由于`garray`模块下的对象及方法较多，支持`int`/`string`/`interface{}`三种数据类型，这里便不一一列举。`garray`下包含了多种数据类型的`slice`，可以使用 `garray.New*Array`/`garray.NewSorted*Array` 方法来创建，其中`garray.New*Array`为普通不排序数组，`garray.NewSorted*Array`为排序数组(当创建`interface{}`类型的数组时，创建时可以指定自定义的排序函数)。

## 示例1，普通数组
```go
package main

import (
    "fmt"
    "github.com/gogf/gf/container/garray"
)


func main () {
    // 创建普通的int类型数组，并关闭默认的并发安全特性
    a := garray.NewIntArray(true)

    // 添加数据项
    for i := 0; i < 10; i++ {
        a.Append(i)
    }

    // 获取当前数组长度
    fmt.Println(a.Len())

    // 获取当前数据项列表
    fmt.Println(a.Slice())

    // 获取指定索引项
    fmt.Println(a.Get(6))

    // 在指定索引前插入数据项
    a.InsertAfter(9, 11)
    // 在指定索引后插入数据项
    a.InsertBefore(10, 10)
    fmt.Println(a.Slice())

    // 修改指定索引的数据项
    a.Set(0, 100)
    fmt.Println(a.Slice())

    // 搜索数据项，返回搜索到的索引位置
    fmt.Println(a.Search(5))

    // 删除指定索引的数据项
    a.Remove(0)
    fmt.Println(a.Slice())

    // 并发安全，写锁操作
    a.LockFunc(func(array []int) {
        // 将末尾项改为100
        array[len(array) - 1] = 100
    })

    // 并发安全，读锁操作
    a.RLockFunc(func(array []int) {
        fmt.Println(array[len(array) - 1])
    })

    // 清空数组
    fmt.Println(a.Slice())
    a.Clear()
    fmt.Println(a.Slice())
}
```
执行后，输出结果为：
```html
10
[0 1 2 3 4 5 6 7 8 9]
6
[0 1 2 3 4 5 6 7 8 9 10 11]
[100 1 2 3 4 5 6 7 8 9 10 11]
5
[1 2 3 4 5 6 7 8 9 10 11]
100
[1 2 3 4 5 6 7 8 9 10 100]
[]
```

## 示例2，排序数组

排序数组的方法与普通数组类似，但是带有自动排序功能及唯一性过滤功能。

```go
package main

import (
    "fmt"
    "github.com/gogf/gf/container/garray"
)


func main () {
    // 自定义排序数组，降序排序(SortedIntArray管理的数据是升序)
    a := garray.NewSortedArray(func(v1, v2 interface{}) int {
        if v1.(int) < v2.(int) {
            return 1
        }
        if v1.(int) > v2.(int) {
            return -1
        }
        return 0
    })

    // 添加数据
    a.Add(2)
    a.Add(3)
    a.Add(1)
    fmt.Println(a.Slice())

    // 添加重复数据
    a.Add(3)
    fmt.Println(a.Slice())

    // 检索数据，返回最后对比的索引位置，检索结果
    // 检索结果：0: 匹配; <0:参数小于对比值; >0:参数大于对比值
    fmt.Println(a.Search(1))

    // 设置不可重复
    a.SetUnique(true)
    fmt.Println(a.Slice())
    a.Add(1)
    fmt.Println(a.Slice())
}
```
执行后，输出结果：
```html
[3 2 1]
[3 3 2 1]
3 0
[3 2 1]
[3 2 1]
```


## 示例3，JSON序列化/反序列
`garray`模块下的所有容器类型均实现了标准库`json`数据格式的序列化/反序列化接口。
1. `Marshal`
    ```go
    package main

    import (
        "encoding/json"
        "fmt"
        "github.com/gogf/gf/container/garray"
    )

    func main() {
        type Student struct {
            Id     int
            Name   string
            Scores *garray.IntArray
        }
        s := Student{
            Id:     1,
            Name:   "john",
            Scores: garray.NewIntArrayFrom([]int{100, 99, 98}),
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
        "github.com/gogf/gf/container/garray"
    )

    func main() {
        b := []byte(`{"Id":1,"Name":"john","Scores":[100,99,98]}`)
        type Student struct {
            Id     int
            Name   string
            Scores *garray.IntArray
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
