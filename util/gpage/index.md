
[TOC]

分页管理由```gpage```模块实现，gpage提供了强大的动态分页及静态分页功能，并且为开发者自定义分页样式提供了极高的灵活度。

**使用方式**：
```go
import "github.com/gogf/gf/util/gpage"
```

**接口文档**：[godoc.org/github.com/gogf/gf/util/gpage](https://godoc.org/github.com/gogf/gf/util/gpage)
```go
type Page
    func New(TotalSize, perPage int, CurrentPage interface{}, url string, router ...*ghttp.Router) *Page
    func (page *Page) EnableAjax(actionName string)
    func (page *Page) FirstPage(styles ...string) string
    func (page *Page) GetContent(mode int) string
    func (page *Page) GetLink(url, text, title, style string) string
    func (page *Page) GetUrl(pageNo int) string
    func (page *Page) LastPage(styles ...string) string
    func (page *Page) NextPage(styles ...string) string
    func (page *Page) PageBar(styles ...string) string
    func (page *Page) PrevPage(styles ...string) string
    func (page *Page) SelectBar() string
    func (page *Page) SetUrlTemplate(template string)
```



我们这里需要着重说明的是三个方法，```New```，```GetContent```，```EnableAjax```。

# 创建分页对象

在```New```方法中，前面三个参数很简单，见名知意。

第四个参数```url string```是当前请求页面的URL，可以是完整的URL地址，如：```http://xxx.xxx.xxx/list?type=10#anchor```；也可以是URI绝对路径地址，如：```/list?type=10#anchor```；该参数是用于分页管理器计算分页URL地址的基础。

第五个参数```route...string```是一个可选参数，表示当前请求页面的路由匹配规则（如：```/user/list/:page```，或者```/order/list/*order-page```），当使用静态分页时，该参数为必传项，以便分页管理器能够智能替换静态URI中对应的分页参数。

具体使用示例请查看后续章节。


# 预定义分页样式

方法```GetContent```提供了预定义的常见的分页样式，以便于开发者快速使用。当预定义的样式无法满足开发者需求时，开发者可以使用公开的方法来自定义分页样式（或者进行方法重载来实现自定义），也可以使用正则替换指定预定义的分页样式中的部分内容来实现自定义。

具体使用示例请查看后续章节。

# 使用Ajax分页功能

方法```EnableAjax```给定一个Ajax方法名，用于实现Ajax分页，但是需要注意的是，该Ajax方法名称需要前后端约定统一，并且该Ajax方法只有一个URL参数。以下是一个Ajax方法的客户端定义示例：
```javascript
function DoAjax(url) {
	// 这里读取URL的内容并根据业务逻辑进行内容展示
}
```

具体使用示例请查看后续章节。