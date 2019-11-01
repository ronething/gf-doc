# 错误提示

任何时候，我们都可以通过`gvalid.SetDefaultErrorMsgs`方法来批量设置默认的错误提示信息（特别是针对多语言环境中），但是需要注意的是，修改是全局变化的，请注意可能会对其他模块校验信息的影响。通常建议为针对特定的校验单独配置不同的校验错误提示信息。


修改默认错误信息方法：
https://godoc.org/github.com/gogf/gf/util/gvalid
```go
func SetDefaultErrorMsgs(msgs map[string]string)
```

默认的错误信息如下：

```go
var defaultMessages = map[string]string {
    "required"             : "字段不能为空",
    "required-if"          : "字段不能为空",
    "required-unless"      : "字段不能为空",
    "required-with"        : "字段不能为空",
    "required-with-all"    : "字段不能为空",
    "required-without"     : "字段不能为空",
    "required-without-all" : "字段不能为空",
    "date"                 : "日期格式不正确",
    "date-format"          : "日期格式不正确",
    "email"                : "邮箱地址格式不正确",
    "phone"                : "手机号码格式不正确",
    "telephone"            : "电话号码格式不正确",
    "passport"             : "账号格式不合法，必需以字母开头，只能包含字母、数字和下划线，长度在6~18之间",
    "password"             : "密码格式不合法，密码格式为任意6-18位的可见字符",
    "password2"            : "密码格式不合法，密码格式为任意6-18位的可见字符，必须包含大小写字母和数字",
    "password3"            : "密码格式不合法，密码格式为任意6-18位的可见字符，必须包含大小写字母、数字和特殊字符",
    "postcode"             : "邮政编码不正确",
    "id-number"            : "身份证号码不正确",
    "qq"                   : "QQ号码格式不正确",
    "ip"                   : "IP地址格式不正确",
    "ipv4"                 : "IPv4地址格式不正确",
    "ipv6"                 : "IPv6地址格式不正确",
    "mac"                  : "MAC地址格式不正确",
    "url"                  : "URL地址格式不正确",
    "domain"               : "域名格式不正确",
    "length"               : "字段长度为:min到:max个字符",
    "min-length"           : "字段最小长度为:min",
    "max-length"           : "字段最大长度为:max",
    "between"              : "字段大小为:min到:max",
    "min"                  : "字段最小值为:min",
    "max"                  : "字段最大值为:max",
    "json"                 : "字段应当为JSON格式",
    "xml"                  : "字段应当为XML格式",
    "array"                : "字段应当为数组",
    "integer"              : "字段应当为整数",
    "float"                : "字段应当为浮点数",
    "boolean"              : "字段应当为布尔值",
    "same"                 : "字段值不合法",
    "different"            : "字段值不合法",
    "in"                   : "字段值不合法",
    "not-in"               : "字段值不合法",
    "regex"                : "字段值不合法",
}
```
