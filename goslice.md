##golang 参数传值
首先，golang官方表示golang的函数调用参数均为值传递，不是指针传递或引用传递。经测试引申出来，当参数变量为指针或隐式指针类型，参数传递方式也是传值（指针本身的copy）。
after they are evaluated, the parameters of the call are passed by value to the
function and the called function begins execution.
##解释
关于最有争议的slice传参是传值还是传引用（或指针）的问题，做一个剖析。
首先，认为slice传参是引用传递的大都是根据以下代码情况得出的结果：

```
package main
import "fmt"

func main(){
    slice := make([]int, 3, 5)
    fmt.Println("before:", slice)
    changeSliceMember(slice)
    fmt.Println("after:", slice)
}

func changeSliceMember(slice []int) {
    if len(slice) > 1 {
	slice[0] = 9
    }
}

```

函数执行结果为：
```
befor:[0 0 0]
after:[9 0 0]
```
很显然，slice[0]值在函数内操作过之后确实在main中将结果保留了下来。但这不足以表明参数传递方式，参数slice无论值还是引用，不影响slice[0]地址的值。换句话说，只要参数是地址数据，都可以改变其指向的值的内容。
slice是结构体和指针的混合体，它的每项值，是以指针为内容存储存于values中，因此可以改变其值；slice属性values、count 和 capacity等均是值传递。但values的值（指针）所指向的堆栈，是存储单元，因此可以保留更改。
因此，以指针和结构体的角度看，都是值传递。
证明slice传参过程为值传递的例子
如下代码：

```go
package main
import "fmt"

func main() {
	slice := make([]int, 3, 5)

	fmt.Println("main-befor send:", slice)
	sliceValueAppend(slice)
	fmt.Println("main-after send:", slice)
}

func sliceValueAppend(slice []int) {
	slice = append(slice, 4)
	fmt.Println("fun-after append:", slice)
}
```

执行结果为：

```
main-befor send:[0 0 0]
fun-after append:[0 0 0 4]
main-after send:[0 0 0]
```

因此看出传输方式为值传递。
## 使用指针变量的例子
使用指针变量作为参数更清晰:
```go
package main

func main() {
	value := new(int)
	modifyFunc(value)
	println("main:", value)
}

func modifyFunc(value *int) {
	value = nil
	println("modifyFunc:", value)
}
```
执行结果：
```
modifyFunc: 0x0
main: 0xc820049f30
```

可以看出，即使传值为指针，仍未改变变量value在main中的值。
## golang中的引用
golang引用的存在
golang在函数传参过程中均为值传递，但不代表golang没有传引用的地方。
golang闭包访问其外部环境变量时，是通过引用访问的。
闭包举例
```go
package main
import "fmt"

func main() {
	value := new(int)
	fmt.Println("before:", value)

	func () {
		value = nil
	}()
	fmt.Println("after:", value)
}
```
输出为：
```
before: 0xc82000a360
after: <nil>
```

其实闭包的引用传递，相当于全局变量的使用，只是对该值的处理过程进行闭包处理，没什么不好理解的。

## 总结

* 在golang中，基础类型和自定义类型均以传值的方式进行参数传递，包括但不限于slice、map、chan。
* golang对闭包传值方式是引用传值。

