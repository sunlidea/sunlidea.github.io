---
title: Go语言中反射Reflection的原理及应用
date: 2019-05-23 16:31:37
urlname: go-reflection
tags:
- Go
categories:
- Go
---

作为一种静态编译型语言，Go语言中的反射Reflection特性提供了程序运行时检查，修改和创建变量和函数的功能。对于缺乏泛型generic支持的Go语言而言，通过Reflection也可以间接实现泛型的功能。本来主要探究Go语言中Reflection的原理及应用。

## reflect.Type和reflect.Value

先来看一下Reflection包里提供的最基本的两个概念：reflect.Type 和 reflect.Value，对应的两个基本函数为：

```go

// TypeOf returns the reflection Type that represents the dynamic type of i.
// If i is a nil interface value, TypeOf returns nil.
func TypeOf(i interface{}) Type {
	eface := *(*emptyInterface)(unsafe.Pointer(&i))
	return toType(eface.typ)
}

// ValueOf returns a new Value initialized to the concrete value
// stored in the interface i. ValueOf(nil) returns the zero Value.
func ValueOf(i interface{}) Value {
	if i == nil {
		return Value{}
	}

	// TODO: Maybe allow contents of a Value to live on the stack.
	// For now we make the contents always escape to the heap. It
	// makes life easier in a few places (see chanrecv/mapassign
	// comment below).
	escapes(i)

	return unpackEface(i)
}

```

可以看到，两个函数的参数都是空接口interface{}类型，可见反射Reflection与接口类型Interface密切相关。要了解reflect.Type 和 reflect.Value两个概念，我们需要先理清楚接口类型Interface。

<!--more -->

## 反射Reflection与接口Interface

反射建立在类型系统上，因此探究反射就有必要先整理清楚Go语言中的类型。

Go语言是一种静态类型statically typed的语言，每个变量在编译时就具有一个固定的类型，例如int, []string,包括自定义类型MyType等等。

接口Interfaces是一种重要类型，该类型表示固定方法的集合(fixed sets of methods)。任何实现了接口代表的方法集的具体变量，都可以赋值给对应接口变量。

抽象来说，一个接口变量底层存储了一对值：赋值给该接口的具体值 和 该值的完整的类型描述符。例如：

```go

var r io.Reader
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
if err != nil {
    return nil, err
}
r = tty

```

经典的接口类型io.Reader定义了一个Read方法

```go

type Reader interface {
        Read(p []byte) (n int, err error)
}

```

