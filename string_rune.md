## string和rune
字符串字面量(string literal)使用UTF-8(8bit)进行储存，其包含了go定义的断字符，string本质上是byte数组的slice

同时go使用了一种独有的数据类型，将每个字面值以rune为类型储存，其数据类型等同于int32，每个rune对应一个字符

string使用双引号，rune使用单引号，另外还有一种反引号" `` "，表示原生字符串字面量，其与string的区别是不支持任何转义，但是可以多行书写，常用于HTML和正则表达式

以下例子说明了string和rune的区别
```go
s := "hello 你好"
r := 'hello 你好'
len(s) // 12 按byte算，中文每个字3个byte
len(r) // 8
```

对于string的index，获取的是byte
```go
func main() {
	s := "hello 你好"
	for i:=0;i<len(s);i++ {
		fmt.Printf("%v ",s[i])
	}
}
/**
104 101 108 108 111 32 228 189 160 229 165 189 
**/
```

因此使用for loop对字符串循环，其编号为每个字面值所对应byte的起始位置，其因为UTF-8的长度的关系可能并不连续
```go
const nihongo = "日本語"
for index, runeValue := range nihongo {
    fmt.Printf("%#U starts at byte position %d\n", runeValue, index)
}

// U+65E5 '日' starts at byte position 0
// U+672C '本' starts at byte position 3
// U+8A9E '語' starts at byte position 6
```

要想获得每个rune占用的byte数可以使用utf8包
```go
const nihongo = "日本語"
for i, w := 0, 0; i < len(nihongo); i += w {
    runeValue, width := utf8.DecodeRuneInString(nihongo[i:]) //这里的width返回的就是rune占用的字节数
    fmt.Printf("%#U starts at byte position %d\n", runeValue, i)
    w = width
}
```

## string转换为rune
将string直接转换为rune的slice，这样每个字面值对应的字节数都为标准的int32
```go
var s string = "abc"
rs := []rune(s)
```