
# XSS变量处理

默认情况下，模板引擎对所有的变量输出并没有使用HTML转码处理，也就是说，如果开发者处理不好，可能会存在XSS漏洞。

不用担心，`GF`框架当然已经充分考虑到这点，并且为开发者提供了比较灵活的配置参数来控制是否默认转义变量输出的`HTML`内容。该特性可以通过`AutoEncode`配置项，或者`SetAutoEncode`方法来开启/关闭。

> 需要注意的是，该特性并不会影响`include`模板内置函数。

使用示例：

1. 配置文件

    ```toml
    [viewer]
        delimiters  =  ["${", "}"]
        autoencode  =  true
    ```
1. 示例代码

    ```go
    package main

    import (
        "fmt"
        "github.com/gogf/gf/frame/g"
    )

    func main() {
        result, _ := g.View().ParseContent("姓名: ${.name}", g.Map{
            "name": "<script>alert('john');</script>",
        })
        fmt.Println(result)
    }
    ```
1. 执行输出

    ```
    姓名: &lt;script&gt;alert(&#39;john&#39;);&lt;/script&gt;
    ```






