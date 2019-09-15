[TOC]

# Map转换

`gconv.Map`支持将任意的`map`或`struct`/`*struct`类型转换为常用的 `map[string]interface{}` 类型。当转换参数为`struct`/`*struct`类型时，支持自动识别`struct`的 `gconv`/`json` 标签，并且可以通过`Map`方法的第二个参数`tags`指定自定义的转换标签，以及标签解析的优先级。
如果转换失败，返回`nil`。

注意：默认情况下，当属性中两个标签 `gconv`和`json` 同时存在时，`gconv`标签的优先级更高。

> 属性标签：当转换`struct`/`*struct`类型时，如果属性带有 `gconv`/`json` 标签，也支持 `-`及`omitempty` 标签属性。当使用 `-` 标签属性时，表示该属性不执行转换；当使用 `omitempty` 标签属性时，表示当属性为空时（空指针`nil`, 数字`0`, 字符串`""`, 空数组`[]`等）不执行转换。具体请查看随后示例。

转换方法：
```go
func Map(i interface{}, tags...bool) map[string]interface{}
func MapDeep(i interface{}, tags...bool) map[string]interface{}
```
其中，`MapDeep`支持递归转换，即会递归转换属性中的`struct`/`*struct`对象。

### 示例1，基本示例
```go
package main

import (
    "fmt"
    "github.com/gogf/gf/util/gconv"
)

func main() {
    type User struct {
        Uid  int    `json:"uid"`
        Name string `json:"name"`
    }
    // 对象
    fmt.Println(gconv.Map(User{
        Uid      : 1,
        Name     : "john",
    }))
    // 对象指针
    fmt.Println(gconv.Map(&User{
        Uid      : 1,
        Name     : "john",
    }))

    // 任意map类型
    fmt.Println(gconv.Map(map[int]int{
        100 : 10000,
    }))
}
```
执行后，输出结果如下：
```html
map[uid:1 name:john]
map[uid:1 name:john]
map[100:10000]
```

### 示例2，属性标签

```go
package main

import (
    "fmt"
    "github.com/gogf/gf/util/gconv"
)

func main() {
    type User struct {
        Uid      int
        Name     string `gconv:"-"`
        NickName string `gconv:"nickname, omitempty"`
        Pass1    string `gconv:"password1"`
        Pass2    string `gconv:"password2"`
    }
    user := User{
        Uid      : 100,
        Name     : "john",
        Pass1    : "123",
        Pass2    : "456",
    }
    fmt.Println(gconv.Map(user))
}
```
示例中可以使用`gconv`标签，也使用`json`标签。
执行后，输出结果为：
```
map[Uid:100 password1:123 password2:456]
```
### 示例3，自定义标签

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/util/gconv"
)

func main() {
	type User struct {
		Id   int    `json:"uid"`
		Name string `my-tag:"nick-name" json:"name"`
	}
	user := &User{
		Id:   1,
		Name: "john",
	}
	g.Dump(gconv.Map(user, "my-tag"))
}
```
执行后，输出结果为：
```json
{
	"nick-name": "john",
	"uid": 1
}
```




### 示例4，递归转换

当参数为`map`/`struct`/`*struct`类型时，如果键值/属性为一个对象（或者对象指针）时，`Map`方法将会讲对象转换为结果的一个键值。我们可以使用`MapDeep`方法递归转换参数的子对象。

使用示例：

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/util/gconv"
)

func main() {
	type Ids struct {
		Id         int    `json:"id"`
		Uid        int    `json:"uid"`
	}
	type Base struct {
		Ids
		CreateTime string `json:"create_time"`
	}
	type User struct {
		Base
		Passport   string `json:"passport"`
		Password   string `json:"password"`
		Nickname   string `json:"nickname"`
	}
	user := new(User)
	user.Id         = 1
	user.Uid        = 100
	user.Nickname   = "John"
	user.Passport   = "johng"
	user.Password   = "123456"
	user.CreateTime = "2019"
	g.Dump(gconv.MapDeep(user))
}
```
执行后，终端输出结果为：
```html
{
	"create_time": "2019",
	"id": 1,
	"nickname": "John",
	"passport": "johng",
	"password": "123456",
	"uid": 100
}
```
