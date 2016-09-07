## 高级GO并发模式

==============

### GO支持并发
GO原生的内部语法级的就有runtime支持，而非其他语言那样需要外部的库来支持。
这个极大的改变了我们编程的模式。

## goroutine & channel

Goroutines 可以独立执行在一个地址空间：
```go
go f()
go g(1,2)
```
但是goroutine需要交互数据，所以就有了Channels 是有类型的允许goroutine去同步和交换信息：
```
c:=make(chan int)
go func(){c<-3}()
n:=<-c
```

上面的例子只有，c获得数据才会继续，不然会block.

### ping pong的例子

```go
type Ball struct{ hits int }

func main() {
    table := make(chan *Ball)
    go player("ping", table)
    go player("pong", table)

    table <- new(Ball) // game on; toss the ball
    time.Sleep(1 * time.Second)
    <-table // game over; grab the ball
}

func player(name string, table chan *Ball) {
    for {
        ball := <-table
        ball.hits++
        fmt.Println(name, ball.hits)
        time.Sleep(100 * time.Millisecond)
        table <- ball
    }
}

```

如果我们不开始往table中放入数据，那么go会检测到**死锁**，报错。
```go
type Ball struct{ hits int }

func main() {
    table := make(chan *Ball)
    go player("ping", table)
    go player("pong", table)

    // table <- new(Ball) // game on; toss the ball
    time.Sleep(1 * time.Second)
    <-table // game over; grab the ball
}

func player(name string, table chan *Ball) {
    for {
        ball := <-table
        ball.hits++
        fmt.Println(name, ball.hits)
        time.Sleep(100 * time.Millisecond)
        table <- ball
    }
}
```

错误如下：

```
[output]
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
```
上面的如果没有关闭，会造成泄漏，所以要在适合的时机关闭，这里有一种简单的方法，select方法
-----------
可以用panic 来查看它的内存地址：
```go
type Ball struct{ hits int }

func main() {
    table := make(chan *Ball)
    go player("ping", table)
    go player("pong", table)

    table <- new(Ball) // game on; toss the ball
    time.Sleep(1 * time.Second)
    <-table // game over; grab the ball

    panic("show me the stacks")
}

func player(name string, table chan *Ball) {
    for {
        ball := <-table
        ball.hits++
        fmt.Println(name, ball.hits)
        time.Sleep(100 * time.Millisecond)
        table <- ball
    }
}
```

[output]
```go
RunKillClose
pong 1
ping 2
pong 3
ping 4
pong 5
ping 6
pong 7
ping 8
pong 9
ping 10
pong 11
ping 12
panic: show me the stacks

goroutine 1 [running]:
panic(0xff2c0, 0x1040a190)
    /usr/local/go/src/runtime/panic.go:500 +0x720
main.main()
    /tmp/sandbox258996763/main.go:24 +0x1c0
```
----------------------
### Select
长时间运行的程序需要清理。
让我们看一个程序能够处理通讯、周期事件、以及退出任务。
核心是GO的select,有点类似switch。
```go
select {
case xc <- x:
    // 发送x进入xc
case y := <-yc:
    // 接受y从yc管道
}
```

