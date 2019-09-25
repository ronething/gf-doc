[TOC]

# `gres`资源管理

`GF`框架从`v1.9`版本开始提供了资源管理的特性，由`gres`模块实现，该模块具有以下特点：
- 可将任意的文件打包为`Go`内容，支持开发者自定义加解密；
- 资源管理器内容完全基于内存，并且内容只读，无法动态修改；
- 资源管理器默认整合支持到了WebServer、配置管理、模板引擎中；
- 任意文件如网站静态文件、配置文件等可编译到二进制文件中，也可编译到发布的可执行文件中；
- 开发者可只需编译发布一个可执行文件，除了方便了软件分发，也为保护软件知识产权内容提供了可能；


**使用方式**：
```go
import "github.com/gogf/gf/os/gres"
```

**接口文档**： 

https://godoc.org/github.com/gogf/gf/os/gres

```go
func Add(content []byte, prefix ...string) error
func Contains(path string) bool
func Dump()
func GetContent(path string) []byte
func IsEmpty() bool
func Get(path string) *File
func GetWithIndex(path string, indexFiles []string) *File
func ScanDir(path string, pattern string, recursive ...bool) []*File
func ScanDirFile(path string, pattern string, recursive ...bool) []*File
func Load(path string, prefix ...string) error

func Pack(srcPaths string, keyPrefix ...string) ([]byte, error)
func PackToGoFile(srcPath, goFilePath, pkgName string, keyPrefix ...string) error
func PackToFile(srcPaths, dstPath string, keyPrefix ...string) error
func Unpack(path string) ([]*File, error)
func UnpackContent(content []byte) ([]*File, error)
type File
    func (f *File) Close() error
    func (f *File) Content() []byte
    func (f *File) FileInfo() os.FileInfo
    func (f *File) Name() string
    func (f *File) Open() (io.ReadCloser, error)
    func (f *File) Read(b []byte) (n int, err error)
    func (f *File) Readdir(count int) ([]os.FileInfo, error)
    func (f *File) Seek(offset int64, whence int) (int64, error)
    func (f *File) Stat() (os.FileInfo, error)
type Resource
    func Instance(name ...string) *Resource
    func New() *Resource
    func (r *Resource) Add(content []byte, prefix ...string) error
    func (r *Resource) Contains(path string) bool
    func (r *Resource) Dump()
    func (r *Resource) Get(path string) *File
    func (r *Resource) GetContent(path string) []byte
    func (r *Resource) GetWithIndex(path string, indexFiles []string) *File
    func (r *Resource) IsEmpty() bool
    func (r *Resource) Load(path string, prefix ...string) error
    func (r *Resource) ScanDir(path string, pattern string, recursive ...bool) []*File
    func (r *Resource) ScanDirFile(path string, pattern string, recursive ...bool) []*File
```
1. 通过`Pack*`/`Unpack*`方法可以实现对任意文件的打包/解包功能，可以打包到二进制文件或者Go代码文件；
1. 资源管理由`Resource`对象实现，可实现对打包内容的添加，文件的检索查找，以及对目录的遍历等功能；
1. 资源文件由`File`对象实现，该文件对象和`os.File`文件对象类似，并且该对象实现了`http.File`接口；
1. `ScanDir`用于针对于特定目录下的文件/目录检索，并且支持递归检索；
1. `ScanDirFile`用于针对于特定目录下的文件检索，并且支持递归检索；
1. 通过`Dump`方法在终端打印出`Resource`资源对象所有的文件列表，资源管理器中的文件分隔符号统一为`/`；
1. 此外，`gres`资源管理模块提供了默认的`Resource`对象，并通过包方法提供了对该默认对象的操作；

## 文件/目录打包

开发者对文件/目录的打包可以自定义通过`Pack*`方法实现，并通过`Unpack*`方法自行对二进制文件进行打包；或者通过`import`方式导入打包的`Go`资源文件。

开发者也可以通过`gf`命令行工具的`pack`命令实现对任意文件的打包，具体请查看[CLI工具链](toolchain/cli.md)章节。通过命令行工具的`pack`命令也是比较推荐的打包方式。

> 可以通过`Pack`方法直接生成打包后的`[]byte`内容，开发者可以对该内容进行自定义加密以额外增加文件内容的安全性。随后可以在程序中对该`[]byte`解密后，并通过`UnpackContent`方法可以对该内容进行解包，使用该打包内容。


## 打包生成`Go`文件

