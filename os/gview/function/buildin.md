
[TOC]




# 内置函数

## text
```go
{{.value | text}}
```
将`value`变量值去掉HTML标签，仅显示文字内容（并且去掉`script`标签）。
示例：
```go
{{"<div>测试</div>"|text}}
// 输出: 测试
```

## htmlencode/encode/html
```go
{{.value | htmlencode}}
{{.value | encode}}
{{.value | html}}
```
将`value`变量值进行html转义。
示例：
```go
{{"<div>测试</div>"|html}}
// 输出: &lt;div&gt;测试&lt;/div&gt;
```

## htmldecode/decode
```go
{{.value | htmldecode}}
{{.value | decode}}
```
将`value`变量值进行html反转义。
示例：
```go
{{"&lt;div&gt;测试&lt;/div&gt;" | htmldecode}}
// 输出: <div>测试</div>
```

## urlencode/urlquery/url
```go
{{.url | url}}
```
将`url`变量值进行`url`转义。
示例：
```go
{{"https://goframe.org" | url}}
// 输出: https%3A%2F%2Fgoframe.org
```

## urldecode
```go
{{.url | urldecode}}
```
将`url`变量值进行`url`反转义。
示例：
```go
{{"https%3A%2F%2Fgoframe.org"|urldecode}}
// 输出: https://goframe.org
```


## date
```go
{{.timestamp | date .format}}
{{date .format .timestamp}}
{{date .format}}
```
将`timestamp`时间戳变量进行时间日期格式化，类似PHP的`date`方法，`format`参数支持[PHP date](http://php.net/manual/en/function.date.php)方法格式，亦可参考【[gtime模块](os/gtime/index.md)】。

当`timestamp`变量为`空`(或者`0`)时，表示以当前时间作为时间戳参数执行打印。


示例：
```go
{{1540822968 | date "Y-m-d"}}
{{"1540822968" | date "Y-m-d H:i:s"}}
{{date "Y-m-d H:i:s"}}
// 输出: 
// 2018-10-29
// 2018-10-29 22:22:48
// 2018-12-05 10:22:00
```

## compare
```go
{{compare .str1 .str2}}
{{.str2 | compare .str1}}
```
将`str1`和`str2`进行字符串比较，返回值：
- 0 : `str1` == `str2`
- 1 : `str1` > `str2`
- -1 : `str1` < `str2`

示例：
```go
{{compare "A" "B"}}
{{compare "1" "2"}}
{{compare 2 1}}
{{compare 1 1}}
// 输出: 
// -1
// -1
// 1
// 0
```

## replace
```go
{{.str | replace .search .replace}}
{{replace .search .replace .str}}
```
将`str`中的`search`替换为`replace`。
示例：
```go
{{"I'm中国人" | replace "I'm" "我是"}}
// 输出:
// 我是中国人
```

## substr
```go
{{.str | substr .start .length}}
{{substr .start .length .str}}
```
将`str`从`start`索引位置（索引从0开始）进行字符串截取`length`，支持中文，类似PHP的`substr`函数。
示例：
```go
{{"我是中国人" | substr 2 -1}}
{{"我是中国人" | substr 2  2}}
// 输出:
// 中国人
// 中国
```

## strlimit
```go
{{.str | strlimit .length .suffix}}
```
将`str`字符串截取`length`长度，支持中文，超过长度则追加`suffix`字符串到末尾。
示例：
```go
{{"我是中国人" | strlimit 2  "..."}}
// 输出:
// 我是...
```

## concat
```go
{{concat .str1 .str2 .str3...}}
```
拼接字符串。
示例：
```go
{{concat "我" "是" "中" "国" "人"}}
// 输出:
// 我是中国人
```

## hidestr
```go
{{.str | hidestr .percent .hide}}
```
将`str`字符串按照`percent`百分比从字符串中间向两边隐藏字符（主要用于姓名、手机号、邮箱地址、身份证号等的隐藏），隐藏字符由`hide`变量定义。
支持中文，支持email格式。
示例：
```go
{{"热爱GF热爱生活" | hidestr 20  "*"}}
{{"热爱GF热爱生活" | hidestr 50  "*"}}
// 输出:
// 热爱GF*爱生活
// 热爱****生活
```

## highlight
```go
{{.str | highlight .key .color}}
```
将`str`字符串中的关键字`key`按照定义的颜色`color`进行前置颜色高亮。
示例：
```go
{{"热爱GF热爱生活" | highlight "GF" "red"}}
// 输出:
// 热爱<span style="color:red;">GF</span>热爱生活
```

## toupper/tolower
```go
{{.str | toupper}}
{{.str | tolower}}
```
将`str`字符串进行大小写转换。
示例：
```go
{{"gf" | toupper}}
{{"GF" | tolower}}
// 输出:
// GF
// gf
```

## nl2br
```go
{{.str | nl2br}}
```
将`str`字符串中的`\n/\r`替换为html中的`<br />`标签。
示例：
```go
{{"Go\nFrame" | nl2br}}
// 输出:
// Go<br />Frame
```

## dump
```go
{{dump .var}}
```
格式化打印变量，功能类似于`g.Dump`方法。
示例：
```go
gview.Assign("var", g.Map{
    "name" : "john",
})
```
```go
{{dump .var}}
// 输出:
// <!--
// {
//     name: "john"
// }
// -->
```