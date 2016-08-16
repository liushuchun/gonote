Go语言并发的设计模式和应用场景
以下设计模式和应用场景来自Google IO上的关于Goroutine的PPT: https://talks.golang.org/2012/concurrency.slide

本文的示例代码在: https://github.com/hit9/Go-patterns-with-channel

生成器

在Python中我们可以使用yield关键字来让一个函数成为生成器，在Go中我们可以使用信道来制造生成器(一种lazy load类似的东西)。

当然我们的信道并不是简单的做阻塞主线的功能来使用的哦。

下面是一个制作自增整数生成器的例子，直到主线向信道索要数据，我们才添加数据到信道

func xrange() chan int{ // xrange用来生成自增的整数
    var ch chan int = make(chan int)

    go func() { // 开出一个goroutine
        for i := 0; ; i++ {
            ch <- i  // 直到信道索要数据，才把i添加进信道
        }
    }()

    return ch
}

func main() {

    generator := xrange()

    for i:=0; i < 1000; i++ {  // 我们生成1000个自增的整数！
        fmt.Println(<-generator)
    }
}
这不禁叫我想起了Python中可爱的xrange, 所以给了生成器这个名字！

服务化

比如我们加载一个网站的时候，例如我们登入新浪微博，我们的消息数据应该来自一个独立的服务，这个服务只负责 返回某个用户的新的消息提醒。

如下是一个使用示例:

func get_notification(user string) chan string{
   /*
    * 此处可以查询数据库获取新消息等等..
    */
    notifications := make(chan string)

    go func() { // 悬挂一个信道出去
        notifications <- fmt.Sprintf("Hi %s, welcome to weibo.com!", user)
    }()

    return notifications
}

func main() {
    jack := get_notification("jack") //  获取jack的消息
    joe := get_notification("joe") // 获取joe的消息

    // 获取消息的返回
    fmt.Println(<-jack)
    fmt.Println(<-joe)
}
多路复合

上面的例子都使用一个信道作为返回值，可以把信道的数据合并到一个信道的。 不过这样的话，我们需要按顺序输出我们的返回值（先进先出）。

如下，我们假设要计算很复杂的一个运算 100-x , 分为三路计算， 最后统一在一个信道中取出结果:

func do_stuff(x int) int { // 一个比较耗时的事情，比如计算
    time.Sleep(time.Duration(rand.Intn(10)) * time.Millisecond) //模拟计算
    return 100 - x // 假如100-x是一个很费时的计算
}

func branch(x int) chan int{ // 每个分支开出一个goroutine做计算并把计算结果流入各自信道
    ch := make(chan int)
    go func() {
        ch <- do_stuff(x)
    }()
    return ch
}

func fanIn(chs... chan int) chan int {
    ch := make(chan int)

    for _, c := range chs {
        // 注意此处明确传值
        go func(c chan int) {ch <- <- c}(c) // 复合
    }

    return ch
}


func main() {
    result := fanIn(branch(1), branch(2), branch(3))

    for i := 0; i < 3; i++ {
        fmt.Println(<-result)
    }
}
select监听信道

Go有一个语句叫做select，用于监测各个信道的数据流动。

如下的程序是select的一个使用例子，我们监视三个信道的数据流出并收集数据到一个信道中。

func foo(i int) chan int {
    c := make(chan int)
    go func () { c <- i }()
    return c
}


func main() {
    c1, c2, c3 := foo(1), foo(2), foo(3)

    c := make(chan int)

    go func() { // 开一个goroutine监视各个信道数据输出并收集数据到信道c
        for {
            select { // 监视c1, c2, c3的流出，并全部流入信道c
            case v1 := <- c1: c <- v1
            case v2 := <- c2: c <- v2
            case v3 := <- c3: c <- v3
            }
        }
    }()

    // 阻塞主线，取出信道c的数据
    for i := 0; i < 3; i++ {
        fmt.Println(<-c) // 从打印来看我们的数据输出并不是严格的1,2,3顺序
    }
}
有了select, 我们在 多路复合中的示例代码中的函数fanIn还可以这么来写(这样就不用开好几个goroutine来取数据了):

func fanIn(branches ... chan int) chan int {
    c := make(chan int)

    go func() {
        for i := 0 ; i < len(branches); i++ { //select会尝试着依次取出各个信道的数据
            select {
            case v1 := <- branches[i]: c <- v1
            }
        }
    }()

    return c
}
使用select的时候，有时需要超时处理, 其中的timeout信道相当有趣:

