[TOC]

# Struct参数传递

在`gdb`中，`Data`/`Where`/`And`/`Or`链式方法支持任意的`string/map/slice/struct/*struct`数据类型参数，该特性为`gdb`提供了很高的灵活性。当使用`struct`/`*struct`对象作为输入参数时，将会被自动解析为`map`类型，只有`struct`的**公开属性**能够被转换，并且支持 `orm`/`gconv`/`json` 标签，用于定义转换后的键名，即与表字段的映射关系。

例如:
```go
type User struct {
    Uid      int    `orm:"user_id"`
    Name     string `orm:"user_name"`
    NickName string `orm:"nick_name"`
}
// 或者
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
其中，`struct`的属性应该是公开属性（首字母大写），`orm`标签对应的是数据表的字段名称。表字段的对应关系标签既可以使用`orm`，也可以用`gconv`，还可以使用传统的`json`标签，但是当三种标签都存在时，`orm`标签的优先级更高。为避免将`struct`对象转换为`JSON`数据格式返回时与`JSON`编码标签冲突，推荐使用`orm`标签来实现数据库`ORM`的映射关系。更详细的转换规则请查看【[gconv.Map转换](util/gconv/map.md)】章节。

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

