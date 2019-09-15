Ajax分页与其他分页方式的区别在于，分页链接会使用Javascript方法来实现，该Javascript方法是分页方法，参数固定为该分页对应的分页URL地址。

完整示例如下：
```go
package main

import (
    "github.com/gogf/gf/os/gview"
    "github.com/gogf/gf/net/ghttp"
    "github.com/gogf/gf/util/gpage"
)

func main() {
    s := ghttp.GetServer()
    s.BindHandler("/page/ajax", func(r *ghttp.Request){
        page := gpage.New(100, 10, r.Get("page"), r.URL.String(), r.Router)
        page.EnableAjax("DoAjax")
        buffer, _ := gview.ParseContent(`
        <html>
            <head>
                <style>
                    a,span {padding:8px; font-size:16px;}
                    div{margin:5px 5px 20px 5px}
                </style>
                <script src="https://cdn.bootcss.com/jquery/2.0.3/jquery.min.js"></script>
                <script>
                function DoAjax(url) {
                     $.get(url, function(data,status) {
                         $("body").html(data);
                     });
                }
                </script>
            </head>
            <body>
                <div>{{.page}}</div>
            </body>
        </html>
        `, g.Map{
            "page" : page.GetContent(1),
        })
        r.Response.Write(buffer)
    })
    s.SetPort(8199)
    s.Run()
}
```

在该示例中，我们定义了一个```DoAjax(url)```方法用来执行分页操作，为演示需要它逻辑很简单，会加载指定分页页面的内容并覆盖掉当前页面的分页内容。
```javascript
function DoAjax(url) {
     $.get(url, function(data,status) {
         $("body").html(data);
     });
}
```