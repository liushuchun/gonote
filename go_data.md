# array 数组
1. 指针数组 [n]*T，数组指针 *[n]T。
2. 数组是值类型，赋值和传参会复制整个数组，⽽而不是指针
3. 数组⻓长度必须是常量，且是类型的组成部分。[2]int 和 [3]int 是不同类型
4. ⽀支持 "=="、"!=" 操作符，因为内存总是被初始化过的


```
a := [3]int{1, 2} // 未初始化元素值为 0。
b := [...]int{1, 2, 3, 4} // 通过初始化值确定数组⻓长度。
c := [5]int{2: 100, 4:200} // 使⽤用索引号初始化元素。
d := [...]struct {
name string
age uint8
}{
    {"user1", 10}, // 可省略元素类型。
    {"user2", 20}, // 别忘了最后⼀一⾏行的逗号。
}

```

也支持多维数组。
```
a := [2][3]int{{1, 2, 3}, {4, 5, 6}}
b := [...][2]int{{1, 1}, {2, 2}, {3, 3}} // 第 2 纬度不能⽤用 "..."。

```

#slice
需要说明，slice 并不是数组或数组指针。它通过内部指针和相关属性引⽤用数组⽚片段，以实现变⻓长⽅方案。
1.  引⽤用类型。但⾃自⾝身是结构体，值拷⻉贝传递。
2.  属性 len 表⽰示可⽤用元素数量，读写操作不能超过该限制。
3.  属性 cap 表⽰示最⼤大扩张容量，不能超出数组限制。
4.  如果 slice == nil，那么 len、cap 结果都等于 0。 重要

### append
通常以 2 倍容量重新分配底层数组。在⼤大批量添加数据时，建议⼀一次性分配⾜足够⼤大的空
间，以减少内存分配和数据复制开销。或初始化⾜足够⻓长的 len 属性，改⽤用索引号进⾏行操
作。及时释放不再使⽤用的 slice 对象，避免持有过期数组，造成 GC ⽆无法回收。

### copy
函数 copy 在两个 slice 间复制数据，复制⻓长度以 len ⼩小的为准。两个 slice 可指向同⼀一
底层数组，允许元素区间重叠。
```
data := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
s := data[8:]
s2 := data[:5]
copy(s2, s) // dst:s2, src:s
fmt.Println(s2)
fmt.Println(data)

[output]:
[8 9 2 3 4]
[8 9 2 3 4 5 6 7 8 9]
```

# Map
引⽤用类型，哈希表。键必须是⽀支持相等运算符 (==、!=) 类型，⽐比如 number、string、
pointer、array、struct，以及对应的 interface。值可以是任意类型，没有限制。
map结构也经常常用，它和php中的array（）几乎一模一样，是一个key-value的hash结构。key可以是除了func类型，array,slice,map类型之外的类型。
它的使用也是非常简单:
```
m:=map[string]string{}

m["key1"] = "val1"


```

map结构和slice是一样的，是一个指针。赋值的时候是将指针复制给新的变量




预先给 make 函数⼀一个合理元素数量参数，有助于提升性能。因为事先申请⼀一⼤大块内存，
可避免后续操作时频繁扩张。
`m := make(map[string]int, 1000)`
常⻅见操作：
```
m := map[string]int{
"a": 1,
}
if v, ok := m["a"]; ok { // 判断 key 是否存在。
println(v)
}

println(m["c"]) // 对于不存在的 key，直接返回 \0，不会出错。

m["b"] = 2 // 新增或修改。

delete(m, "c") // 删除。如果 key 不存在，不会出错。

println(len(m)) // 获取键值对数量。cap ⽆无效。

for k, v := range m { // 迭代，可仅返回 key。随机顺序返回，每次都不相同。
    println(k, v)
}
//不能保证迭返回次序，通常是随机结果，具体和版本实现有关。
//从 map 中取回的是⼀一个 value 临时复制品，对其成员的修改是没有任何意义的。

map 在使用之前必须用 make 来创建（不是 new）；一个值为 nil 的 map 是空的，并且不能赋值。
func main() {
    m = make(map[string]Vertex)
    m["Bell Labs"] = Vertex{
        40.68433, 74.39967,
    }
    fmt.Println(m["Bell Labs"])
}


for 循环的 range 格式可以对 slice 或者 map 进行迭代循环。
for i, v := range pow {
        fmt.Printf("2**%d = %d\n", i, v)
    }
```

