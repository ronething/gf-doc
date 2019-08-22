[TOC]

# 请求输出

数据输出对象定义如下：
```go
// 服务端请求返回对象。
// 注意该对象并没有实现http.ResponseWriter接口，而是依靠ghttp.ResponseWriter实现。
type Response struct {
    ResponseWriter
    length  int             // 请求返回的内容长度(byte)
    Server  *Server         // 所属Web Server
    Writer  *ResponseWriter // ResponseWriter的别名
    request *Request        // 关联的Request请求对象
}
```
可以看到```ghttp.Response```对象继承了标准库的```http.ResponseWriter```对象，因此完全可以使用标准库的方法来进行输出控制。

当然，建议使用```ghttp.Response```封装的方法来控制输出，```ghttp.Response```的数据输出使用```Write*```相关方法实现，并且数据输出采用了Buffer机制，数据的处理效率比较高，任何时候可以通过```OutputBuffer```方法输出缓冲区数据到客户端，并清空缓冲区数据。

相关方法（API详见： [godoc.org/github.com/gogf/gf/g/net/ghttp#Response](https://godoc.org/github.com/gogf/gf/net/ghttp#Response)）：
```go
func (r *Response) Buffer() []byte
func (r *Response) BufferLength() int
func (r *Response) ClearBuffer()
func (r *Response) ContentSize() int
func (r *Response) OutputBuffer()
func (r *Response) SetBuffer(buffer []byte)

func (r *Response) RedirectBack()
func (r *Response) RedirectTo(location string)
func (r *Response) ServeFile(path string)
func (r *Response) ServeFileDownload(path string, name ...string)
func (r *Response) SetAllowCrossDomainRequest(allowOrigin string, allowMethods string, maxAge ...int)
func (r *Response) CORS(options CORSOptions)
func (r *Response) CORSDefault()

func (r *Response) Write(content ...interface{})
func (r *Response) Writef(format string, params ...interface{})
func (r *Response) Writefln(format string, params ...interface{})
func (r *Response) Writeln(content ...interface{})

func (r *Response) WriteXml(content interface{}, rootTag ...string) error
func (r *Response) WriteJson(content interface{}) error
func (r *Response) WriteJsonP(content interface{}) error
func (r *Response) WriteStatus(status int, content ...string)

func (r *Response) WriteTpl(tpl string, params map[string]interface{}, funcmap ...map[string]interface{}) error
func (r *Response) WriteTplContent(content string, params map[string]interface{}, funcmap ...map[string]interface{}) error

func (r *Response) ParseTpl(tpl string, params map[string]interface{}, funcmap ...map[string]interface{}) ([]byte, error)
func (r *Response) ParseTplContent(content string, params map[string]interface{}, funcmap ...map[string]interface{}) ([]byte, error)
```
此外，需要提一下，Header的操作可以通过标准库的方法来实现，例如：
```go
r.Header().Set("Content-Type", "text/plain; charset=utf-8")
```

## 实现允许跨域请求

我们可以使用`CORS`或者`CORSDefault`来实现服务端允许跨域请求。

> `SetAllowCrossDomainRequest`也是跨域使用的方法，新版本中已废弃。

1. `CORS`方法允许开发者自定义跨域请求参数，参数类型为`CORSOptions`对象；

    `CORSOptions`定义：
    ```go
    // See https://www.w3.org/TR/cors/ .
    // 服务端允许跨域请求选项
    type CORSOptions struct {
        AllowOrigin      string // Access-Control-Allow-Origin
        AllowCredentials string // Access-Control-Allow-Credentials
        ExposeHeaders    string // Access-Control-Expose-Headers
        MaxAge           int    // Access-Control-Max-Age
        AllowMethods     string // Access-Control-Allow-Methods
        AllowHeaders     string // Access-Control-Allow-Headers
    }
    ```
1. `CORSDefault`方法使用默认的跨域选项配置来设置跨域，大部分情况下使用该方法即可；

    使用示例：
    ```go
    package main

    import (
        "github.com/gogf/gf/g"
        "github.com/gogf/gf/g/frame/gmvc"
        "github.com/gogf/gf/g/net/ghttp"
    )

    type Order struct {
        gmvc.Controller
    }

    func (o *Order) Get() {
        o.Response.Write("GET")
    }

    func main() {
        s := g.Server()
        s.BindHookHandlerByMap("/api.v2/*any", map[string]ghttp.HandlerFunc {
        "BeforeServe"  : func(r *ghttp.Request) {
            r.Response.CORSDefault()
        },
        })
        s.BindControllerRest("/api.v2/{.struct}", new(Order))
        s.SetPort(8199)
        s.Run()
    }
    ```
    随后随便找个支持`jQuery`的站点（例如：https://www.baidu.com ），使用`F12`快捷键打开`console`窗口，执行以下`Javascript`指令检测是否跨域请求成功：
    ```javascript
    $.get("http://localhost:8199/api.v2/order", function(result){
        console.log(result)
    });
    ```
    效果如下：
    ![](/images/ghttp.response.cors.png)