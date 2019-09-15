# 容器日志搜集工具套件

## 仓库地址
  * [https://github.com/johngcn/k8s-log](https://github.com/johngcn/k8s-log)

## 关于项目

### 1. 使用到的模块
```go
"github.com/gogf/gf/os/gcron"
"github.com/gogf/gf/os/genv"
"github.com/gogf/gf/os/gfile"
"github.com/gogf/gf/os/glog"
"github.com/gogf/gf/os/gproc"
"github.com/gogf/gf/os/gtime"
"github.com/gogf/gf/util/gconv"
"github.com/gogf/gf/container/gmap"
"github.com/gogf/gf/container/gset"
"github.com/gogf/gf/encoding/gjson"
"github.com/gogf/gf/os/gfsnotify"
"github.com/gogf/gf/os/gmlock"
"github.com/gogf/gf/text/gregex"
"github.com/gogf/gf/database/gkafka"
"github.com/gogf/gf/os/gcache"
"github.com/gogf/gf/container/garray"
"github.com/gogf/gf/util/gstr"
```

### 2. 你能从这个项目了解什么？
* `kafka`客户端的使用；
* `gcron`定时任务模块的使用；
* `gtime`时间模块的使用；
* `garray`排序数组作为内存排序缓冲区的使用；
* 以上各种模块的使用示例；
