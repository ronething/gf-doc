# 复杂类型参数
`ghttp.Server`对象支持智能的参数类型解析（不区分请求提交方式及请求提交类型），数组参数可以通过例如`array[]=1&array[]=2&array[]=3`这样的方式传递，假如`v=1&v=2&v=3`这样的提交参数将会被解析为变量`v=3`。同时也支持`map[a]=1&map[b]=2&map[c]=3`这样的`map`提交方式，复杂类型解析请参考以下表格示例：

Parameter | Variable
---|---
`v=m&v=n` | `map[v:n]`
`v1=m&v2=n` | `map[v1:m v2:n]`
`v[]=m&v[]=n` | `map[v:[m n]]`
`v[a][]=m&v[a][]=n` | `map[v:map[a:[m n]]]`
`v[a]=m&v[b]=n` | `map[v:map[a:m b:n]]`
`v[a][a]=m&v[a][b]=n` | `map[v:map[a:map[a:m b:n]]]`
`v=m&v[a]=n` | `error`