timeout := time.After(1 * time.Second) // timeout 是一个计时信道, 如果达到时间了，就会发一个信号出来

for is_timeout := false; !is_timeout; {
    select { // 监视信道c1, c2, c3, timeout信道的数据流出
    case v1 := <- c1: fmt.Printf("received %d from c1", v1)
    case v2 := <- c2: fmt.Printf("received %d from c2", v2)
    case v3 := <- c3: fmt.Printf("received %d from c3", v3)
    case <- timeout: is_timeout = true // 超时
    }
}
结束标志

在Go并发与并行笔记一我们已经讲过信道的一个很重要也很平常的应用，就是使用无缓冲信道来阻塞主线，等待goroutine结束。

这样我们不必再使用timeout。

那么对上面的timeout来结束主线的方案作个更新：

func main() {

    c, quit := make(chan int), make(chan int)

    go func() {
        c <- 2  // 添加数据
        quit <- 1 // 发送完成信号
    } ()

    for is_quit := false; !is_quit; {
        select { // 监视信道c的数据流出
        case v := <-c: fmt.Printf("received %d from c", v)
        case <-quit: is_quit = true // quit信道有输出，关闭for循环
        }
    }
}
菊花链



简单地来说，数据从一端流入，从另一端流出，看上去好像一个链表，不知道为什么要取这么个尴尬的名字。。

菊花链的英文名字叫做: Daisy-chain, 它的一个应用就是做过滤器，比如我们来筛下100以内的素数(你需要先知道什么是筛法)

程序有详细的注释，不再说明了。

/*
 *  利用信道菊花链筛法求某一个整数范围的素数
 *  筛法求素数的基本思想是：把从1开始的、某一范围内的正整数从小到大顺序排列，
 *  1不是素数，首先把它筛掉。剩下的数中选择最小的数是素数，然后去掉它的倍数。
 *  依次类推，直到筛子为空时结束
 */
package main

import "fmt"

func xrange() chan int{ // 从2开始自增的整数生成器
    var ch chan int = make(chan int)

    go func() { // 开出一个goroutine
        for i := 2; ; i++ {
            ch <- i  // 直到信道索要数据，才把i添加进信道
        }
    }()

    return ch
}


func filter(in chan int, number int) chan int {
    // 输入一个整数队列，筛出是number倍数的, 不是number的倍数的放入输出队列
    // in:  输入队列
    out := make(chan int)

    go func() {
        for {
            i := <- in // 从输入中取一个

            if i % number != 0 {
                out <- i // 放入输出信道
            }
        }
    }()

    return out
}


func main() {
    const max = 100 // 找出100以内的所有素数
    nums := xrange() // 初始化一个整数生成器
    number := <-nums  // 从生成器中抓一个整数(2), 作为初始化整数

    for number <= max { // number作为筛子，当筛子超过max的时候结束筛选
        fmt.Println(number) // 打印素数, 筛子即一个素数
        nums = filter(nums, number) //筛掉number的倍数
        number = <- nums  // 更新筛子
    }
}
随机数生成器

信道可以做生成器使用，作为一个特殊的例子，它还可以用作随机数生成器。如下是一个随机01生成器:

func rand01() chan int {
    ch := make(chan int)

    go func () {
        for {
            select { //select会尝试执行各个case, 如果都可以执行，那么随机选一个执行
            case ch <- 0:
            case ch <- 1:
            }
        }
    }()

    return ch
}


func main() {
    generator := rand01() //初始化一个01随机生成器

    //测试，打印10个随机01
    for i := 0; i < 10; i++ {
        fmt.Println(<-generator)
    }
}
定时器

我们刚才其实已经接触了信道作为定时器, time包里的After会制作一个定时器。

看看我们的定时器吧！

/*
 * 利用信道做定时器
 */

package main

import (
    "fmt"
    "time"
)


func timer(duration time.Duration) chan bool {
    ch := make(chan bool)

    go func() {
        time.Sleep(duration)
        ch <- true // 到时间啦！
    }()

    return ch
}

func main() {
    timeout := timer(time.Second) // 定时1s

    for {
        select {
        case <- timeout:
            fmt.Println("already 1s!") // 到时间
            return  //结束程序
        }
    }
}
TODO

Google的应用场景例子。