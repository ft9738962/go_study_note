## func的综述

## func的声明
每一个函数都包含一个名字、一个形参列表、一个可选的返回列表以及函数体
```go
func name(parameter-list) (result-list) {
    body
}
```
其中形参列表paramter-list：
- 指定了一组变量的参数名和参数类型
  
返回列表result-list：
- 指定了函数返回值的类型
- 当返回一个未命名的返回值或没有返回值的，可省略圆括号

当有返回列表时，必须显式以return语句结束，除非函数明确不会走完整个执行流程

当返回列表定义了返回值的名称时，可以使用裸返回
```go
func name(a int) (b int) {
    b = a + 1
    returns
}
```
裸返回可以消除重复代码，但易读性较差。鉴于这个原因，应谨慎使用。

## func的错误处理
对于许多函数，并不总能够保证成功返回，因此当函数调用发生错误时返回一个附加的结果作为错误值，习惯上将错误值作为最后一个结果返回。

如果错误只有一种情况，结果通常设置为布尔类型；更多时候，错误的原因可能多种多样，错误的类型选择error，error是内置的接口类型，没有错误
```go
value, ok := cache.Lookup(key) //bool类型错误
if !ok {
    body
}

resp, err := http.Get(url) //error类型错误
if err != nil {
    return nil, err
}
```

## 函数变量
函数变量也有类型，其空值为nil
```go
func square(n int) int {} //func(int) int
func negative(n int) int {} //func(int) int
func product(m, n int) int {} //func(int, int) int
```
函数变量可以和空值比较，但它们之间不能比较

## 匿名函数
```go
func(r rune) rune {return r + 1}
```