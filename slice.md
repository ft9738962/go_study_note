## **slice的模型**

slice有两种产生方法：
- 从已有的数组或slice中截取一部分，slice不占用内存，只是指向原数组或原slice
- 完全新建一个slice，并加入数据，这相当于（隐性的）生成了一个底层数组underlying array，这个数组会占用内存，然后再将新建的slice指向它

总结一下，slice都会有一个对应的数组，因此slice有两个属性:
1. length （数组的长度，从slice的始端到slice的末端）
2. capacity （容量的长度，从slice的始端到底层数组的末端）

单纯的声明一个空slice变量不会指向任何数组

### 举例
有一个10个数据的数组
```go
arr := [10]int{0,1,2,3,4,5,6,7,8,9}
fmt.Println(len(arr)) //10

sli := arr[1:6] //截取5个数据
fmt.Println(len(sli), cap(sli), sli) // 5 9 [1,2,3,4,5]

//声明一个空slice
var sli []int
fmt.Println(len(sli), cap(sli), sli) // 0 0 []

//需要注意
var sli []int
sli := []int{}
//两者并不相同，前者表示没有任何初始数据，其值为slice类型的默认零值类型：nil，还没被分配底层数组，因此也没有内存指针
//后者的值为[]，已分配底层数组，有内存指针
```

因为slice是指向底层数组或底层slice的，因此对slice本身的修改，会影响到底层数组或底层slice（底层slice又会把改动传递给它的底层数组）

## **slice的添加元素方法：append**
当slice达到原有capacity的上限时，再添加元素，将触发slice的capacity的自动扩容

```go
var a []int // nil
a = append(a, 1 , 3) // {1,3} len:2 cap:2
a = append(a, 2) // {1,3,2} len:3 cap:4 注意capacity因为系统自动扩容，会有冗余
a = append(a, 3, 5, 7) // {1,3,2,3,5,7} len:5 cap:8 同上
a = a[:3] // {1,3,2} len:3 cap:8 注意虽然因为重新截取，slice长度变小了，但是被分配的capacity并不会回收，因此之前的赋值仍然在内存上
a = a[:5] // {1,3,2,3,5} len:5 cap:8

//另外可以将一个slice的元素全部添加到另一个slice
b := []int{8,9}
a = append(a, b...) //{1,3,2,3,5,8,9} 注意只能一次添加一个slice的全部元素
```

capacity的自动扩容在前1024个元素，按照2倍的速度扩容：
- 2 -> 4
- 4 -> 8
- 8 -> 16
- ......
- 512 -> 1024

在1024以后，按照大约1.25倍的速度扩容

## **slice的完整切片表示法**
一般slice的截取法可以用
```go
a := [5]int{1,2,3,4,5}
b := a[1:4] // [2,3,4] len:3 cap:4
//完整切片表示法
c := a[1:4:4] // [2,3,4] len:3 cap:3 切片数字第一个表示slice和底层数组的起始位，第二个数字表示slice的结束位（不包含），第三个数字表示底层数组的结束位（不包含）
d := a[3:3:3] // []
```

## **make定制slice**
内置函数make可以快速定制slice
```go
a := make([]int, 3) // [0 0 0] len:3 cap:3 注意用make生成的slice会被填充数值类型的默认值（不是空缺默认值）
b := make([]int, 3, 5) // [0 0 0] len:3 cap:5 make的第一个参数是slice类型，第二个参数是默认值slice长度，第三个参数是底层数组长度
c := make([]int, 0, 5) // [] len:0 cap:5

//将一个slice进行处理的最高效方法
a := ["abc","edf","ghi"]
b := make([]string, 0, len(a)) //用make建立0长度，但是有底层数组长度的空slice

for _, v := range a {
    b = append(b, strings.ToUpper(v)) // 用append插入元素
}
fmt.Println(b) // ["ABC","EDF","GHI"]
```

## **slice之间的复制copy**
内置函数copy可以将一个slice的元素复制到另一个slice中，可以复制的元素数量由两个slice中的长度小者决定
```go
a := []int{1,2,3,4,5,6}
b := []int{9,8,7}

n = copy(a[1:4], b[0:2]) // [1,9,8,4,5,6] copy会返回实际复制的元素个数 n = min(len(a[1:4]), len(b[0:2]))
```