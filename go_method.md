# 方法
方法的定义
⽅方法总是绑定对象实例，并隐式将实例作为第⼀一实参 (receiver)。
    1. 只能为当前包内命名类型定义⽅方法。
    2. 参数 receiver 可任意命名。如⽅方法中未曾使⽤用，可省略参数名。
    3. 参数 receiver 类型可以是 T 或 *T。基类型 T 不能是接⼝口或指针。
    4. 不⽀支持⽅方法重载，receiver 只是参数签名的组成部分。
    5. 可⽤用实例 value 或 pointer 调⽤用全部⽅方法，编译器⾃自动转换。

没有构造和析构⽅方法，通常⽤用简单⼯工⼚厂模式返回对象实例。
```
type Queue struct {
elements []interface{}
}
func NewQueue() *Queue { // 创建对象实例。
return &Queue{make([]interface{}, 10)}
}
func (*Queue) Push(e interface{}) error { // 省略 receiver 参数名。
panic("not implemented")
}
// func (Queue) Push(e int) error { // Error: method redeclared: Queue.Push
// panic("not implemented")
// }
func (self *Queue) length() int { // receiver 参数名可以是 self、this 或其他。
return len(self.elements)
}

```
⽅方法不过是⼀一种特殊的函数，只需将其还原，就知道 receiver T 和 *T 的差别。

## 方法集
>每个类型都有与之关联的⽅方法集，这会影响到接⼝口实现规则。
⽤用实例 value 和 pointer 调⽤用⽅方法 (含匿名字段) 不受⽅方法集约束，编译器总是查找全部方法，并⾃自动转换receiver 实参。

根据调用者不同，方法分为两种表现形式：
```
instance.method(args...) ---> <type>.func(instance, args...)
```
