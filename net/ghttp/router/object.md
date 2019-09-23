
[TOC]

# 执行对象注册

执行对象注册是在注册时便给定一个实例化的对象，以后每一个请求都交给该对象(同一对象)处理，该对象常驻内存不释放。由于相比较控制器注册来说，执行对象注册方式在处理请求的时候不需要不停地创建/销毁控制器对象，因此请求处理效率会高很多。

这种注册方式的缺点也很明显，服务端进程在启动时便需要初始化这些执行对象，并且这些对象需要自行负责对自身数据的并发安全维护（往往struct对象的成员变量应当是并发安全的，每个请求执行完毕后struct对象不会销毁，其成员变量也不会释放）。

执行对象的定义没有严格要求，也没有强行要求继承`gmvc.Controller`控制器基类，因为在请求进入时没有自动初始化流程，内部的成员变量需要自行维护（包括变量初始化，变量销毁等）。

> <font color=OrangeRed size=3>注意事项：</font>执行对象和控制器的路由注册有很大不同，注册的对象都应当是**同一个**。例如以下的注册方式是错误的：
```go
s.BindObjectMethod("GET:/api/user/{id}", new(User), "Show")
s.BindObjectMethod("POST:/api/user/{id}", new(User), "Edit")
```
这样会造成注册的方法其实是两个不同的`User`对象，而应当使用以下方式注册：
```go
user := new(User)
s.BindObjectMethod("GET:/api/user/{id}", user, "Show")
s.BindObjectMethod("POST:/api/user/{id}", user, "Edit")
```

## 执行对象注册

我们可以通过```ghttp.BindObject```方法完成执行对象的注册。

