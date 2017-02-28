# 池的概念

连接池、线程池我的理解就是服务员或者小姐重复多次接客，而非接一次客就被回收了。



在Golang的_**sync**_包中，有一个_**Pool**_结构，乍一看容易以为就是一个容器，类似于共享内存，而实际上理解它不能从共享内存这样简单的角度，而是要从其名字出发，即所谓_**池**_。

工程开发中我们常听的有几种池，比如连接池和线程池，大多数时候都与资源申请和分配有关。比如数据库连接池，由于建立TCP连接消耗比较大，比较合适的处理方法是连接一个缓存池，池内包含一定数量的已经建立好的数据库连接引用，客户端在需要进行数据库操作时直接从池中取出一个连接引用，而不用再进行高消耗的建立握手连接的过程。

# sync包的Pool {#syncpool}

Golang中的`sync.Pool`正是用于实现这样的池功能。有几点非常关键的：

1. 池中的数据应该是临时的，随时有可能被GC清除（手动或自动）
2. _**func \(p \*Pool\) Get**_，从池中取走一个项目，即从池中去除的该项目并返回给调用方
3. 如果_**Get**_返回nil，并且`p.New`非nil，_**Get**_会返回`p.New`的返回结果
4. _**func \(p \*Pool\) Put**_，往池中放入一个项目
5. 池是一个可被共享的资源，这也意味着客户端刚刚_**Put**_进去的项目不一定就是被_**Get**_出来的结果

所以Pool的New成员应该是包含了资源的创建的逻辑，比如一个数据库连接池没有任何连接实例引用时，调用_**Get**_会去调用_**New**_，这时应该是在_**New**_中创建数据库连接并返回给到客户端，而客户端合适的行为应该是完成数据库操作后把连接_**Put**_到池中。

# 应用

池的应用应该说比较多也比较常用，并不局限与我们比较熟悉的连接池、线程池。[Godocs\#Pool](http://godoc.golangtc.com/pkg/sync/#Pool)中就举了一个例子，fmt维护了一个输出缓冲区池，当多个goroutine要执行输出时就可以从这个池中取出资源。

其中有句话我觉得是比较重要的：

> An appropriate use of a Pool is to manage a group of temporary items silently shared among and potentially reused by concurrent independent clients of a package. Pool provides a way to amortize allocation overhead across many clients.

确实是如此，一方面我们有New功能将默认创建资源的操作内聚到Pool中，而这个池又可以根据当前被取用的频率情况动态伸缩，因为当_**Get**_不到时就会去_**New**_并_**Put**_入池中，而当一段时间后空闲时又由GC来清理池中的资源。

不得不说，在平常的工程中，多注意注意哪里适合用到池的概念，或许可以帮助我们节省不少资源消耗。



