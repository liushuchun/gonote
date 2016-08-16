Comma-ok断言的语法是：value, ok := element.(T)。element必须是接口类型的变量，T是普通类型。如果断言失败，ok为false，否则ok为true并且value为变量的值。来看个例子：
```
package main

import (
    "fmt"
)

type Html []interface{}

func main() {
    html := make(Html, 5)
    html[0] = "div"
    html[1] = "span"
    html[2] = []byte("script")
    html[3] = "style"
    html[4] = "head"
    for index, element := range html {
        if value, ok := element.(string); ok {
            fmt.Printf("html[%d] is a string and its value is %s\n", index, value)
        } else if value, ok := element.([]byte); ok {
            fmt.Printf("html[%d] is a []byte and its value is %s\n", index, string(value))
        }
    }
}
```
其实Comma-ok断言还支持另一种简化使用的方式：value := element.(T)。但这种方式不建议使用，因为一旦element.(T)断言失败，则会产生运行时错误。如：
```
package main

import (
    "fmt"
)

func main() {
    var val interface{} = "good"
    fmt.Println(val.(string))
    // fmt.Println(val.(int))
}
```
以上的代码中被注释的那一行会运行时错误。这是因为val实际存储的是string类型，因此断言失败。

还有一种转换方式是switch测试。既然称之为switch测试，也就是说这种转换方式只能出现在switch语句中。可以很轻松的将刚才用Comma-ok断言的例子换成由switch测试来实现：
```
package main

import (
    "fmt"
)

type Html []interface{}

func main() {
    html := make(Html, 5)
    html[0] = "div"
    html[1] = "span"
    html[2] = []byte("script")
    html[3] = "style"
    html[4] = "head"
    for index, element := range html {
        switch value := element.(type) {
        case string:
            fmt.Printf("html[%d] is a string and its value is %s\n", index, value)
        case []byte:
            fmt.Printf("html[%d] is a []byte and its value is %s\n", index, string(value))
        case int:
            fmt.Printf("invalid type\n")
        default:
            fmt.Printf("unknown type\n")
        }
    }
}
```