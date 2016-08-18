# 函数
> 不⽀支持 嵌套 (nested)、重载 (overload) 和 默认参数 (default parameter)。

函数是第⼀一类对象，可作为参数传递。建议将复杂签名定义为函数类型，以便于阅读。
```
    func test(fn func() int) int {
        return fn()
    }
    type FormatFunc func(s string, x, y int) string // 定义函数类型。
    
    func format(fn FormatFunc, s string, x, y int) string {
        return fn(s, x, y)
    }
    func main() {
        s1 := test(func() int { return 100 }) // 直接将匿名函数当参数。
        s2 := format(func(s string, x, y int) string {
        return fmt.Sprintf(s, x, y)
            }, "%d, %d", 10, 20)
        println(s1, s2)
    }
```

## 变参
变参本质上就是 slice。只能有⼀一个，且必须是最后⼀一个。
```
func test(s string, n ...int) string {
    var x int
    for _, i := range n {
    x += i
    }
    return fmt.Sprintf(s, x)
    }
    func main() {
    println(test("sum: %d", 1, 2, 3))
}
```
使⽤用 slice 对象做变参时，必须展开。
```
func main() {
    s := []int{1, 2, 3}
    println(test("sum: %d", s...))
}
```

## 延迟执行defer
命名返回参数允许 defer 延迟调⽤用通过闭包读取和修改。
```
func add(x, y int) (z int) {
    defer func() {
        z += 100
    }()
    z = x + y
    return
}
func main() {
    println(add(1, 2)) // 输出: 103
}

```

## GO中的defer注意点
特性：

--------------
### defer表达式中*变量的值在defer表达式被定义时就已经明确*

```go

func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}
```

上面的这段代码，defer表达式中用到了i这个变量，i在初始化之后的值为0，接着程序执行到defer表达式这一行，表达式所用到的i的值就为0了，接着，表达式被放入list，等待在return的时候被调用。所以，后面尽管有一个i++语句，仍然不能改变表达式 fmt.Println(i)的结果。

所以，程序运行结束的时候，输出的结果是0而不是1。
-----------
### defer表达式的调用顺序是按照先进后出的方式
```go

func b() {
    defer fmt.Print(1)
    defer fmt.Print(2)
    defer fmt.Print(3)
    defer fmt.Print(4)
}
```
前面已经提到过，defer表达式会被放入一个类似于栈(stack)的结构，所以调用的顺序是后进先出的。所以，上面这段代码输出的结果是4321而不是1234。在实际的编码中应该主意，程序后面的defer表达式会被优先执行。

### defer表达式中可以修改函数中的命名返回值.

```go
func c() (i int) {
    defer func() { i++ }()
    return 1
}
```
上面的示例程序，返回值变量名为i，在defer表达式中可以修改这个变量的值。所以，虽然在return的时候给返回值赋值为1，后来defer修改了这个值，让i自增了1，所以，函数的返回值是2而不是1。

这三个特性需要好好掌握，不然会有很多坑。



## 匿名函数
匿名函数可赋值给变量，做为结构字段，或者在 channel ⾥里传送。
闭包复制的是原对象指针，这就很容易解释延迟引⽤用现象。
```
func test() func() {
x := 100
fmt.Printf("x (%p) = %d\n", &x, x)
return func() {
fmt.Printf("x (%p) = %d\n", &x, x)
}
}
func main() {
f := test()
f()
}
```

