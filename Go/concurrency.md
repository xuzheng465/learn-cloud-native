#### Go Concurrency Patterns: Pipelines and cancellation



pipline是一系列用channel连接起来的阶段。每个阶段都是一组运行相同函数的goroutine。

在每个阶段，goroutine：

* 通过入站channel接收来自上游的数值
* 对该数据进行一些操作，通常产生新的值
* 通过出站channel向下游发送数据

除了第一个阶段只有出站channel，最后一个阶段只有入站channel，其他每个阶段都有任意个入站channel和出站channel。

第一个阶段称为source（源）或是producer（生产者）。最后一个阶段称为sink（槽）或是consumer（消费者）



简单的例子，数的平方

一个三阶段的pipeline

第一阶段gen，功能是将一列整数转换成channel，这个channel可以将这列整数输出。

gen函数启动一个goroutine，在通道上发送整数，并在所有数值发送完毕后关闭该通道。

```go
func gen(nums ...int) <-chan int {
  out := make(chan int)
  go func() {
    for _, n := range nums {
      out <- n
    }
    close(out)
  }()
  return out
}
```



第二阶段，sq，从一个通道接收整数，并返回一个通道，发射每个接收的整数的平方。在入站通道关闭后，这个阶段将所有的值发送到下游，它将关闭出站通道。

```go
```

