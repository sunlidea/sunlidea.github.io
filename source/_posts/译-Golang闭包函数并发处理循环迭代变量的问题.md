---
title: '[译]Golang闭包函数并发处理循环迭代变量的问题'
date: 2018-09-29 13:04:04
urlname: golang-loop-closures-goroutines
tags:
- Golang
categories:
- Golang
---

总结这一段时间团队在使用Go语言编程时，遇到最多的问题就是在闭包函数并发处理循环迭代变量时处理不当的问题。所以就想总结下这种情况，查资料时看到Go在Github的wiki页面上有一篇专门介绍该问题的文章，可见这个问题被询问的频率之高。这篇文章总结的挺好的，就把它翻译了下来，留作之后查资料值之用。

原文地址: [CommonMistakes](https://github.com/golang/go/wiki/CommonMistakes)

相关资料: [closures_and_goroutines](https://golang.org/doc/faq#closures_and_goroutines)

## 简介

当新手程序员开始使用Go或熟练的Go程序员开始使用新概念时，他们中的许多人都会犯一些常见的错误。目前为止，在Go的邮件列表中出现最频繁的错误之一，就是在循环迭代时将迭代数据传入并发的goroutines引发的错误。

<!-- more -->

## 在循环迭代变量上使用goroutines

在Go中迭代时，操作者可能会尝试使用goroutines并行处理数据。例如，你可以使用闭包来编写类似下面的代码：

```golang

for _, val := range values {
	go func() {
		fmt.Println(val)
	}()
}

```

上面的for循环可能没有达到你所预期的效果，因为这里的在整个循环中只初始化了一个val变量，这个val变量在每次循环时被重新赋值为当前切片元素。因为这里的闭包函数只绑定到了这个val变量，所以很有可能当你运行这个代码时，你会看到每个迭代打印的都是最后一个元素，而不是序列中的每个值，因为goroutines很可能会在循环结束后才开始执行。

写这个闭包循环的正确方式如下:

```golang

for _, val := range values {
	go func(val interface{}) {
		fmt.Println(val)
	}(val)
}

```

通过将val作为参数传入闭包函数中，val在每次迭代时都已经赋值为了当前切片元素，并且传入的val值被放在了当前goroutine的堆栈上，所以当最终goroutine执行的时候，每个切片元素都能正确输出。

另一种正确处理的方式如下，
需要注意的是，在循环体内声明的变量不在迭代之间共享，因此可以在闭包函数中单独使用。以下代码使用公共索引变量i来创建单独的val，这是可以产生预期的结果：

```golang

for i := range valslice {
	val := valslice[i]
	go func() {
		fmt.Println(val)
	}()
}

```

请注意，如果不将此闭包函数作为goroutine执行，代码将会按预期运行。以下示例打印出1到10之间的整数。这时候即使闭包函数仍然使用同一个变量（在这种情况下，就是i），但它们在变量变化之前就执行了，从而产生了预期的行为。

```golang

for i := 1; i <= 10; i++ {
	func() {
		fmt.Println(i)
	}()
}

```

下面是另一种相似的情况:

```golang

for _, val := range values {
	go val.MyMethod()
}

func (v *val) MyMethod() {
        fmt.Println(v)
}

```

上面的例子也会打印出最后一个元素的值，原因与最上面并发时闭包的处理相同。要解决此问题，请在循环内声明另一个变量。

```golang

for _, val := range values {
        newVal := val
	go newVal.MyMethod()
}

func (v *val) MyMethod() {
        fmt.Println(v)
}

```