
# gjson

`gjson`模块实现了强大的`JSON`编码/解析，支持层级检索、运行时修改、常用数据格式转换等特点。

特点：
1. 支持数据层级检索；
2. 支持运行时数据修改；
3. 支持`JSON`、`XML`、`YAML/YML`、`TOML`、`Struct`数据格式相互转换；

**使用方式**：
```go
import "github.com/gogf/gf/encoding/gjson"
```

**接口文档**： 

https://godoc.org/github.com/gogf/gf/encoding/gjson


## 示例1，数据层级检索
```go
data :=
    `{
        "users" : {
            "count" : 100,
            "list"  : [
                {"name" : "小明",  "score" : 60},
                {"name" : "John", "score" : 99.5}
            ]
        }
    }`
if j, err := gjson.DecodeToJson([]byte(data)); err != nil {
    glog.Error(err)
} else {
    fmt.Println("John Score:", j.GetFloat32("users.list.1.score"))
}
```

可以看到，`gjson.Json`对象可以通过非常灵活的层级筛选功能(```j.GetFloat32("users.list.1.score")```)检索到对应的变量信息。这是`gjson`包的一大特色。
    
## 示例2，处理键名本身带有层级符号"`.`"的情况
```go
data :=
    `{
        "users" : {
            "count" : 100
        },
        "users.count" : 101
    }`
if j, err := gjson.DecodeToJson([]byte(data)); err != nil {
    glog.Error(err)
} else {
    j.SetViolenceCheck(true)
    fmt.Println("Users Count:", j.GetInt("users.count"))
}
```
运行之后打印出的结果为```101```。当键名存在"."号时，检索优先级：键名->层级，因此并不会引起歧义。
    
## 示例3，支持运行时的数据修改
```go
data :=
    `{
        "users" : {
            "count" : 100
        }
    }`
if j, err := gjson.DecodeToJson([]byte(data)); err != nil {
    glog.Error(err)
} else {
    j.Set("users.count",  2)
    j.Set("users.list",  []string{"John", "小明"})
    c, _ := j.ToJson()
    fmt.Println(string(c))
}
```
执行后输出结果为：
```{"users":{"count":1,"list":["John","小明"]}}```
当`JSON`数据通过`gjson`包读取后，可以通过`Set`方法改变内部变量的内容，当然也可以`新增/删除`内容，当需要删除内容时，设定的值为`nil`即可。`gjson`包的数据运行时修改特性非常强大，在该特性的支持下，各种数据结构的编码/解析显得异常的灵活方便。

    
## 示例4，JSON转换为其他数据格式
```go
data :=
    `{
        "users" : {
            "count" : 100,
            "list"  : ["John", "小明"]
        }
    }`
if j, err := gjson.DecodeToJson([]byte(data)); err != nil {
    glog.Error(err)
} else {
    c, _ := j.ToJson()
    fmt.Println("JSON:")
    fmt.Println(string(c))
    fmt.Println("======================")

    fmt.Println("XML:")
    c, _ = j.ToXmlIndent()
    fmt.Println(string(c))
    fmt.Println("======================")

    fmt.Println("YAML:")
    c, _ = j.ToYaml()
    fmt.Println(string(c))
    fmt.Println("======================")

    fmt.Println("TOML:")
    c, _ = j.ToToml()
    fmt.Println(string(c))
}
```
`gjson`支持将`JSON`转换为其他常见的数据格式，目前支持：`JSON`、`XML`、`YAML/YML`、`TOM`L、`Struct`数据格式之间的相互转换。
    
    
    
    
    