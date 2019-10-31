# 基本示例
我们来看几个HTTP客户端请求的例子：

1. 发送`GET`请求，并打印出返回值
    ```go
    if response, err := ghttp.Get("https://goframe.org"); err != nil {
        panic(err)
    } else {
        defer response.Close()
        fmt.Println(response.ReadAllString())
    }
    ```

1. 发送`GET`请求，下载远程文件
    ```go
    if response, err := ghttp.Get("https://goframe.org/cover.png"); err != nil {
        panic(err)
    } else {
        defer response.Close()
        gfile.PutBytes("/Users/john/Temp/cover.png", response.ReadAll())
    }
    ```
    下载文件操作，小文件下载非常简单。需要注意的是，如果远程文件比较大时，服务端会分批返回数据，因此会需要客户端发多个`GET`请求，每一次通过`Header`来请求分批的文件范围长度，感兴趣的同学可自行研究相关细节。

1. 发送`POST`请求，并打印出返回值
    ```go
    if response, err := ghttp.Post("http://127.0.0.1:8199/form", "name=john&age=18"); err != nil {
        panic(err)
    } else {
        defer response.Close()
        fmt.Println(response.ReadAllString())
    }
    ```
    传递多参数的时候用户可以使用`&`符号进行连接，注意参数值往往需要通过`gurl.Encode`编码一下。

1. 发送`POST`请求，参数为`map`类型，并打印出返回值
    ```go
    if response, err := ghttp.Post("http://127.0.0.1:8199/form", g.Map{
        "submit"   : "1",
        "callback" : "http://127.0.0.1/callback?url=http://baidu.com",
    })); err != nil {
        panic(err)
    } else {
        defer response.Close()
        fmt.Println(response.ReadAllString())
    }
    ```
    传递多参数的时候用户可以使用`&`符号进行连接，也可以直接使用`map`（其实之前也提到，任意数据类型都支持，包括`struct`）。

1. 发送`POST`请求，参数为`JSON`数据，并打印出返回值
    ```go
    if response, err := ghttp.Post("http://127.0.0.1:8199/api/user", `{"passport":"john","password":"123456","password-confirm":"123456"}`); err != nil {
        panic(err)
    } else {
        defer response.Close()
        fmt.Println(response.ReadAllString())
    }
    ```
    可以看到，通过`ghttp`客户端发送`JSON`数据请求非常方便，直接通过`Post`方法提交即可，客户端会自动将请求的`Content-Type`设置为`application/json`。

1. 发送`DELETE`请求，并打印出返回值
    ```go
    if response, err := ghttp.Delete("http://127.0.0.1:8199/user", "10000"); err != nil {
        panic(err)
    } else {
        defer response.Close()
        fmt.Println(response.ReadAllString())
    }
    ```

