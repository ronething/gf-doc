[TOC]

> 该章节为手把手安装Go开发环境及IDE配置教程，仅针对Golang新手，老司机请忽略。

# Go开发环境安装


## Step1 - 下载Go开发包

访问Go国内镜像站下载页面 [https://golang.google.cn/dl/](https://golang.google.cn/dl/)，并在页面最上方的版本中选择你当前的系统版本，会下载最新版本的Go开发包:
![](/images/downloadgo.png)

## Step2 - 安装引导

访问官方安装介绍页面 [https://golang.google.cn/doc/install](https://golang.google.cn/doc/install)，按照当前系统版本执行对应的安装流程即可。

Windows(`msi`)和MacOS(`pkg`)推荐使用安装包的方式来安装。作者当前MacOS安装包(`pkg`)安装过程如下图所示：

![](/images/goinstall-macos-1.png)

![](/images/goinstall-macos-2.png)

![](/images/goinstall-macos-3.png)



Go的开发包升级也是同样的过程。


# IDE开发环境安装

目前`Go`的`IDE`有两款比较流行，一款是`VSCode+Plugins`（免费），另一款是`JetBrains`公司的`Goland`（收费）。由于`JetBrains`也是`GoFrame`框架的赞助商，因此我们优先推荐使用`Goland`来作为开发IDE，下载及注册请参考网上教程（[百度](https://www.baidu.com/s?wd=goland%20安装) 或 [Google](https://www.google.com/search?q=goland+安装)）。

`JetBrains`的官方网站为：[https://www.jetbrains.com](https://www.jetbrains.com/?from=GoFrame)


## Goland的使用

我们来创建第一个`Go`程序吧，老规矩，上`hello world`。

### Step1. 打开IDE
![](/images/goland0.png)


### Step2. 创建项目
这里需要注意的是`Go`安装文件的路径(`SDK`)，[官方安装文档](https://golang.google.cn/doc/install)有详细说明，请仔细阅读。

其中的`Location`随意选择一个本地路径即可。

![](/images/goland2.png)


### Step3. 创建程序
新建一个`go`文件，叫做`hello.go`，并输入以下代码:
```go
package main

import "fmt"

func main() {
    fmt.Println("hello world!")
}

```
![](/images/goland3.png)


### Step4. 执行运行
菜单栏 `Run` - `Run` - `go build hello.go`。

![](/images/goland4.png)

![](/images/goland5.png)

![](/images/goland6.png)

恭喜你，第一个`Go`程序便成功了！