对于变量tty，其具体类型是 * os.File，[* os.File](https://golang.org/pkg/os/#File)类型实现的具体方法的完整列表如下：

```go

    func (f *File) Chdir() error
    func (f *File) Chmod(mode FileMode) error
    func (f *File) Chown(uid, gid int) error
    func (f *File) Close() error
    func (f *File) Fd() uintptr
    func (f *File) Name() string
    func (f *File) Read(b []byte) (n int, err error)
    func (f *File) ReadAt(b []byte, off int64) (n int, err error)
    func (f *File) Readdir(n int) ([]FileInfo, error)
    func (f *File) Readdirnames(n int) (names []string, err error)
    func (f *File) Seek(offset int64, whence int) (ret int64, err error)
    func (f *File) SetDeadline(t time.Time) error
    func (f *File) SetReadDeadline(t time.Time) error
    func (f *File) SetWriteDeadline(t time.Time) error
    func (f *File) Stat() (FileInfo, error)
    func (f *File) Sync() error
    func (f *File) SyscallConn() (syscall.RawConn, error)
    func (f *File) Truncate(size int64) error
    func (f *File) Write(b []byte) (n int, err error)
    func (f *File) WriteAt(b []byte, off int64) (n int, err error)
    func (f *File) WriteString(s string) (n int, err error)

```

可以看到类型* os.File实现了 Read(b []byte) (n int, err error) 方法，因此实现了接口类型io.Reader定义，因此可以赋值给io.Reader接口类型的变量r。当操作了r=tty之后，r的底层就存储了一对值(tty, * os.File)，也就是实现了io.Reader接口的具体变量值tty 和 完整描述tty类型的描述符* os.File。

当然除了Read方法，* os.File类型也实现了其他的一些方法，只是因为接口类型io.Reader值只定义了Read方法，当操作了r=tty之后，之后操作接口变量r的时候只能调用tty的Read方法，但是底层存储上，r仍然保存了对于该类型方法的完整描述。因此我们可以实现下面的操作：

```go

var w io.Writer
w = r.(io.Writer)

```

接口类型io.Writer定义如下：

```go

// Writer is the interface that wraps the basic Write method.
type Writer interface {
    Write(p []byte) (n int, err error)
}

```

因为* os.File类型也实现了Write方法，所以* os.File类型的变量也实现了接口io.Writer，因此可以将* os.File类型的变量赋值给接口变量w。又r底层保留了tty的全部类型描述，因此可以实现w = r.(io.Writer)的操作，将r赋值给w。赋值之后，w底层将包含（tty，* os.File），与r所持有的一对变量是相同的。

继续 我们可以这样做：

```go

var empty interface{}
empty = w

```

因为空接口interface{}没有定义任何方法，也就意味着任何变量都实现了空接口，所以任何变量都可以赋值给空接口interface{}。对于例子中的empty，该变量底层仍旧包含了一个信息对(tty，* os.File)，和上面的w和r所持有的相同。并且这里不需要类型断言，因为静态地知道空接口可以包含任何值。在我们将值从Reader移动到Writer的示例中，我们需要显式得使用类型断言，因为Writer的方法不是Reader方法的子集。

也就是说一个空接口可以保存任何值，并包含我们可能需要的有关该值的所有信息(具体值，关于该值的完整类型描述)。

总而言之，接口类型的变量底层总是包含(赋值给该接口的具体值，关于该值得完整类型描述)的信息对，注意这里只能是具体值，不能是接口值。


## 接口和反射对象的相互转换

我们再回到文章开头时给出的Reflection的基础概念：reflect.Type 和 reflect.Value，对应的两个基本函数为：

```go

func ValueOf(i interface{}) Value
func TypeOf(i interface{}) Type

```

接口实现的底层可以抽象表示为一个信息对(实现该接口的具体值，描述该具体值完整类型信息的类型描述符)。两个函数入参是空接口类型interface{}的变量，上面讲到，任何值都可以赋值给空接口类型的变量，并且空接口类型的变量底层会保存需要的有关该值的所有信息(具体值，关于该值的完整类型描述)。

因此两个函数的含义也就自解释了：

- ValueOf是取出空接口变量i底层包含的**具体值的信息**，以上面的empty为例，也就是tty的信息；
- TypeOf则是取出空接口变量i底层包含的描述具体值的**类型信息**，以上面的empty为例，也就是* os.File代表的信息。

对应的reflect.Value和reflect.Type的含义也就是：

- reflect.Value表示接口变量的具体值的信息
- reflect.Type表示接口变量的描述具体值的类型信息

现在看下两个函数的使用

```go

	x := 3.1415926
	v := reflect.ValueOf(x)
	t := reflect.TypeOf(x)

```

可以看到虽然传入ValueOf的变量为float64类型的变量x，但是因为ValueOf定义时的参数是interface{}类型，因此在调用reflect.ValueOf(x)时，x首先被存储在一个新的空接口变量i中，i底层的抽象表示为(x, float64)，然后i再作为参数传入ValueOf。ValueOf再拆解空接口变量i，提取出关于具体值x的信息；同样的，TypeOf提取出描述具体值x的类型的float64的信息。

也就是说，通过ValueOf和TypeOf，反射Reflection实现了从接口值(空接口变量i)到反射对象(reflect.Value,reflect.Type)的转换。

反射是相互的，因此通过下面的操作，反射Reflection也可以实现从反射对象(reflect.Value)到接口值(空接口变量i)得转换：

```go

// Interface returns v's current value as an interface{}.
// It is equivalent to:
//	var i interface{} = (v's underlying value)
// It panics if the Value was obtained by accessing
// unexported struct fields.
func (v Value) Interface() (i interface{}) {
	return valueInterface(v, true)
}

```

继续，我们可以通过类型断言，获取到具体值y

```go

y := v.Interface().(float64) // y will have type float64.
fmt.Println(y)

```

因此，可以说，**反射Reflection实现了从接口值(空接口变量i)到反射对象(reflect.Value,reflect.Type)的相互转换**。

## 通过反射对象，修改原始具体值

开头提到，Go语言中反射Reflection特性提供了程序运行时检查，修改和创建变量的功能。上面介绍到的ValueOf和TypeOf可以说提供了检查变量信息的功能，那么怎样去通过反射的对象，修改到原始的具体值呢。下面是一个例子：

```go

	x := 3.141592
	p := reflect.ValueOf(&x)
	v := p.Elem()
	v.SetFloat(2.333333)
	fmt.Printf("value of x:%f\n", x)

```

从要实现反射对象修改原始具体值这一功能的设计角度出发，我们看下为什么要这样设计，为什么要传入&x，为什么要调用p.Elem()。
假设我们传入x，而不是&x(当然下面这段程序会panic)。一个基本原则是Go语言是值传递的，当调用reflect.ValueOf(x)时，其实传入ValueOf时候会首先生成一个x的拷贝，我们称为copyX，ValueOf内部操作的真实值是copyX，已经丢失了原始值x的地址信息，那么通过v去更改原始值x也就无从实现。所以我们需要传入指向x的指针&x，&x保存了原始值x的地址信息，才有可能通过&x生成的反射对象reflect.Value去修改原始值x。

```go

   //panic: reflect: reflect.Value.SetFloat using unaddressable value

	x := 3.141592
	v := reflect.ValueOf(x)
	v.SetFloat(2.333333)
	fmt.Printf("value of x:%f\n", x)

```

再来，为什么要有v := p.Elem()，然后通过v去修改原始值x？p实际是x的指针变量&x，我们要修改的是x，而不是p本身，因此通过p.Elem()取出指向的原始值x，再进行修改。类比下面的Go语言中最基本的指针操作，p.Elem()相当于* p的功能，是对指针变量p进行解引用，进而通过* p操作原始对象。

```go

	x := 3.141592
	p := &x
	//not p = 2.333333
	*p = 2.333333
	fmt.Printf("value of x:%f\n", x)

```

因此，通过反射对象修改原始具体值，我们需要传入原始值的指针。

## 通过反射新建变量

再来看通过反射创建新变量的过程。

```go

	x := 3.141592
	typ := reflect.TypeOf(x)

	p := reflect.New(typ)
	v := p.Elem()
	v.SetFloat(2.333333)
	y := v.Interface().(float64)

	//value of y:2.333333
	fmt.Printf("value of y:%f\n", y)
	//value of p:0xc0000c0550
	fmt.Printf("value of p:%v\n", p.Interface())
	//value of x:3.141592
	fmt.Printf("value of x:%f\n", x)

```

主要是通过reflect.New来实现，传入reflect.Type类型的变量typ，reflect.New会生成一个指向类型typ的空值的指针。可以通过该指针设置具体值。如上所示，最终得到的y是个float64类型的新变量。

```go

// New returns a Value representing a pointer to a new zero value
// for the specified type. That is, the returned Value's Type is PtrTo(typ).
func New(typ Type) Value {
	if typ == nil {
		panic("reflect: New(nil)")
	}
	t := typ.(*rtype)
	ptr := unsafe_New(t)
	fl := flag(Ptr)
	return Value{t.ptrTo(), ptr, fl}
}

```

## 利用反射创建函数

Reflection包提供了MakeFunc函数用于创建新函数。[MakeFunc](https://golang.org/pkg/reflect/#MakeFunc)的定义如下，

```go

func MakeFunc(typ Type, fn func(args []Value) (results []Value)) Value

```

MakeFunc实现的功能是：返回一个入参typ代表的函数类型的新函数，我们称为newFn，该新函数newFn封装了入参的函数fn，当调用newFn时，newFn执行了下面一系列的操作：

- 将入参变量(typ定义的入参)转换为[]reflect.Value的切片，假设该切片为args
- 运行 results := fn(args)，results的类型是[]reflect.Value
- 将[]reflect.Value的results转换为typ定义的出参变量

来看下面给出的一个最简单的🌰，说明下MakeFunc的功能。

- 我们首先定义了MakeFunc第二个入参fn的类型的函数sayHello（为了专注于MakeFunc的操作说明，并没有在sayHello中做相关非法性判断），sayHello假设输入的args只有一个string值，执行fmt.Sprintf("hello,%s", name)将新的字符串返回。
- 之后我们在TestMakeFunc定义了一个 func(name string) string 函数类型的变量hello，将hello传入makeFunction，利用reflect.TypeOf(f)获取到对应的反射对象类型为reflect.Type的typ，注意此时typ持有的描述空接口f的具体值的类型为func(name string) string。
- 接着调用wf := reflect.MakeFunc(typ, sayHello)，返回值wf类型为reflect.Value，wf持有的是一个具体类型为func(name string) string的新函数
- 将wf.Interface()返回后，因为wf实际持有的具体值类型为func(name string) string，因此可以通过类型断言转换为reHello，调用reHello("leo")，执行以下的各个步骤，最后就会输出的结果是"hello,leo"
    1. 该新函数reHello内部将入参name转换为[]reflect.Value
    2. 之后调用sayHello(args []reflect.Value)，sayHello处理后(加上了”hello“前缀)，将结果result作为[]reflect.Value返回
    3. 将[]reflect.Value类型的结果转换为string（因为wf持有的变量类型就是func(name string) string,出参是个单一的string类型) 


```go

func sayHello(args []reflect.Value) []reflect.Value {
	name := args[0].Interface().(string)
	str := fmt.Sprintf("hello,%s", name)

	result := []reflect.Value{
		reflect.ValueOf(str),
	}
	return result
}

func makeFunction(f interface{}) interface{} {
	typ := reflect.TypeOf(f)
	wf := reflect.MakeFunc(typ, sayHello)
	return wf.Interface()
}

func TestMakeFunc(t *testing.T) {
	var hello func(name string) string
	reHello := makeFunction(hello).(func(name string) string)
	//output: hello,leo
	fmt.Println(reHello("leo"))
}

```

上面的sayHello的例子主要是为了展示MakeFunc的原理，那么MakeFunc的具体应用有哪些呢？

### 通过MakeFunc实现泛型

Go语言中没有原生支持泛型(呼声很高，2.0应该会提供...)，通过MakeFunc可以实现泛型的功能。

下面给出源代码/go/src/reflect/example_test.go给出的示例，实现了两个数互相交换的功能。

这个例子的关键点是要注意，利用了前面介绍的通过反射对象修改原始具体值的功能。以intSwap为例，调用makeSwap(&intSwap)是传入的是变量intSwap的指针，在makeSwap中:

- v := reflect.MakeFunc(fn.Type(), swap)创建了一个新的具有fn具体类型的新函数v，当传入makeSwap的是intSwap，则该类型为func(int, int) (int, int)；当传入的是floatSwap，则该类型为func(float64, float64) (float64, float64)，不管哪种类型，调用v的时候都会调用swap实现交换功能。这也是对泛型进行了间接的实现。
- fn := reflect.ValueOf(fptr).Elem()取出了传入的函数指针真正指向的函数变量，通过fn.Set(v)将MakeFunc创建的新函数v赋值给了原始值，也就是说此时intSwap的值已经修改为了MakeFunc创建的新函数v，调用intSwap，其实就是在调用v。通过这一操作其实可以实现优雅的rpc反射调用，完成调用远程函数就像调用本地函数一样的特性。

```go

func ExampleMakeFunc() {
	// swap is the implementation passed to MakeFunc.
	// It must work in terms of reflect.Values so that it is possible
	// to write code without knowing beforehand what the types
	// will be.
	swap := func(in []reflect.Value) []reflect.Value {
		return []reflect.Value{in[1], in[0]}
	}

	// makeSwap expects fptr to be a pointer to a nil function.
	// It sets that pointer to a new function created with MakeFunc.
	// When the function is invoked, reflect turns the arguments
	// into Values, calls swap, and then turns swap's result slice
	// into the values returned by the new function.
	makeSwap := func(fptr interface{}) {
		// fptr is a pointer to a function.
		// Obtain the function value itself (likely nil) as a reflect.Value
		// so that we can query its type and then set the value.
		fn := reflect.ValueOf(fptr).Elem()

		// Make a function of the right type.
		v := reflect.MakeFunc(fn.Type(), swap)

		// Assign it to the value fn represents.
		fn.Set(v)
	}

	// Make and call a swap function for ints.
	var intSwap func(int, int) (int, int)
	makeSwap(&intSwap)
	fmt.Println(intSwap(0, 1))

	// Make and call a swap function for float64s.
	var floatSwap func(float64, float64) (float64, float64)
	makeSwap(&floatSwap)
	fmt.Println(floatSwap(2.72, 3.14))

	// Output:
	// 1 0
	// 3.14 2.72
}


```

当然反射Reflection中还提供了很多其他操作，这个留待之后探究~~

## 参考

<https://blog.golang.org/laws-of-reflection>
<https://medium.com/capital-one-tech/learning-to-use-go-reflection-822a0aed74b7>
