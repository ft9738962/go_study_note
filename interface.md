## Interface的综述

Go 中的 interface 是一种类型，更准确的说是一种抽象类型 abstract type，一个 interface 就是包含了一系列行为的 method 集合，interface 的定义很简单：
```go
package io

type Writer interface {
    Write(p []byte) (n int, err error)
}
```
Go 中的 interface 不同于其它语言，它是隐式的 implicitly，这意味着对于一个已有类型，你可以不用更改任何代码就可以让其满足某个 interface。

如果一个 concrete type 实现了某个 interface，我们说这个 concrete type 实现了 interface 包含的所有 method，**必须是所有的 method**。

## Interface的使用目的
在 Go 的标准库 fmt 中有一系列的方法可以很好的诠释 interface 是如何应用到实践当中的。
```go
package fmt

func Fprintf(w io.Writer, format string, args ...interface{}) (int, error)

func Printf(format string, args ...interface{}) (int, error) {
    return Fprintf(os.Stdout, format, args...)
}

func Sprintf(format string, args ...interface{}) string {
    var buf bytes.Buffer
    Fprintf(&buf, format, args...)
    return buf.String()
}
```
Fprintf 中的前缀 F 表示 File，意思是格式化的输出被输出到函数指定的第一个 File 类型的参数中。

在 Printf 函数中，调用 Fprintf 时指定的输出是标准输出，这正是 Printf 的功能：Printf formats according to a format specifier and writes to standard output，根据指定的格式化要求输出到标准输出，os.Stdout 的类型是 *os.File 。

同样在 Sprintf 函数中，调用 Fprintf 时指定的输出是一个指向某个 memory buffer 的指针，其类似一个 *os.File。

虽然 bytes.Buffer 和 os.Stdout 是不同的，但是它们都可以被用于调用同一个函数 Fprintf，就是因为 Fprintf 的第一个参数是接口类型 io.Writer ，而 bytes.Buffer 和 os.Stdout 都实现了这个 interface，即它们都实现了 Write 这个 method，这个 interface 并不是一个 File 却完成了类似 File的功能。

Fprintf 其实并不关心它的第一个参数是一个 file 还是一段 memory，它只是调用了 Write method。这正是 interface 所关注的，**只在乎行为，不在乎其值**，这种能力让我们可以非常自由的向 Fprintf 传递任何满足 io.Writer 的 concrete type，这是 Go interface 带来的 substitutability 可替代性，object-oriented programming 的一种重要特性。

再看一个类似的例子
```go
package main

import (
    "fmt"
)

type ByteCounter int


func (c *ByteCounter) Write(p []byte) (int, error) {
    *c += ByteCounter(len(p)) // convert int to ByteCounter
    return len(p), nil
}

func main() {

    var c ByteCounter
    c.Write([]byte("hello"))
    fmt.Println(c) // 5  #1
    fmt.Fprintf(&c, "hello") #2 
    fmt.Println(c) // 10  #3
}
```

ByteCounter 实现了 Write method，它满足 io.Writer interface，Write method 计算传给它的 byte slice 的长度并且赋值给自身，所以 #1 输出到标准输出的是它的值 5，正如前文所言，调用 fmt.Fprintf 时再次调用了 c 的 Write method，所以 #3 输出是 10。

这就是 Go 中的 interface 所具有的最基本的功能：作为一种 abstract type，实现各种 concrete type 的行为统一。

##定义Interface
在 Go 的标准库 io package 中定义了很多有用的 interface：
```go
package io

type Reader interface {
    Read(p []byte)(n intm err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}
```
Reader 代表任何可以去读 byte 的类型，类似的，Writer 代表任何可以写入 byte 的类型，Closer 代表任何可以执行 close 的类型。除此之外，我们还能在 io package 中发现其它组合式的 interface：
```go
package io

type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```
ReaderWrite 和 ReadWriteCloser 的语法形式和 struct 的 embedding 非常类似，被称为 embedding interface，通过这种形式可以便捷的实现一个新的 interface 而不必写出 interface 包含的所有 method。

注意 Go 标准库中对组合 interface 的命名，你应该总是遵循这样的规则。

ReaderWrite 也可以用非 embedding 的形式实现：
```go
type ReadWriter interface {
    Read(p []byte)(n int err error)
    Write(p []byte) (n int, err error)
}
```
或者是二者的组合：
```go
type ReadWriter interface {
    Read(p []byte)(n int err error)
    Writer
}
```
以上的几种实现都是等价的，但是这里更推崇使用 embedding 的形式。

## Interface的实现使用
如果一个 concrete type 实现了某个 interface type，其值可以赋值给 interface type 的 instance。
```go
var w io.Writer
w = os.Stdout
w = new(byte.Buffer)

var rwc io.ReadWriteCloser
rwc = os.Stdout
```
甚至赋值符号右边也可以是 interface instance，只要它们的关系满足左边 interface type method 集合是右边 interface type method 集合的子集即可：
```go
w = rwc
```
w method 集合 {Writer} 是 rwc method 集合 {Reader, Writer, Closer} 的子集。

在 Struct 中，在一个类型 T 上直接调用 receiver 是 *T 的 method 是合法的，只要 T 是以 variable 的形势存在，看个例子：
```go
package main

import (
    "fmt"
)

type T struct {
}

func (t *T) String() string {
    return ""
}
```
但是这样是不可以的：
```go
T{}.String()
```

Struct 这个微妙的细节也体现在 interface 的赋值上，只有 *T 实现了 String 所以 *T 才能满足 interface Stringer，这样是可以的：
```go
var _ fmt.Stringer = &t
```
