# gconv

`gf`框架提供了非常强大的类型转换包```gconv```，可以实现将任何数据类型转换为指定的数据类型，对常用基本数据类型之间的无缝转换，同时也支持任意类型到`struct`对象的转换。由于`gconv`模块内部大量使用了断言而非反射(仅`struct`转换使用到了反射)，因此执行的效率非常高。

**使用方式**：
```go
import "github.com/gogf/gf/util/gconv"
```

**接口文档**： 

https://godoc.org/github.com/gogf/gf/util/gconv
```go
// 基本类型
func Bool(i interface{}) bool
func Float32(i interface{}) float32
func Float64(i interface{}) float64
func Int(i interface{}) int
func Int16(i interface{}) int16
func Int32(i interface{}) int32
func Int64(i interface{}) int64
func Int8(i interface{}) int8
func String(i interface{}) string
func Uint(i interface{}) uint
func Uint16(i interface{}) uint16
func Uint32(i interface{}) uint32
func Uint64(i interface{}) uint64
func Uint8(i interface{}) uint8

// slice类型
func Bytes(i interface{}) []byte
func Ints(i interface{}) []int
func Floats(i interface{}) []float64
func Strings(i interface{}) []string
func Interfaces(i interface{}) []interface{}
func Structs(params interface{}, pointer interface{}, mapping ...map[string]string) (err error)
func StructsDeep(params interface{}, pointer interface{}, mapping ...map[string]string) (err error)

// 时间类型
func Time(i interface{}, format ...string) time.Time
func TimeDuration(i interface{}) time.Duration

// Map转换, 支持的类型包括：任意map或struct
func Map(value interface{}, tags ...string) map[string]interface{}
func MapDeep(value interface{}, tags ...string) map[string]interface{}

// 对象转换
func Struct(params interface{}, pointer interface{}, mapping ...map[string]string) error
func StructDeep(params interface{}, pointer interface{}, mapping ...map[string]string) error

// 根据类型名称执行基本类型转换(非struct转换)
func Convert(i interface{}, t string, extraParams ...interface{}) interface{}
```


## 基准性能测试

测试转换变量值为`123456789`，类型`int`。

```shell
john@john-B85M:~/Workspace/Go/GOPATH/src/github.com/gogf/gf/util/gconv$ go test *.go -bench=".*" -benchmem
goos: linux
goarch: amd64
BenchmarkString-4               20000000                71.8 ns/op            24 B/op          2 allocs/op
BenchmarkInt-4                  100000000               22.2 ns/op             8 B/op          1 allocs/op
BenchmarkInt8-4                 100000000               24.5 ns/op             8 B/op          1 allocs/op
BenchmarkInt16-4                50000000                23.8 ns/op             8 B/op          1 allocs/op
BenchmarkInt32-4                100000000               24.1 ns/op             8 B/op          1 allocs/op
BenchmarkInt64-4                100000000               21.7 ns/op             8 B/op          1 allocs/op
BenchmarkUint-4                 100000000               22.2 ns/op             8 B/op          1 allocs/op
BenchmarkUint8-4                50000000                25.6 ns/op             8 B/op          1 allocs/op
BenchmarkUint16-4               50000000                32.1 ns/op             8 B/op          1 allocs/op
BenchmarkUint32-4               50000000                27.7 ns/op             8 B/op          1 allocs/op
BenchmarkUint64-4               50000000                28.1 ns/op             8 B/op          1 allocs/op
BenchmarkFloat32-4              10000000               155 ns/op              24 B/op          2 allocs/op
BenchmarkFloat64-4              10000000               177 ns/op              24 B/op          2 allocs/op
BenchmarkTime-4                  5000000               240 ns/op              72 B/op          4 allocs/op
BenchmarkTimeDuration-4         50000000                26.2 ns/op             8 B/op          1 allocs/op
BenchmarkBytes-4                10000000               149 ns/op             128 B/op          3 allocs/op
BenchmarkStrings-4              10000000               223 ns/op              40 B/op          3 allocs/op
BenchmarkInts-4                 20000000                55.0 ns/op            16 B/op          2 allocs/op
BenchmarkFloats-4               10000000               186 ns/op              32 B/op          3 allocs/op
BenchmarkInterfaces-4           20000000                66.6 ns/op            24 B/op          2 allocs/op
PASS
ok      command-line-arguments  35.356s

```
