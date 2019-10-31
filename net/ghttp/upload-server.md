# 文件上传

文件上传的功能比较常用，我们来看一个使用`gf`框架的`Web Server`服务端处理文件上传的例子：

```go
package main

import (
	"github.com/gogf/gf/frame/g"
	"github.com/gogf/gf/net/ghttp"
	"github.com/gogf/gf/os/gfile"
	"io"
)

// Upload uploads file to /tmp .
func Upload(r *ghttp.Request) {
	f, h, e := r.FormFile("upload-file")
	if e != nil {
		r.Response.Write(e)
	}
	defer f.Close()
	savePath := "/tmp/" + gfile.Basename(h.Filename)
	file, err := gfile.Create(savePath)
	if err != nil {
		r.Response.Write(err)
		return
	}
	defer file.Close()
	if _, err := io.Copy(file, f); err != nil {
		r.Response.Write(err)
		return
	}
	r.Response.Write("upload successfully")
}

// UploadShow shows uploading page.
func UploadShow(r *ghttp.Request) {
	r.Response.Write(`
    <html>
    <head>
        <title>GF UploadFile Demo</title>
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

func main() {
	s := g.Server()
	s.Group("/upload", func(g *ghttp.RouterGroup) {
		g.ALL("/", Upload)
		g.ALL("/show", UploadShow)
	})
	s.SetPort(8199)
	s.Run()
}
```

访问  http://127.0.0.1:8199/upload/show  选择需要上传的文件，提交之后可以看到文件上传成功到服务器上。

服务端处理文件上传比较简单，但是需要注意以下几点：
1. 服务端在上传处理中需要使用`defer f.Close()`关闭掉临时上传文件指针；
1. 将上传文件转存到其他文件目录时，需要通过`gfile.Create`新建文件，并通过`io.Copy(file, f)`将文件内存写入到新文件中，这里使用的是流式读写方式，就算上传的文件容量有数G，也丝毫不会影响服务端的内存占用；
1. 需要使用`defer file.Close()`关闭创建的文件指针；

HTTP客户端上传文件的例子请参考后续的【[HTTP客户端-使用示例](net/ghttp/client/example.md)】章节

