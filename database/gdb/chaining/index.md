
[TOC]

`gdb`链式操作使用方式简单灵活，是`GF`框架官方推荐的数据库操作方式。

# 链式操作

链式操作可以通过数据库对象的`db.Table`/`db.From`方法或者事务对象的`tx.Table`/`tx.From`方法，基于指定的数据表返回一个链式操作对象`*Model`，该对象可以执行以下方法。

接口文档：
https://godoc.org/github.com/gogf/gf/database/gdb#Model

```go
func (md *Model) LeftJoin(joinTable string, on string) *Model
func (md *Model) RightJoin(joinTable string, on string) *Model
func (md *Model) InnerJoin(joinTable string, on string) *Model

func (md *Model) Fields(fields string) *Model
func (md *Model) Limit(start int, limit int) *Model
func (md *Model) Data(data...interface{}) *Model
func (md *Model) Batch(batch int) *Model
func (md *Model) Filter() *Model
func (md *Model) Safe(safe...bool) *Model

func (md *Model) Where(where interface{}, args...interface{}) *Model
func (md *Model) And(where interface{}, args ...interface{}) *Model
func (md *Model) Or(where interface{}, args ...interface{}) *Model

func (md *Model) GroupBy(groupby string) *Model
func (md *Model) OrderBy(orderby string) *Model

func (md *Model) Insert() (sql.Result, error)
func (md *Model) Replace() (sql.Result, error)
func (md *Model) Save() (sql.Result, error)
func (md *Model) Update() (sql.Result, error)
func (md *Model) Delete() (sql.Result, error)

func (md *Model) Select() (Result, error)
func (md *Model) All() (Result, error)
func (md *Model) One() (Record, error)
func (md *Model) Value() (Value, error)
func (md *Model) Count() (int, error)

func (md *Model) Struct(objPointer interface{}) error
func (md *Model) Structs(objPointerSlice interface{}) error
func (md *Model) Scan(objPointer interface{}) error

func (md *Model) Chunk(limit int, callback func(result Result, err error) bool)
func (md *Model) Option(option int) *Model
func (md *Model) ForPage(page, limit int) (*Model)
```

## `Insert/Replace/Save`
1. **Insert**
	使用```insert into```语句进行数据库写入，如果写入的数据中存在`Primary Key`或者`Unique Key`的情况，返回失败，否则写入一条新数据；
3. **Replace**
	使用```replace into```语句进行数据库写入，如果写入的数据中存在`Primary Key`或者`Unique Key`的情况，会删除原有的记录，必定会写入一条新记录；
5. **Save**
	使用```insert into```语句进行数据库写入，如果写入的数据中存在`Primary Key`或者`Unique Key`的情况，更新原有数据，否则写入一条新数据；

## `Data`方法

`Data`方法用于传递数据参数，用于数据写入/更新等写操作，支持的参数为`string/map/slice/struct/*struct`。例如，在进行`Insert`操作时，开发者可以传递任意的`map`类型，如: `map[string]string`/`map[string]interface{}`/`map[interface{}]interface{}`等等，也可以传递任意的`struct`对象或者其指针`*struct`。

## `Where`方法

`Where`（包括`And`/`Or`）方法用于传递查询条件参数，支持的参数为任意的`string/map/slice/struct/*struct`类型。

# 链式安全

该章节比较重要，可能对于新手来说有点绕，也是新手比较常问的问题之一，如果没有看特别明白的同学请多看几遍。其实"链式安全"只是模型操作的两种方式区别：一种会修改当前`model`对象（不安全，默认），一种不会（安全）但是模型属性修改/条件叠加需要使用赋值操作，仅此而已。

