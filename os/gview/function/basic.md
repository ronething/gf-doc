[TOC]

# 基础函数

变量可以使用符号 `|` 在函数间传递

```go
{{.value | Func1 | Func2}}
```

使用括号

```go
{{printf "nums is %s %d" (printf "%d %d" 1 2) 3}}
```

## and

```go
{{and .X .Y .Z}}
```

`and`会逐一判断每个参数，将返回第一个为空的参数，否则就返回最后一个非空参数

## call

```go
{{call .Field.Func .Arg1 .Arg2}}
```

`call`可以调用函数，并传入参数

调用的函数需要返回 1 个值 或者 2 个值，返回两个值时，第二个值用于返回`error`类型的错误。返回的错误不等于`nil`时，执行将终止。

## index

`index`支持`map`, `slice`, `array`, `string`，读取指定类型对应下标的值。

```go
{{index .Maps "name"}}
```

## len

```go
{{printf "The content length is %d" (.Content|len)}}
```

返回对应类型的长度，支持类型：`map`, `slice`, `array`, `string`, `chan`。

## not

`not`返回输入参数的否定值。

例如，判断是否变量是否为空：
```go
{{if not .Var}}
// 执行为空操作(.Var为空, 如: nil, 0, "", 长度为0的slice/map)
{{else}}
// 执行非空操作(.Var不为空)
{{end}}
```

## or

```go
{{or .X .Y .Z}}
```

`or`会逐一判断每个参数，将返回第一个非空的参数，否则就返回最后一个参数。

## print

同`fmt.Sprint`。

## printf

同`fmt.Sprintf`。

## println

同`fmt.Sprintln`。

## urlquery

```go
{{urlquery "http://johng.cn"}}
```

将返回

```go
http%3A%2F%2Fjohng.cn
```

## eq / ne / lt / le / gt / ge

这类函数一般配合在`if`中使用

```go
`eq`: arg1 == arg2
`ne`: arg1 != arg2
`lt`: arg1 < arg2
`le`: arg1 <= arg2
`gt`: arg1 > arg2
`ge`: arg1 >= arg2
```

`eq`和其他函数不一样的地方是，支持多个参数。
```go
{{eq arg1 arg2 arg3 arg4}}
```
和下面的逻辑判断相同:
```go
arg1==arg2 || arg1==arg3 || arg1==arg4 ...
```

与`if`一起使用

```go
{{if eq true .Var1 .Var2 .Var3}}...{{end}}
```

```go
{{if lt 100 200}}...{{end}}
```

例如，判断变量不为空时执行：
```go
{{if .Var}}
// 执行非空操作(.Var不为空)
{{else}}
// 执行为空操作(.Var为空, 如: nil, 0, "", 长度为0的slice/map)
{{end}}
```

### 对比函数改进

`GF`框架模板引擎对标准库自带的对比模板函数`eq/ne/lt/le/gt/ge`做了必要的改进，以便支持任意数据类型的比较。例如，在标准库模板中的以下比较：
```go
{{eq 1 "1"}}
```
将会引发错误：
```html
panic: template: at <eq 1 "1">: error calling eq: incompatible types for comparison
```
由于比较的两个参数不是同一数据类型，因此引发了`panic`错误。

在`GF`框架的模板引擎中，将会自动将两个参数进行数据转换，转换为同一类型后再进行比较，这样的开发体验更好、灵活性更高。如果两个参数均为整型变量（或者整型字符串），那么将会转换为整型进行比较，否则转换为字符串变量进行比较（区分大小写）。

### 改进运行示例

我们来看一个`GF`框架的模板引擎中的对比模板函数运行示例。

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/frame/g"
)

func main() {
	tplContent := `
eq:
eq "a" "a": {{eq "a" "a"}}
eq "1" "1": {{eq "1" "1"}}
eq  1  "1": {{eq  1  "1"}}

ne:
ne  1  "1": {{ne  1  "1"}}
ne "a" "a": {{ne "a" "a"}}
ne "a" "b": {{ne "a" "b"}}

lt:
lt  1  "2": {{lt  1  "2"}}
lt  2   2 : {{lt  2   2 }}
lt "a" "b": {{lt "a" "b"}}

le:
le  1  "2": {{le  1  "2"}}
le  2   1 : {{le  2   1 }}
le "a" "a": {{le "a" "a"}}

gt:
gt  1  "2": {{gt  1  "2"}}
gt  2   1 : {{gt  2   1 }}
gt "a" "a": {{gt "a" "a"}}

ge:
ge  1  "2": {{ge  1  "2"}}
ge  2   1 : {{ge  2   1 }}
ge "a" "a": {{ge "a" "a"}}
`
	content, err := g.View().ParseContent(tplContent, nil)
	if err != nil {
		panic(err)
	}
	fmt.Println(content)
}
```
运行后，输出结果为：
```html
eq:
eq "a" "a": true
eq "1" "1": true
eq  1  "1": true

ne:
ne  1  "1": false
ne "a" "a": false
ne "a" "b": true

lt:
lt  1  "2": true
lt  2   2 : false
lt "a" "b": true

le:
le  1  "2": true
le  2   1 : false
le "a" "a": true

gt:
gt  1  "2": false
gt  2   1 : true
gt "a" "a": false

ge:
ge  1  "2": false
ge  2   1 : true
ge "a" "a": true
```



