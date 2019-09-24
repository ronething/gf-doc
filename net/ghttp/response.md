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
可以看到`ghttp.Response`对象继承了标准库的`http.ResponseWriter`对象，因此完全可以使用标准库的方法来进行输出控制。

当然，建议使用`ghttp.Response`封装的方法来控制输出，`ghttp.Response`的数据输出使用`Write*`相关方法实现，并且数据输出采用了Buffer机制，数据的处理效率比较高，任何时候可以通过`OutputBuffer`方法输出缓冲区数据到客户端，并清空缓冲区数据。

相关方法：godoc.org/github.com/gogf/gf/g/net/ghttp#Response

此外，需要提一下，Header的操作可以通过标准库的方法来实现，例如：
```go
r.Header().Set("Content-Type", "text/plain; charset=utf-8")
```
