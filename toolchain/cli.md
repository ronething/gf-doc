[TOC]


# `gf tool chain`

从`GF2.0`版本开始提供了`gf`命令行开发辅助工具，将会随着框架发展不断完善，作为未来框架发展的一个重要组成部分，工具开源项目地址：https://github.com/gogf/gf-cli

工具安装成功后，可以通过`gf`或者`gf -h`查看所有支持的命令。复杂的命令可以通过`gf help COMMAND`或者`gf COMMAND -h`查看更详细的使用帮助信息，例如：`gf help gen`、`gf gen -h`。

> 当前帮助文档以`gf cli v0.3.0`版本为例进行介绍，内容信息可能会有滞后，最新的具体详细介绍请查看工具帮助信息。

```
$ gf
USAGE
    gf COMMAND [ARGUMENT] [OPTION]

COMMAND
    get        install or update GF to system in default...
    gen        automatically generate go files for ORM models...
    init       initialize an empty GF project at current working directory...
    help       show more information about a specified command
    pack       packing any file/directory to a resource file, or a go file
    build      cross-building go project for lots of platforms...
    update     update current gf binary to latest one (you may need root/admin permission)
    install    install gf binary to system (you may need root/admin permission)
    version    show version info

OPTION
    -?,-h      show this help or detail for specified command
    -v,-i      show version information

ADDITIONAL
    Use 'gf help COMMAND' or 'gf COMMAND -h' for detail about a command, which has '...' in the tail of their comments.
```


## `install` 工具安装

使用方式：`./gf install`

该命令往往是在`gf`命令行工具下载到本地后执行（注意执行权限），用于将`gf`命令安装到系统环境变量默认支持的目录路径中，以便于在系统任何的地方直接可以使用`gf`工具。

> 部分系统需要管理员权限支持。

## `update` 工具更新

使用方式：`gf update`

该命令用以检测`gf`命令行工具的版本，并自动执行版本更新。

> 部分系统需要管理员权限支持。

## `build` 交叉编译

使用方式：`gf build FILE [OPTION]`

目前支持绝大部分常见系统的直接交叉编译。

## `pack` 二进制打包

使用方式：`gf pack SRC DST`

该命令用以将任意的文件打包为二进制文件，或者`Go`代码文件，可将任意文件打包后随着可执行文件一同发布。

## `gen` 代码生成命令

使用方式：`gf gen model [PATH] [OPTION]`

`gen`命令用以自动化生成模板代码，目前支持从数据库直接生成模型文件。

## `get` 依赖包下载

使用方式：`gf get PACKAGE`

`gf get`命令和`go get`命令类似，内部自动提供了代理设置功能，并智能识别并设置最快的下载代理地址。



## `init` 项目初始化

使用方式：`gf init [NAME]`

我们可以使用`init`命令在当前目录生成一个示例的`GF`空框架项目，并可给定可选的项目名称参数。生成的项目目录结构仅供参考，根据业务项目具体情况可自行调整。

> `GF`框架开发推荐统一使用官方的`go module`特性进行依赖包管理，因此空项目根目录下也有一个`go.mod`文件。

## `version` 工具版本查看

使用方式：
- `gf -v`
- `gf -i`
- `gf version`

用以查看当前`gf`命令行工具版本信息。

## `help` 命令行帮助

使用方式：
- `gf -h`
- `gf -?`
- `gf help [COMMAND]`  
- `gf [COMMAND] -h`

任何不懂的，就用`help`看看吧。















