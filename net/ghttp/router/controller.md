
[TOC]

**为防止多种注册方式对开发者引起混淆，并且控制器注册方式效率较低，因此未来不再推荐使用控制器注册方式。**

# 控制器注册

这种方式将每一个请求都当做一个控制器对象来处理，比较类似且媲美于PHP的请求执行模式，当一个请求进来之后，立即初始化一个新的控制器对象进行处理，处理完成之后释放控制器资源。这种路由注册方式的优点是简单、安全、OOP设计，每个请求的控制器严格数据隔离，成员变量无法相互共享。

在这种模式下，我们可以给`控制器struct`对象定义一些非并发安全的成员变量，在请求中可以进行随意操作，不会影响并发安全性，因为请求一旦完成，所有的成员变量随着控制器对象销毁而销毁。

控制器注册方式当然也支持```{.struct}```及```{.method}```内置变量。

> `控制器注册`方式与`执行对象注册`方式非常类似，两者除了运行机制不一样以外，其他细节大同小异。建议在开始本章节阅读之前先了解一下[执行对象注册方式](net/ghttp/router/object)。

## 控制器注册

### 基本使用

我们可以通过```BindController```方法完成控制器的注册。

[github.com/gogf/gf/blob/master/.example/frame/mvc/controller/demo/user.go](https://github.com/gogf/gf/blob/master/.example/frame/mvc/controller/demo/user.go)

```go
package demo

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/frame/gmvc"
)

type User struct {
    gmvc.Controller
}

func init() {
    s := g.Server()
    s.BindController("/user",                       new(User))
    s.BindController("/user/{.method}/{uid}",       new(User), "Info")
    s.BindController("/user/{.method}/{page}.html", new(User), "List")
}

func (u *User) Index() {
    u.Response.Write("User")
}

func (u *User) Info() {
    u.Response.Write("Info - Uid: ", u.Request.Get("uid"))
}

func (u *User) List() {
    u.Response.Write("List - Page: ", u.Request.Get("page"))
}
```
以上注册的路由列表：
```html
/user
/user/index
/user/info
/user/list
/user/{.method}/{uid}
/user/{.method}/{page}.html
```
路由注册必须提供注册的```pattern```，往往是一个精准匹配规则的URI，注册时会将所有控制器的公开方法映射到指定URI末尾(示例代码中，```s.BindController("/user", new(User))```，其中的```new(User)```也可以使用```&User{}```来替换)。注册的控制器参数是一个```ghttp.Controller```接口，参数直接传递自定义的控制器对象指针即可(```&ControllerUser{}```，实际上只要继承了```gmvc.Controller```基类，控制器的指针对象便已经自动实现了```ghttp.Controller```接口)。

该示例也展示了内置变量```{.method}```来使用方法名称，**当注册的规则中带有内置变量时，路由将不会自动将方法名称追加到所给规则末尾，而是使用方法名称替换该内置变量**。

框架通过解析该对象指针获取对应的控制器方法，生成反射类型，处理请求时再根据该反射类型自动生成对应的控制器对象，处理客户端请求，处理完后自动销毁该控制器对象。

### 指定方法

控制器方法注册原理类似于执行对象方法注册，只公开控制器中的特定方法。看下面这个例子，执行后```ControllerMethod```的```Name```和```Age```方法将被注册到Web Server提供服务，而```Info```方法却不会对外公开。
```go
package demo

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/frame/gmvc"
)

type Method struct {
    gmvc.Controller
}

func init() {
    // 第三个参数指定主要注册的方法，其他方法不注册，方法名称会自动追加到给定路由后面，构成新路由
    // 以下注册会中注册两个新路由: /method/name, /method/age
    g.Server().BindController("/method", new(Method), "Name, Age")
}

func (c *Method) Name() {
    c.Response.Write("John")
}

func (c *Method) Age() {
    c.Response.Write("18")
}

func (c *Method) Info() {
    c.Response.Write("Info")
}
```
启动外层的```main.go```，我们尝试着访问 http://127.0.0.1:8199/method/info ，因为没有对该方法执行注册，因此会发现返回```404```；而 http://127.0.0.1:8199/method/name 及 http://127.0.0.1:8199/method/age 却能够正常访问。这个例子比较简单，没有用复杂的动态路由，因此不在赘述。

## 绑定路由方法

> 注意`BindControllerMethod`和`BindController`的区别：
`BindControllerMethod`只是将控制器中的指定方法与指定路由规则进行绑定，第三个`method`参数只能指定一个方法名称；
`BindController`是注册控制器，会自动按照方法命名生成一系列默认的路由规则(URI后缀形式)，第三个`methods`参数可以指定多个注册的方法名称。


```go
package demo

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/frame/gmvc"
)

type Method struct {
    gmvc.Controller
}

func init() {
    // 绑定路由到指定的方法执行，以下注册只会注册一个路由: /method-name
    g.Server().BindControllerMethod("/method-name", new(Method), "Name")
}

func (c *Method) Name() {
    c.Response.Write("John")
}

func (c *Method) Age() {
    c.Response.Write("18")
}

func (c *Method) Info() {
    c.Response.Write("Info")
}

```



## RESTful控制器注册

这种方式注册的控制器，运行模式和“控制器注册”模式相同。我们可以通过```BindControllerRest```方法完成RESTful控制器的注册。

以下是一个示例：

[github.com/gogf/gf/blob/master/.example/frame/mvc/controller/demo/rest.go](https://github.com/gogf/gf/blob/master/.example/frame/mvc/controller/demo/rest.go)

```go
package demo

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/frame/gmvc"
)

type Rest struct {
    gmvc.Controller
}

func init() {
    g.Server().BindControllerRest("/rest", &Rest{})
}

// RESTFul - GET
func (c *Rest) Get() {
    c.Response.Write("RESTFul HTTP Method GET")
}

// RESTFul - POST
func (c *Rest) Post() {
    c.Response.Write("RESTFul HTTP Method POST")
}

// RESTFul - DELETE
func (c *Rest) Delete() {
    c.Response.Write("RESTFul HTTP Method DELETE")
}

// 该方法无法映射，将会无法访问到
func (c *Rest) Hello() {
    c.Response.Write("Hello")
}
```

## 构造方法`Init`与析构方法`Shut`

```ghttp.Controller```接口中的```Init```和```Shut```是两个在HTTP请求流程中被Web Server自动调用的特殊方法(类似构造函数和析构函数的作用)。```gmvc.Controller```基类中已经实现了这两个方法，用户自定义的控制器类直接继承```gmvc.Controller```即可。**如果需要自定义请求初始化以及请求结束时的一些业务逻辑操作，可以在自定义控制器中重载这两个方法**。

1. `Init`回调方法

	控制器初始化方法，参数是当前请求的对象。```gmvc.Controller```基类中的Init方法是对自身成员对象的初始化。

    方法定义：
    ```go
    // "构造函数"控制器方法
    func (c *Controller) Init(r *ghttp.Request) {
        
    }
    ```

1. `Shut`回调方法

	当请求结束时被Web Server自动调用，可以用于控制器一些收尾处理的操作。默认不执行任何操作。

    方法定义：
    ```go
    // "析构函数"控制器方法
    func (c *Controller) Shut() {

    }
    ```

## `Exit`, `ExitAll`与`ExitHook`

1. `Exit`: 仅退出当前执行的逻辑方法，如: 当前HOOK方法、服务方法，不退出后续的逻辑处理，可用于替代`return`；
1. `ExitAll`: 强行退出当前执行流程，当前执行方法的后续逻辑以及后续所有的逻辑方法将不再执行，常用于权限控制；
1. `ExitHook`: 当路由匹配到多个HOOK方法时，默认是按照路由匹配优先级顺序执行HOOK方法。当在HOOK方法中调用`ExitHook`方法后，后续的HOOK方法将不会被继续执行，作用类似HOOK方法覆盖；



