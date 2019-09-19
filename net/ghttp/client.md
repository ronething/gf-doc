[TOC]


# HTTP客户端

## ghttp.Client

`gf`框架提供了强大易用的HTTP客户端，同样由`ghttp`模块实现。

方法列表：
https://godoc.org/github.com/gogf/gf/net/ghttp#Client
```go
type Client
    func NewClient() *Client
    func (c *Client) Get(url string) (*ClientResponse, error)
    func (c *Client) Put(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Post(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Delete(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Connect(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Head(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Options(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Patch(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) Trace(url string, data ...interface{}) (*ClientResponse, error)
    func (c *Client) DoRequest(method, url string, data ...interface{}) (*ClientResponse, error)

    func (c *Client) GetContent(url string, data ...interface{}) string
    func (c *Client) PutContent(url string, data ...interface{}) string
    func (c *Client) PostContent(url string, data ...interface{}) string
    func (c *Client) DeleteContent(url string, data ...interface{}) string
    func (c *Client) ConnectContent(url string, data ...interface{}) string
    func (c *Client) HeadContent(url string, data ...interface{}) string
    func (c *Client) OptionsContent(url string, data ...interface{}) string
    func (c *Client) PatchContent(url string, data ...interface{}) string
    func (c *Client) TraceContent(url string, data ...interface{}) string
    func (c *Client) DoRequestContent(method string, url string, data ...interface{}) string

    func (c *Client) SetBasicAuth(user, pass string)
    func (c *Client) SetBrowserMode(enabled bool)
    func (c *Client) SetCookie(key, value string)
    func (c *Client) SetCookieMap(cookieMap map[string]string)
    func (c *Client) SetHeader(key, value string)
    func (c *Client) SetHeaderRaw(header string)
    func (c *Client) SetPrefix(prefix string)
    func (c *Client) SetTimeOut(t time.Duration)
```
简要说明：
1. 我们可以使用`NewClient`创建一个自定义的HTTP客户端对象`Client`，随后可以使用该对象执行请求。
1. 客户端提供了一系列以`HTTP Method`命名的方法，调用这些方法将会发起对应的`HTTP Method`请求。常用的方法当然是`Get`和`Post`方法，同时`DoRequest`是核心的请求方法，用户可以调用该方法实现自定义的`HTTP Method`发送请求。
1. 请求返回结果为`*ClientResponse`对象，可以通过该结果对象获取对应的返回结果，通过`ReadAll`方法可以获得返回的内容。以`*Content`命名的请求方法结果为字符串内容，如果请求失败时，得到的是空字符串。
1. 可以看到，客户端的请求参数的数据参数`data`数据类型为`interface{}`类型，也就是说可以传递任意的数据类型，常见的参数数据类型为`string`/`map`，如果参数为`map`类型，参数值将会进行`urlencode`编码。

### 工具方法
`ghttp`模块也提供了独立的包函数来实现HTTP请求，函数列表如下：

```go
func Get(url string) (*ClientResponse, error)
func Put(url string, data ...interface{}) (*ClientResponse, error)
func Post(url string, data ...interface{}) (*ClientResponse, error)
func Delete(url string, data ...interface{}) (*ClientResponse, error)
func Head(url string, data ...interface{}) (*ClientResponse, error)
func Patch(url string, data ...interface{}) (*ClientResponse, error)
func Connect(url string, data ...interface{}) (*ClientResponse, error)
func Options(url string, data ...interface{}) (*ClientResponse, error)
func Trace(url string, data ...interface{}) (*ClientResponse, error)
func DoRequest(method, url string, data ...interface{}) (*ClientResponse, error)

func GetContent(url string, data ...interface{}) string
func PutContent(url string, data ...interface{}) string
func PostContent(url string, data ...interface{}) string
func DeleteContent(url string, data ...interface{}) string
func HeadContent(url string, data ...interface{}) string
func PatchContent(url string, data ...interface{}) string
func ConnectContent(url string, data ...interface{}) string 
func OptionsContent(url string, data ...interface{}) string
func TraceContent(url string, data ...interface{}) string 
func RequestContent(method string, url string, data ...interface{}) string 
```
函数说明与`Client`对象的方法说明一致，一般情况下直接使用`ghttp`对应的HTTP包方法来实现HTTP请求即可，非常简便。当需要自定义HTTP请求的一些细节(例如超时时间、Cookie、Header等)时，就得依靠自定义的`Client`对象来处理了。

