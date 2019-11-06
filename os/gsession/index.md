
[TOC]

`GF`框架提供了完善的`Session`管理能力，由`gsession`模块实现。由于`Session`机制在`HTTP`服务中最常用，因此后续章节中将着重以`HTTP`服务为示例介绍`Session`的使用。

# 接口文档

https://godoc.org/github.com/gogf/gf/os/gsession

任何时候都可以通过`ghttp.Request`获取`Session`对象，因为`Cookie`和`Session`都是和请求会话相关，因此都属于`Request`的成员对象，并对外公开。`GF`框架的`Session`默认过期时间是`24小时`。

`SessionId`默认通过`Cookie`来传递，并且也支持客户端通过`Header`传递`SessionId`，`SessionId`的识别名称可以通过`ghttp.Server`的`SetSessionIdName`进行修改。

此外，需要说明的是，`Session`的操作是支持`并发安全`的，这也是框架在对`Session`的设计上不采用直接以`map`的形式操作数据的原因。在`HTTP`请求流程中，我们可以通过`ghttp.Request`对象来获取`Session`对象，并执行相应的数据操作。



# `gsession`模块

`Session`的管理功能由独立的`gsession`模块实现，并已完美整合到了`ghttp.Server`中。由于该模块是解耦独立的，因此可以应用到更多不同的场景中，例如：`TCP`通信、`gRPC`接口服务等等。

在`gsession`模块中有比较重要的三个对象/接口：
1. `gsession.Manager`：管理`Session`对象、`Storage`持久化存储对象、以及过期时间控制。
1. `gsession.Session`：单个`Session`会话管理对象，用于`Session`参数的增删查改等数据管理操作。
1. `gsession.Storage`：这是一个接口定义，用于`Session`对象的持久化存储、数据写入/读取、存活更新等操作，开发者可基于该接口实现自定义的持久化存储特性。该接口定义如下：
    https://github.com/gogf/gf/blob/master/os/gsession/gsession_storage.go
    ```go
    type Storage interface {
        // New creates a custom session id.
        // This function can be used for custom session creation.
        New(ttl time.Duration) (id string)

        // Get retrieves session value with given key.
        // It returns nil if the key does not exist in the session.
        Get(id string, key string) interface{}

        // GetMap retrieves all key-value pairs as map from storage.
        GetMap(id string) map[string]interface{}

        // GetSize retrieves the size of key-value pairs from storage.
        GetSize(id string) int

        // Set sets key-value session pair to the storage.
        // The parameter <ttl> specifies the TTL for the session id (not for the key-value pair).
        Set(id string, key string, value interface{}, ttl time.Duration) error

        // SetMap batch sets key-value session pairs with map to the storage.
        // The parameter <ttl> specifies the TTL for the session id(not for the key-value pair).
        SetMap(id string, data map[string]interface{}, ttl time.Duration) error

        // Remove deletes key with its value from storage.
        Remove(id string, key string) error

        // RemoveAll deletes all key-value pairs from storage.
        RemoveAll(id string) error

        // GetSession returns the session data as *gmap.StrAnyMap for given session id from storage.
        //
        // The parameter <ttl> specifies the TTL for this session, and it returns nil if the TTL is exceeded.
        // The parameter <data> is the current old session data stored in memory,
        // and for some storage it might be nil if memory storage is disabled.
        //
        // This function is called ever when session starts.
        GetSession(id string, ttl time.Duration, data *gmap.StrAnyMap) (*gmap.StrAnyMap, error)

        // SetSession updates the data for specified session id.
        // This function is called ever after session, which is changed dirty, is closed.
        // This copy all session data map from memory to storage.
        SetSession(id string, data *gmap.StrAnyMap, ttl time.Duration) error

        // UpdateTTL updates the TTL for specified session id.
        // This function is called ever after session, which is not dirty, is closed.
        UpdateTTL(id string, ttl time.Duration) error
    }
    ```

# 存储实现方式

`gsession`实现并为开发者提供了常见的三种`Session`存储实现方式：

1. 基于文件存储（默认）：单节点部署方式下比较高效的持久化存储方式；
1. 基于纯内存存储：性能最高效，但是无法持久化保存，重启即丢失；
1. 基于`Redis`存储：远程`Redis`节点存储`Session`数据，支持应用多节点部署；

三种方式各有优劣，详细介绍请查看后续章节。
