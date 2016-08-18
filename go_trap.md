# GO中的defer注意点
特性：

--------------
##defer表达式中变量的值在defer表达式被定义时就已经明确

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
## defer表达式的调用顺序是按照先进后出的方式
```go

func b() {
    defer fmt.Print(1)
    defer fmt.Print(2)
    defer fmt.Print(3)
    defer fmt.Print(4)
}
```
前面已经提到过，defer表达式会被放入一个类似于栈(stack)的结构，所以调用的顺序是后进先出的。所以，上面这段代码输出的结果是4321而不是1234。在实际的编码中应该主意，程序后面的defer表达式会被优先执行。

## defer表达式中可以修改函数中的命名返回值.

```go
func c() (i int) {
    defer func() { i++ }()
    return 1
}
```
上面的示例程序，返回值变量名为i，在defer表达式中可以修改这个变量的值。所以，虽然在return的时候给返回值赋值为1，后来defer修改了这个值，让i自增了1，所以，函数的返回值是2而不是1。

这三个特性需要好好掌握，不然会有很多坑。