这种是比较常用的方式，我们这里直接使用`gf`工具来进行打包，不添加自定义的加密。

### 1. `gf pack`生成`Go`文件

比较推荐的方式是将`Go`文件直接生成到`boot`启动目录，并设置生成`Go`文件的包名为`boot`，这样该资源文件将会被自动引入到项目中。我们将项目的`config,public,template`三个目录的文件打包到`Go`文件，打包命令为：`gf pack config,public,template boot/data.go -n boot`

生成的`Go`文件内容类似于：
```go
import "github.com/gogf/gf/os/gres"

func init() {
	if err := gres.Add([]byte("\x50\x4b\x03\x04\x14\x00\x08...")); err != nil {
		panic(err)
	}
}
```

### 2. 使用打包的`Go`资源文件

由于项目的`main`入口程序文件会引入`boot`包，例如像这样（`module`名称为`my-app`）：
```go
import (
	_ "my-app/boot"
)
```
随后可以在项目的任何地方使用`gres`模块来访问打包的资源文件。

> 如果使用`GF`推荐的[项目目录结构](start/index.md)，在目录结构中会存在`boot`目录（对应包名也是`boot`），用于程序启动设置。因此如果将Go资源文件打包到`boot`目录下，那么将会被自动编译进可执行文件中。

### 3. 打印资源管理文件列表
可以通过`Dump()`方法打印出当前资源管理器中所有的文件列表，输出内容类似于：
```html
2019-09-15T13:36:28+00:00   0.00B config
2019-07-27T07:26:12+00:00   1.34K config/config.toml
2019-09-15T13:36:28+00:00   0.00B public
2019-06-25T17:03:56+00:00   0.00B public/resource
2018-12-04T12:50:16+00:00   0.00B public/resource/css
2018-12-17T12:54:26+00:00   0.00B public/resource/css/document
2018-12-17T12:54:26+00:00   4.20K public/resource/css/document/style.css
2018-08-24T01:46:58+00:00  32.00B public/resource/css/index.css
2019-05-23T03:51:24+00:00   0.00B public/resource/image
2018-08-20T05:02:08+00:00  24.01K public/resource/image/cover.png
2019-05-23T03:51:24+00:00   4.19K public/resource/image/favicon.ico
2018-08-23T01:44:50+00:00   4.19K public/resource/image/gf.ico
2018-12-04T13:04:34+00:00   0.00B public/resource/js
2019-06-27T11:06:12+00:00   0.00B public/resource/js/document
2019-06-27T11:06:12+00:00  11.67K public/resource/js/document/index.js
2019-09-15T13:36:28+00:00   0.00B template
2019-02-02T09:08:56+00:00   0.00B template/document
2018-12-04T12:49:08+00:00   0.00B template/document/include
2018-12-04T12:49:08+00:00 329.00B template/document/include/404.html
2019-03-06T01:52:56+00:00   3.42K template/document/index.html
...
```

## 自定义打包/解包

### 自定义打包示例

我们将项目根目录下的`public`和`config`目录打包为`data.bin`二进制文件，并通过`gaes`加密算法对生成的二进制内容进行加密。

```go
package main

import (
	"github.com/gogf/gf/crypto/gaes"
	"github.com/gogf/gf/os/gfile"
	"github.com/gogf/gf/os/gres"
)

var (
	CryptoKey = []byte("x76cgqt36i9c863bzmotuf8626dxiwu0")
)

func main() {
	binContent, err := gres.Pack("public,config")
	if err != nil {
		panic(err)
	}
	binContent, err = gaes.Encrypt(binContent, CryptoKey)
	if err != nil {
		panic(err)
	}
	if err := gfile.PutBytes("data.bin", binContent); err != nil {
		panic(err)
	}
}
```

### 自定义解包示例

我们使用将刚才打包生成的`data.bin`，需要解密和解包两步操作。

```go
package main

import (
	"github.com/gogf/gf/crypto/gaes"
	"github.com/gogf/gf/os/gfile"
	"github.com/gogf/gf/os/gres"
)

var (
	CryptoKey = []byte("x76cgqt36i9c863bzmotuf8626dxiwu0")
)

func main() {
	binContent := gfile.GetBytes("data.bin")
	binContent, err := gaes.Decrypt(binContent, CryptoKey)
	if err != nil {
		panic(err)
	}
	if err := gres.Add(binContent); err != nil {
		panic(err)
	}
	gres.Dump()
}
```
最后，我们使用`gres.Dump()`打印出添加成功的文件列表。









