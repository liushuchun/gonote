go 的 vet 工具可以用来检查 go 代码中可以通过编译但仍然有可能存在错误的代码。例如下面的代码本应调用`fmt.Fprintln()`却错误的调用了`fmt.Println()`，虽然可以编译成功，但实际上运行后并不会把错误信息发到 stderr：

```
// testvet.go
package
 main


import
"fmt"
import
"os"
func
 main() {
    fmt.Println(os.Stderr, fmt.Errorf("error msg"))
}

```

通过 vet 可以检测出这样的错误：

```
$ go vet testvet.go
testvet.go:: first argument to Println is os.Stderr

```

根据[vet 的文档](https://golang.org/cmd/vet/)，可以使用三种方式调用 vet：

> By package, from the go tool:

```
go vet package/path/name
vets the package whose path is provided.

```

> By files:

```
go tool vet source/directory/*.go
vets the files named, all of which must be in the same package.

```

> By directory:

```
go tool vet source/directory
recursively descends the directory, vetting each package it finds.

```

vet 的检查结果仅供参考，有时 vet 也会给出错误的检查结果。下面是一个例子：

```
// testvet2.go
package
 main


import
"fmt"
type S struct{
}


func (this *S) Printf(i int) {
    fmt.Println(i)
}



func main() {
    s := &S{}
    s.Printf(1)
}

```

上面的程序可以正常编译运行，而使用 go vet 检查后会报错：

```
$ go tool vet testvet2.go
testvet.go:16: constant 1 not a string in call to Printf

```

可以看出，S.Printf\(\) 接受一个 int 类型的参数，但 vet 却认为应该给它传一个 string 类型的参数。如果把 S 的 Printf 方法改成其他名字，使用 go vet 检查就不会报错了。对此，[Go 开发者给出答复](https://github.com/golang/go/issues/12294#issuecomment-140292129)是：

> This is working as expected.
>
> Your choices are 1\) rename your Printf function; 2\) don't run vet.



