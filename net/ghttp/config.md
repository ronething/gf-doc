
[TOC]

# 配置管理对象
https://godoc.org/github.com/gogf/gf/net/ghttp#ServerConfig
```go
// HTTP Server 设置结构体，静态配置
type ServerConfig struct {
    // 底层http对象配置
    Addr              string                // 监听IP和端口，监听本地所有IP使用":端口"(支持多个地址，使用","号分隔)
    HTTPSAddr         string                // HTTPS服务监听地址(支持多个地址，使用","号分隔)
    HTTPSCertPath     string                // HTTPS证书文件路径
    HTTPSKeyPath      string                // HTTPS签名文件路径
    Handler           http.Handler          // 默认的处理函数
    ReadTimeout       time.Duration         // 读取超时
    WriteTimeout      time.Duration         // 写入超时
    IdleTimeout       time.Duration         // 等待超时
    MaxHeaderBytes    int                   // 最大的header长度
    TLSConfig         tls.Config

    // 静态文件配置
    IndexFiles        []string              // 默认访问的文件列表
    IndexFolder       bool                  // 如果访问目录是否显示目录列表
    ServerAgent       string                // Server Agent
    ServerRoot        string                // 服务器服务的本地目录根路径(检索优先级比StaticPaths低)
    SearchPaths       []string              // 静态文件搜索目录(包含ServerRoot，按照优先级进行排序)
    StaticPaths       []staticPathItem      // 静态文件目录映射(按照优先级进行排序)
    FileServerEnabled bool                  // 是否允许静态文件服务(通过静态文件服务方法调用自动识别)

    // COOKIE
    CookieMaxAge      int                   // Cookie有效期
    CookiePath        string                // Cookie有效Path(注意同时也会影响SessionID)
    CookieDomain      string                // Cookie有效Domain(注意同时也会影响SessionID)

    // SESSION
    SessionMaxAge     int                   // Session有效期
    SessionIdName     string                // SessionId名称

    // IP访问控制
    DenyIps           []string              // 不允许访问的ip列表，支持ip前缀过滤，如: 10 将不允许10开头的ip访问
    AllowIps          []string              // 仅允许访问的ip列表，支持ip前缀过滤，如: 10 将仅允许10开头的ip访问

    // 路由访问控制
    DenyRoutes        []string              // 不允许访问的路由规则列表
    Rewrites          map[string]string     // URI Rewrite重写配置

    // 日志配置
    LogPath           string                // 存放日志的目录路径(默认为空，表示不写文件)
    LogHandler        LogHandler            // 自定义日志处理回调方法(默认为空)
    LogStdPrint       bool                  // 是否打印日志到终端(默认开启)
    ErrorLogEnabled   bool                  // 是否开启error log(默认开启)
    AccessLogEnabled  bool                  // 是否开启access log(默认关闭)

    // 其他设置
    NameToUriType     int                   // 服务注册时对象和方法名称转换为URI时的规则
    GzipContentTypes  []string              // 允许进行gzip压缩的文件类型
    DumpRouteMap      bool                  // 是否在程序启动时默认打印路由表信息
    RouterCacheExpire int                   // 路由检索缓存过期时间(秒)
}
```


# 配置管理方法
https://godoc.org/github.com/gogf/gf/net/ghttp#Server

Web Server的配置比较丰富，所有的配置均可在创建`ghttp.Server`对象后使用`SetConfig`方法进行统一配置；也可以使用Server对象的`Set*/Enable*`方法进行特定配置的设置。主要注意的是，配置项在Server执行`Start`之后便不能再修改。

# 常用配置介绍

## 关闭路由表打印

在WebServer启动的时候默认会打印出当前注册的所有路由信息(包括HOOK注册)，这对于开发者来说非常有用。如果不想启动时打印路由表信息，可以通过以下方式关闭：
```go
g.Server().SetDumpRouteMap(false)
```
此外，我们也可以通过以下方式获取路由表信息(不自动打印)，随后我们可以自定义处理：
```go
routes := g.Server().GetRouteMap()
```








