---
title: 'defer or not defer, that is the question'
date: 2019-06-14 18:19:09
urlname: go-defer-or-not-defer
tags:
- Go
categories:
- Go
---

defer是Go语言的一种特殊的流程控制机制，通常用于简化执行各种资源清理和释放操作，比如关闭文件，释放锁，断开连接等等。使用defer可以更加简洁清晰的实现这种清理操作，但同时就像任何事物都有两面性，这种简洁性有时也会带来性能的下降，有些场景因此应该避免使用defer。本文主要是关于对应于不同的场景，如何选择性得正确得使用defer。

## defer概念

> 下面的叙述中，统一用外围函数代指包含defer的外层函数

defer语句调用一个函数，该函数的执行被推迟到外围函数返回的那一刻，外围函数的返回存在两种情况：正常执行到了return语句 或者 函数所在的goroutine发生了崩溃(panic)。也就是说如果外围函数通过显式return语句返回，defer语句真正执行的时机则是在该return语句设置任何结果参数之后，但在外围函数返回其调用者之前。

<!--more -->

基于defer的特性，其天然适用于各种资源的关闭操作，如果我们不使用defer语句，则下面的例子中，虽然有调用```src.Close()```但是如果程序在os.Create的时候出错执行了return操作，那么```src.Close()```就不会被执行到，src也就不会被正确关闭。当然我们可以在os.Create的错误判断的地方多加入一行src.Close()，就像下面代码中被注释掉的那行一样。不过如果我们的程序本身更复杂，返回操作更多，那么实现src的正确关闭就会困难很多。

```go

func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)
    if err != nil {
        //src.Close()
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}

```

所以官方推荐使用defer进行关闭，就像下面的代码，在打开资源的```os.Open```和```os.Create```之后，紧接着就defer对应的关闭操作。defer语句一定会在CopyFile返回给调用者之前执行，确保了资源的关闭。

```go

func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}

```

defer语句允许我们考虑在打开资源之后立即关闭每个资源，保证无论函数中的返回语句的位置和数量如何，文件都将被关闭。

## Defer的陷阱

因为官方的推荐和各种教程的演示，我想Gopher往往都养成了类似的习惯，打开资源后立马进行defer关闭操作，就可以放心得接续写接下来的代码了。但是这样真的就可以放心了吗？

以下面的加锁解锁为例，为了保证client.seq的并发安全的递增，我们在调用时首先加锁，之后defer语句确保了锁的释放。可是如果我们在send函数里接下来执行非常耗时的操作呢？比如执行网络发送操作。这时候锁mutex就会一直处于加锁状态，直到send函数执行完之后才会执行```defer client.mutex.Unlock()```，并发的其他send请求就要一直处于等待状态，大大降低了并发的效率。

```go

func (client *Client) send(call *Call)  {
	client.mutex.Lock()
	defer client.mutex.Unlock()
    
	client.seq++

 	//do someting
    //do someting
    //do someting
    
}

```

因为我们只是想利用锁确保client.seq的并发安全，如果我们改成下面的代码，对seq进行操作之后立马解锁，其他并发的send操作就可以马上被执行到，而不用等到send函数全部执行，效率大大增加。

```go

func (client *Client) send(call *Call) {
	client.mutex.Lock()
	client.seq++
	client.mutex.Unlock()

	//do someting
    //do someting
    //do someting
}

```

或者如果我们仍旧想使用defer，我们可以这样修改，可以封装一个方法单独执行seq的操作。

```go

func (client *Client) send(call *Call) {

	client.incrSeq()
	
	//do someting
    //do someting
    //do someting
}

func (client *Client) incrSeq() {
	client.mutex.Lock()
	defer client.mutex.Unlock()
	client.seq++
}

```

从上面的例子可以了解到，defer用于资源的释放时，要等待外围的函数全部执行完毕，如果我们在外围函数中对于资源的使用早早就结束了，接下来还要执行非常耗时的操作，那么这种场景下利用defer释放资源是非常不明智的，会极大的影响资源的使用效率。

在这样的场景下，我们要么不使用defer，在资源使用完毕时直接释放；如果使用defer，就围绕着资源的使用周期另外封装函数，确保资源的尽早释放。

简而言之，在使用defer时不要因为其简洁性而忽略了可能带来的性能问题，需要根据具体场景谨慎决定是否使用defer和怎样使用defer。
