[TOC]


# 配置引入

由于`boot`和`router`包使用了`init`包初始化方式来进行相关配置，因此我们需要使用：
```go
import _ "PATH"
```
方式来引入。

# main包
当然每个项目至少存在一个`package main`，用于程序的入口执行。

`/main.go`
```go
package main

import (
	_ "github.com/gogf/gf-demos/boot"
	_ "github.com/gogf/gf-demos/router"
	"github.com/gogf/gf/frame/g"
)

func main() {
	g.Server().Run()
}
```

# 编译运行
我们可以使用IDE执行运行，也可以使用以下命令编译运行。
```shell
$ go build main.go
$ ./main
```
执行后，注册的路由列表如下：
```
  SERVER  | ADDRESS | DOMAIN  | METHOD | P |        ROUTE        |                                   HANDLER                                    | HOOK  
|---------|---------|---------|--------|---|---------------------|------------------------------------------------------------------------------|------|
  default | :8199   | default | ALL    | 2 | /user/checknickname | github.com/gogf/gf-demos/app/controller/user.(*ctl_Controller).CheckNickName |       
|---------|---------|---------|--------|---|---------------------|------------------------------------------------------------------------------|------|
  default | :8199   | default | ALL    | 2 | /user/checkpassport | github.com/gogf/gf-demos/app/controller/user.(*ctl_Controller).CheckPassport |       
|---------|---------|---------|--------|---|---------------------|------------------------------------------------------------------------------|------|
  default | :8199   | default | ALL    | 2 | /user/issignedin    | github.com/gogf/gf-demos/app/controller/user.(*ctl_Controller).IsSignedIn    |       
|---------|---------|---------|--------|---|---------------------|------------------------------------------------------------------------------|------|
  default | :8199   | default | ALL    | 2 | /user/signin        | github.com/gogf/gf-demos/app/controller/user.(*ctl_Controller).SignIn        |       
|---------|---------|---------|--------|---|---------------------|------------------------------------------------------------------------------|------|
  default | :8199   | default | ALL    | 2 | /user/signout       | github.com/gogf/gf-demos/app/controller/user.(*ctl_Controller).SignOut       |       
|---------|---------|---------|--------|---|---------------------|------------------------------------------------------------------------------|------|
  default | :8199   | default | ALL    | 2 | /user/signup        | github.com/gogf/gf-demos/app/controller/user.(*ctl_Controller).SignUp        |       
|---------|---------|---------|--------|---|---------------------|------------------------------------------------------------------------------|------|
```

# 接口测试

我们通过`curl`命令来对其中两个接口执行简单的测试。

## 1. 用户注册 - `/user/signup`
注册一个账号`test001`，昵称为`john`，密码为`123456`。
```shell
$ curl -d 'nickname=john&passport=test001&password=123456&password2=123456' http://127.0.0.1:8199/user/signup
{"data":null,"err":0,"msg":"ok"}
```
我们再次使用刚才的信息注册一次试试。
```shell
$ curl -d 'nickname=john&passport=test001&password=123456&password2=123456' http://127.0.0.1:8199/user/signup
{"data":null,"err":1,"msg":"账号 test001 已经存在"}
```
可以看到注册失败了，相同名称只能注册一个账号。

## 2.用户登录 - `/user/signin`
我们用刚才注册的账号登录。
```shell
$ curl -d 'passport=test001&password=123456' http://127.0.0.1:8199/user/signin
{"data":null,"err":0,"msg":"ok"}
```





