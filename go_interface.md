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

举个例子：
接口类型是方法的合集，如下：
```
type Human struct {
    name string
    age int
    phone string
}

type Student struct {
    Human //an anonymous field of type Human
    school string
    loan float32
}

type Employee struct {
    Human //an anonymous field of type Human
    company string
    money float32
}

// A human likes to stay... err... *say* hi
func (h *Human) SayHi() {
    fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
}

// A human can sing a song, preferrably to a familiar tune!
func (h *Human) Sing(lyrics string) {
    fmt.Println("La la, la la la, la la la la la...", lyrics)
}

// A Human man likes to guzzle his beer!
func (h *Human) Guzzle(beerStein string) {
    fmt.Println("Guzzle Guzzle Guzzle...", beerStein)
}

// Employee's method for saying hi overrides a normal Human's one
func (e *Employee) SayHi() {
    fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
        e.company, e.phone) //Yes you can split into 2 lines here.
}

// A Student borrows some money
func (s *Student) BorrowMoney(amount float32) {
    loan += amount // (again and again and...)
}

// An Employee spends some of his salary
func (e *Employee) SpendSalary(amount float32) {
    e.money -= amount // More vodka please!!! Get me through the day!
}

// INTERFACES
type Men interface {
    SayHi()
    Sing(lyrics string)
    Guzzle(beerStein string)
}

type YoungChap interface {
    SayHi()
    Sing(song string)
    BorrowMoney(amount float32)
}

type ElderlyGent interface {
    SayHi()
    Sing(song string)
    SpendSalary(amount float32)
}
```

从上面可以看出，接口能够满足不同类型的结构体。这里接口men同时被Student和Employee实现。



