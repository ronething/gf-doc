[TOC]

# 对象转换

对象转换在请求处理中非常常见。我们推荐将输入和输出定义为`struct`结构体对象，以便于后续的输入输出参数的维护。

`GF`框架支持非常方便的对象转换，支持将客户端提交的参数如`Query`参数、表单参数、内容参数、`JSON/XML`等参数非常便捷地转换为指定的`struct`结构体，并且支持提交参数与`struct`属性的映射关系维护。

对象转换方法使用`Request`对象的`Get*Struct`方法，具体请参考API文档：
https://godoc.org/github.com/gogf/gf/net/ghttp

# 参数映射

## 默认规则

客户端提交的参数如果需要映射到服务端定义的`struct`上，可以采用默认的映射关系，这一点非常方便。默认的转换规则如下：
1. `struct`中需要匹配的属性必须为**`公开属性`**(首字母大写)。
2. 参数名称会自动按照 **`不区分大小写`** 且 **忽略`-/_/空格`符号** 的形式与`struct`属性进行匹配。 
3. 如果匹配成功，那么将键值赋值给属性，如果无法匹配，那么忽略该键值。

以下是几个匹配的示例：
```html
map键名    struct属性     是否匹配
name       Name           match
Email      Email          match
nickname   NickName       match
NICKNAME   NickName       match
Nick-Name  NickName       match
nick_name  NickName       match
nick name  NickName       match
NickName   Nick_Name      match
Nick-name  Nick_Name      match
nick_name  Nick_Name      match
nick name  Nick_Name      match
```

> 更详细的规则请参考【[gconv.Struct](util/gconv/struct.md)】章节。

## 自定义规则

自定义的参数映射规则可以通过为`struct`属性绑定`tag`实现，`tag`名称可以为`p/param/params`。例如：

```go
type User struct{
    Id    int
    Name  string
    Pass1 string `p:"password1"`
    Pass2 string `p:"password2"`
}
```
其中我们使用了`p`标签来指定该属性绑定的参数名称。`password1`参数将会映射到`Pass1`属性，`password2`将会映射到`Pass2`属性上。其他属性采用默认的转换规则即可，无序设置`tag`。

此外，`Get*Struct`转换方法也支持自定义的映射关系输入，通过`mapping`参数输入，该参数在对象转换时的映射优先级最高，只是大部分场景中都是采用`tag`形式，该参数往往不会使用到。

# 示例1，基本使用












