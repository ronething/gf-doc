
[TOC]

`Web Server`提供服务需要回调函数/方法/对象/控制器的支持，`ghttp`包支持多种路由注册模式，为开发者提供非常强大和灵活的接口功能。路由注册是整个Web Server最核心的部分，也是`gf`框架中最精心设计的一个模块。

<!-- 
> 以下章节中部分示例按照MVC模式进行目录管理（控制器需要分别通过独立的包```init```方法进行自动注册），所有示例代码存放于：[github.com/gogf/gf/blob/master/.example/frame/mvc/](https://github.com/gogf/gf/blob/master/.example/frame/mvc) 目录中，每个示例无法独立运行（只是独立注册服务，没有```main```模块），需要访问示例结果的话，需要执行外层的```main.go```入口程序。（少部分示例位于 [github.com/gogf/gf/blob/master/.example/net/ghttp/server/](https://github.com/gogf/gf/blob/master/.example/net/ghttp/server/)  目录中，可独立运行）
-->

接口文档：
https://godoc.org/github.com/gogf/gf/net/ghttp

# g与ghttp包

在随后的章节示例代码中，我们将会看到频繁的```g.Server()```及```ghttp.GetServer()```混用，其实它们获取的都是同一个`Web Server`单例对象指针。其中```ghttp.GetServer()```是`ghttp`包**原生单例**`Web Server`对象指针获取方法。而```g.Server()```是框架通用**对象管理器**提供的方法，框架```g.*```对象管理器封装了常用的一些对象方法，具体请参看后续的【[对象管理](frame/g/index.md)】章节。虽然这种方式模块间耦合性比较高，但使用简便，也是推荐的使用方式。


# 路由注册介绍

本章开始之前，我们再来看一下本手册开头的`Hello World`程序：
```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/", func(r *ghttp.Request) {
        r.Response.Write("哈喽世界！")
    })
    s.Run()
}
```
其中，使用```BindHandler```方法进行路由注册的方式叫做```回调函数注册```，是最简单的一种路由注册方式。通过给指定的Web Server上对应的URI注册一个可执行的方法，当客户端访问该URI时，Web Server便自动调用对应注册的回调函数来执行处理。在回调函数注册中，每个注册函数都会有一个```ghttp.Request```对象参数指针，表示每个请求特定的独立的请求处理对象，回调函数可以通过该对象获取提交请求参数，也可以返回处理结果数据。

## 路由注册方式比较

在详细讲解每一种注册方式之前，先看看每种注册方式各自的优缺点，以便在不同的业务场景中选择更适合的注册方式。如果暂时不理解这个表格没有关系，可以在了解完每一种注册方式之后再回过头来看，也许会更清晰。

|  注册方式        |  使用难度  |  安全系数  |  执行性能  | 内存消耗  |
| ---                   |     ---       | --- | --- | ---|
|  回调函数注册  |  高  |  低  |  高  | 低 |
|  执行对象注册  |  中  |  中  |  中  | 中 |
|  控制器方式注册     |  低  |   高 |  低  |  高 |

比较指标说明：
1. 使用难度：主要指对于执行流程以及数据管理维护的复杂度；
1. 安全系数：主要指在异步多协程下的数据安全管理维护；
1. 执行性能：执行性能，相对比较的结果；
1. 内存消耗：内存消耗，相对比较的结果；

**为何要设计三种注册方式？**


以上的三种方式对应的是三种```使用习惯```：

1. **回调函数注册**：这种方式的路由注册不限制给定的回调函数是一个对象方法还是包方法，它仅仅需要一个函数的内存地址指针即可，使用比较灵活；
1. **执行对象注册**：使用一个实例化的`struct`对象进行路由注册，大多数的go web框架也仅提供这种方式，大部分场景下也推荐使用这种方式进行注册；
1. **控制器方式注册**： 非常类似于PHP的执行机制，变量的管理维护比较安全，struct对象的管理更类似于OOP编程，由于内部在运行时使用了反射机制，因此对于性能没有过高要求的场景可以考虑这种方式；

具体详细的介绍及使用请继续查看后续对应的章节。



## 路由注册接口文档

服务的注册仅有以下7个方法：

```go
func (s *Server) BindHandler(pattern string, handler HandlerFunc) error

func (s *Server) BindObject(pattern string, obj interface{}, methods ...string) error
func (s *Server) BindObjectMethod(pattern string, obj interface{}, method string) error
func (s *Server) BindObjectRest(pattern string, obj interface{}) error

func (s *Server) BindController(pattern string, c Controller, methods ...string) error
func (s *Server) BindControllerMethod(pattern string, c Controller, method string) error
func (s *Server) BindControllerRest(pattern string, c Controller) error
```

其中，```BindHandler```方法用于特定的回调函数注册，```BindObject*```方法用于对象相关注册，```BindController*```方法用于控制器相关注册。

需要注意的是，控制器注册```BindController*```系列方法第二个参数为控制器接口，给定的参数必须实现```ghttp.Controller```接口。简便的做法是用户自定义的控制器直接继承```gmvc.Controller```基类即可，```gmvc.Controller```已经实现了对应的接口方法。

路由注册使用的```pattern```参数格式在【[路由规则](net/ghttp/router/rules.md)】章节已经有介绍，这里便不再过多赘述。




## 域名路由注册方法

我们可以通过Server的以下方法获得Domain对象：
```go
func (s *Server) Domain(domains string) *Domain
```
其中```domains```参数支持多个域名绑定，使用```,```号分隔。

路由注册支持绑定域名，以下是对应的**接口文档**：
```go
func (d *Domain) BindHandler(pattern string, handler HandlerFunc) error

func (d *Domain) BindObject(pattern string, obj interface{}, methods ...string) error
func (d *Domain) BindObjectMethod(pattern string, obj interface{}, method string) error
func (d *Domain) BindObjectRest(pattern string, obj interface{}) error

func (d *Domain) BindController(pattern string, c Controller, methods ...string) error
func (d *Domain) BindControllerMethod(pattern string, c Controller, method string) error
func (d *Domain) BindControllerRest(pattern string, c Controller) error
```
各项参数说明和```Server```的路由注册对应方法一致，只不过在```Domain```对象的底层会自动将方法绑定到指定的域名列表中，只有对应的域名才能提供访问。

我们来看一个简单的例子，我们将前面的`Hello World`程序改成如下形式：
```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    g.Server().Domain("localhost").BindHandler("/", func(r *ghttp.Request) {
        r.Response.Write("Hello World!")
    })
    g.Server().Run()
}
```
我们再次使用 http://127.0.0.1/ 进行访问，发现Web Server返回```404```，为什么呢？因为该程序中的回调函数只注册到了```localhost```域名中，因此只能通过 http://localhost/ 访问，其他域名自然无法访问。


## 服务路由注册管理

服务的路由注册有两种方式：`统一路由注册管理`和`独立路由注册管理`。

### 统一路由注册管理
`统一路由注册`是官方推荐的路由注册管理方式。

1. **优点**
  * 统一的路由表注册、管理、维护；
  * 很容易根据路由对服务进行定位；
  * 易于对路由进行修改，不易出现路由注册冲突；
2. **缺点**
  * 当服务比较多时统一由一个地方进行路由维护，本身耦合性比较强；
  * 必须对外公开可供路由模块调用的注册的地址(方法)，或者结构/对象；

示例：
1. 控制器源码：
    ```go
    package ctlDoc

    import (
        "github.com/gogf/gf/net/ghttp"
        "github.com/gogf/gf/frame/g"
    )

    // 必须公开服务方法
    func Index(r *ghttp.Request) {
        // 业务逻辑部分
    }
    ```
1. 路由注册源码：
    ```go
    package router

    import (
        "github.com/gogf/gf/frame/g"
        "github.com/gogf/gf/net/ghttp"
        "github.com/gogf/gf-home/app/ctl/doc"
    )

    func init() {
        g.Server().BindHandler("/doc/*path", ctlDoc.Index)
    }
    ```
1. `main`模块的加载方式：
    ```go
    package main

    import (
        "github.com/gogf/gf/frame/g"
        _ "github.com/gogf/gf-home/boot"
        _ "github.com/gogf/gf-home/router"
    )

    func main() {
        g.Server().Run()
    }
    ```

### 独立路由注册管理

独立路由注册，指的是实现/统一定义好项目的路由注册规范，而后每个服务按照包的形式分开独立进行注册管理。例如，规范 指定的**路由前缀**由指定的服务使用，事先分配好路由资源(可设定好路由表)。如`/api`前缀归API服务使用，`/admin`前缀归后台管理使用，以此类推，每个服务下根据拿到的路由资源可继续再规范化拆分。
1. **优点**
  * 路由资源分配下的服务独立负责各自的路由管理；
  * 每个服务的包独立设计，可不公开任何的结构/方法，灵活度高，耦合性低内聚性强；
2. **缺点**
  * 必须事先定义路由注册规范，建立路由表规划设计；
  * 由于各服务路由独立运作，但仍有可能出现路由冲突的风险；
  * 同一路由资源下，出现耦合性比较强的多个服务时，需要小心对路由进行解耦再分配；

在这种方式下，所有的路由注册独立在**控制器包**的```init```初始化方法中完成(```init```是Go语言内置的```模块初始化方法```，并且一个包中支持多个```init```方法)，一个包可以包含多个文件，每个文件都可以有一个init初始化方法，可以分开注册，在使用的时候会通过同一个包引入进程序，自动调用初始化方法完成注册。


示例：

1. 控制器源码：
    ```go
    package controller

    import (
        "github.com/gogf/gf/net/ghttp"
        "github.com/gogf/gf/frame/g"
    )

    // 统一在各自包的init包初始化方法中进行路由注册
    func init() {
        g.Server().BindHandler("/api/*path", index)
    }

    // 服务方法不需要对外公开
    func index(r *ghttp.Request) {
        // 业务逻辑部分
    }
    ```
1. `main`模块源码：
    ```go
    package main

    import (
    	"github.com/gogf/gf/frame/g"
        "github.com/gogf/gf/net/ghttp"
        _ "PATH/TO/YOUR/PROJECT/app/ctl/api"
    )

    func main() {
        g.Server().SetPort(8199)
        g.Server().Run()
    }
    ```
    其中通过：
    ```go
    import _ "PATH/TO/YOUR/PROJECT/app/ctl/api"
    ```
    这样一条语句完成了对包中的所有控制器的引入和注册（当然，包中的```init```应当实现注册方法调用），在demo包中包含了多个控制器、执行对象、回调函数的注册，demo包具体的控制器注册以及相关逻辑我们将在后续章节继续介绍。