无法做出任何相关修改.
```
type user struct{ name string }
m := map[int]user{ // 当 map 因扩张⽽而重新哈希时，各键值项存储位置都会发⽣生改变。 因此，map
1: {"user1"}, // 被设计成 not addressable。 类似 m[1].name 这种期望透过原 value
} // 指针修改成员的⾏行为⾃自然会被禁⽌止。
m[1].name = "Tom" // Error: cannot assign to m[1].name
```
正确的做法是完整替换掉value或*使用指针*。
```
m2 := map[int]*user{
1: &user{"user1"},
}
m2[1].name = "Jack" // 返回的是指针复制品。透过指针修改原对象是允许的。
```

可以在迭代时安全删除键值。但如果期间有新增操作，那么就不知道会有什么意外了。
```
for i := 0; i < 5; i++ {
m := map[int]string{
0: "a", 1: "a", 2: "a", 3: "a", 4: "a",
5: "a", 6: "a", 7: "a", 8: "a", 9: "a",
}
for k := range m {
m[k+k] = "x"
delete(m, k)
}
fmt.Println(m)
}
```
output
```
map[12:x 16:x 2:x 6:x 10:x 14:x 18:x]
map[12:x 16:x 20:x 28:x 36:x]
map[12:x 16:x 2:x 6:x 10:x 14:x 18:x]
map[12:x 16:x 2:x 6:x 10:x 14:x 18:x]
map[12:x 16:x 20:x 28:x 36:x]
```

# Go语言slice和map机制
array是固定长度的数组，这个和C语言中的数组是一样的，使用前必须确定数组长度。但是和C中的数组相比，又是有一些不同的：

1. Go中的数组是值类型，换句话说，如果你将一个数组赋值给另外一个数组，那么，实际上就是将整个数组拷贝一份

2. 如果Go中的数组作为函数的参数，那么实际传递的参数是一份数组的拷贝，而不是数组的指针。这个和C要区分开。因此，在Go中如果将数组作为函数的参数传递的话，那效率就肯定没有传递指针高了。这个是不是有点陷阱的感觉？

3. array的长度也是Type的一部分，这样就说明[10]int和[20]int是不一样的。

array的结构用图示表示是这样的：
![图片](http://oa8dpdexh.bkt.clouddn.com//201206142215457646.png)

len表示的是数组的长度，后面的int存储的是实际的数据。

slice类型,Slice是Go程序中最常用的表示序列数组的类型。为什么最经常用它呢？
1. Slice可变长
定义完一个slice变量之后，不需要为它的容量而担心，你随时可以往slice里面加数据

比如：
```
v:=[]string{}

v=append(v, "hello")

这里附带说一下，slice和array的写法很容易混

v:=[2]string{"str1", "str2"} //这个是array

m:=[]string{"str1","str2"} //这个是slice
```
写法上默念：array有长度slice没长度

2. Slice是一个指针，引用

指针比值可就小多了，因此，我们将slice作为函数参数传递比将array作为函数参数传递。

slice是一个指针，它指向的是一个array 单元，它有两个基本函数len和cap。

![图2](http://oa8dpdexh.bkt.clouddn.com//201206142215564342.png)

slice是一个带有point（指向数组的指针），Len（数组中实际有值的个数），Cap（数组的容量）

比如上面的这个slice，它指向的数组是[3]int,其中的前两个有值，第三个为空

则:
```
len(slic) = 2

cap(slic) = 3
```
append函数就理解为往slice中加入一个值，如果未达到容量（len<cap）那么就直接往数组中加值，如果达到容量（len = cap）那么就增加一个新的元素空间，将值放在里面。




# Struct

值类型，赋值和传参会复制全部内容。可⽤用 "_" 定义补位字段，⽀支持指向⾃自⾝身类型的指针成员。
```
type Node struct {
    _ int
    id int
    data *byte
    next *Node
}
```
顺序初始化必须包含全部字段，否则会出错。
```
type User struct {
    name string
    age int
}
u1 := User{"Tom", 20}
u2 := User{"Tom"} // Error: too few values in struct initializer
```

可定义字段标签，⽤用反射读取。标签是类型的组成部分。
```
var u1 struct { name string "username" }
var u2 struct { name string }
u2 = u1 // Error: cannot use u1 (type struct { name string "username" }) as
// type struct { name string } in assignment
```

空结构 "节省" 内存，⽐比如⽤用来实现 set 数据结构，或者实现没有 "状态" 只有⽅方法的 "静
态类"。
```
var num struct{}
set:=make(map[string]struct{})
set["a"]=null
```

# 面向对象
>⾯面向对象三⼤大特征⾥里，Go 仅⽀支持封装，尽管匿名字段的内存布局和⾏行为类似继承。没有
class 关键字，没有继承、多态等等。




