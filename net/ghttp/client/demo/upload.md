# 文件上传

`gf`的HTTP客户端封装并极大简化了文件上传功能，直接上例子：

1. 客户端

    https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/upload/client.go

    ```go
    package main

    import (
        "fmt"
        "github.com/gogf/gf/os/glog"
        "github.com/gogf/gf/net/ghttp"
    )

    func main() {
        path := "/home/john/Workspace/Go/github.com/gogf/gf/version.go"
        r, e := ghttp.Post("http://127.0.0.1:8199/upload", "name=john&age=18&upload-file=@file:" + path)
        if e != nil {
            glog.Error(e)
        } else {
            fmt.Println(r.ReadAllString())
            r.Close()
        }
    }
    ```

    注意到了吗？文件上传参数格式使用了 `参数名=@file:文件路径` ，HTTP客户端将会自动解析**文件路径**对应的文件内容并读取提交给服务端。原本复杂的文件上传操作被`gf`进行了封装处理，用户只需要使用 `@file:+文件路径` 来构成参数值即可。
    > 其中，`文件路径`请使用本地文件绝对路径。
1. 服务端

    https://github.com/gogf/gf/blob/master/.example/net/ghttp/client/upload/server.go

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
