
[TOC]

`gdb`链式操作使用方式简单灵活，是官方推荐的数据库操作方式。

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
```
可以看到，如果是分开执行链式操作，链式的每一个操作都会修改已有的`Model`对象，查询条件会自动叠加。这种使用方式中，每次我们需要操作`user`用户表，都得使用`g.DB().Table("user")`这样的语法创建一个新的用户模型对象，相对来说会比较繁琐。

> 默认情况下，基于性能以及GC优化考虑，模型对象为`非链式安全`，防止产生过多的临时模型对象。

## Safe方法

当然，我们可以通过`Safe`方法设置当前模型为`链式安全`的对象，后续的每一个链式操作都将返回一个新的`Model`对象，该`Model`对象可重复使用。
```go
// 定义一个用户模型单例
user := g.DB().Table("user").Safe()
```
```go
m := m.Where("status IN(?)", g.Slice{1,2,3})
if vip {
    // 查询条件通过赋值叠加
    m = m.And("money>=?", 1000000)
} else {
    // 查询条件通过赋值叠加
    m = m.And("money<?",  1000000)
}
r, err := m.Select()
```
可以看到，示例中得用户模型单例对象`user`可以重复使用，而不用担心被“污染”的问题。在这种链式安全的方式下，我们可以创建一个用户单例对象`user`，并且可以重复使用到后续的各种查询中。存在多个查询条件时，条件的叠加需要通过模型赋值操作（`m = m.xxx`）来实现。

> 使用`Safe`方法标记之后，每一个链式操作都将会创建一个新的临时模型对象，从而实现链式安全。这种使用方式在模型操作中比较常见。

## Clone方法

此外，我们也可以使用`Clone`方法克隆当前模型，创建一个新的模型来实现链式安全，由于是新的模型对象，因此并不担心会修改已有的模型对象的问题。多个查询条件的叠加无需模型赋值。例如：
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
r, err := m.Select()
```



# Struct参数传递

在`gdb`中，`Data`/`Where`/`And`/`Or`链式方法支持任意的`string/map/slice/struct/*struct`数据类型参数，该特性为`gdb`提供了很高的灵活性。当使用`struct`/`*struct`对象作为输入参数时，将会被自动解析为`map`类型，只有`struct`的**公开属性**能够被转换，并且支持 `gconv`/`json` 标签，用于定义转换后的键名，即与表字段的映射关系。

例如:
```go
type User struct {
    Uid      int    `gconv:"user_id"`
    Name     string `gconv:"user_name"`
    NickName string `gconv:"nick_name"`
}
// 或者
type User struct {
    Uid      int    `json:"user_id"`
    Name     string `json:"user_name"`
    NickName string `json:"nick_name"`
}
```
其中，`struct`的属性应该是公开属性（首字母大写），`gconv`标签对应的是数据表的字段名称。表字段的对应关系标签既可以使用`gconv`，也可以使用传统的`json`标签，但是当两种标签都存在时，`gconv`标签的优先级更高。为避免将`struct`对象转换为`JSON`数据格式返回时与`JSON`编码标签冲突，推荐使用`gconv`标签来实现映射关系。具体转换规则请查看【[gconv.Map转换](util/gconv/map.md)】章节。

# Struct结果输出

链式操作提供了3个方法对查询结果执行`struct`对象转换/输出。

1. `Struct`: 将查询结果转换为一个`struct`对象，查询结果应当是特定的一条记录，并且`pointer`参数应当为`struct`对象的指针地址（`*struct`或者`**struct`），使用方式例如：
    ```go
    type User struct {
        Id         int
        Passport   string
        Password   string
        NickName   string
        CreateTime gtime.Time
    }
    user := new(User)
    err  := db.Table("user").Where("id", 1).Struct(user)
    ```
    或者
    ```go
    user ：= &User{}
    err  := db.Table("user").Where("id", 1).Struct(user)
    ```
    前两种方式都是预先初始化对象（提前分配内存），推荐的方式：
    ```go
    user := (*User)(nil)
    err  := db.Table("user").Where("id", 1).Struct(&user)
    ```
    这种方式只有在查询到数据的时候才会执行初始化及内存分配。注意在用法上的区别，特别是传递参数类型的差别（前两种方式传递的参数类型是`*User`，这里传递的参数类型其实是`**User`）。
1. `Structs`: 将多条查询结果集转换为一个`[]struct/[]*struct`数组，查询结果应当是多条记录组成的结果集，并且`pointer`应当为数组的指针地址，使用方式例如：
    ```go
    users := ([]User)(nil)
    // 或者 var users []User
    err := db.Table("user").Structs(&users)
    ```
    或者
    ```go
    users := ([]*User)(nil)
    // 或者 var user []*User
    err := db.Table("user").Structs(&users)
    ```
1. `Scan`: 该方法会根据输入参数`pointer`的类型选择调用`Struct`还是`Structs`方法。如果结果是特定的一条记录，那么调用`Struct`方法；如果结果是`slice`类型则调用`Structs`方法。

