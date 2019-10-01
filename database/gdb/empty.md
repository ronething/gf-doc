[TOC]

# 空值处理

空值主要影响于写入/更新操作，如`Insert`, `Replace`, `Update`, `Save`操作。

当 `map`/`struct` 中存在空值如 `nil`,`""`,`0` 时，默认情况下，`gdb`将会将其当做正常的输入参数，因此这些参数也会被更新到数据表。如以下操作（以`map`为例，`struct`同理）：
```go
// UPDATE `user` SET `name`='john',update_time=null WHERE `id`=1
g.DB("user").Data(g.Map{
    "name"        : "john",
    "update_time" : nil,
}).Where("id", 1).Update()
```

## Option方法
我们可以通过`Option`方法来过滤掉这些空值，并给定`gdb.OPTION_OMITEMPTY`，例如，以上示例可以修改为：
```go
// UPDATE `user` SET `name`='john' WHERE `id`=1
g.DB("user").Option(gdb.OPTION_OMITEMPTY).Data(g.Map{
    "name"        : "john",
    "update_time" : nil,
}).Where("id", 1).Update()
```

对于`struct`的空值过滤操作示例：
```go
type User struct {
    Id         int    `orm:"id"`
    Passport   string `orm:"passport"`
    Password   string `orm:"password"`
    NickName   string `orm:"nickname"`
    CreateTime string `orm:"create_time"`
    UpdateTime string `orm:"update_time"`
}
user := User{
    Id        : 1,
    NickName  : "john",
    UpdateTime: gtime.Now().String(),
}
g.DB("user").Option(gdb.OPTION_OMITEMPTY).Data(user).Save()
// INSERT INTO `user`(`id`,`nickname`,`update_time`) VALUES(1,'john','2019-10-01 12:00:00') ON DUPLICATE KEY UPDATE `id`=VALUES(`id`) `nickname`=VALUES(`nickname`) `update_time`=VALUES(`update_time`)
```

> 注意，批量写入/更新操作中`Option`方法将会失效，因为在批量操作中，必须保证每个写入记录的字段是统一的。

## 更新特定字段
此外，我们也可以更新特定的字段，通过`Fields`方法指定。例如：
```go
// INSERT INTO `user`(`id`,`passport`) VALUES(1,'john')
db.Table("user").Fields("id, passport").Data(g.Map{
    "id":       1,
    "passport": "john",
    "password": "123456",
    "nickname": "John",
}).Insert()
```

对于`struct`的空值过滤及特定字段更新操作示例：
```go
type User struct {
    Id         int    `orm:"id"`
    Passport   string `orm:"passport"`
    Password   string `orm:"password"`
    NickName   string `orm:"nickname"`
    CreateTime string `orm:"create_time"`
    UpdateTime string `orm:"update_time"`
}
user := User{
    Id        : 1,
    NickName  : "john",
    UpdateTime: gtime.Now().String(),
}
g.DB("user").Option(gdb.OPTION_OMITEMPTY).Fields("id,nickname").Data(user).Save()
// INSERT INTO `user`(`id`,`nickname`) VALUES(1,'john') ON DUPLICATE KEY UPDATE `id`=VALUES(`id`) `nickname`=VALUES(`nickname`)
```
