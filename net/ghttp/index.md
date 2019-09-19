

[TOC]

`gf`框架提供了非常强大的Web Server模块，由```ghttp```模块实现。集成了丰富完善的Web Server相关组件，例如：Router、Cookie、Session、路由注册、配置管理、模板引擎、缓存控制等等，支持热重启、热更新、多域名、多端口、多服务、HTTPS、Rewrite等等特性。

接口文档地址： 

https://godoc.org/github.com/gogf/gf/net/ghttp

# 哈喽世界

老规矩，我们先来一个`Hello World`：

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
这便是一个最简单的Web Server，默认情况下它不支持静态文件处理，只有一个功能，访问 http://127.0.0.1/ 的时候，它会返回`哈喽世界！`。

任何时候，您都可以通过```g.Server()```方法获得一个默认的Web Server对象，该方法采用```单例模式```设计，也就是说，多次调用该方法，返回的是同一个Web Server对象。通过`Run()`方法执行Web Server的监听运行，在没有任何额外设置的情况下，它默认监听`80`端口。

关于其中的路由注册，我们将会在后续的章节中介绍，我们继续来看看如何创建一个支持静态文件的Web Server。


# 静态服务

创建并运行一个支持静态文件的Web Server：

```go
package main

import (
    "github.com/gogf/gf/frame/g"
)

func main() {
    s := g.Server()
    s.SetIndexFolder(true)
    s.SetServerRoot("/home/www/")
    s.Run()
}
```
创建了Web Server对象之后，我们可以使用```Set*```方法来设置Web Server的属性，我们这里的示例中涉及到了两个属性设置方法：
1. ```SetIndexFolder```用来设置是否允许列出Web Server主目录的文件列表（默认为`false`）；
1. ```SetServerRoot```用来设置Web Server的主目录（类似其他Web Server中的```DocumentRoot```地址，默认为空，在某些时候，Web Server仅提供接口服务，因此Web Server的主目录为非必需参数）；

Web Server默认情况下是没有任何主目录的设置，只有设置了主目录，才支持对应主目录下的静态文件的访问。

# 多端口监听

`ghttp.Server`同时支持多端口监听，只需要往```SetPort```参数设置绑定多个端口号即可（当然，针对于`HTTPS`服务，我们也同样可以通过```SetHTTPSPort```来设置绑定并支持多个端口号的监听，`HTTPS`服务的介绍请参看后续对应章节）。

我们来看一个例子：

```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/", func(r *ghttp.Request){
        r.Response.Writeln("go frame!")
    })
    s.SetPort(8100, 8200, 8300)
    s.Run()
}
```

执行以上示例后，我们访问以下URL将会得到期望的相同结果：
```shell
http://127.0.0.1:8100/
http://127.0.0.1:8200/
http://127.0.0.1:8300/
```

# 多服务支持

`ghttp.Server`支持多Web Server运行，下面我们来看一个例子：

```go
package main

import (
    "github.com/gogf/gf/frame/g"
)

func main() {
    s1 := g.Server("s1")
    s1.SetPort(8080)
    s1.SetIndexFolder(true)
    s1.SetServerRoot("/home/www/static1")
    s1.Start()

    s2 := g.Server("s2")
    s2.SetPort(8088)
    s2.SetIndexFolder(true)
    s2.SetServerRoot("/home/www/static2")
    s2.Start()

    g.Wait()
}
```
可以看到我们在支持多个Web Server的语句中，给```g.Server```方法传递了不同的参数（参数可以为任意类型，常用字符串或者整型识别），该参数为Web Server的名称，之前我们提到```g.Server```方法采用了单例设计模式，该参数用于标识不同的Web Server，因此需要保证唯一性。

如果需要获取同一个Web Server，那么传入同一个名称即可。例如在多个goroutine中，或者不同的模块中，都可以通过```g.Server```获取到同一个Web Server对象。

# 域名&多域名

同一个Web Server支持多域名绑定，并且不同的域名可以绑定不同的服务。

我们来看一个简单的例子：

```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func Hello1(r *ghttp.Request) {
    r.Response.Write("127.0.0.1: Hello1!")
}

func Hello2(r *ghttp.Request) {
    r.Response.Write("localhost: Hello2!")
}

func main() {
    s := g.Server()
    s.Domain("127.0.0.1").BindHandler("/", Hello1)
    s.Domain("localhost").BindHandler("/", Hello2)
    s.Run()
}
```
我们访问 http://127.0.0.1/ 和 http://localhost/ 可以看输出不同的内容。

此外，```Domain```方法支持多个域名参数，使用英文“,”号分隔，例如：
```
s.Domain("localhost1,localhost2,localhost3").BindHandler("/", Hello2)
```
这条语句的表示将Hello2方法注册到指定的3个域名中(`localhost1~3`)，对其他域名不可见。

需要注意的是：`Domain`方法的参数必须是**准确的**域名，**不支持泛域名形式**，例如：```*.johng.cn```或者```.johng.cn```是不支持的，```api.johng.cn```或者```johng.cn```才被认为是正确的域名参数。


# 强大路由特性

`ghttp.Server`提供了比任何同类框架更加出色的路由特性，我们先来看一个简单的示例：
```go
package main

import (
    "github.com/gogf/gf/net/ghttp"
    "github.com/gogf/gf/frame/g"
)

func main() {
    s := g.Server()
    s.BindHandler("/{class}-{course}/:name/*act", func(r *ghttp.Request) {
        r.Response.Writeln(r.Get("class"))
        r.Response.Writeln(r.Get("course"))
        r.Response.Writeln(r.Get("name"))
        r.Response.Writeln(r.Get("act"))
    })
    s.SetPort(8199)
    s.Run()
}
```
这是一个混合的路由规则示例，用于展示某个班级、某个学科、某个学生、对应的操作，运行后，我们可以通过例如该地址： http://127.0.0.1:8199/class3-math/john/score 看到测试结果。在页面上你可以看得到对应的路由规则都被一一解析，业务层可以根据解析的参数进行对应的业务逻辑处理。

由于路由注册的知识点比较多，并且`gf`框架的路由又比较强大，因此具体的路由注册管理介绍请查看后续【[路由控制](net/ghttp/router/rules.md)】章节。

# 平滑重启特性


`ghttp.Server`**内置** 支持Web Server平滑重启特性，详细介绍请参考【[平滑重启特性](net/ghttp/graceful.md)】章节。


# HTTPS服务支持


`ghttp.Server`支持HTTPS服务，并且也同时支持单进程提供HTTP&HTTPS服务，HTTPS的详细介绍请参考【[HTTPS服务](net/ghttp/https.md)】章节。


# Web Server配置管理

Web Server的配置管理往往不是必须的，因为大多数场景下使用默认的Web Server配置即可。如果想要更进一步进行配置，请查看【[Server配置管理](net/ghttp/config.md)】章节。




更多功能及特性请继续阅读后续章节。
