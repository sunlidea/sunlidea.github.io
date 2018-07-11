---
title: Go语言中slice使用注意事项
date: 2018-06-28 17:39:34
tags:
---
Go语言中slice类型算是比较特殊的一种数据类型，用起来很方便，在代码中的使用率也很高。不过要注意的是，slice虽然很方便，但同时也有很多陷阱需要注意。本文结合具体实例阐述了使用slice的一些注意事项。
### slice定义
Go语言中slice类型可以理解为是数组array类型的描述符，包含了三个因素:
1. 指向底层数组的指针
2. slice目前使用到的底层数组的元素个数，即长度
3. 底层数组的最大长度，即容量

因此当我们定义如下一个切片变量
```go
s := make([]int, 5, 10)
```
s即为指向了一个最大长度为10的底层数组，目前切片s使用到的长度为5。

<!-- more -->

## 注意事项
基于slice的定义，在使用slice时，有以下几点注意事项：

### 一. 切分操作


对slice进行切分操作会生成一个新的slice变量，新slice和原来的slice指向同一个底层数组，只不过指向的起始位置可能不同，长度及容量可能也不相同。

- 当从左边界有截断时，会改变新切片容量大小
- 左边界默认0，最小为0；右边界默认slice的长度，最大为slice的容量
- 因为指向同一个底层数组，对新slice的操作会影响到原来的slice

