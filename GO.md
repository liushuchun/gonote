# 常量
在常亮当中，如果不提供类型和初始值，视作与上一个常量相同。
```
const (
    s = "abc"
    x         // x="abc"
    )
```

常量值还可以是 len、cap、unsafe.Sizeof 等编译期可确定结果的函数返回值
```
const (
    a = "abc"
    b = len(a)
    c = unsafe.Sizeof(b)
)
```

## 枚举
关键字 iota 定义常量组中从 0 开始按⾏行计数的⾃自增枚举值。
```
const (
    Sunday = iota // 0
    Monday // 1，通常省略后续⾏行表达式。
    Tuesday // 2
    Wednesday // 3
    Thursday // 4
    Friday // 5
    Saturday // 6
)

```
也可以计算
```
const (
    _ = iota // iota = 0
    KB int64 = 1 << (10 * iota) // iota = 1
    MB // 与 KB 表达式相同，但 iota = 2
    GB
    TB
    )
```
关键字 iota 定义常量组中从 0 开始按⾏行计数的⾃自增枚举值。
```
const (
A, B = iota, iota << 10 // 0, 0 << 10
C, D // 1, 1 << 10
)
```

如果 iota ⾃自增被打断，须显式恢复。
```
const (
    A = iota // 0
    B // 1
    C = "c" // c
    D // c，与上⼀一⾏行相同。
    E = iota // 4，显式恢复。注意计数包含了 C、D 两⾏行。
    F // 5
)
```

# 引用类型
> 引⽤用类型包括 slice、map 和 channel。它们有复杂的内部结构，除了申请内存外，还需要初始化相关属性。
## new 和make 区别
内置函数 `new` 计算类型⼤大⼩小，为其分配零值内存，返回指针。
而 `make` 会被编译器翻译成具体的创建函数，由其分配内存和初始化成员结构，返回对象⽽而⾮非指针。
## new 的主要特性
首先 new 是内建函数，你可以从 http://golang.org/pkg/builtin/#new 这儿看到它，它的定义也很简单：
`func new(Type) *Type`
官方文档对于它的描述是：
> 内建函数 new 用来分配内存，它的第一个参数是一个类型，不是一个值，它的返回值是一个指向新分配类型零值的指针

## make的主要特性
make 也是内建函数，你可以从 http://golang.org/pkg/builtin/#make 这儿看到它，它的定义比 new 多了一个参数，返回值也不同：
`func make(Type, size IntegerType) Type`

> 内建函数 make 用来为 slice，map 或 chan 类型分配内存和初始化一个对象(注意：只能用在这三种类型上)，跟 new 类似，第一个参数也是一个类型而不是一个值，跟 new 不同的是，make 返回类型的引用而不是指针，而返回值也依赖于具体传入的类型.

```
Slice: 第二个参数 size 指定了它的长度，它的容量和长度相同。
你可以传入第三个参数来指定不同的容量值，但必须不能比长度值小。
比如 make([]int, 0, 10)

Map: 根据 size 大小来初始化分配内存，不过分配后的 map 长度为 0，如果 size 被忽略了，那么会在初始化分配内存时分配一个小尺寸的内存

Channel: 管道缓冲区依据缓冲区容量被初始化。如果容量为 0 或者忽略容量，管道是没有缓冲区的

```

## 总结
new 的作用是初始化一个指向类型的指针(*T)，make 的作用是为 slice，map 或 chan 初始化并返回引用(T)。

内置函数 new 计算类型⼤大⼩小，为其分配零值内存，返回指针。⽽make会被编译器翻译成具体的创建函数，由其分配内存和初始化成员结构，返回对象⽽而⾮非指针。
```
a := []int{0, 0, 0} // 提供初始化表达式。
a[1] = 10
b := make([]int, 3) // makeslice
b[1] = 10
c := new([]int)
c[1] = 10 // Error: invalid operation: c[1] (index of type *[]int)
```

# 字符串
字符串是不可变值类型，内部⽤用指针指向 UTF-8 字节数组。
* 默认值是空字符串 ""。
* ⽤用索引号访问某字节，如 s[i]。
* 不能⽤用序号获取字节元素指针，&s[i] ⾮非法。
* 不可变类型，⽆无法修改字节数组。
* 字节数组尾部不包含 NULL。

使⽤用 "`" 定义不做转义处理的原始字符串，⽀支持跨⾏行。
```
s := `a
b\r\n\x00
c`
```
连接跨⾏行字符串时，"+" 必须在上⼀一⾏行末尾，否则导致编译错误。
```
s := "Hello, " +
"World!"
s2 := "Hello, "
+ "World!" // Error: invalid operation: + untyped string
```
⽀支持⽤用两个索引号返回⼦子串。⼦子串依然指向原字节数组，仅修改了指针和⻓长度属性。
```
s := "Hello, World!"
s1 := s[:5] // Hello
s2 := s[7:] // World!
s3 := s[1:5] // ello
```
用for循环遍历字符串时，也有byte和rune两种方式。
```
func main() {
    s := "abc汉字"
    for i := 0; i < len(s); i++ { // byte
    fmt.Printf("%c,", s[i])
    }
    fmt.Println()
    for _, r := range s { // rune
    fmt.Printf("%c,", r)
    }
}
```
输出
```
a,b,c,æ,±,,å,,,
a,b,c,汉,字,
```

# 指针
⽀支持指针类型 *T，指针的指针 **T，以及包含包名前缀的 *<package>.T。
• 默认值 nil，没有 NULL 常量。

• 操作符 "&" 取变量地址，"*" 透过指针访问⺫⽬目标对象。
• 不⽀支持指针运算，不⽀支持 "->" 运算符，直接⽤用 "." 访问目标成员。

```
func main() {
    type data struct{ a int }
    var d = data{1234}
    var p *data
    p = &d
    fmt.Printf("%p, %v\n", p, p.a) // 直接⽤用指针访问⺫⽬目标对象成员，⽆无须转换。
}
```

指针没法做自增自减的运算。
不过可以转化为 uintptr 做运算
```
    d := struct {
    s string
    x int
    }{"abc", 100}
    p := uintptr(unsafe.Pointer(&d)) // *struct -> Pointer -> uintptr
    p += unsafe.Offsetof(d.x) // uintptr + offset
    p2 := unsafe.Pointer(p) // uintptr -> Pointer
    px := (*int)(p2) // Pointer -> *int
    *px = 200 // d.x = 200
    fmt.Printf("%#v\n", d)
```

# FOR 循环
```
    s := "abc"
    for i, n := 0, len(s); i < n; i++ { // 常⻅见的 for 循环，⽀支持初始化语句。
    println(s[i])
    }
    n := len(s)
    for n > 0 { // 替代 while (n > 0) {}
    println(s[n]) // 替代 for (; n > 0;) {}
    n--
    }
    for { // 替代 while (true) {}
    println(s) // 替代 for (;;) {}
    }
```
# switch循环
如需要继续下⼀一分⽀支，可使⽤用 fallthrough，但不再判断条件。