### 一些重要说明

1. `ghttp`客户端默认关闭了`KeepAlive`功能以及对服务端`TLS`证书的校验功能，如果需要启用可自定义客户端的`Transport`属性；
1. `Post`/`PostContent`方法提交的请求类型(`Content-Type`)默认为`application/x-www-form-urlencoded`，当`data`参数为`JSON`类型时，将会被自动识别此时请求的类型为`application/json`；

### 数组及复杂类型参数

数组参数可以通过例如`array=1&array=2&array=3`这样的方式传递，但是推荐复杂数据类型使用`JSON`数据格式传递。

由于`gf`框架的`WebServer`基于`net/http`标准库，因此遵循标准库的参数设计方式，`array=1&array=2&array=3`这样的提交参数将会被解析为数组变量`array`，不支持`array[]=1&array[]=2&array[]=3`这样的数组提交方式，也不支持`map[a]=1&map[b]=2&map[c]=3`这样的`map`提交方式。

## ghttp.ClientResponse

`ClientResponse`为HTTP对应请求的返回结果对象，该对象继承于`http.Response`，可以使用`http.Response`的所有方法。在此基础之上增加了以下两个方法：
```go
func (r *ClientResponse) GetCookie(key string) string
func (r *ClientResponse) ReadAll() []byte
func (r *ClientResponse) ReadAllString() string
func (r *ClientResponse) Close()
```
这里也要提醒的是，**ClientResponse需要手动调用`Close`方法关闭**，也就是说，不管你使用不使用返回的`ClientResponse`对象，你都需要将该返回对象赋值给一个变量，并且手动调用其`Close`方法进行关闭（往往使用`defer r.Close()`）。需要手动关闭返回对象这一点，与标准库的HTTP客户端请求对象操作相同。



## 基本示例

我们来看几个HTTP客户端请求的例子：

1. 发送GET请求，并打印出返回值
    ```go
    if r, e := ghttp.Get("https://goframe.org"); e != nil {
        panic(e)
    } else {
        defer r.Close()
        fmt.Println(r.ReadAllString())
    }
    ```

1. 发送GET请求，下载远程文件
    ```go
    if r, e := ghttp.Get("https://goframe.org/cover.png"); e != nil {
        panic(e)
    } else {
        defer r.Close()
        gfile.PutBinContents("/Users/john/Temp/cover.png", r.ReadAll())
    }
    ```
    下载文件操作，小文件下载非常简单。需要注意的是，如果远程文件比较大时，服务端会分批返回数据，因此会需要客户端发多个GET请求，每一次通过Header来请求分批的文件范围长度，感兴趣的同学可自行研究相关细节。

1. 发送POST请求，并打印出返回值
    ```go
    if r, e := ghttp.Post("http://127.0.0.1:8199/form", "name=john&age=18"); e != nil {
        panic(e)
    } else {
        defer r.Close()
        fmt.Println(r.ReadAllString())
    }
    ```
    传递多参数的时候用户可以使用`&`符号进行连接。

1. 发送POST请求，参数为`map`类型，并打印出返回值
    ```go
    if r, e := ghttp.Post("http://127.0.0.1:8199/form", g.Map{
        "submit"   : "1",
        "callback" : "http://127.0.0.1/callback?url=http://baidu.com",
    })); e != nil {
        panic(e)
    } else {
        defer r.Close()
        fmt.Println(r.ReadAllString())
    }
    ```
    传递多参数的时候用户可以使用`&`符号进行连接，也可以直接使用`map`（其实之前也提到，任意数据类型都支持，包括`struct`）。

1. 发送POST请求，参数为`JSON`数据，并打印出返回值
    ```go
    if r, e := ghttp.Post("http://127.0.0.1:8199/api/user", `{"passport":"john","password":"123456","password-confirm":"123456"}`); e != nil {
        panic(e)
    } else {
        defer r.Close()
        fmt.Println(r.ReadAllString())
    }
    ```
    可以看到，通过`ghttp`客户端发送`JSON`数据请求非常方便，直接通过`Post`方法提交即可，客户端会自动将请求的`Content-Type`设置为`application/json`。

1. 发送DELETE请求，并打印出返回值
    ```go
    if r, e := ghttp.Delete("http://127.0.0.1:8199/user", "10000"); e != nil {
        panic(e)
    } else {
        defer r.Close()
        fmt.Println(r.ReadAllString())
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
            gfile.PutBinContents("/tmp/" + name, buffer)
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

