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

