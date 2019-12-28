
[TOC]


# 调试模式

为便于开发阶段调试，`gdb`支持调试模式，可以使用以下方式开启调试模式：
```go
// 是否开启调试服务
func (db DB) SetDebug(debug bool)
```
随后在ORM的操作过程中，所有的执行语句将会打印到终端进行展示。
同时，我们可以通过以下方法获得调试过程中执行的所有SQL语句：
```go
// 获取已经执行的SQL列表
func (db DB) GetQueriedSqls() []*Sql
```
使用示例：
```go
package main

import (
    "fmt"
    "github.com/gogf/gf/database/gdb"
)

var db gdb.DB

// 初始化配置及创建数据库
func init () {
    gdb.AddDefaultConfigNode(gdb.ConfigNode {
       Host    : "127.0.0.1",
       Port    : "3306",
       User    : "root",
       Pass    : "123456",
       Name    : "test",
       Type    : "mysql",
       Role    : "master",
       Charset : "utf8",
    })
    db, _ = gdb.New()
}

func main() {
    db.SetDebug(true)
    // 执行3条SQL查询
    for i := 1; i <= 3; i++ {
        db.Table("user").Where("uid=?", i).One()
    }
    // 构造一条错误查询
    db.Table("user").Where("no_such_field=?", "just_test").One()
}
```
执行后，输出结果如下：
```shell
2018-08-31 13:54:32.913 [DEBU] SELECT * FROM user WHERE uid=1 LIMIT 1
2018-08-31 13:54:32.915 [DEBU] SELECT * FROM user WHERE uid=2 LIMIT 1
2018-08-31 13:54:32.915 [DEBU] SELECT * FROM user WHERE uid=3 LIMIT 1
2018-08-31 13:54:32.915 [ERRO] SELECT * FROM user WHERE no_such_field='just_test' LIMIT 1
Error: Error 1054: Unknown column 'no_such_field' in 'where clause'
1.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_base.go:120
2.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_base.go:174
3.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_model.go:378
4.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_model.go:301
5.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_model.go:306
6.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_model.go:311
7.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/database/gdb/mysql/gdb_debug.go:30
```



# 日志输出

日志输出请查看`ORM`的使用配置章节。

# 类型识别

使用`gdb`查询数据时，返回的数据类型将会被自动识别映射到`Go变量类型`。例如: 当字段类型为`int(xx)`时，查询到的字段值类型将会被识别会`int`类型；当字段类型为`varchar(xxx)`/`char(xxx)`/`text`等类型时将会被自动识别为`string`类型。以下以`mysql`类型为例，介绍数据库类型与Go变量类型的自动识别映射关系:

|数据库类型 | Go变量类型
|---|---
|`*char`   | `string`
|`*text`   | `string`
|`*binary` | `bytes`
|`*blob`   | `bytes`
|`*int`    | `int`
|`bit`     | `int`
|`big_int` | `int64`
|`float`   | `float64`
|`double`  | `float64`
|`decimal` | `float64`
|`bool`    | `bool`
|`其他`     | `string`

这一特性对于需要将查询结果进行编码，并通过例如`JSON`方式直接返回给客户端来说将会非常友好。

# 类型转换

`gdb`的数据记录结果（```Value```）支持非常灵活的类型转换，并内置支持常用的数十种数据类型的转换。```Result```/```Record```的类型转换请查看后续【[ORM高级特性](database/gdb/senior.md)】章节。

> `Value`类型是`*gvar.Var`类型的别名，因此可以使用`gvar.Var`数据类型的所有转换方法，具体请查看【[通用动态变量](container/gvar/index.md)】章节

使用示例：

首先，数据表定义如下：
```sql
# 商品表
CREATE TABLE `goods` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(300) NOT NULL COMMENT '商品名称',
  `price` decimal(10,2) NOT NULL COMMENT '商品价格',
  ...
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```
其次，数据表中的数据如下：
```html
id   title     price
1    IPhoneX   5999.99
```
最后，完整的示例程序如下：
```go
package main

import (
    "fmt"
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/os/glog"
)

func main() {
	g.Config().SetPath("/home/john/Workspace/github.com/gogf/gf/geg/frame")
    db := g.Database()
    if r, err := db.Table("goods").Where("id=?", 1).One(); err == nil {
        fmt.Printf("goods    id: %d\n",   r["id"].Int())
        fmt.Printf("goods title: %s\n",   r["title"].String())
        fmt.Printf("goods proce: %.2f\n", r["price"].Float32())
    } else {
        glog.Error(err)
    }
}
```
执行后，输出结果为：
```shell
goods    id: 1
goods title: IPhoneX
goods proce: 5999.99
```

# 继承支持

`gdb`模块针对于`struct`继承提供了良好的支持，包括参数传递、结果处理。例如：
```go
type Base struct {
    Uid        int    `orm:"uid"`
    CreateTime string `orm:"create_time"`
}
type User struct {
    Base
    Passport   string `orm:"passport"`
    Password   string `orm:"password"`
    Nickname   string `orm:"nickname"`
}
```
并且，无论多少层级的`struct`继承，`gdb`的参数传递和结果处理都支持。