## 默认情况
在默认情况下，`gdb`是`非链式安全`的，也就是说链式操作的每一个方法都将对当前操作的`Model`属性进行修改，因此该`Model`对象**不可以重复使用**。例如，当存在多个分开查询的条件时，我们可以这么来使用`Model`对象：
```go
user := g.DB().Table("user")
user.Where("status IN(?)", g.Slice{1,2,3})
if vip {
    // 查询条件自动叠加，修改当前模型对象
    user.Where("money>=?", 1000000)
} else {
    // 查询条件自动叠加，修改当前模型对象
    user.Where("money<?",  1000000)
}
//  vip: SELECT * FROM user WHERE status IN(1,2,3) AND money >= 1000000
// !vip: SELECT * FROM user WHERE status IN(1,2,3) AND money < 1000000
r, err := user.Select()
//  vip: SELECT COUNT(1) FROM user WHERE status IN(1,2,3) AND money >= 1000000
// !vip: SELECT COUNT(1) FROM user WHERE status IN(1,2,3) AND money < 1000000
n, err := user.Count()
```
可以看到，如果是分开执行链式操作，链式的每一个操作都会修改已有的`Model`对象，查询条件会自动叠加，因此`user`对象不可重复使用，否则条件会不停叠加。并且在这种使用方式中，每次我们需要操作`user`用户表，都得使用`g.DB().Table("user")`这样的语法创建一个新的`user`模型对象，相对来说会比较繁琐。

> 默认情况下，基于性能以及GC优化考虑，模型对象为`非链式安全`，防止产生过多的临时模型对象。

## Clone方法

此外，我们也可以手动调动`Clone`方法克隆当前模型，创建一个新的模型来实现链式安全，由于是新的模型对象，因此并不担心会修改已有的模型对象的问题。例如：
```go
// 定义一个用户模型单例
user := g.DB().Table("user")
```

```go
// 克隆一个新的用户模型
m := user.Clone()
m.Where("status IN(?)", g.Slice{1,2,3})
if vip {
    m.And("money>=?", 1000000)
} else {
    m.And("money<?",  1000000)
}
//  vip: SELECT * FROM user WHERE status IN(1,2,3) AND money >= 1000000
// !vip: SELECT * FROM user WHERE status IN(1,2,3) AND money < 1000000
r, err := m.Select()
//  vip: SELECT COUNT(1) FROM user WHERE status IN(1,2,3) AND money >= 1000000
// !vip: SELECT COUNT(1) FROM user WHERE status IN(1,2,3) AND money < 1000000
n, err := m.Count()
```

## Safe方法

当然，我们可以通过`Safe`方法设置当前模型为`链式安全`的对象，后续的每一个链式操作都将返回一个新的`Model`对象，该`Model`对象可重复使用。但需要特别注意的是，模型属性的修改，或者操作条件的叠加，需要通过变量赋值的方式（`m = m.xxx`）覆盖原有的模型对象来实现。例如：
```go
// 定义一个用户模型单例
user := g.DB().Table("user").Safe()
```
```go
m := user.Where("status IN(?)", g.Slice{1,2,3})
if vip {
    // 查询条件通过赋值叠加
    m = m.And("money>=?", 1000000)
} else {
    // 查询条件通过赋值叠加
    m = m.And("money<?",  1000000)
}
//  vip: SELECT * FROM user WHERE status IN(1,2,3) AND money >= 1000000
// !vip: SELECT * FROM user WHERE status IN(1,2,3) AND money < 1000000
r, err := m.Select()
//  vip: SELECT COUNT(1) FROM user WHERE status IN(1,2,3) AND money >= 1000000
// !vip: SELECT COUNT(1) FROM user WHERE status IN(1,2,3) AND money < 1000000
n, err := m.Count()
```
可以看到，示例中的用户模型单例对象`user`可以重复使用，而不用担心被“污染”的问题。在这种链式安全的方式下，我们可以创建一个用户单例对象`user`，并且可以重复使用到后续的各种查询中。但是存在多个查询条件时，条件的叠加需要通过模型赋值操作（`m = m.xxx`）来实现。

> 使用`Safe`方法标记之后，每一个链式操作都将会创建一个新的临时模型对象（内部自动使用`Clone`实现模型克隆），从而实现链式安全。这种使用方式在模型操作中比较常见。





