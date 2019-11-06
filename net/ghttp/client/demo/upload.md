[TOC]

# 文件上传

`gf`支持非常方便的表单文件上传功能，并且HTTP客户端对上传功能进行了必要的封装并极大简化了上传功能调用。

## 服务端

https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/upload/server.go

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/os/gfile"
	"io"
)

// Upload uploads files to /tmp .
func Upload(r *ghttp.Request) {
	saveDir := "/tmp/"
	for _, item := range r.GetMultipartFiles("upload-file") {
		file, err := item.Open()
		if err != nil {
			r.Response.Write(err)
			return
		}
		defer file.Close()

		f, err := gfile.Create(saveDir + gfile.Basename(item.Filename))
		if err != nil {
			r.Response.Write(err)
			return
		}
		defer f.Close()

		if _, err := io.Copy(f, file); err != nil {
			r.Response.Write(err)
			return
		}
	}
	r.Response.Write("upload successfully")
}

// UploadShow shows uploading simgle file page.
func UploadShow(r *ghttp.Request) {
	r.Response.Write(`
    <html>
    <head>
        <title>GF Upload File Demo</title>
    </head>
        <body>
            <form enctype="multipart/form-data" action="/upload" method="post">
                <input type="file" name="upload-file" />
                <input type="submit" value="upload" />
            </form>
        </body>
    </html>
    `)
}

// UploadShowBatch shows uploading multiple files page.
func UploadShowBatch(r *ghttp.Request) {
	r.Response.Write(`
    <html>
    <head>
        <title>GF Upload Files Demo</title>
    </head>
        <body>
            <form enctype="multipart/form-data" action="/upload" method="post">
                <input type="file" name="upload-file" />
                <input type="file" name="upload-file" />
                <input type="submit" value="upload" />
            </form>
        </body>
    </html>
    `)
}

func main() {
	s := g.Server()
	s.Group("/upload", func(g *ghttp.RouterGroup) {
		g.ALL("/", Upload)
		g.ALL("/show", UploadShow)
		g.ALL("/batch", UploadShowBatch)
	})
	s.SetPort(8199)
	s.Run()
}
```
该服务端提供了3个接口：
1. http://127.0.0.1:8199/upload/show  地址用于展示单个文件上传的H5页面；
1. http://127.0.0.1:8199/upload/batch  地址用于展示多个文件上传的H5页面；
1. http://127.0.0.1:8199/upload  接口用于真实的表单文件上传，该接口同时支持单个文件或者多个文件上传；

我们这里访问  http://127.0.0.1:8199/upload/show  选择需要上传的单个文件，提交之后可以看到文件上传成功到服务器上。

服务端处理文件上传比较简单，但是需要注意以下几点：
1. 服务端在上传处理中需要使用`defer file.Close()`关闭掉临时上传文件指针；
1. 将上传文件转存到其他文件目录时，需要通过`gfile.Create`新建文件，并通过`io.Copy(file, f)`将文件内存写入到新文件中，由于这里使用的是流式读写方式，因此对于服务端的内存占用会比较友好；
1. 需要使用`defer f.Close()`关闭创建的文件指针；


## 客户端
    
### 单文件上传

https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/upload/client.go

```go
package main

import (
    "fmt"

    "github.com/gogf/gf/net/ghttp"
    "github.com/gogf/gf/os/glog"
)

func main() {
    path := "/home/john/Workspace/Go/github.com/gogf/gf/version.go"
    r, e := ghttp.Post("http://127.0.0.1:8199/upload", "upload-file=@file:"+path)
    if e != nil {
        glog.Error(e)
    } else {
        fmt.Println(string(r.ReadAll()))
        r.Close()
    }
}
```

注意到了吗？文件上传参数格式使用了 `参数名=@file:文件路径` ，HTTP客户端将会自动解析**文件路径**对应的文件内容并读取提交给服务端。原本复杂的文件上传操作被`gf`进行了封装处理，用户只需要使用 `@file:+文件路径` 来构成参数值即可。其中，`文件路径`请使用本地文件绝对路径。

首先运行服务端程序之后，我们再运行这个上传客户端（注意修改上传的文件路径为本地真实文件路径），执行后可以看到文件被成功上传到服务器的指定路径下。

### 多文件上传

https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/upload-batch/client.go

```go
package main

import (
	"fmt"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/os/glog"
)

func main() {
	path1 := "/Users/john/Pictures/logo1.png"
	path2 := "/Users/john/Pictures/logo2.png"
	r, e := ghttp.Post(
		"http://127.0.0.1:8199/upload",
		fmt.Sprintf(`upload-file=@file:%s&upload-file=@file:%s`, path1, path2),
	)
	if e != nil {
		glog.Error(e)
	} else {
		fmt.Println(string(r.ReadAll()))
		r.Close()
	}
}
```
可以看到，多个文件上传提交参数格式为`参数名=@file:xxx&参数名=@file:xxx...`，也可以使用`参数名[]=@file:xxx&参数名[]=@file:xxx...`的形式。

首先运行服务端程序之后，我们再运行这个上传客户端（注意修改上传的文件路径为本地真实文件路径），执行后可以看到文件被成功上传到服务器的指定路径下。