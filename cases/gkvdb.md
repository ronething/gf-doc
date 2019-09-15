# gkvdb高性能Key-Value嵌入式事务数据库

## 仓库地址
  * [https://github.com/johngcn/gkvdb](https://github.com/johngcn/gkvdb)

## 关于项目

### 1. 使用到的模块
```go
"github.com/gogf/gf/os/gfile"
"github.com/gogf/gf/encoding/ghash"
"github.com/gogf/gf/container/gmap"
"github.com/gogf/gf/container/gtype"
"github.com/gogf/gf/os/gmlock"
"github.com/gogf/gf/os/grpool"
"github.com/gogf/gf/container/glist"
"github.com/gogf/gf/os/gcache"
"gitee.com/johng/gkvdb/gkvdb/gfilespace"
"github.com/gogf/gf/os/gtime"
```

### 2. 你能从这个项目了解什么？
* 《Key-Value数据库是如何炼成的》
* 常用数据结构操作，二进制文件操作，并发安全控制；
* 缓存控制、内存锁、文件碎片管理、文件指针池、Goroutine池等等；
* 等等等等；
