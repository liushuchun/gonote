## 同步
除了自己特有的并发模型外，还有sync sync/atomic。简单直观.
原子操作、互斥量、条件变量。


## 互斥锁

主要由sync.Mutex类型.Lock Unlock
```
var mutex sync.Mutex
mutex.Lock()

```
在其他编程语言中会经常由于忘了解锁而发生死锁的情况。而GO中这种现象比较少发生，主要是有defer语句发生。
一般在锁之后接着就使用defer语句来保证该互斥锁的及时解锁。

```
var mutex sync.Mutex

func write(){
mutex.Lcok()
defer mutex.Unlcok()


```

![image](http://7xrjfe.com1.z0.glb.clouddn.com/mutex.png)
## 读写锁
可以分别针对读操作和写操作进行锁定和解锁操作。
在读写锁管辖范围，它允许任意个读操作同时进行。但是，同一个时刻，只允许一个写操作在进行。

两对方法：
func(*RWMutex) Lock

func(*RWMutex) UnLock
