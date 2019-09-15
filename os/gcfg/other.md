
# 独立使用cfg包进行配置文件管理

由于`gf`框架的所有组件都是低耦合设计的，因此可以单独使用`cfg`模块进行配置管理，这种方式相对来说比较简单。请查看以下示例。
为示例需要这里将同一个配置文件的内容拆分为两个配置文件，并且使用了不同的配置文件格式，```redis.toml```和```memcache.yml```，都放到一个配置文件管理目录 ```/var/www/config/```下进行统一管理(需要注意的是)。在平时的开发中，一般统一使用一种配置文件格式即可。

配置文件的内容如下：
```toml
# redis.toml - redis服务器配置信息
[[redis-cache]]
    db   = 0
    host = "192.168.0.100"
    port = 6379

[[redis-cache]]
    db   = 1
    host = "192.168.0.100"
    port = 6379

[[redis-disk]]
    db   = 0
    host = "192.168.0.100"
    port = 6380

[[redis-disk]]
    db   = 1
    host = "192.168.0.100"
    port = 6380
```

```yaml
# memcache.yml - memcache服务器配置信息
- host:   192.168.0.101
  port:   11211
  expire: 60
- host:   192.168.0.102
  port:   11211
  expire: 60
```
分别读取指定`redis`配置信息以及所有的`memcache`配置信息：
```go
package main

import (
    "fmt"
    "github.com/gogf/gf/os/gcfg"
)

func main() {
    c              := gcfg.New("/home/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/geg/os/gcfg")
    redisConfig    := c.GetArray("redis-cache", "redis.toml")
    memConfig      := c.GetArray("", "memcache.yml")
    fmt.Println(redisConfig)
    fmt.Println(memConfig)
}

// 输出：
// [map[db:0 host:192.168.0.100 port:6379] map[db:1 host:192.168.0.100 port:6379]]
// [map[port:11211 expire:60 host:192.168.0.101] map[expire:60 host:192.168.0.102 port:11211]]
```
以上示例代码中，我们可以通过```pattern```参数指定获取的参数项，如果获取配置文件的所有配置信息，那么```pattern```传递一个**空的字符串**即可。第二个文件参数指定需要操作的配置文件名称，不同的配置文件需要传递不同的配置文件参数，因此相对来说会比较繁琐。我们可以将所有这些基础配置都放到统一的配置文件中，这样我们可以只需要通过```pattern```参数来做区分不同的配置项即可，方便调用，因此我们可以做如下改进。

将所有配置放到```config.toml```配置文件中，此时```config.toml```的配置文件内容如下：
```toml
# redis配置
[[redis-cache]]
    db   = 0
    host = "192.168.0.100"
    port = 6379

[[redis-cache]]
    db   = 1
    host = "192.168.0.100"
    port = 6379

[[redis-disk]]
    db   = 0
    host = "192.168.0.100"
    port = 6380

[[redis-disk]]
    db   = 1
    host = "192.168.0.100"
    port = 6380

# memcache配置
[[memcache]]
    host   = "192.168.0.101"
    port   = 11211
    expire = 60

[[memcache]]
    host   = "192.168.0.102"
    port   = 11211
    expire = 60
```
那么假如我们通过以下程序获取memcache的配置：
```go
package main

import (
    "fmt"
    "github.com/gogf/gf/os/gcfg"
)

func main() {
    c := gcfg.New("/var/www/config")
    v := c.GetArray("memcache")
    fmt.Println(v)
}

// 输出：
// [map[port:11211 expire:60 host:192.168.0.101] map[expire:60 host:192.168.0.102 port:11211]]
```
可以看到，读取配置的方式简化了不少。因此在实际项目开发中，建议尽可能地将配置（特别是频繁使用的配置）统一存放到```config.toml```文件中进行统一管理，大大提高开发及维护效率。

此外有一点需要提醒的是，如果调用的方法与配置项的类型不匹配时，```pattern```将会检索失败，返回空值（对应类型的空值）。例如，假如配置项是map类型，而使用```GetArray```进行获取时，将会得到一个nil。
