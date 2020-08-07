## 方法的声明
方法的声明和普通函数的声明类似，只是在函数名字前面多了一个参数。这个参数把这个方法绑定到这个参数对应的类型上
```go
package geometry

type Point struct {
    X, Y float64
}

//普通函数
func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

//Point类型的方法
func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}
```
其中附加的参数p成为方法的接受者。调用方法的时候，接受者在方法名的前面，和声明保持一致
```go
p := Point{1, 2}
q := Point{4, 6}
fmt.Println(Distance(p, q)) // 函数调用
fmt.Println(p.Distance(q)) // 方法调用
```
前一个声明一个包级别的函数（称为geometry.Distance)，第二个声明一个类型Point的方法，名字为Point.Distance

表达式p.Distance称作为选择子(selector)，但选择子也用于选择结构类型中的某些字段值，比如p.X。由于方法和字段来自于同一个命名空间，因此在Point结构类型中声明一个X的方法会与字段X冲突，编译器会报错。

```go
type Path []Point
func (path path) Distance() float64 {
    sum := 0.0
    for i := range path {
        if i > 0 {
            sum += path[i-1].Distance(path[i])
        }
    }
}
```
上个例子中Path是一个命名的slice类型，并不是一个结构体，但也可以给它定义方法，**Go可以将方法绑定到任何类型上**，只要它的类型不是指针类型或接口类型

## 指针接受者的方法
如果函数需要更新一个变量，或如果一个实参太大，希望避免复制整个实参，因此必须使用指针来传递变量的地址，这也同样适用于更新接受者，可以将它绑定到指针类型
```go
func (p *Point) ScaleBy(factor float64) {
    p.X *= factor
    p.Y *= factor
}
```
这个方法的名字是(*Point).ScaleBy。\*圆括号是必须的，如果没有圆括号，表达式会被解析为\*(Point.ScaleBy)。

命名类型（Point）与指向它们的指针（*Point）是唯一可以出现在接受者声明处的类型。而且为防止混淆，不允许本身是指针的类型进行方法声明：
```go
type P *int
func (P) f() {} // 编译错误：非法的接受者类型
```

通过提供 \*Point能够调用(*Point).ScaleBy方法，比如：
```go
r := &Point{1, 2}
r.ScaleBy(2)
fmt.Println(*r) // {2, 4}

p := Point{1, 2}
pptr := &p
pptr.ScaleBy(2)
fmt.Println(p) // {2, 4}
```

如果接受者p是Point类型的变量，但方法要求一个*Point接受者，可以使用简写：
```go
p.ScaleBy(2)
```
实际上编译器会对变量进行&p的隐式转换，只有变量才允许这么做，包括结构体字段，如p.X和数组或者slice的元素，比如perim[0]

## 结构体内嵌类型的方法
```go
type Point struct {
    X, Y float64
}

type ColoredPoint struct {
    Point
    Color color.RGBA
}
```
这里的ColoredPoint不仅内嵌了Point类型提供的字段X, Y，还包含了Point类型定义的方法，可以供ColoredPoint直接调用，相当于编译器额外生成了以下代码
```go
func (p ColoredPoint) Distance(q Point) float64 {
    return p.Point.Distance(q)
}

func (p *ColoredPoint) ScaleBy(factor float64) {
    p.Point.ScaledBy(factor)
}
```
编译器处理选择子的时候，先查找直接声明的方法，如果没有则再查找内嵌字段的方法；当同一个查找级别中有同名方法时，编译器会报告选择子不明确的错误。

方法只能在明明的类型和指向它们的指针中声明，但内嵌使能够在未命名的结构体类型中声明方法，以一个互斥锁和map为例，互斥锁将会保护map和数据：
```go
// 不适用内嵌
var (
    mu sync.Mutex // 保护mapping
    mappping = make(map[string]string)
)

func Lookup(key string) string {
    mu.Lock()
    v := mapping[key]
    mu.Unlock()
    return v
}

=========================================================
// 未命名结构体使用内嵌
var cache = struct {
    sync.Mutex
    mapping map[string]string
} {
    mapping: make(map[string]string),
}

func Lookup(key string) string {
    cache.Lock()
    v := cache.mapping[key]
    cache.Unlock()
    return v
}
```
由于将sync.Mutex内嵌，它的Lock和Unlock方法也包含在了结构体中，可以直接使用cache变量本身加锁