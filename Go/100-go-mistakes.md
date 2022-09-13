

## Ch 8 Concurrency: Foundations



### 8.1 #55 concurrency vs parallelism

Unlike parallelism, which is about doing the same thing multiple times at once, concurrency is about structure.

并发与并行不同，并行是指同时多次做同一件事，而并发是关于结构。



Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once.

Concurrency is about dealing with things happening out-of-order

Parallelism is about things actually happening at the same time.



From "Concurrency in Go", cocurrency is a property of the code; paralleism is a property of the running program.

We do not write ==parallel== code, only ==concurrent== code that we hope will be run in parallel.

我们不写并行（parallel）代码，只写并发（concurrency）代码，并希望代码能并行地(in parallel) 运行

Parallelism is a property of the **runtime** of our program.



### 8.2 #56 Thinking concurrency is always faster

The overall performance of a solution depends on many factors, such as 

* the efficiency of our structure (concurrency), 
* which parts can be tackled in parallel, 
* and the level of contention among the computation units.



OS级线程是由操作系统在CPU核上进行上下文切换的，

goroutine是由Go Runtime在操作系统线程上进行上下文切换的。



If we’re not sure that a parallel version will be faster, the right approach may be to start with a simple sequential version and build from there using profiling and benchmarks.

如果我们不确定并行版本是否会更快，正确的方法可能是先从简单的顺序版本开始，然后使用剖析和基准测试来建立。

<img src="../assets/go-concurrency-mech.jpg" alt="go-concurrency-mech" width="800px"/>

### 8.3 #57 channel vs mutex

Whenever we want to share a state or access a shared resource, mutexes ensure exclusive access to this resource. Conversely, channels are a mechanic for signaling with or without data (chan struct{} or not). Coordination or ownership transfer should be achieved via channels.

每当我们想共享一个状态或访问一个共享资源时，mutex就会确保对这个资源的独占访问。反之，channel是一种机制，用于发送有或无数据的信号（chan struct{}或无数据）。协调或所有权的转移应该通过通道来实现



### 8.4 #58 race problem

`-race`

If the effects of a goroutine must be observed by another goroutine, use a synchronization mechanism such as a **lock** or **channel** communication to establish a relative ordering.

```go
i := 0
ch := make(chan struct{})
go func() {
  <- ch
  fmt.Println(i)
}()
i++
ch <- struct{}{}
```

var inc < chan send < chan rec < var read



### 8.6 #59 workload type

CPU-bound

IO-bound

memory-bound

If the workload is **I/O-bound**, the answer mainly depends on the external system. How many concurrent accesses can the system cope with if we want to maximize throughput?

If the workload is **CPU-bound**, a best practice is to rely on GOMAXPROCS.



### 8.7 #60: Misunderstanding Go contexts

A Context carries a deadline, a cancellation signal, and other values across API boundaries



`context.Context` exports an `Err` method that returns `nil` if the `Done` channel isn't yet closed.

It returns a non-nil error explaining why the `Done` channel was closed

综上所述，要成为一名熟练的Go开发者，我们必须了解什么是**context**以及如何使用它。在Go中，`context.Context`在标准库和外部库中无处不在。正如我们所提到的，一个context允许我们携带一个最后期限、一个取消信号和/或一个键值列表。一般来说，用户等待的函数应该带一个上下文，因为这样做可以让上游的调用者决定什么时候应该中止调用这个函数。





### #66 not using nil channels



```go
func merge(ch1, ch2 <- chan int) <- chan int {
  ch := make(chan int, 1)
  
  go func() {
    for ch1 != nil || ch2 != nil {
      select {
      case v, open := <-ch1:
        if !open {
          ch1 = nil
          break
        }
        ch <-v
       case v, open := <-ch2:
        if !open {
          ch2 = nil
          break
        }
        ch <- v
      }
    }
    close(ch)
  }()
  
  return ch
}

```

设置为nil的好处是select不会反复检查这个为nil的channel

我们可以使用nil通道来实现一个优雅的状态机，它将从选择语句中删除一个分支。



### #67: Being puzzled about channel size

同步用unbuffered channel



### #68: forgetting about possible side effects with string formatting

`String()` 方法可能会造成data race



### #69: creating data races with append

在并发情况下处理slice时，我们必须记住，在slice上使用append并不总是race-free的。根据分片的情况和它是否满了，行为会发生变化。如果分片是满的，append是无竞争的。否则，多个goroutines可能会竞争更新同一个数组索引，从而导致数据竞争。



### #70: using mutexes inaccurately with slices and maps

当对map操作，只有mutex加锁，因为map var只是个只像runtime.hmap的指针

两个方法解决：

1. 对整个方法加锁
2. 复制map，操作新map



### #71: misusing `sync.WaitGroup`

```go
// correct way 1
func listing() {
  wg := sync.WaitGroup{}
  var v uint64
  wg.Add(3)
  for i:=0; i<3; i++ {
    go func() {
      atomic.AddUint64(&v, 1)
      wg.Done()
    }()
  }
  wg.Wait()
  fmt.Println(v)
}
// correct way 2
func listing1() {
  wg := sync.WaitGroup{}
  var v uint64
  for i:=0; i<3; i++ {
    wg.Add(1)
    go func() {
      atomic.AddUint64(&v, 1)
      wg.Done()
    }()
  }
  wg.Wait()
  fmt.Println(v)
}
```

### #72: Forgetting about `sync.Cond`

我们需要找到一种方法，每当余额被更新到多个goroutines时，重复广播通知。幸运的是，Go有一个解决方案：sync.Cond。



Cond实现了一个条件变量，是等待或宣布一个事件发生的goroutines的汇合点。



#73: Not using errgroup

首先，我们通过提供父级上下文创建一个`*errgroup.Group`。在每个迭代中，我们使用`g.Go`来触发一个新的goroutine中的调用。这个方法将一个`func() error`作为输入，用一个闭包来包裹对foo的调用，并处理结果和错误。作为与我们第一个实现的主要区别，如果我们得到一个错误，我们会从这个闭包中返回它。然后，`g.Wait`允许我们等待所有的goroutine完成。

另一个好处就是共享上下文。假如我们要触发三个并行的调用

第一个在1ms以内返回一个错误

第二个和第三个在5s以内返回结果或错误

我们想返回一个错误，如果有的话。因此，没有必要等到第二和第三次调用完成。使用`errgroup.WithContext`创建一个共享上下文，用于所有的并行调用。因为第一次调用在1毫秒内返回一个错误，它将取消上下文，从而取消其他`goroutines`。因此，我们不必等待5秒来返回错误。这是使用`errgroup`时的另一个好处。

g.Go所调用的进程必须**context aware**。否则，取消上下文不会有任何影响。



#74: Copying a sync type

作为一个经验法则，当多个goroutines需要访问一个共同的同步元素时，我们必须确保它们都依赖于同一个实例。这条规则适用于同步包中定义的所有类型。使用指针是解决这个问题的一种方法：我们可以有一个指向同步元素的指针，或者一个指向包含同步元素的结构的指针。
