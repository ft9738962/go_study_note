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
//两者并不相同，前者表示没有任何初始数据，其值为slice类型的默认零值类型：nil，没有内存指针
//后者的值为[]，有内存指针
```

因为slice是指向底层数组或底层slice的，因此对slice本身的修改，会影响到底层数组或底层slice（底层slice又会把改动传递给它的底层数组）

## **slice的添加元素方法：append**
当slice达到原有capacity的上限时，再添加元素，将触发slice的capacity的自动扩容