[github.com/gogf/gf/blob/master/.example/frame/mvc/controller/demo/object.go](https://github.com/gogf/gf/blob/master/.example/frame/mvc/controller/demo/object.go)

```go
package demo

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

type Object struct {}

func init() {
    g.Server().BindObject("/object", new(Object))
}

func (o *Object) Index(r *ghttp.Request) {
    r.Response.Write("object index")
}

func (o *Object) Show(r *ghttp.Request) {
    r.Response.Write("object show")
}
```
可以看到，执行对象在进行路由注册时便生成了一个对象(执行对象在Web Server启动时便生成)，此后不管多少请求进入，Web Server都是将请求转交给该对象对应的方法进行处理。需要注意的是，公开方法的定义，必须为以下形式：
```go
func(r *ghttp.Request) 
```
否则无法完成注册，调用注册方法时会有错误提示，例如：
```shell
panic: interface conversion: interface {} is xxx, not func(*ghttp.Request)
```
该示例执行后可以通过，可以通过 http://127.0.0.1:8199/object/show 查看效果。

### 默认路由方法

控制器中的```Index```方法是一个特殊的方法，例如，当注册的路由规则为```/user```时，HTTP请求到```/user```时，将会自动映射到控制器的```Index```方法。也就是说，访问地址```/user```和```/user/index```将会达到相同的执行效果。对后续的控制器注册方式同理。

### 路由内置变量

当使用`BindObject`方法进行执行对象注册时，在路由规则中可以使用两个内置的变量：```{.struct}```和```{.method}```，前者表示当前**对象名称**，后者表示当前注册的**方法名**。这两个内置变量使得开发者可以非常灵活地自定义路由规则。

我们来看一个例子：
[github.com/gogf/gf/blob/master/.example/frame/mvc/controller/demo/buildin-vars.go](https://github.com/gogf/gf/blob/master/.example/frame/mvc/controller/demo/buildin-vars.go)
```go
package demo

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

type Order struct { }

func init() {
    g.Server().BindObject("/{.struct}-{.method}", new(Order))
}

func (o *Order) List(r *ghttp.Request) {
    r.Response.Write("List")
}
```
启动外层的```main.go```，我们尝试着访问 http://127.0.0.1:8199/order-list ，可以看到页面输出```List```。如果路由规则中不使用内置变量，那么默认的情况下，方法将会被追加到指定的路由规则末尾。

### 命名风格规则

当方法名称带有多个`单词`(按照字符大写区分单词)时，路由控制器默认会自动使用英文连接符号```-```进行拼接，因此访问的时候方法名称需要带```-```号。例如，方法名为```UserName```时，生成的URI地址为```user-name```；方法名为```ShowListItems```时，生成的URI地址为```show-list-items```；以此类推。对后续的控制器注册方式同理。

此外，我们可以通过```ghttp.Server.SetNameToUriType```方法来设置`struct/method`名称与uri的转换方式。支持的方式目前有`4`种，对应`4`个常量定义：
```go
URI_TYPE_DEFAULT  = 0      // 路由注册时对象和方法名称转换为URI时，全部转为小写，单词以'-'连接符号连接
URI_TYPE_FULLNAME = 1      // 不处理名称，以原有名称构建成URI
URI_TYPE_ALLLOWER = 2      // 仅转为小写，单词间不使用连接符号
URI_TYPE_CAMEL    = 3      // 采用驼峰命名方式
```
我们来看一个示例：
[github.com/gogf/gf/blob/master/.example/net/ghttp/server/name.go](https://github.com/gogf/gf/blob/master/.example/net/ghttp/server/name.go)
```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

type User struct {}

func (u *User) ShowList(r *ghttp.Request) {
    r.Response.Write("list")
}

func main() {
    s1 := g.Server(1)
    s2 := g.Server(2)
    s3 := g.Server(3)
    s4 := g.Server(4)

    s1.SetNameToUriType(ghttp.URI_TYPE_DEFAULT)
    s2.SetNameToUriType(ghttp.URI_TYPE_FULLNAME)
    s3.SetNameToUriType(ghttp.URI_TYPE_ALLLOWER)
    s4.SetNameToUriType(ghttp.URI_TYPE_CAMEL)

    s1.BindObject("/{.struct}/{.method}", new(User))
    s2.BindObject("/{.struct}/{.method}", new(User))
    s3.BindObject("/{.struct}/{.method}", new(User))
    s4.BindObject("/{.struct}/{.method}", new(User))

    s1.SetPort(8100)
    s2.SetPort(8200)
    s3.SetPort(8300)
    s4.SetPort(8400)

    s1.Start()
    s2.Start()
    s3.Start()
    s4.Start()

    g.Wait()
}
```
这个示例采用了`多Server`运行方式，将不同的名称转换方式使用了不同的Server来配置运行，因此我们可以方便地在同一个程序中，访问不同的Server（通过不同的端口绑定）看到不同的结果。执行以上示例后，可以分别访问以下URL地址得到期望的结果：
```html
http://127.0.0.1:8100/user/show-list
http://127.0.0.1:8200/User/ShowList
http://127.0.0.1:8300/user/showlist
http://127.0.0.1:8300/user/showList
```

### 对象方法注册

假如控制器中有若干公开方法，但是我只想注册其中几个，其余的方法我不想对外公开，怎么办？

我们可以通过```BindObject```传递**第三个非必需参数**替换实现，参数支持传入**多个**方法名称，多个名称以英文```,```号分隔（**方法名称参数区分大小写**）。

示例：
```go
package demo

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

type Object struct {}

func init() {
    obj := new(Object)
    g.Server().BindObject("/object", obj)
    g.Server().BindObjectMethod("/object-show", obj, "Show")
}

func (o *Object) Index(r *ghttp.Request) {
    r.Response.Write("object index")
}

func (o *Object) Show(r *ghttp.Request) {
    r.Response.Write("object show")
}
```


## 绑定路由方法


我们可以通过```BindObjectMethod```方法绑定指定的路由到指定的方法执行（**方法名称参数区分大小写**）。

> 注意`BindObjectMethod`和`BindObject`的区别：
`BindObjectMethod`只是将对象中的指定方法与指定路由规则进行绑定，第三个`method`参数只能指定一个方法名称；
`BindObject`是注册执行对象，会自动按照方法命名生成一系列默认的路由规则(URI后缀形式)，第三个`methods`参数可以指定多个注册的方法名称。

来看一个例子：

```go
package demo

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

type ObjectMethod struct {}

func init() {
    obj := &ObjectMethod{}
    g.Server().BindObjectMethod("/object-method-show1", obj, "Show1")
    g.Server().Domain("localhost").BindObjectMethod("/object-method-show2", obj, "Show2")
}

func (o *ObjectMethod) Show1(r *ghttp.Request) {
    r.Response.Write("show 1")
}

func (o *ObjectMethod) Show2(r *ghttp.Request) {
    r.Response.Write("show 2")
}

func (o *ObjectMethod) Show3(r *ghttp.Request) {
    r.Response.Write("show 3")
}
```
这个例子比较简单，同时也演示了域名绑定路由到执行对象的指定方法负责执行。


## RESTful对象注册

```RESTful```设计方式的控制器，通常用于API服务。**在这种模式下，HTTP的Method将会映射到控制器对应的方法名称**，例如：```POST```方式将会映射到控制器的```Post```方法中(公开方法，首字母大写)，```DELETE```方式将会映射到控制器的```Delete```方法中,以此类推。其他非HTTP Method命名的方法，即使是定义的包公开方法，将无法完成自动注册，对于应用端不可见。当然，如果控制器并未定义对应HTTP Method的方法，该Method请求下将会返回 ```HTTP Status 404```。

我们可以通过```ghttp.BindObjectRest```方法完成REST对象的注册。

[github.com/gogf/gf/blob/master/.example/frame/mvc/controller/demo/object_rest.go](https://github.com/gogf/gf/blob/master/.example/frame/mvc/controller/demo/object_rest.go)

```go
package demo

import "github.com/gogf/gf/net/ghttp"

// 测试绑定对象
type ObjectRest struct {}

func init() {
    ghttp.GetServer().BindObjectRest("/object-rest", &ObjectRest{})
}

// RESTFul - GET
func (o *ObjectRest) Get(r *ghttp.Request) {
    r.Response.Write("RESTFul HTTP Method GET")
}

// RESTFul - POST
func (c *ObjectRest) Post(r *ghttp.Request) {
    r.Response.Write("RESTFul HTTP Method POST")
}

// RESTFul - DELETE
func (c *ObjectRest) Delete(r *ghttp.Request) {
    r.Response.Write("RESTFul HTTP Method DELETE")
}

// 该方法无法映射，将会无法访问到
func (c *ObjectRest) Hello(r *ghttp.Request) {
    r.Response.Write("Hello")
}
```

## 构造方法`Init`与析构方法`Shut`

执行对象中的```Init```和```Shut```是两个在HTTP请求流程中被Web Server自动调用的特殊方法(类似`构造函数`和`析构函数`的作用)。当struct对象中定义了这两个方法名称时，使用执行对象方式进行路由注册时将会被自动注册到路由表中。

1. `Init`回调方法

	执行对象收到请求时的初始化方法，在服务接口调用之前被回调执行。

    方法定义：
    ```go
    // "构造函数"执行对象方法
    func (obj *Object) Init(r *ghttp.Request) {

    }
    ```

1. `Shut`回调方法

	当请求结束时被Web Server自动调用，可以用于执行对象执行一些收尾处理的操作。

    方法定义：
    ```go
    // "析构函数"执行对象方法
    func (obj *Object) Shut(r *ghttp.Request) {

    }
    ```

## `Exit`, `ExitAll`与`ExitHook`

1. `Exit`: 仅退出当前执行的逻辑方法，如: 当前HOOK方法、服务方法，不退出后续的逻辑处理，可用于替代`return`；
1. `ExitAll`: 强行退出当前执行流程，当前执行方法的后续逻辑以及后续所有的逻辑方法将不再执行，常用于权限控制；
1. `ExitHook`: 当路由匹配到多个HOOK方法时，默认是按照路由匹配优先级顺序执行HOOK方法。当在HOOK方法中调用`ExitHook`方法后，后续的HOOK方法将不会被继续执行，作用类似HOOK方法覆盖；


使用示例：
```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/os/glog"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    p := "/"
    s := g.Server()
    s.BindHandler(p, func(r *ghttp.Request) {
        r.Response.Writeln("start")
        r.Exit()
        r.Response.Writeln("end")
    })
    s.BindHookHandlerByMap(p, map[string]ghttp.HandlerFunc{
        ghttp.HOOK_BEFORE_SERVE : func(r *ghttp.Request){
            glog.To(r.Response.Writer).Println("BeforeServe")
        },
        ghttp.HOOK_AFTER_SERVE  : func(r *ghttp.Request){
            glog.To(r.Response.Writer).Println("AfterServe")
        },
    })
    s.SetPort(8199)
    s.Run()
}
```
其中我们使用了两个回调注册`ghttp.HOOK_BEFORE_SERVE`和`ghttp.HOOK_AFTER_SERVE`，以测试`Exit`方法是否会对回调方法产生影响。

执行后访问 http://127.0.0.1:8199 ，可以看到页面输出了以下内容：
```
2018-11-06 18:57:52.153 BeforeServe
start
2018-11-06 18:57:52.153 AfterServe
```
并没有输出`end`字符串，表明在调用`r.Exit()`后当前的业务流程被立即退出了。