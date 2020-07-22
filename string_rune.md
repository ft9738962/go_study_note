## string和rune
string本质上是byte数组的slice，显示为字符串字面值

字符串字面值(string literal)使用UTF-8进行储存，其包含了go定义的断字符，将每个字面值以rune为类型储存，其数据类型等同于int32

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