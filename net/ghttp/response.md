[TOC]

# 请求输出

数据输出对象定义如下：
```go
type Response struct {
	*ResponseWriter                 // Underlying ResponseWriter.
	Server          *Server         // Parent server.
	Writer          *ResponseWriter // Alias of ResponseWriter.
	Request         *Request        // According request.
}
```
可以看到`ghttp.Response`对象实现了标准库的`http.ResponseWriter`接口。数据输出使用`Write*`相关方法实现，并且数据输出采用了`Buffer`机制，因此数据的处理效率比较高。任何时候可以通过`OutputBuffer`方法输出缓冲区数据到客户端，并清空缓冲区数据。

接口文档：https://godoc.org/github.com/gogf/gf/net/ghttp#Response

简要说明:
1. `Write*`方法用于数据的返回，可为任意的数据格式，内部通过断言对返回参数做自动分析；
1. `WriteJson*`/`WriteXml`方法用于特定数据格式的返回，这是为开发者提供的简便方法；
1. `WriteTpl*`方法用于模板内容返回，可以解析并返回模板文件，也可以直接解析并返回模板内容；
1. 其他方法详见接口文档；

此外，需要提一下，Header的操作可以通过标准库的方法来实现，例如：
```go
Response.Header().Set("Content-Type", "text/plain; charset=utf-8")
```
