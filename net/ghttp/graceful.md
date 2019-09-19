
[TOC]


# 基本介绍

`平滑重启`(`热重启`)是指Web Server在重启的时候不会中断已有请求的执行。该特性在不同的项目版本发布的时候特别有用，例如，当需要先后发布两个版本：A、B，那么在A执行的过程当中，我们可以将B的程序发布`直接覆盖`A的程序，并使用平滑重启特性(使用`Web`或者`命令行`)无缝地将请求过渡到新版本的服务中。

需要注意的是，该特性仅限于```*nix```系统(Linux/Unix/FreeBSD等等)，在Windows下仅支持完整重启功能(请求无法平滑过渡)。

# 管理功能

gf框架支持非常方便的`Web管理功能`，也就是说我们可以通过Web页面/接口直接进行Web Server的重启/关闭等管理操作。同时，gf框架也支持通过`命令行终端指令`(仅限```*nix```系统)的形式进行Web Server的重启/关闭等管理操作。


## Web管理

我们先来看一下Web Server中涉及到管理操作方法有哪些：
```go
func (s *Server) Restart(newExeFilePath...string) error
func (s *Server) Shutdown() error

func (s *Server) EnableAdmin(pattern ...string)
```
`Restart`用于重启服务（```*nix```系统下为平滑重启，```windows```下为完整重启），```Shutdown```用于关闭服务，```EnableAdmin```用于将管理页面注册到指定的路由规则上，默认地址是```/debug/admin```(我们可以指定一个私密的管理地址，也可以对该路由规则绑定`ghttp.HOOK_BEFORE_SERVE`事件回调来进行页面鉴权)。

以下对其中两个方法做详细说明。
1. **Restart**

	`Restart`的参数可指定自定义重启的可执行文件路径(`newExeFilePath`)，不传递时默认为原可执行文件路径。特别是在windows系统下，当可执行文件正在使用时，无法对其进行文件替换更新（新版本文件替换老版本文件）。当指定自定义的可执行文件路径后，Web Server重启时将会执行新版本的可执行文件，不再使用老版本文件，这种特性简化了在某些系统上的版本更新流程。

1. **EnableAdmin**
	* 首先，该方法为用户管理Web Server提供了简便的页面和接口，在单Web Server下管理非常方便，直接访问管理页面点击对应链接即可。需要注意的是，由于带有管理功能，如果是在生产环境上，建议自定义该管理地址为一个私密地址。
	* 同时，```EnableaAdmim```提供的restart接口也支持自定义可执行文件路径，直接通过GET参数往restart接口传递```newExeFilePath```变量即可，例如： http://127.0.0.1/debug/admin/restart?newExeFilePath=xxxxxxx 
	* 此外，在大多数时候，Web Server往往不只有1个节点，因此大多数服务管理运维中，例如：重启操作，当然不是直接访问每个Web Server的admin页面手动执行重启操作。而是充分利用admin页面提供的功能接口，通过接口控制来实现统一的Web Server管理控制。

### 示例1：基本使用
```go
package main

import (
    "time"
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/os/gproc"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/", func(r *ghttp.Request){
        r.Response.Writeln("哈喽！")
    })
    s.BindHandler("/pid", func(r *ghttp.Request){
        r.Response.Writeln(gproc.Pid())
    })
    s.BindHandler("/sleep", func(r *ghttp.Request){
        r.Response.Writeln(gproc.Pid())
        time.Sleep(10*time.Second)
        r.Response.Writeln(gproc.Pid())
    })
    s.EnableAdmin()
    s.SetPort(8199)
    s.Run()
}
```
我们通过以下几个步骤来测试平滑重启：
1. 访问 http://127.0.0.1:8199/pid 查看当前进程的pid

	![](/images/Selection_999144.png)

3. 访问 http://127.0.0.1:8199/sleep ，这个页面将会执行10秒，用于测试重启时该页面请求执行是否会断掉

	![](/images/Selection_999145.png)

5. 访问 http://127.0.0.1:8199/debug/admin ，这是```s.EnableAdmin```后默认注册的一个Web Server管理页面

	![](/images/QQ截图20180601192828.png)

