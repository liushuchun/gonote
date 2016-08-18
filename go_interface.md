# 接口
定义：
> 接⼝口是⼀一个或多个⽅方法签名的集合，任何类型的⽅方法集中只要拥有与之对应的全部⽅方法，
就表⽰示它 "实现" 了该接⼝口，⽆无须在该类型上显式添加接⼝口声明。
所谓对应⽅方法，是指有相同名称、参数列表 (不包括参数名) 以及返回值。当然，该类型还
可以有其他⽅方法。
    1. • 接⼝口命名习惯以 er 结尾，结构体。
    2. • 接⼝口只有⽅方法签名，没有实现。
    3. • 接⼝口没有数据字段。
    4. • 可在接⼝口中嵌⼊入其他接⼝口。
    5. • 类型可实现多个接⼝口。

============

例子
```
type Stringer interface {
    String() string
}
type Printer interface {
    Stringer // 接⼝口嵌⼊入。
    Print()
}
type User struct {
    id int
    name string
}
func (self *User) String() string {
    return fmt.Sprintf("user %d, %s", self.id, self.name)
}
func (self *User) Print() {
    fmt.Println(self.String())
}

func main() {
    var t Printer = &User{1, "Tom"} // *User ⽅方法集包含 String、Print。
    t.Print()
}
```