```go

	a := make([]int, 5, 10)

	b := a[:3]
	//b  len: 3 cap: 10
	fmt.Println("b ", "len:", len(b), "cap:", cap(b))

	//当从左边界有截断时 会改变新切片容量大小
	c := a[2:4]
	//c  len: 2 cap: 8
	fmt.Println("c ", "len:", len(c), "cap:", cap(c))

	//左边界默认0 最小为0 | 右边界默认slice的长度 最大为slice的容量
	d := a[:cap(a)]
	//d  len: 10 cap: 10
	fmt.Println("d ", "len:", len(d), "cap:", cap(d))

```
[运行示例代码 - slice切分操作](https://play.golang.org/p/3GO0cIbZ9u)
>本文中出现的示例代码展示于The Go Playground

### 二. 赋值及函数间传递

#### *第一种情况*


```go
	a := []int{1, 2, 3, 4, 5}
	b := a

	//对b进行操作会影响到a
	b[0] = 10
	// 10 2 3 4 5
	fmt.Println(a)
```
[运行示例代码](https://play.golang.org/p/lcjiulu-X2)

如上所示，则a, b指向同一个底层数组，且长度及容量因素相同，对b进行的操作会影响到a。 

#### *第二种情况*

```go

func main() {
	a := []int{1, 2, 3, 4, 5}
	modifySlice(a)
	//[10 2 3 4 5]
	fmt.Println(a)
}

func modifySlice(s []int) {
	s[0] = 10
}

```
[运行示例代码](https://play.golang.org/p/ioOXLoAz3W)

如上所示，将slice作为参数在函数间传递的时候是值传递，产生了一个新的slice，只不过新的slice仍然指向原来的底层数组，所以通过新的slice也能改变原来的slice的值。

#### *第三种情况*

```go

func main() {

	a := []int{1, 2, 3, 4, 5}
	modifySlice(a)
	//[1 2 3 4 5]
	fmt.Println(a)
}

func modifySlice(s []int) {
	s = []int{10, 20, 30, 40, 50}
}

```
[运行示例代码](https://play.golang.org/p/LbFovzP-Rj)

但是，如上所示，在调用函数的内部，将s整体赋值一个新的slice，并不会改变a的值，因为modifySlice函数内对s重新的整体赋值，让s指向了一个新的底层数组，而不是传递进来之前的a指向的那个数组，之后s的任何操作都不会影响到a了。

### 三. append操作

append操作最容易踩坑，下面详细说明一下。
- append函数定义:  
```go
    func append(s []T, x ...T) []T
```
 
- Append基本原则：对于一个slice变量，若slice容量足够，append直接在原来的底层数组上追加元素到slice上；如果容量不足，append操作会生成一个新的更大容量的底层数组。 

#### *第一种情况*

```go
func main() {

	a := make([]int, 2, 4)

	//通常append操作都是将返回值赋值给自己,
	//此处为了方便说明,没有这样做
	b := append(a, 1)
	//改变b的前2个元素值 会影响到a
	b[0] = 99

	//a: [99 0]    &a[0]: 0x10410020  len: 2  cap: 4
	fmt.Println("a:", a, " &a[0]:", &a[0], " len:", len(a), " cap:", cap(a))
	//b: [99 0 1]  &b[0]: 0x10410020  len: 3  cap: 4
	fmt.Println("b:", b, " &b[0]:", &b[0], " len:", len(b), " cap:", cap(b))
}

```
[运行示例代码](https://play.golang.org/p/rQQy4u0vCq)

如上所示，对a进行append操作，若append后的新slice的实际元素个数没有超出原来指向的底层数组的容量，所以仍然使用原来的底层数组a, b的第一个值的地址一样, 改变b的前2个元素也会影响到a。

其实这时候a,b指向的同一个底层数组的第3位(索引2)已经变成了数值1，但是对slice而言，除了底层数组，还有长度，容量两个因素，这时候a的长度仍然是2，所以输出的a的值没有变化。

#### *第二种情况*

```go

func main() {

	a := make([]int, 2, 4)

	//通常append操作都是将返回值赋值给自己,
	//此处为了方便说明,没有这样做
	b := append(a, 1, 2, 3)
	//改变b的前2个元素值 不会影响到a
	b[0] = 99

	//a: [0 0]         &a[0]: 0x10410020  len: 2  cap: 4
	fmt.Println("a:", a, " &a[0]:", &a[0], " len:", len(a), " cap:", cap(a))
	//b: [99 0 1 2 3]  &b[0]: 0x10454000  len: 5  cap: 8
	fmt.Println("b:", b, " &b[0]:", &b[0], " len:", len(b), " cap:", cap(b))
}


```

[运行示例代码](https://play.golang.org/p/e-gvTVx4vZ)

如上所示，若append后的新slice即b的实际元素个数已经超出了原来的a指向的底层数组的容量，那么就会分配给b一个新的底层数组，可以看到，a，b第一个元素的地址已经不同，改变b的前两个元素值也不会影响到a，同时容量也发生了变化。

#### *第三种情况*

```go

func main() {

	a := make([]int, 2, 4)
	a[0] = 10
	a[1] = 20
	//a: [10 20]  &a[0]: 0x10410020  len: 2  cap: 4
	fmt.Println("a:", a, " &a[0]:", &a[0], " len:", len(a), " cap:", cap(a))

	//进行append操作
	b := append(a[:1], 1)

	//a: [10 1]   &a[0]: 0x10410020  len: 2  cap: 4
	fmt.Println("a:", a, " &a[0]:", &a[0], " len:", len(a), " cap:", cap(a))
	
	//b: [10 1]   &b[0]: 0x10410020  len: 2  cap: 4
	fmt.Println("b:", b, " &b[0]:", &b[0], " len:", len(b), " cap:", cap(b))
}

```
[运行示例代码](https://play.golang.org/p/Efb42G5Wt9)

如上所示，若append后的b的实际元素个数没有超过原来的a指向的底层数组的容量，那么a，b指向同一个底层数组。
注意此时append的操作对象是：对a进行切分之后的切片，只取了a的第一个值，相当于一个新切片，长度为1，和a指向同一个底层数组，我们称这个切分后的新切片为c吧，那么就相当于b其实是基于c切片进行append的，直接在长度1之后追加元素，所以append之后a的第二个元素变成了1。**所以切分操作和append操作放一起的时候，一定要小心**

#### *第四种情况*

```go

func main() {

	a := make([]int, 2, 4)
	a[0] = 10
	a[1] = 20
	//a: [10 20]       &a[0]: 0x10410020  len: 2  cap: 4
	fmt.Println("a:", a, " &a[0]:", &a[0], " len:", len(a), " cap:", cap(a))

	//进行append操作
	//append是在第一个元素后开始追加，所以要超过容量，至少要追加4个，而不是之前例子的3个
	b := append(a[:1], 1, 2, 3, 4)

	//a: [10 20]       &a[0]: 0x10410020  len: 2  cap: 4
	fmt.Println("a:", a, " &a[0]:", &a[0], " len:", len(a), " cap:", cap(a))
	
	//b: [10 1 2 3 4]  &b[0]: 0x10454020  len: 5  cap: 8
	fmt.Println("b:", b, " &b[0]:", &b[0], " len:", len(b), " cap:", cap(b))
}

```
[运行示例代码](https://play.golang.org/p/wzNbO9vDJ0)

如上所示，这种情况主要用来与第三种情况对比，如果append的元素数较多，超过了原来的容量，直接采用了新的底层数组，也就不会影响到a了。

上述的四种情况所用例子都比较简单，所以比较容易看清。要小心如果在函数间传递slice，调用函数采用append进行操作，可能会改变原来的值的，如下所示：

```go

func main() {

	a := make([]int, 2, 4)
	a[0] = 10
	a[1] = 20
	//a: [10 20]  &a[0]: 0x10410020  len: 2  cap: 4
	fmt.Println("a:", a, " &a[0]:", &a[0], " len:", len(a), " cap:", cap(a))

	testAppend(a[:1])

	//a: [10 1]   &a[0]: 0x10410020  len: 2  cap: 4
	fmt.Println("a:", a, " &a[0]:", &a[0], " len:", len(a), " cap:", cap(a))

}

func testAppend(s []int) {
	//进行append操作
	s = append(s, 1)
	//s: [10 1]  &s[0]: 0x10410020  len: 2  cap: 4
	fmt.Println("s:", s, " &s[0]:", &s[0], " len:", len(s), " cap:", cap(s))
}

```
[运行示例代码]( https://play.golang.org/p/mvUZkmBW8u)


参考资料:

https://blog.golang.org/go-slices-usage-and-internals
