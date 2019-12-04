[TOC]

# 错误处理

## 结果为空判断

`GF`框架`v1.8.0`之前的版本通过判断返回的结果集是否为`nil`来判断是否查询结果为空。

从`v1.8.0`版本之后，对数据库操作的返回错误增加`sql.ErrNoRows`错误，因此可以通过判断返回的`error`结果是否等于`sql.ErrNoRows`来判断结果集是否为空(`nil`)。除此之外的其他错误为执行错误。

## 数据方法定义

首先我们需要对定义数据库读取的接口做一些说明。

我们来看一个方法定义：
```go
func GetPayedOrder() (gdb.Result, error) {
	r, err := g.DB().Table("order").Where("status", 1).All()
	if err != nil && err != sql.ErrNoRows {
		return nil, err
	}
	return r, nil
}
```
也可以这样：
```go
func GetPayedOrder() (r gdb.Result, err error) {
	r, err = g.DB().Table("order").Where("status", 1).All()
	if err != nil && err == sql.ErrNoRows {
		err = nil
	}
	return
}
```
这是Golang风格的方法定义，使用多返回值，并返回错误类型`error`。这种定义是Golang推荐的，并且也是严谨的方式，产生的错误努力往调用的上层抛，由调用方决定如何处理错误。当你无法决定如何处理产生的错误时，那就按照这样往上抛吧。

一般来说，应当由流程控制方法来决定如何处理该错误（例如：接口控制器），流程控制方法可以根据负责的业务需求决定，是否可以因为错误产生而终止执行流程，还是可以忽略错误（这时直接使用`_`符号忽略返回的错误变量即可）继续执行。

## 错误处理示例1
一些场景中，往往使用定义的结构体来管理数据集，例如这样的：
```go
type Order struct {
	Id         int
	Status     int
	PayedTime  *gtime.Time
	CreateTime *gtime.Time
}
```

```go
func GetPayedOrder() ([]*Order, error) {
	orders := ([]*Order)(nil)
	err := g.DB().Table("order").Where("status", 1).Structs(&orders)
	if err != nil && err != sql.ErrNoRows {
		return nil, err
	}
	return orders, nil
}
```
也可以这样：
```go
func GetPayedOrder() (orders []*Order, err error) {
	err = g.DB().Table("order").Where("status", 1).Structs(&orders)
	if err != nil && err == sql.ErrNoRows {
		err = nil
	}
	return
}
```
其中的数据集类型既可以使用`orders []*Order`，也可以使用`orders []Order`，两种定义方式没有太大的区分，喜好而定。

注意这里的`err`变量，当没有查询到数据时，是不会进行对象转换的，因此当没有数据时，会返回`sql.ErrNoRows`错误以便通知开发者没有数据。

这里还有一点细节，`orders := ([]*Order)(nil)`这样的初始化方式使得`orders`变量本身就是`nil`（预先没有执行对象初始化及内存分配），只有党查询到数据并且执行成功对象转换后`orders`才会有值（此时会自动分配内存），这是推荐的使用方式，这种方式对于GC来说比较友好。

## 错误处理示例2

单条记录的对象查询方式是这样的：
```go
func GetOrderInfo(id int) (*Order, error) {
	order := (*Order)(nil)
	err := g.DB().Table("order").Where("id", id).Struct(&order)
	if err != nil && err != sql.ErrNoRows {
		return nil, err
	}
	return order, nil
}
```
也可以这样：
```go
func GetOrderInfo(id int) (order *Order, err error) {
	err = g.DB().Table("order").Where("id", id).Struct(&order)
	if err != nil && err == sql.ErrNoRows {
		err = nil
	}
	return
}
```
同样的，只有当查询到数据记录之后，`order`才会执行内存分配，否则返回`nil`。







