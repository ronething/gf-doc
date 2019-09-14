
# 分组路由

`GF`框架支持分组路由的注册方式，可以给分组路由指定一个`prefix`前缀（可选），在该分组下的所有路由注册都将注册在该路由前缀下。分组路由注册方式也是推荐的路由注册方式。

**接口文档**：

```go
func (s *Server) Group(prefix string, groups ...func(g *RouterGroup)) *RouterGroup 
func (d *Domain) Group(prefix string, groups ...func(g *RouterGroup)) *RouterGroup 

func (g *RouterGroup) ALL(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) GET(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) PUT(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) POST(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) DELETE(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) PATCH(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) HEAD(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) CONNECT(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) OPTIONS(pattern string, object interface{}, params...interface{})
func (g *RouterGroup) TRACE(pattern string, object interface{}, params...interface{})

// REST路由注册
func (g *RouterGroup) REST(pattern string, object interface{})

// 分组路由批量注册
func (g *RouterGroup) Bind(items []GroupItem)
```

其中：
1. `Group`方法用户创建一个分组路由对象，并且支持在指定域名对象上创建；
1. `Bind`方法用于批量路由注册，每一个路由注册项为`Slice`类型的参数，且参数数量应该>=3，具体使用请见后续示例；
1. `REST`方法用户注册`RESTful`风格的路由，需给定一个执行对象或者控制器对象；
1. `ALL`方法用于注册所有的`HTTP Method`到指定的函数/对象/控制器上；
1. 其他以`HTTP Method`命名的方法用以绑定指定的`HTTP Method`路由；

