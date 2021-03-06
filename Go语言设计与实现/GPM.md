# GPM

# GPM模型

## 调度器

### 概述

- 线程是操作系统调度时的最基本单元，而 Linux 在调度器并不区分进程和线程的调度，它们在不同操作系统上也有不同的实现，但是在大多数的实现中线程都属于进程：
- 多个线程可以属于同一个进程并共享内存空间。因为多线程不需要创建新的虚拟内存空间，所以它们也不需要内存管理单元处理上下文的切换，线程之间的通信也正是基于共享的内存进行的，与重量级的进程相比，线程显得比较轻量。
- 虽然线程比较轻量，但是在调度时也有比较大的额外开销。每个线程会都占用 1M 以上的内存空间(goroutine是2k)，在切换线程时不止会消耗较多的内存，恢复寄存器中的内容还需要向操作系统申请或者销毁资源

### 任务窃取调度器

G-P-M模型中引入的处理器p是线程和goroutine的中间层

处理器持有一个由可运行的 Goroutine 组成的环形的运行队列 `runq`，还反向持有一个线程。调度器在调度时会从处理器的队列中选择队列头的 Goroutine 放到线程 M 上执行。如下所示的图片展示了 Go 语言中的线程 M、处理器 P 和 Goroutine 的关系。

![https://img.draveness.me/2020-02-02-15805792666151-golang-gmp.png](https://img.draveness.me/2020-02-02-15805792666151-golang-gmp.png)

问题

- 某些 Goroutine 可以长时间占用线程，造成其它 Goroutine 的饥饿；
- 垃圾回收需要暂停整个程序（Stop-the-world，STW），最长可能需要几分钟的时间，导致整个程序无法工作；

### 抢占式调度器

**基于协作的抢占式调度器**

抢占是通过编译器插入函数实现的，还是需要函数调用作为入口才能触发抢占，所以这是一种**协作式的抢占式调度。**

1. 编译器会在调用函数前插入 `[runtime.morestack](https://draveness.me/golang/tree/runtime.morestack)`；
2. Go 语言运行时会在垃圾回收暂停程序、系统监控发现 Goroutine 运行超过 10ms 时发出抢占请求 `StackPreempt`；
3. 当发生函数调用时，可能会执行编译器插入的 `[runtime.morestack](https://draveness.me/golang/tree/runtime.morestack)`，它调用的 `[runtime.newstack](https://draveness.me/golang/tree/runtime.newstack)` 会检查 Goroutine 的 `stackguard0` 字段是否为 `StackPreempt`；
4. 如果 `stackguard0` 是 `StackPreempt`，就会触发抢占让出当前线程；

**基于信号的抢占式调度器**

解决了垃圾回收和栈扫描时存在的问题

### 数据结构

1. G — 表示 Goroutine，它是一个待执行的任务；
2. M — 表示操作系统的线程，它由操作系统的调度器调度和管理，最多只会有 `GOMAXPROCS`个活跃线程能够正常运行
    
    Go 语言会使用私有结构体 `[runtime.m](https://draveness.me/golang/tree/runtime.m)`表示操作系统线程，其中 g0 是持有调度栈的 Goroutine，`curg`是在当前线程上运行的用户 Goroutine，这也是操作系统线程唯一关心的两个 Goroutine。
    
    g0 是一个运行时中比较特殊的 Goroutine，它会深度参与运行时的调度过程，包括 Goroutine 的创建、大内存分配和 CGO 函数的执行
    
3. P — 表示处理器，它可以被看做运行在线程上的本地调度器；
    
    调度器中的处理器 P 是线程和 Goroutine 的中间层，它能提供线程需要的上下文环境，也会负责调度线程上的等待队列，通过处理器 P 的调度，每一个内核线程都能够执行多个 Goroutine，它能在 Goroutine 进行一些 I/O 操作时及时让出计算资源，提高线程的利用率。
    
    因为调度器在启动时就会创建 `GOMAXPROCS` 个处理器，所以 Go 语言程序的处理器数量一定会等于 `GOMAXPROCS`
    

### 调度循环

- Go 语言有两个运行队列，其中一个是处理器本地的运行队列，另一个是调度器持有的全局运行队列，只有在本地运行队列没有剩余空间时才会使用全局队列。

### 触发调度

- 主动挂起
- 系统调用
- 协作式调度
- 系统监控

## 网络轮询器

网络轮询器是 Go 语言运行时用来处理 I/O 操作的关键组件，它使用了操作系统提供的 I/O 多路复用机制增强程序的并发处理能力

运行时的调度器和系统调用都会通过 `[runtime.netpoll](https://draveness.me/golang/tree/runtime.netpoll)` 与网络轮询器交换消息，获取待执行的 Goroutine 列表，并将待执行的 Goroutine 加入运行队列等待处理。

所有的文件 I/O、网络 I/O 和计时器都是由网络轮询器管理的。

**I/O多路复用**

用来处理同一个事件循环中的多个 I/O 事件，它可以同时阻塞监听了一组文件描述符的状态。

## 系统监控

- 内部启动了一个不会中止的循环，在循环的内部会轮询网络、抢占长期运行或者处于系统调用的 Goroutine 以及触发垃圾回收

休眠时间规则：

- 初始的休眠时间是 20μs；
- 最长的休眠时间是 10ms；
- 当系统监控在 50 个循环中都没有唤醒 Goroutine 时，休眠时间在每个循环都会倍增；

它除了会检查死锁之外，还会在循环中完成以下的工作：

- 运行计时器 — 获取下一个需要被触发的计时器；
- 轮询网络 — 获取需要处理的到期文件描述符；
- 抢占处理器 — 抢占运行时间较长的或者处于系统调用的 Goroutine；
- 垃圾回收 — 在满足条件时触发垃圾收集回收内存；