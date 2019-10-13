# 文件上传

文件上传的功能比较常用，我们来看一个使用gf框架的Web Server服务端处理文件上传的例子：

```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/os/gfile"
    "github.com/gogf/gf/net/ghttp"
)

// 执行文件上传处理，上传到系统临时目录 /tmp
func Upload(r *ghttp.Request) {
    if f, h, e := r.FormFile("upload-file"); e == nil {
        defer f.Close()
        name   := gfile.Basename(h.Filename)
        buffer := make([]byte, h.Size)
        f.Read(buffer)
        gfile.PutBytes("/tmp/" + name, buffer)
        r.Response.Write(name + " uploaded successly")
    } else {
        r.Response.Write(e.Error())
    }
}

// 展示文件上传页面
func UploadShow(r *ghttp.Request) {
    r.Response.Write(`
    <html>
    <head>
        <title>上传文件</title>
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
    s.BindHandler("/upload",      Upload)
    s.BindHandler("/upload/show", UploadShow)
    s.SetPort(8199)
    s.Run()
}
```

访问  http://127.0.0.1:8199/upload/show  选择需要上传的文件，提交之后可以看到文件上传成功到服务器上。

服务端处理文件上传比较简单，**但是需要注意的是，服务端在上传处理中需要使用```f.Close()```关闭掉临时上传文件指针**。

HTTP客户端上传文件的例子请参考后续的【[HTTP客户端-文件上传](net/ghttp/client.md)】章节

