# Golang method with pointer receiver
这个是我真实遇到的问题，在stackoverflow上看到，例子：
```go
package main

import (
    "fmt"
)

type IFace interface {
    SetSomeField(newValue string)
    GetSomeField() string
}

type Implementation struct {
    someField string
}

func (i Implementation) GetSomeField() string {
    return i.someField
}

func (i Implementation) SetSomeField(newValue string) {
    i.someField = newValue
}

func Create() IFace {
    obj := Implementation{someField: "Hello"}
    return obj // <= Offending line
}

func main() {
    a := Create()x
    a.SetSomeField("World")
    fmt.Println(a.GetSomeField())
}

```

setSomeField不能修改值，因为传值的方式来定义方法而不是指针。
但是当我修改为指针方法，我原本以为会工作的，如下：
```go
func (i *Implementation) SetSomeField(newValue string) { ...

```
编译错误如下：
```
prog.go:26: cannot use obj (type Implementation) as type IFace in return argument:
Implementation does not implement IFace (GetSomeField method has pointer receiver)
```
解决方法如下：
指向这个结构体的指针要实现该接口。修改代码如下：
```go
package main

import (
    "fmt"
)

type IFace interface {
    SetSomeField(newValue string)
    GetSomeField() string
}

type Implementation struct {
    someField string
}    

func (i *Implementation) GetSomeField() string {
    return i.someField
}

func (i *Implementation) SetSomeField(newValue string) {
    i.someField = newValue
}

func Create() *Implementation {
    return &Implementation{someField: "Hello"}
}

func main() {
    var a IFace
    a = Create()
    a.SetSomeField("World")
    fmt.Println(a.GetSomeField())
}

```
算是一个理解的问题:
```
var impl Implementation
var iface IFace = &impl
```
因为他是一个:
```
 *impl 而不是 impl.


```