7. 随后我们点击```restart```管理链接，Web Server将会立即平滑重启(```*nix```系统下)

	![](/images/QQ截图20180601192916.png)

    同时在终端也会输出以下信息:
    ```shell
    2018-05-18 11:02:04.812 11511: http server started listening on [:8199]
    2018-05-18 11:02:09.172 11511: server reloading
    2018-05-18 11:02:09.172 11511: all servers shutdown
    2018-05-18 11:02:09.176 16358: http server restarted listening on [:8199]
    ```
6. 我们可以发现在整个操作中，```sleep```页面的执行并没有被中断，继续等待几秒，当```sleep```执行完成后，页面输出内容为：

	![](/images/Selection_999148.png)
    
8. 可以发现，```sleep```页面输出的进程pid和之前的不一样了，代表请求的执行被新的进程平滑接管，旧的服务进程也随之销毁；

### 示例2：HTTPS支持

```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/", func(r *ghttp.Request){
        r.Response.Writeln("哈罗！")
    })
    s.EnableHTTPS("/home/john/temp/server.crt", "/home/john/temp/server.key")
    s.EnableAdmin()
    s.SetPort(8200)
    s.Run()
}
```
gf框架的平滑重启特性对于HTTPS的支持也是相当友好和简便，操作步骤如下：
1. 访问 https://127.0.0.1:8200/debug/admin/restart 平滑重启HTTPS服务；
2. 访问 https://127.0.0.1:8200/debug/admin/shutdown 平滑关闭Web Server服务；

在命令行终端可以看到以下输出信息：
```shell
2018-05-18 11:13:05.554 17278: https server started listening on [:8200]
2018-05-18 11:13:21.270 17278: server reloading
2018-05-18 11:13:21.270 17278: all servers shutdown
2018-05-18 11:13:21.278 17319: https server reloaded listening on [:8200]
2018-05-18 11:13:34.895 17319: server shutting down
2018-05-18 11:13:34.895 17269: all servers shutdown
```
### 示例3：多服务及多端口
gf框架的平滑重启特性相当强大及稳定，不仅仅支持单一服务单一端口监听管理，同时也支持多服务多端口等复杂场景的监听管理。
```go
package main

import (
    "github.com/gogf/gf/frame/g"
)

func main() {
    s1 := g.Server("s1")
    s1.EnableAdmin()
    s1.SetPort(8100, 8200)
    s1.Start()

    s2 := g.Server("s2")
    s2.EnableAdmin()
    s2.SetPort(8300, 8400)
    s2.Start()

    g.Wait()
}
```
以上示例演示的是两个Web Server ```s1```及```s2```，分别监听```8100```，```8200```及```8300```，```8400```。我们随后访问 http://127.0.0.1:8100/debug/admin/reload 平滑重启服务，然后再通过 http://127.0.0.1:8100/debug/admin/shutdown 平滑关闭服务，最终在终端打印出的信息如下：
```shell
2018-05-18 11:26:54.729 18111: http server started listening on [:8400]
2018-05-18 11:26:54.729 18111: http server started listening on [:8100]
2018-05-18 11:26:54.729 18111: http server started listening on [:8300]
2018-05-18 11:26:54.729 18111: http server started listening on [:8200]
2018-05-18 11:27:08.203 18111: server reloading
2018-05-18 11:27:08.203 18111: all servers shutdown
2018-05-18 11:27:08.207 18124: http server reloaded listening on [:8300]
2018-05-18 11:27:08.207 18124: http server reloaded listening on [:8400]
2018-05-18 11:27:08.207 18124: http server reloaded listening on [:8200]
2018-05-18 11:27:08.207 18124: http server reloaded listening on [:8100]
2018-05-18 11:27:19.379 18124: server shutting down
2018-05-18 11:27:19.380 18102: all servers shutdown
```

## 命令行管理

gf框架除了提供Web方式的管理能力以外，也支持命令行方式来进行管理，由于命令行采用了```信号量```进行管理，因此仅支持```*nix```系统。

### 重启服务
使用```SIGUSR1```信号量实现，**使用方式**：
```shell
kill -SIGUSR1 进程ID
```

### 关闭服务
使用```SIGINT/SIGQUIT/SIGKILL/SIGHUP/SIGTERM```其中任意一个信号量来实现，**使用方式**：
```shell
kill -SIGTERM 进程ID
```

## 其他管理方式

由于gf框架的Web Server采用了单例设计，因此任何地方都可以通过```g.Server(名称)```或者```ghttp.GetServer(名称)```来获得对应Web Server的单例对象，随后通过```Restart```和```Shutdown```方法可以实现对该Web Server的管理。
