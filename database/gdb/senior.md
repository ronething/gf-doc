
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
2018-08-31 13:54:32.913 [DEBU] SELECT * FROM user WHERE uid=?, [1], 2018-08-31 13:54:32.912, 2018-08-31 13:54:32.913, 1 ms, DB:Query
2018-08-31 13:54:32.915 [DEBU] SELECT * FROM user WHERE uid=?, [2], 2018-08-31 13:54:32.914, 2018-08-31 13:54:32.915, 1 ms, DB:Query
2018-08-31 13:54:32.915 [DEBU] SELECT * FROM user WHERE uid=?, [3], 2018-08-31 13:54:32.915, 2018-08-31 13:54:32.915, 0 ms, DB:Query
2018-08-31 13:54:32.915 [ERRO] SELECT * FROM user WHERE no_such_field=?, [just_test], 2018-08-31 13:54:32.915, 2018-08-31 13:54:32.915, 0 ms, DB:Query
Error: Error 1054: Unknown column 'no_such_field' in 'where clause'
1.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_base.go:120
2.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_base.go:174
3.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_model.go:378
4.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_model.go:301
5.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_model.go:306
6.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/database/gdb/gdb_model.go:311
7.	/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/database/gdb/mysql/gdb_debug.go:30
```

# 查询缓存

`gdb`支持对查询结果的缓存处理，并支持手动的缓存清理，一切都由业务层调用端自主决定。需要注意的是，查询缓存仅支持链式操作，且在事务操作下不可用。
相关方法：
```go
// 查询缓存/清除缓存操作，需要注意的是，事务查询不支持缓存。
// 当time < 0时表示清除缓存， time=0时表示不过期, time > 0时表示过期时间，time过期时间单位：秒；
// name表示自定义的缓存名称，便于业务层精准定位缓存项(如果业务层需要手动清理时，必须指定缓存名称)，
// 例如：查询缓存时设置名称，清理缓存时可以给定清理的缓存名称进行精准清理。
func (md *Model) Cache(time int, name ... string) *Model
```
使用示例：
```go
package main

import (
    "github.com/gogf/gf/database/gdb"
    "github.com/gogf/gf/util/gutil"
)

func main() {
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
    db, err := gdb.New()
    if err != nil {
        panic(err)
    }
    // 开启调试模式，以便于记录所有执行的SQL
    db.SetDebug(true)

    // 执行2次查询并将查询结果缓存3秒，并可执行缓存名称(可选)
    for i := 0; i < 2; i++ {
        r, _ := db.Table("user").Cache(3, "vip-user").Where("uid=?", 1).One()
        gutil.Dump(r.ToMap())
    }

    // 执行更新操作，并清理指定名称的查询缓存
    db.Table("user").Cache(-1, "vip-user").Data(gdb.Map{"name" : "smith"}).Where("uid=?", 1).Update()

    // 再次执行查询，启用查询缓存特性
    r, _ := db.Table("user").Cache(3, "vip-user").Where("uid=?", 1).One()
    gutil.Dump(r.ToMap())
}
```
执行后输出结果为（测试表数据结构仅供示例参考）：
```shell
2018-08-31 13:56:58.132 [DEBU] SELECT * FROM user WHERE uid=?, [1], 2018-08-31 13:56:58.131, 2018-08-31 13:56:58.132, 1 ms, DB:Query
{
	"datetime": "",
	"name": "smith",
	"uid": "1"
}

{
	"datetime": "",
	"name": "smith",
	"uid": "1"
}

2018-08-31 13:56:58.144 [DEBU] UPDATE `user` SET `name`=? WHERE uid=?, [smith 1], 2018-08-31 13:56:58.133, 2018-08-31 13:56:58.144, 11 ms, DB:Exec
2018-08-31 13:56:58.144 [DEBU] SELECT * FROM user WHERE uid=?, [1], 2018-08-31 13:56:58.144, 2018-08-31 13:56:58.144, 0 ms, DB:Query
{
	"datetime": "",
	"name": "smith",
	"uid": "1"
}
```

# 日志输出

> 章节文档待完善，看到了请提醒作者。

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
    Uid        int    `json:"uid"`
    CreateTime string `json:"create_time"`
}
type User struct {
    Base
    Passport   string `json:"passport"`
    Password   string `json:"password"`
    Nickname   string `json:"nickname"`
}
```
并且，无论多少层级的`struct`继承，`gdb`的参数传递和结果处理都支持。

