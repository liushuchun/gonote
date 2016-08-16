## 信号

```
sigRecv1:=make(chan os.Signal,1)
sigs1:=[]os.Signal{syscall.SIGINT,syscall.SIGQUIT}
signal.Notify(sigRecv1,sigs1...)


sigRecv2:=make(chan os.Signal,1)
sigs2:=[]os.Signal{syscall.SIGINT,syscall.SIGQUIT}
signal.Notify(sigRecv2,sigs2...)

// 接着我们用两个for循环来接收消息.考虑到主程序会退出，所以用waitgroup等待完成后退出.

var wg sync.WaitGroup
wg.Add(2)
go func(){
    for sig:=range sigRecv1{
        fmt.Printf("Received a signal from sigRecv1%v",sig)
    }
    fmt.Printf("End. [sigRecv1]\n")
}()


go func(){
    for sig:=range sigRecv2{
        fmt.Printf("Received a signal from sigRecv1%v",sig)
    }
    fmt.Printf("End. [sigRecv2]\n")
}()

wg.Wait()
```
`int socket(int domain,int type,int protocol)`
![image](http://7xrjfe.com1.z0.glb.clouddn.com/ip.png)
`socket type 类型`
![image](http://7xrjfe.com1.z0.glb.clouddn.com/sockettype.png)
## TCP UDP
`tcp 连结类型`

![image](http://7xrjfe.com1.z0.glb.clouddn.com/tcp1.png)
可能会传回io.EOF,表明连接结束
```
var dataBuffer bytes.Buffer
b:=make([byte,10])

for {
    
    n,err:=conn.Read(b)
    if err!=nil{
        if err==io.EOF{
            fmt.Println("The connection is closed.")
            conn.Close()
        } else{
            fmt.Printf("Read Error:%s\n",err)
        }
        break
    }
    dataBuffer.Write(b[:n])

}

```

bufio 是 Buffered I/O缩写.

由于net.Conn类型实现了接口类型io.Reader中的Read接口，所以该接口的类型的一个实现类型。因此，我们可以使用bufio.NewReader函数来包装变量conn,像这样：
 reader:=bufio.NewReader(conn)
 可以调用reader变量的ReadBytes("\n")来接受一个byte类型的参数.
 当然很多情况下并不是查找一个但直接字符那么简单。比如,http协议中规定,消息头部的信息的末尾是两个空行,即是字符串"\r\n\r\n",
 writer:=bufio.NewWriter(conn)来写入数据
 ### close()
 关闭连接
 
 ### LocalAddr & RemoteAddr 方法
 conn.RmeoteAddr().Network() .String()
 
 ### SetDeadline, SetReadDeadline,SetWrteDeadline
 ```
 b:=make([]byte,10)
 conn.SetDeadline(time.Now().Add(2*time.Second))
 for {
    n,err:=conn.Read(b)
 }
 ```
 我们通过调用time.Now()方法当前绝对时间加2s.2s后的那一刻.假设之后的第二次迭代时间超过2s.则第三次迭代尚未开始，就已经超时了，所以改为.
  ```
 b:=make([]byte,10)
 for {
  conn.SetDeadline(time.Now().Add(2*time.Second))

    n,err:=conn.Read(b)
 }
 ```
 
 ### 一个server 的简单实现
 ![image](http://7xrjfe.com1.z0.glb.clouddn.com/servergo.png)
 
 缓冲器中的缓存机制，在很多时候，它会读取比足够多更多一点的数据到其中的缓冲器。会产生提前读取的问题.
 
 net/http在 net/tcp的基础上构建了非常好用的接口，除此以外，标准库，net/rcp中api为我们提供了两个go程序之间建立通讯和交换数据的另外一种方式。
 远程过程调用（remote procedure call）也是基于TCP/IP协议栈的。
 Unix系统中，POSIX标准中定义的线程及其属性和操作方法被广泛认可.
 Linux中称为NPTL。
 
 ## 多线程
 
 线程一个id, TID.
 编写高并发程序的建议：
1.  控制临界区的纯度。临界区中的仅仅包含操作共享数据的代码。
1.  控制临界区的粒度。
1.  减少临界区中代码的执行耗时。
1.  避免长时间的持有互斥变量。
1.  优先使用院子操作而不是互斥量。

GO语言是操作系统提供的内核线程之上搭建了一个特有的两级线程模型。
Goroutine代表的正确的含义：
不要用共享内存的方式来通信。作为替代，应该用通信作为手段来共享内存。
把数据放在共享内存区域中供多个线程中的程序访问的这种方式虽然在基本思想上非常简单，但是却让并发访问控制变得异常复杂。只有做好了很多约束和限制，才有可能让这些简单的方法正确的实施。但是正确性的同时，也需要有可伸缩性。
Go语言不推荐用共享内存区的方式传递数据。作为替代，优先使用Channel。作为多个Goroutine之间的传递数据，并且还会保证其过程的同步。
GO语言线程3个必须知道的核心元素。
M：machine.一个M代表了一个内核线程。
P：processor.一个P代表了M所需的上下文Content.
G:Goroutine.一个G代表了对一段需要被并发执行的GO语言代码的封装。

### GO并发编程.
type IntChan chan int. 
元素类型为int 的通道类型。
这样的通道是双向类型的。

如果向一个nil（被关闭的channel）发送消息，则这个进程会被永久阻塞。
需要注意的是：当我们向一个通道发送消息的时候，我们得到的是一个值的copy，当这个copy形成之后，之后对原来的值的修改都不会影响通道中的值。  

## select例子：
select语句和switch语句不同的是，跟在每个case 后面的只能是针对某个通道的发送语句或者接受语句。
针对每个case 都有初始化了一个通道。通道的类型不受任何约束。元素的类型和容量是任意的。
### 分支选择规则
重做到右，从上到下。
但是当系统发现第一个满足选择条件的case时，运行时系统就会执行该case所包含的语句。这也意味着其他case 会被忽略。如果同时有多个case满足条件，那么运行时，系统会通过一个伪随机算法决定那个case会被执行。例如下面的代码，发送5个范围在【1，3】的整数：
```
package main

import (
    "fmt"
)

func main() {
    chanCap := 5
    ch7 := make(chan int, chanCap)
    for i := 0; i < chanCap; i++ {
        select {
        case ch7 <- 1:
        case ch7 <- 2:
        case ch7 <- 3:
        }

    }
    for i := 0; i < chanCap; i++ {
        fmt.Printf("%v\n", <-ch7)
    }
}

```
但是，如果所有的case 都不满足（并且没有default case），就会阻塞住。直到某个case 被触发。
所以通常情况下，default case 都是必要的.且default只能包含一个default case.
两个变量赋值， e,ok:=<-chan
第二个变量指的是通道是否被关闭。


```
package main

import (
    "fmt"
)

func main() {
    go func(){
        for {
            select{
            case e,ok:=<-ch11:
                if !ok{
                    fmt.Println("End.")
                    break
                }else{
                    fmt.Printf("%d\n",e)ß
                }

            }
        }
    }
}


//修改方案如下，不然会死锁。
func main(){
    go func(){
        var e int
        ok:=true   //声明在外，方便外层使用
        for {
            select{
            case e,ok=<-ch11:
                if !ok{
                    fmt.Println("End.")
                    break

                }else{
                    fmt.Printf("%d\n",e)
               }
            }
        if !ok{
            break
        }
        }
    }
}
```

有的时候我们需要早点关闭流程。这里我们添加新的超时行为。
```
timeout:=make(chan bool,1)
go func(){
    time.Sleep(time.Millisecond)
    timeout<-false
}
```
在原来的基础上，添加
```
go func(){
    var e int
    ok := true
    for {
       select{
       case e,ok=<-ch11:
        ...
        case ok=<-timeout:
        fmt.Println("Timeout.")
        break
       }
}
    if !ok{
     break
    }
}
```
## 非缓冲的Channel
![图片](http://oa8dpdexh.bkt.clouddn.com/2C552445-4DF9-4813-8069-3087049D8011.png)

缓冲通道：由于元素的传递是异步的，所以发送操作在成功向通道发送元素值之后就会立即结束。
非缓冲通道：等待能够接受该元素值的那个接收操作。并且只有确保成功接收，才会真正的完成执行。


```
package main

import (
    "fmt"
    "time"
)

func main() {
    unbufChan := make(chan int)
    go func() {
        fmt.Printf("Sleep a second ...\n")
        time.Sleep(time.Second)
        num := <-unbufChan
        fmt.Printf("Received a integer %d.\n", num)

    }()
    num := 1
    fmt.Printf("Send integer %d ...\n", num)
    unbufChan <- num
    fmt.Printf("Done")
}

```


#### select 语句与非缓冲通道
在使用select语句向某个非缓冲通道发送元素的时候，我们需要打起鸡血。因为，与操作缓冲通道的select语句相比，它被阻塞的概率非常之大。其基本原因依然是非缓冲通道会以同步的方式传递元素值。


## time包与Channel
### 定时器
首先，结构体类型，time.Timer. new的两种方法：
```
time.NewTimer(Duration) & time.AfterFunc.

Duration=3*time.Hour+36*.time.Minute

Timer{
    Reset()
    Stop()
}
//到达时间，是通过字段C（chan time.Time）缓冲通道。 时间一道向自己发送一个time.Time类型元素。绝对到期时间。


```
重构之前的结构：
```
case <-time.NewTimer(time.Millisecond).C:
    fmt.Println("Timeout")
    ok=false
    break
```

定时器是可以被复用的。所以在case中的接受雨具中初始化是浪费的。下面的使用方式更好：
```
go func(){
   var timer *time.Timer
   for{
        select{
            case <-func()<-chan time.Time{
                if timer==nil{
                    timer=time.NewTimer(time.Millisecond)
                }else{
                    timer.Reset(time.Millisedcond)
                }
                return timer.C
            }():
            fmt.Println("Timeout")
            ok=false
            break
        }
   }
}
```

### 断续器
断续器与定时器的使用场景不同，当作超时触发器是不合适的。因为它对到期事件的传达虽然可能被放弃，当绝对不会被延误。断续器一旦被初始化，它所有的到期时间都是固定的了。
固定不变的到期时间恰恰使断续器非常适合被作为定时任务的触发器。
例如要求两次执行之间的最短间隔时间为10分钟。我们可以这样编写满足这一需求的代码：


## 高并发负载均衡器
QPS(Query Per Second,每秒查询量) & TPS(Transactions Per Second,每秒事物处理量)。

切片和数组都不是并发安全的。所以用一个chan来存储结果。

