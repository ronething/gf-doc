# ghttp.ClientResponse

`ClientResponse`为HTTP对应请求的返回结果对象，该对象继承于`http.Response`，可以使用`http.Response`的所有方法。在此基础之上增加了以下两个方法：
```go
func (r *ClientResponse) GetCookie(key string) string
func (r *ClientResponse) ReadAll() []byte
func (r *ClientResponse) ReadAllString() string
func (r *ClientResponse) Close()
```
这里也要提醒的是，**ClientResponse需要手动调用`Close`方法关闭**，也就是说，不管你使用不使用返回的`ClientResponse`对象，你都需要将该返回对象赋值给一个变量，并且手动调用其`Close`方法进行关闭（往往使用`defer r.Close()`）。需要手动关闭返回对象这一点，与标准库的HTTP客户端请求对象操作相同。
