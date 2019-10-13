[TOC]
# 使用示例

## 基本示例
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


## 文件上传

`gf`的HTTP客户端封装并极大简化了文件上传功能，直接上例子：

1. 客户端

    https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/upload/client.go

    ```go
    package main

    import (
        "fmt"
        "github.com/gogf/gf/os/glog"
        "github.com/gogf/gf/net/ghttp"
    )

    func main() {
        path := "/home/john/Workspace/Go/github.com/gogf/gf/version.go"
        r, e := ghttp.Post("http://127.0.0.1:8199/upload", "name=john&age=18&upload-file=@file:" + path)
        if e != nil {
            glog.Error(e)
        } else {
            fmt.Println(r.ReadAllString())
            r.Close()
        }
    }
    ```

    注意到了吗？文件上传参数格式使用了 `参数名=@file:文件路径` ，HTTP客户端将会自动解析**文件路径**对应的文件内容并读取提交给服务端。原本复杂的文件上传操作被`gf`进行了封装处理，用户只需要使用 `@file:+文件路径` 来构成参数值即可。
    > 其中，`文件路径`请使用本地文件绝对路径。
1. 服务端

    https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/upload/server.go

    ```go
    package main

    import (
        "github.com/gogf/gf/frame/g"
        "github.com/gogf/gf/os/gfile"
        "github.com/gogf/gf/net/ghttp"
    )

    // 执行文件上传处理，上传到系统临时目录 /tmp
    func Upload(r *ghttp.Request) {
        if f, h, e := r.FormFile("upload-file"); e == nil {
            defer f.Close()
            name   := gfile.Basename(h.Filename)
            buffer := make([]byte, h.Size)
            f.Read(buffer)
            gfile.PutBytes("/tmp/" + name, buffer)
            r.Response.Write(name + " uploaded successly")
        } else {
            r.Response.Write(e.Error())
        }
    }

    // 展示文件上传页面
    func UploadShow(r *ghttp.Request) {
        r.Response.Write(`
        <html>
        <head>
            <title>上传文件</title>
        </head>
            <body>
                <form enctype="multipart/form-data" action="/upload" method="post">
                    <input type="file" name="upload-file" />
                    <input type="submit" value="upload" />
                </form>
            </body>
        </html>
        `)
    }

    func main() {
        s := g.Server()
        s.BindHandler("/upload",      Upload)
        s.BindHandler("/upload/show", UploadShow)
        s.SetPort(8199)
        s.Run()
    }
    ```
    访问  http://127.0.0.1:8199/upload/show  选择需要上传的文件，提交之后可以看到文件上传成功到服务器上。

    文件上传比较简单，**但是需要注意的是，服务端在上传处理中需要使用 `f.Close()` 关闭掉临时上传文件打开的指针**。

## 自定义Header

http客户端发起请求时可以自定义发送给服务端的Header内容，该特性使用`SetHeader`/`SetHeaderRaw`方法实现。我们来看一个客户端自定义Cookie的例子。

1. 客户端
    
    https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/cookie/client.go
    ```go
    package main

    import (
        "fmt"
        "github.com/gogf/gf/net/ghttp"
    )

    func main() {
        c := ghttp.NewClient()
        c.SetHeader("Cookie", "name=john; score=100")
        if r, e := c.Get("http://127.0.0.1:8199/"); e != nil {
            panic(e)
        } else {
            fmt.Println(r.ReadAllString())
        }
    }
    ```
    通过`ghttp.NewClient()`创建一个自定义的http请求客户端对象，并通过`c.SetHeader("Cookie", "name=john; score=100")`设置自定义的Cookie，这里我们设置了两个示例用的Cookie参数，一个`name`，一个`score`，注意多个Cookie参数使用`;`符号分隔。
1. 服务端
    
    https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/cookie/server.go
    ```go
    package main

    import (
        "github.com/gogf/gf/frame/g"
        "github.com/gogf/gf/net/ghttp"
    )

    func main() {
        s := g.Server()
        s.BindHandler("/", func(r *ghttp.Request){
            r.Response.Writeln(r.Cookie.Map())
        })
        s.SetPort(8199)
        s.Run()
    }
    ```
	服务端的逻辑很简单，直接将接收到的Cookie参数全部打印出来。

1. 执行结果

	客户端代码执行后，终端将会打印出服务端的返回结果，如下：
    ```shell
    map[name:john score:100]
    ```
    可以看到，服务端已经接收到了客户端自定义的Cookie参数。

1. 使用`SetHeaderRaw`自定义Header

    这个方法十分强大，给个例子：
    ```go
    c := ghttp.NewClient()
    c.SetHeaderRaw(`
    accept-encoding: gzip, deflate, br
    accept-language: zh-CN,zh;q=0.9,en;q=0.8
    referer: https://idonottell.you
    cookie: name=john
    user-agent: my test http client
    `)
    if r, e := c.Get("http://127.0.0.1:8199/"); e != nil {
        panic(e)
    } else {
        fmt.Println(r.ReadAllString())
    }
    ```
    Do you get it?

