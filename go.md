# 1.Never start a goroutine without knowing how it will stop

[链接](https://dave.cheney.net/2016/12/22/never-start-a-goroutine-without-knowing-how-it-will-stop)

go中，非常容易高效地创建一个goroutine。上百上千的goroutine被创建也很常见，但是由于太容易使用，也导致了滥用，因为goroutine也会同样耗费一定的内存空间，我们不能无节制的使用他们。

所以每次你创建一个goroutine,你应该知道如何、什么时候这个goroutine会结束，如果你不知道这些，可能会导致潜在的内存泄露。

看一个小例子：

```
ch := somefunction()
go func() {
        for range ch { }
}()
```

长话短说，这个代码依赖于somefunction\(\),程序的结束依赖于somefunction\(\) 的close\(ch\),所以也许ch可能永远都不会关闭，导致goroutine悄无声息的内存泄漏。

我们在设计时，有些goroutine可能直到程序结束才退出，例如一个后台goroutine监控一个配置文件，或者conn.Accept,循环接受请求，这些程序是非常少的，所以不认为他们是一个例外。

# 2.Do not fear first class functions



#  

  


