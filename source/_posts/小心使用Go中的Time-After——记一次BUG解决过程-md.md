---
title: 小心使用Go中的Time.After——记一次BUG解决过程.md
date: 2019-06-06 17:46:29
urlname: go-time-after
tags:
- Go
categories:
- Go
---

最近项目中遇到了一个time.After的错误使用问题，而在常见的gopher学习材料中time.After又常常被用到，以至于我们忽略了背后的性能问题。

## 问题

这一段时间用户增长很快，通讯服务程序出现一个现象: 最初两台配置较差的服务器上通讯进程的CPU使用率和内存占用率开始缓慢升高，之后加速度越来越大o(╯□╰)o，当内存占用足够高的时候就会被linux的OOM killer机制杀死，之后被supervisor拉起之后重复这一现象。

## 查找

考虑是由于用户增加快，这两台配置较差的服务器上触发了程序的隐藏bug。观察现象：通讯进程的CPU使用率和内存占用率缓慢升高，首先想到的是goroutine泄露。但是因为我们有监控每个进程的goroutine数量，发现随着时间的推进，goroutine数量是比较稳定的，并无暴涨的情况。利用pprof的goroutine分析，也未发现异常。

再想到，因为内存占有率的不断升高直到快达到100%而被杀死，那可能是因为垃圾回收方面的问题。利用pprof的heap分析，查看当前程序的内存占用情况：

<!--more -->

![e67fc4a5660cc80dd2f0ad85002a92cd.png](evernotecid://BBA67D57-7922-4B2E-B8C7-BE4DD447858B/appyinxiangcom/12364249/ENResource/p1125)

发现了time.After和time.NewTimer是内存暴涨的原因。去程序里锁定了类似如下一段代码：

```go

for self.Open {
    select {

    case msg := <-self.readChan:
        // do someting

    case <- time.After(time.Second*3):

    }
}

```

在for循环内在一个从读取channel变量readChan中获取消息的逻辑，利用time.After做了超时处理，如果3秒内没有从readChan中读取到数据，就跳出到外循环检查是否连接仍旧是开启(open)状态，之后继续读取。

然后我们点进到time.After的官方定义：

```go

// After waits for the duration to elapse and then sends the current time
// on the returned channel.
// It is equivalent to NewTimer(d).C.
// The underlying Timer is not recovered by the garbage collector
// until the timer fires. If efficiency is a concern, use NewTimer
// instead and call Timer.Stop if the timer is no longer needed.
func After(d Duration) <-chan Time {
	return NewTimer(d).C
}

```

注意到里面的这样一句话 **The underlying Timer is not recovered by the garbage collector until the timer fires.** 也就是说time.After生成的底层timer定时器每次只会在时间到了才会被垃圾回收。

。。。ヽ(*。>Д<)o゜ ヽ(*。>Д<)o゜ ヽ(*。>Д<)o゜

如果只是一次调用time.After倒还好，但是如果像我们上面那段代码，外面存在一个for循环，那也就是说time.After创建的timer会不断累积。虽然对应的timer*到期*之后会被回收，但是对于高并发的程序而言已经是个极大的隐患。如果并发量不足，服务器性能较好，内存配置较高，那么这个问题并不明显。因为最近用户增长量很快外加有两台服务器配置较差，才被暴露出来。

## 解决

那么如何解决呢？仍然先看time.After的官方说明里 **If efficiency is a concern, use NewTimer instead and call Timer.Stop if the timer is no longer needed.** 这是说因为time.After创建的新timer只有在到期之后才会被回收，如果这个特性给你的程序带来了性能问题。那么可以不使用time.After，而直接使用NewTimer创建定时器，如果定时器没到期的时候就决定不再使用它，那么可以显式调用Timer.Stop直接停止该Stop.

如果参照着官方说明的意思，那么我们出问题的那段代码应该改为下面这样：

```go

for self.Open {
    timer := time.NewTimer(time.Second*3)
    select {
    case msg := <-self.readChan:
        //stop the timer
        timer.Stop()

        // do someting
    case <- timer:

    }
}

```

但显然这样仍然存在很大的问题，每次for循环都要创建一个新的timer，意味着可能会有非常多的timer变量被创建，之后又要被回收，并不是一个好的解决方法。当然我们也可以在for循环外只创建一个timer，组合 ```func (t *Timer) Stop() bool``` 和 ```func (t *Timer) Reset(d Duration) bool``` 方法进行timer的复用。不过这两个方法本身也存在很多使用上的问题，值得单独写一篇说明下。。。 不过这里针对这段代码，time.After的使用并非最本质的问题。

细看这段代码，time.After是一个用法上的问题，更深层次的问题是编程思想的问题，总的来说这段代码非常不符合go语言的编程思想。之所以使用timer.After是因为外部的for循环存在被其他情况关闭的可能，利用timer.After其实相当于是在轮询式的检测关闭条件。对于这种情况，go本身提供了原生的channel变量可以用作信号通知。当被关闭时，利用channel变量传递关闭信息break掉for循环即可。此处请出go最著名的一段思想**“Do not communicate by sharing memory; instead, share memory by communicating.”(不要通过共享内存进行通信，要通过通信来共享内存。)**

最终修改后的代码如下(当然是结合了我们程序内的其他代码，此处仅给出示意代码)，当其他的条件引起for循环关闭时，我们通过ctx.Done()这一channel变量传递信号进来，进行break，结束for循环，也就是利用通信来共享内存。

```go

for {
    select {
    case msg := <-self.readChan:
        // do someting

    case <-ctx.Done():
        break
    }
}

```

总结而言，我们首先要注意需要很小心得使用官方的time包，尤其要注意time.After这一封装的方法对于性能的影响。另一方面，
“Do not communicate by sharing memory; instead, share memory by communicating.”每一个gopher可能都读到过这句话，但是真正在实践中怎样贯彻这一编程思想，本文所讲的这个bug解决方案应该是个很好的例子。
