---
date: 2014-03-30
title: Go并发模式：管道和取消
---
由[Air on G](http://air.googol.im/2014/03/15/go-concurrency-patterns-pipelines-and-cancellation.html)翻译.

译自[](http://blog.golang.org/pipelines)[http://blog.golang.org/pipelines](http://blog.golang.org/pipelines)。

这是 Go 官方 blog 的一篇文章，介绍了如何使用 Go 来编写并发程序，并按照程序的演化顺序，介绍了不同模式遇到的问题以及解决的问题。主要解释了用管道模式链接不同的线程，以及如何在某个线程取消工作时，保证所有线程以及管道资源的正常回收。


Go 并发模式：管道和取消

作者：Sameer Ajmani，[blog.golang.org](http://blog.golang.org)，写于 2014 年 3 月 13 日。

### 介绍

Go 本身提供的并发特性，可以轻松构建用于处理流数据的管道，从而高效利用 I/O 和多核 CPU。这篇文章就展示了这种管道的例子，并关注当操作失败时要处理的一些细节，并介绍了如何干净的处理错误的技巧。

### 什么是管道？

Go 语言里没有明确定义管道，而只是把管道当作一类并发程序。简单来说，管道是一系列由 channel 联通的状态（stage），而每个状态是一组运行相同函数的 Goroutine。每个状态上，Goroutine

*   通过流入（inbound）channel 接收上游的数值
*   运行一些函数来处理接收的数据，一般会产生新的数值
*   通过流出（outbound）channel 将数值发给下游

每个语态都会有任意个流入或者流出 channel，除了第一个状态（只有流出 channel）和最后一个状态（只有流入 channel）。第一个状态有时被称作源或者生产者；最后一个状态有时被称作槽（sink）或者消费者。

我们先从一个简单的管道例子开始解释这些想法和技术。之后，我们再来看一些更真实的例子。

### 求平方数

考虑一个管道和三个状态。

第一个状态，`gen`，是一个将一系列整数一一传入 channel 的函数。`gen`函数启动一个 Goroutine，将整数数列发送给 channel，如果所有数都发送完成，关闭这个 channel：

```
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

第二个状态，`sq`，从一个 channel 接收整数，并求整数的平方，发送给另一个 channel。当流入 channel 被关闭，而且状态已经把所有数值都发送给了下游，关闭流出 channel：

```
func sq(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}
```

主函数建立起管道，并执行最终的状态：从第二个状态接收所有的数值并打印，直到 channel 被关闭：

```
func main() {
    // 建立管道
    c := gen(2, 3)
    out := sq(c)

    // 产生输出
    fmt.Println(<-out) // 4
    fmt.Println(<-out) // 9
}
```

因为`sq`有相同类型的流入和流出 channel，我们可以将其组合任意次。我们也可以将`main`函数写成和其他状态类似的范围循环的形式：

```
func main() {
    // 建立管道并产生输出
    for n := range sq(sq(gen(2, 3))) {
        fmt.Println(n) // 16 和 81
    }
}
```

### 扇出，扇入

多个函数可以同时从一个 channel 接收数据，直到 channel 关闭，这种情况被称作_扇出_。这是一种将工作分布给一组工作者的方法，目的是并行使用 CPU 和 I/O。

一个函数同时接收并处理多个 channel 输入并转化为一个输出 channel，直到所有的输入 channel 都关闭后，关闭输出 channel。这种情况称作_扇入_。

我们可以将我们的管道改为同时执行两个`sq`实例，每个都从同样的输入 channel 读取数据。我们还引入新函数，`merge`，来扇入所有的结果：

```
func main() {
    in := gen(2, 3)

    // 在两个从 in 里读取数据的 Goroutine 间分配 sq 的工作
    c1 := sq(in)
    c2 := sq(in)

    // 输出从 c1 和 c2 合并的数据
    for n := range merge(c1, c2) {
        fmt.Println(n) // 4 和 9, 或者 9 和 4
    }
}
```

`merge`对每个流入 channel 启动一个 Goroutine，并将流入的数值复制到流出 channel，由此将一组 channel 转换到一个 channel。一旦启动了所有的`output` Goroutine，`merge`函数会多启动一个 Goroutine，这个 Goroutine 在所有的输入 channel 输入完毕后，关闭流出 channel。

往一个已经关闭的 channel 输出会产生异常（panic），所以一定要保证所有数据发送完成后再执行关闭。[`sync.WaitGroup`](http://golang.org/pkg/sync/#WaitGroup)类型提供了方便的方法，来保证这种同步：

```
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // 为 cs 中每个输入 channel 启动输出 Goroutine。output 从 c 中复制数值，直到 c 被关闭
    // 之后调用 wg.Done
    output := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // 启动一个 Goroutine，当所有 output Goroutine 都工作完后（wg.Done），关闭 out，
    // 保证只关闭一次。这个 Goroutine 必须在 wg.Add 之后启动
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

### 突然关闭

我们的管道函数里有个模式：

*   状态会在所有发送操作做完后，关闭它们的流出 channel
*   状态会持续接收从流入 channel 输入的数值，直到 channel 关闭

这个模式使得每个接收状态可以写为一个`range`循环，并保证所有的 Goroutine 在将所有的数值发送成功给下游后立刻退出。

但是实际的管道，状态不能总是接收所有的流入数值。有时这是设计决定的：接收者可能只需要一部分数值做进一步处理。更常见的情况是，一个状态会由于从早先的状态流入的数值有误而退出。不管哪种情况，接收者都不应该继续等待剩下的数值，而且我们希望早先的状态可以停止生产后续状态不需要的数据。

在我们的管道例子里，如果一个状态无法处理所有的流入数值，试图发送那些数值的 Goroutine 会被永远阻塞住：

```
    // 处理输出的第一个数值
    out := merge(c1, c2)
    fmt.Println(<-out) // 4 或者 9
    return
    // 由于我们不再接收从 out 输出的第二个数值，其中一个输出 Goroutine 会由于试图发送数值而挂起
}
```

这是资源泄漏：Goroutine 会占用内存和运行时资源，而且 Goroutine 栈里的堆引用会一直持有数据，这些数据无法被垃圾回收。Goroutine 本身也无法被垃圾回收，它们必须靠自己退出（而不是被其他人杀死）。

即便下游的状态无法接收所有的流入数值，我们依然需要让管道里的上游状态正常退出。一种方法是修改流出 channel，使其含有缓冲区。缓冲区可以持有固定数量的数值，当缓冲区有空间时，发送操作会立刻完成（，不会产生阻塞）。

在创建 channel 时，如果已经知道要发送数值的数量，缓冲区可以简化代码。比如，我们可以让`gen`把整数列表里的数复制进 channel 缓冲区，而不需使用新的 Goroutine：

```
func gen(nums ...int) <-chan int {
    out := make(chan int, len(nums))
    for _, n := range nums {
        out <- n
    }
    close(out)
    return out
}
```

回到我们管道的阻塞问题上来，我们可以考虑给`merge`的流出 channel 加上缓冲区：

```
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int, 1) // 1 个空间足够应付未读的输入
    // ... 其余未变 ...
```

这个改动当然修正了程序中阻塞 Goroutine 的问题，但这不是好的代码。缓冲区的大小为 1，依赖于我们已经知道我们将要`merge`的数值总数和下游状态要处理的数值总数。这太脆弱了：如果我们从`gen`传入额外的数值，或者下游状态再多读一些数值，我们仍将看到 Goroutine 被阻塞住了。

不使用缓冲区的话，我们需要提供一种方法，让下游状态通知发送者，下游状态将停止接收输入。

### 显式取消

当`main`要在不接收所有来自`out`的数值前退出，就需要告诉所有上游状态的 Goroutine，放弃尝试发送数值的行为。这可以通过发送数值到一个叫做`done`的 channel 来完成。例子里有两个潜在的会被阻塞的发送者，所以给`done`发送了两个数值：

```
func main() {
    in := gen(2, 3)

    // 发布 sq 的工作到两个都从 in 里读取数据的 Goroutine
    c1 := sq(in)
    c2 := sq(in)

    // 处理来自 output 的第一个数值
    done := make(chan struct{}, 2)
    out := merge(done, c1, c2)
    fmt.Println(<-out) // 4 或者 9

    // 通知其他发送者，该退出了
    done <- struct{}{}
    done <- struct{}{}
}
```

发送 Goroutine 将发送操作替换为一个`select`语句，要么把数据发送给`out`，要么处理来自`done`的数值。`done`的类型是个空结构，因为具体数值并不重要：接收事件本身就指明了应当放弃继续发送给 out 的动作。而`output` Goroutine 会继续循环处理流入的 channel，`c`, 而不会阻塞上游状态：


```
func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // 为每个 cs 中的输入 channel 启动一个 output Goroutine。outpu 从 c 里复制数值直到 c 被关闭
    // 或者从 done 里接收到数值，之后 output 调用 wg.Done
    output := func(c <-chan int) {
        for n := range c {
            select {
            case out <- n:
            case <-done:
            }
        }
        wg.Done()
    }
    // ... 其余的不变 ...
```

但是这种方法有个问题：下游的接收者需要知道潜在会被阻塞的上游发送者的数量。追踪这些数量不仅枯燥，还容易出错。

我们需要一种方法，让不知道也不限制数量的 Goroutine，停止往它们下游发送数据的行为。在 Go 里，我们可以通过关闭 channel 来实现这个工作，因为[channel 被关闭时，接收工作会立刻执行，并产生一个符合类型的 0 值](http://golang.org/ref/spec#Receive_operator)。

这就是说，`main`可以容易的通过关闭`done`channel 来释放所有的发送者。关闭是个高效的发送给所有发送者的广播信号。我们扩展管道里的每个函数，让其以参数方式接收`done`，并通过`defer`语句在函数退出时执行关闭操作，这样`main`里所有的退出路径都会触发管道里的所有状态退出。

```
func main() {
    // 构建 done channel，整个管道里分享 done，并在管道退出时关闭这个 channel
    // 以此通知所有 Goroutine 该推出了。
    done := make(chan struct{})
    defer close(done)

    in := gen(done, 2, 3)

    // 发布 sq 的工作到两个都从 in 里读取数据的 Goroutine
    c1 := sq(done, in)
    c2 := sq(done, in)

    // 处理来自 output 的第一个数值
    out := merge(done, c1, c2)
    fmt.Println(<-out) // 4 或者 9

    // done 会通过 defer 调用而关闭
}
```

管道里的每个状态现在都可以随意的提早退出了：`sq`可以在它的循环中退出，因为我们知道如果`done`已经被关闭了，也会关闭上游的`gen`状态。`sq`通过`defer`语句，保证不管从哪个返回路径，它的`out` channel 都会被关闭。

```
func sq(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-done:
                return
            }
        }
    }()
    return out
}

```


下面列出了构建管道的指南：

*   状态会在所有发送操作做完后，关闭它们的流出 channel
*   状态会持续接收从流入 channel 输入的数值，直到 channel 关闭或者其发送者被释放。

管道要么保证足够能存下所有发送数据的缓冲区，要么接收来自接收者明确的要放弃 channel 的信号，来保证释放发送者。

### 对目录做摘要

来考虑一个更现实的管道。

MD5 是一个摘要算法，经常在对文件的校验的时候使用。命令行上使用`md5sum`来打印出一系列文件的摘要数值。

我们的程序类似`md5sum`，但是参数是一个目录，之后会打印出这个目录下所有常规文件的摘要值，以文件路径名排序。

我们的主函数包含一个`MD5All`的辅助函数，返回一个路径名到摘要值的映射，之后排序并打印结果：

```
func main() {
    // 计算指定目录下所有文件的 MD5 值，之后按照目录名排序并打印结果
    m, err := MD5All(os.Args[1])
    if err != nil {
        fmt.Println(err)
        return
    }
    var paths []string
    for path := range m {
        paths = append(paths, path)
    }
    sort.Strings(paths)
    for _, path := range paths {
        fmt.Printf("%x  %s\n", m[path], path)
    }
}
```

`MD5All`函数是我们讨论的焦点。在[`serial.go`](http://blog.golang.org/pipelines/serial.go)文件里，是非并发的函数实现，再扫描目录树时简单读取并计算每个文件。

```
// MD5All 读取文件目录 root 下所有文件，并返回从文件路径到文件内容 MD5 值的映射。如果扫描目录
// 出错或者任何操作失败，MD5All 返回失败。
func MD5All(root string) (map[string][md5.Size]byte, error) {
    m := make(map[string][md5.Size]byte)
    err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        if info.IsDir() {
            return nil
        }
        data, err := ioutil.ReadFile(path)
        if err != nil {
            return err
        }
        m[path] = md5.Sum(data)
        return nil
    })
    if err != nil {
        return nil, err
    }
    return m, nil
}
```

### 并行摘要

在[`parallel.go`](http://blog.golang.org/pipelines/parallel.go)里，我们把`MD5All`分解为两个状态的管道。第一个状态，`sumFiles`，遍历目录，在一个新的 Goroutine 里对每个文件做摘要，并把结果发送到类型为`result`的 channel：

```
type result struct {
    path string
    sum  [md5.Size]byte
    err  error
}
```

`sumFiles`返回两个 channel：一个用来传递`result`，另一个用来返回`filepath.Walk`的错误。遍历函数启动一个新的 Goroutine 来处理每个常规文件，之后检查`done`。如果`done`已经被关闭了，遍历就立刻停止：

```
func sumFiles(done <-chan struct{}, root string) (<-chan result, <-chan error) {
    // 对每个常规文件，启动一个 Goroutine 计算文件内容并发送结果到 c。发送 walk 的结果到 errc
    c := make(chan result)
    errc := make(chan error, 1)
    go func() {
        var wg sync.WaitGroup
        err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if info.IsDir() {
                return nil
            }
            wg.Add(1)
            go func() {
                data, err := ioutil.ReadFile(path)
                select {
                case c <- result{path, md5.Sum(data), err}:
                case <-done:
                }
                wg.Done()
            }()
            // 如果 done 被关闭了，停止 walk
            select {
            case <-done:
                return errors.New("walk canceled")
            default:
                return nil
            }
        })
        // walk 已经返回，所有 wg.Add 的工作都做完了。开启新进程，在所有发送完成后
        // 关闭 c。
        go func() {
            wg.Wait()
            close(c)
        }()
        // 因为 errc 有缓冲区，所以这里不需要 select。
        errc <- err
    }()
    return c, errc
}
```

`MD5All`从`c`接收所有的摘要值。`MD5All`返回早先的错误，通过`defer`关闭`done`：

```
func MD5All(root string) (map[string][md5.Size]byte, error) {
    // MD5All 在返回时关闭 done channel；这个可能在从 c 和 errc 收到所有的值之前被调用
    done := make(chan struct{})
    defer close(done)

    c, errc := sumFiles(done, root)

    m := make(map[string][md5.Size]byte)
    for r := range c {
        if r.err != nil {
            return nil, r.err
        }
        m[r.path] = r.sum
    }
    if err := <-errc; err != nil {
        return nil, err
    }
    return m, nil
}
```

### 受限的并发

在[`parallel.go`](http://blog.golang.org/pipelines/parallel.go)里实现的`MD5All`对每个文件启动一个新的 Goroutine。如果目录里含有很多大文件，这可能会导致申请大量内存，超出机器上的可用内存。

我们可以通过控制并行读取的文件数量来限制内存的申请。在[`bounded.go`](http://blog.golang.org/pipelines/bounded.go)，我们创建固定数量的用于读取文件的 Goroutine，来限制内存使用。现在整个管道有三个状态：遍历树，读取并对文件做摘要，收集摘要值。

第一个状态，`walkFiles`，发送树里的每个常规文件的路径：

```
func walkFiles(done <-chan struct{}, root string) (<-chan string, <-chan error) {
    paths := make(chan string)
    errc := make(chan error, 1)
    go func() {
        // 在 Walk 之后关闭 paths channel
        defer close(paths)
        // 因为 errc 有缓冲区，所以这里不需要 select。
        errc <- filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if info.IsDir() {
                return nil
            }
            select {
            case paths <- path:
            case <-done:
                return errors.New("walk canceled")
            }
            return nil
        })
    }()
    return paths, errc
}
```

中间的状态启动固定数量的`digester` Goroutine，从`paths`接收文件名，并将结果`result`发送到 channel `c`：

```
func digester(done <-chan struct{}, paths <-chan string, c chan<- result) {
    for path := range paths {
        data, err := ioutil.ReadFile(path)
        select {
        case c <- result{path, md5.Sum(data), err}:
        case <-done:
            return
        }
    }
}
```

不象之前的例子，`digester`并不关闭输出 channel，因为多个 Goroutine 会发送到共享的 channel。另一边，`MD5All`中的代码会在所有`digester`完成后关闭 channel：

```
   // 启动固定数量的 Goroutine 来读取并对文件做摘要。
    c := make(chan result)
    var wg sync.WaitGroup
    const numDigesters = 20
    wg.Add(numDigesters)
    for i := 0; i < numDigesters; i++ {
        go func() {
            digester(done, paths, c)
            wg.Done()
        }()
    }
    go func() {
        wg.Wait()
        close(c)
    }()
```

我们也可以让每个`digester`创建并返回自己的输出 channel，但是这就需要一个单独的 Goroutine 来扇入所有结果。

最终从`c`收集到所有结果`result`，并检查从`errc`传入的错误。这个错误的检查不能提早，因为在这个时间点之前，`walkFiles`可能会因为正在发送消息给下游而阻塞：

```
    m := make(map[string][md5.Size]byte)
    for r := range c {
        if r.err != nil {
            return nil, r.err
        }
        m[r.path] = r.sum
    }
    // 检查 Walk 是否失败
    if err := <-errc; err != nil {
        return nil, err
    }
    return m, nil
}
```

### 结论

这篇文章展示了使用 Go 构建流数据管道的技术。要慎重处理这种管道产生的错误，因为管道里的每个状态都可能因为向下游发送数值而阻塞，而下游的状态却不再关心输入的数据。我们展示了如何将关闭 channel 作为“完成”信号广播给所有由管道启动的 Goroutine，并且定义了正确构建管道的指南。

进一步阅读：

[Go 并发模式](http://talks.golang.org/2012/concurrency.slide#1)（[视频](https://www.youtube.com/watch?v=f6kdp27TYZs)）展示了 Go 的并发特性的基础知识，并演示了应用这些知识的方法。

[高级 Go 并发模式](http://blog.golang.org/advanced-Go-concurrency-patterns)（[视频](http://www.youtube.com/watch?v=QDDwwePbDtw)）覆盖了关于 Go 特性更复杂的使用场景，尤其是 select。

Douglas McIlroy 的论文[《一窥级数数列》](http://swtch.com/~rsc/thread/squint.pdf)展示了 Go 使用的这类并发技术是如何优雅地支持复杂计算
