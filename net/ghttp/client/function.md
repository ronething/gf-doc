
# 工具方法
`ghttp`模块也提供了独立的包函数来实现HTTP请求，用于短连接请求，方法如下：

```go
func Get(url string) (*ClientResponse, error)
func Put(url string, data ...interface{}) (*ClientResponse, error)
func Post(url string, data ...interface{}) (*ClientResponse, error)
func Delete(url string, data ...interface{}) (*ClientResponse, error)
func Connect(url string, data ...interface{}) (*ClientResponse, error)
func Head(url string, data ...interface{}) (*ClientResponse, error)
func Options(url string, data ...interface{}) (*ClientResponse, error)
func Patch(url string, data ...interface{}) (*ClientResponse, error)
func Trace(url string, data ...interface{}) (*ClientResponse, error)
func DoRequest(method, url string, data ...interface{}) (*ClientResponse, error)

func GetBytes(url string, data ...interface{}) []byte
func PutBytes(url string, data ...interface{}) []byte
func PostBytes(url string, data ...interface{}) []byte
func DeleteBytes(url string, data ...interface{}) []byte
func ConnectBytes(url string, data ...interface{}) []byte
func HeadBytes(url string, data ...interface{}) []byte
func OptionsBytes(url string, data ...interface{}) []byte
func PatchBytes(url string, data ...interface{}) []byte
func TraceBytes(url string, data ...interface{}) []byte
func DoRequestBytes(method string, url string, data ...interface{}) []byte

func GetContent(url string, data ...interface{}) string
func PutContent(url string, data ...interface{}) string
func PostContent(url string, data ...interface{}) string
func DeleteContent(url string, data ...interface{}) string
func ConnectContent(url string, data ...interface{}) string
func HeadContent(url string, data ...interface{}) string
func OptionsContent(url string, data ...interface{}) string
func PatchContent(url string, data ...interface{}) string
func TraceContent(url string, data ...interface{}) string
func DoRequestContent(method string, url string, data ...interface{}) string
```
函数说明与`Client`对象的方法说明一致，一般情况下直接使用`ghttp`对应的HTTP包方法来实现HTTP请求即可，非常简便。当需要自定义HTTP请求的一些细节(例如超时时间、Cookie、Header等)时，就得依靠自定义的`Client`对象来处理了。