## 订阅的例子
### 找个RSS客户端。
我们在[godoc](https://godoc.org/)搜索例子，有很多建议：
>
// Fetch fetches Items for uri and returns the time when the next
// fetch should be attempted.  On failure, Fetch returns an error.
func Fetch(uri string) (items []Item, next time.Time, err error)
type Item struct{
    Title, Channel, GUID string // a subset of RSS fields
}

但是我要的是stream，而且是多订阅的。
```go
<-chan Item
```

首先定义一个接口：
```go
type Fetcher interface {
    Fetch() (items []Item, next time.Time, err error)
}

func Fetch(domain string) Fetcher {...} // 从urlfetch一些item. 一个接口的实现

type Subscription interface {
    Updates() <-chan Item // item 流
    Close() error         // 关闭流 ,返回一个error如果有错误发生.
}

func Subscribe(fetcher Fetcher) Subscription {...} // 转换一个fetch 为订阅

func Merge(subs ...Subscription) Subscription {...} // merge 很多流.


```
执行：
```go
func main() {
    // 订阅一些，生成一个merged的流。
    merged := Merge(
        Subscribe(Fetch("blog.golang.org")),
        Subscribe(Fetch("googleblog.blogspot.com")),
        Subscribe(Fetch("googledevelopers.blogspot.com")))

    // 等待一段时间就关闭。
    time.AfterFunc(3*time.Second, func() {
        fmt.Println("closed:", merged.Close())
    })

    // 打印该流.
    for it := range merged.Updates() {
        fmt.Println(it.Channel, it.Title)
    }

    panic("show me the stacks")
}
```

订阅：
Subscribe 建立一个新的Subscription 重复地fetches items 直到Close被调用。
```go

func Subscribe(fetcher Fetcher) Subscription {
    s := &sub{
        fetcher: fetcher,
        updates: make(chan Item), // for Updates
    }
    go s.loop()
    return s
}

// sub implements the Subscription interface.
type sub struct {
    fetcher Fetcher   // fetches items
    updates chan Item // delivers items to the user
}

// loop fetches items using s.fetcher and sends them
// on s.updates.  loop exits when s.Close is called.
func (s *sub) loop() {...}
```

implement Subscription

为了实现Subcription 接口，定义Updates 和Close.
```
func (s *sub) Updates() <-chan Item {
    return s.updates
}

func (s *sub) Close() error {
    // TODO: make loop exit loop退出
    // TODO: find out about any error 找到一些错误。
    return err
}
```

loop 做的事情:
1. 周期call Fetch
2. 发送 fetch到的items
3. 退出，并且call Close()接口，report err.

简单的实现
```go
 for {
        if s.closed {
            close(s.updates)
            return
        }
        items, next, err := s.fetcher.Fetch()
        if err != nil {
            s.err = err                 
            time.Sleep(10 * time.Second)
            continue
        }
        for _, item := range items {
            s.updates <- item
        }
        if now := time.Now(); next.After(now) {
            time.Sleep(next.Sub(now))
        }
    }

func (s *naiveSub) Close() error {
    s.closed = true
    return s.err   
}

```

Bug 1: 非同步的访问 s.closed/s.err
```go
  for {
        if s.closed {    //这里
            close(s.updates)
            return
        }
        items, next, err := s.fetcher.Fetch()
        if err != nil {
            s.err = err //这里
            time.Sleep(10 * time.Second)
            continue
        }
        for _, item := range items {
            s.updates <- item
        }
        if now := time.Now(); next.After(now) {
            time.Sleep(next.Sub(now))
        }
    }

func (s *naiveSub) Close() error {
    s.closed = true //这
    return s.err    //这里
}
```

竞态检测器
关于竞态检测器可以看如下的链接：
Introducting the Go Race Detector](https://blog.golang.org/race-detector) [中文翻译版本](http://mikespook.com/2013/06/%E7%BF%BB%E8%AF%91go-%E7%9A%84%E7%AB%9E%E6%80%81%E6%A3%80%E6%B5%8B%E5%99%A8/)

go run -race naivemain.go

Bug 2: time.Sleep 可能保持loop一直运行

```go
 for {
        if s.closed {
            close(s.updates)
            return
        }
        items, next, err := s.fetcher.Fetch()
        if err != nil {
            s.err = err                 
            **time.Sleep(10 * time.Second)**
            continue
        }
        for _, item := range items {
            s.updates <- item
        }
        if now := time.Now(); next.After(now) {
           **time.Sleep(next.Sub(now)) **
        }
    }
```
Bug 3: loop可能被s.updates永久阻塞：
```go
 for {
        if s.closed {
            close(s.updates)
            return
        }
        items, next, err := s.fetcher.Fetch()
        if err != nil {
            s.err = err                 
            time.Sleep(10 * time.Second)
            continue
        }
        for _, item := range items {
            s.updates <- item  //here
        }
        if now := time.Now(); next.After(now) {
            time.Sleep(next.Sub(now))
        }
    }
```

解决方案：
修改Loop body为select以下三种情形：

1. Close调用
2. 调用 Fetch
3. 发送一个item给 s.updates

结构如下：
loop运行它自己的goroutine
select允许loop避免阻塞在任何一个单独的状态.
```go
func (s *sub) loop() {
    ... declare mutable state ...
    for {
        ... set up channels for cases ...
        select {
        case <-c1:
            ... read/write state ...
        case c2 <- x:
            ... read/write state ...
        case y := <-c3:
            ... read/write state ...
        }
    }
}
```

情形1： Close
关闭通讯通过s.closing.
```
type sub struct{
    closing chan chan error
}
```
service(loop)监听s.closing的请求
client(Close) 发送一个request 到s.closing. 退出并返回一个err.

```go
func (s *sub) Close() error {
    errc := make(chan error)
    s.closing <- errc     //ask loop 去结束
    return <-errc         //等待结果。
}
```

```go
var err error // set when Fetch fails
    for {
        select {
        case errc := <-s.closing:
            errc <- err
            close(s.updates) // tells receiver we're done
            return
        }
    }

```



