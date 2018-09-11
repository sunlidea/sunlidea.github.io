---
title: go pprof
urlname: golang-pprof
date: 2018-08-15 14:56:54
tags:
categories:
---

# GO pprof

Profiling工具是Go语言原生的程序分析工具之一，常用来分析Go程序的复杂性和成本，例如其内存使用情况和经常调用的函数，以识别出Go程序中开销比较大的部分。

本文详细介绍Profiling工具的使用。

## 资料1

[资料1](https://golang.org/doc/diagnostics.html#profiling)

可以通过两种方式获取分析数据，一种是利用go test测试程序，另一种就是直接调用net /http/pprof包对程序进行分析。Go运行时(runtime)按照pprof可视化工具所期望的格式提供分析数据。

runtime/pprof包提供了下面几种预定义的分析配置文件Profile：

- cpu：CPU配置文件确定哪一部分程序主动消耗CPU资源（而不是在休眠或等待I/O时）。

- heap：堆配置文件报告内存分配样本;用于监视当前和历史内存使用情况，以及检查内存泄漏。

- threadcreate：线程创建配置文件报告哪一部分程序导致了新OS线程的创建。

- goroutine：Goroutine配置文件报告了所有当前goroutine的堆栈跟踪。

- block：block配置文件显示goroutine阻塞等待同步原语（包括定时器通道）的位置。默认情况下该配置文件不启用;可以调用runtime.SetBlockProfileRate启用它。

- mutex：互斥配置文件报告锁的争用。如果你认为由于互斥争用而未充分利用CPU，可以使用此配置文件。默认情况下不启用互斥配置文件，请参阅runtime.SetMutexProfileFraction以启用它。

### 除了原生的pprof工具，还有哪些分析器可以来分析Go程序？

在Linux上，perf工具可用于分析Go程序。 Perf可以剖析和展开cgo / SWIG代码和内核，因此深入了解本机/内核性能瓶颈非常有用。在macOS上，Instruments套件可以使用profile Go程序。

### 可以在生产环境中启用分析工具吗？

是。在生产中对程序进行概要分析是安全的，但启用某些配置文件（例如CPU配置文件）会增加成本。您应该会看到性能降级。可以通过在生产中打开探测器之前测量探测器的开销来估计性能损失。

您可能希望定期分析您的生产服务。特别是在具有单个过程的许多副本的系统中，定期选择随机副本是安全的选择。选择一个生产过程，每隔Y秒将其分析X秒并保存结果以进行可视化和分析;然后定期重复。可以手动和/或自动检查结果以发现问题。配置文件的收集可能会相互干扰，因此建议一次只收集一个配置文件。

### 可视化分析数据的最好方式？

可以利用 go tool pprof提供配置文件数据的文本，图形和callgrind可视化。

另一种可视化分析数据的方法是火焰图。 火焰图允许特定的祖先路径中移动，因此可以放大/缩小特定的代码段。 [The upstream pprof](https://github.com/google/pprof)已经支持火焰图了。

### 什么是Profile

[profile资料参考](https://golang.org/pkg/runtime/pprof/#Profile)

a profile是堆栈跟踪的集合，显示导致特定事件（例如分配）实例的调用序列。包可以创建和维护自己的配置文件;最常见的用途是跟踪必须显式关闭的资源，例如文件或网络连接。

#### heap Profile

堆配置文件报告最近完成的垃圾回收的统计信息;它省略了更多最近的分配，以避免将配置文件从实时数据偏向垃圾。如果根本没有垃圾收集，堆配置文件将报告所有已知的分配。此异常主要有助于在没有启用垃圾回收的情况下运行的程序，通常用于调试目的。

堆配置文件既追踪当前内存中存活的对象，也追踪自程序启动以来分配的所有对象。 Pprof的-inuse_space，-inuse_objects，-alloc_space和-alloc_objects标志选择要显示的内容，默认为-inuse_space。

#### cpu profile的特殊性？？

CPU配置文件不可用作配置文件。它有一个特殊的API，StartCPUProfile和StopCPUProfile函数，因为它在分析期间将输出流式传输到writer

#### 是否只能使用内置配置文件？

除了运行时runtime提供的之外，用户还可以通过pprof.Profile创建自定义配置文件，并使用现有工具对其进行检查。

## 资料二

[资料二](https://blog.golang.org/profiling-go-programs)

### CPU profile

```

(pprof) top10
Total: 2525 samples
     298  11.8%  11.8%      345  13.7% runtime.mapaccess1_fast64
     268  10.6%  22.4%     2124  84.1% main.FindLoops
     251   9.9%  32.4%      451  17.9% scanblock
     178   7.0%  39.4%      351  13.9% hash_insert
     131   5.2%  44.6%      158   6.3% sweepspan
     119   4.7%  49.3%      350  13.9% main.DFS
      96   3.8%  53.1%       98   3.9% flushptrbuf
      95   3.8%  56.9%       95   3.8% runtime.aeshash64
      95   3.8%  60.6%      101   4.0% runtime.settype_flush
      88   3.5%  64.1%      988  39.1% runtime.mallocgc

```

如上，是执行havlak1的cpu分析数据。启用CPU分析时，Go程序每秒停止大约100次，并记录一份包含在当前正在执行的goroutine堆栈上的程序计数器的样本。该配置文件有2525个样本，因此运行时间超过25秒。在`go tool pprof`输出中，样本中出现的每个函数都有一行。前两列显示函数运行的样本数（而不是等待被调用函数返回），作为原始计数和总样本的百分比。 runtime.mapaccess1_fast64函数在298个样本期间运行，或11.8％。 top10输出按此样本计数排序。第三列显示列表期间的运行总计：前三行占样本的32.4％。第四和第五列显示函数出现的样本数（运行或等待被调用的函数返回）。 main.FindLoops函数在10.6％的样本中运行，但是它在84.1％的样本中位于调用堆栈（它或称为运行的函数）上。

源码对于StartCPUProfile的注释如下，所以采样频率已经被官方硬编码为1秒100次。

> The runtime routines allow a variable profiling rate,
but in practice operating systems cannot trigger signals at more than about 500 Hz, and our processing of the signal is not cheap (mostly getting the stack trace).
100 Hz is a reasonable choice: it is frequent enough to produce useful data, rare enough not to bog down the
system, and a nice round number to make it easy to
convert sample counts to seconds. Instead of requiring
each client to specify the frequency, we hard code it.
