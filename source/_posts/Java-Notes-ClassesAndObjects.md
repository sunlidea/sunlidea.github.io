---
title: Java Notes-ClassesAndObjects
urlname: java-notes-classes-and-objects
date: 2018-08-27 10:57:04
tags: 
- Java
- Java Notes
categories:
- Java
---

本文是学习Oracle The Java Tutorials中ClassesAndObjects篇章的笔记，记录了中间重点的内容，也方便后面自己查阅。

---

<!-- more -->

## Classes

### 定义类 Declaring Classes

```java
class class MyClassMyClass {
    // field, constructor, and 
    // method declarations
} 
```

基础定义:

- 构造函数constructor用户初始化新对象；
- 字段fields的生命提供了类和对象的状态state
- 方法实现了类和对象的行为behavior

```java
class MyClass extends MySuperClass implements YourInterface {
    // field, constructor, and
    // method declarations
}
```

如上，类定义时可以提供更多信息:

- 超级类的名字
- 实现的对应接口

还可以增加修饰符  public | private 

综上，通常而言，类的定义可以包含如下内容，按照顺序依次是：

1. 修饰符，比如public，private，以及其他的一些修饰符
2. 类名，通常首字母大写
3. extends + 父类名即超级类(如果有父类的话)，一个子类只能有一个父类
4. implents + 逗号分割的接口列表(如果实现了接口的话)，一个类能够实现多个接口
5. 类的body，由{}环绕

### 定义成员变量 Declaring Member Variables

存在如下几种成员变量：

- 类中的成员变量-fields
- 方法或者代码块中的变量-local variables
- 方法定义中的变量-parameters

字段fields定义包含如下内容：

- 0个或者更多的修饰符，如public或private
- 字段类型
- 字段名

#### 访问修饰符Access Modifiers

使用的第一个（最左侧）修饰符控制其他类可以访问成员字段的内容。
目前，只考虑public和private。其他访问修饰符将在后面讨论。

- public-该字段可以被其他类访问
- private-该字段只能被自己的类访问

本着封装的精神，将字段设为私有是很常见的。此时可以通过增加能够获取字段值
的public的方法来间接访问

---

方法和类的命名有两点需要注意的地方：

- 类名的首字母大写
- 方法名的第一个单词应该是一个动词

### 定义方法 Defining Methods

下面是一个典型的方法定义：

```java

public double calculateAnswer(double wingSpan, int numberOfEngines,
                              double length, double grossTons) {
    //do the calculation here
}

```
通常而言，方法定义由6部分组成：

1. 修饰符——比如public，private，以及其他的一些修饰符
2. 返回值类型——如果没有返回则为void
3. 方法名
4. 括号中的参数
5. 异常列表
6. 方法body

方法声明的两个组件(方法名和参数类型)组成了方法的签名
方法的签名定义如下：

```java

calculateAnswer(double, int, double, double)

```

#### 方法命名

通常而言，动词+形容词|名词|etc，采用驼峰法

通常，方法在其类中具有唯一名称。但是，由于方法重载，方法可能与其他方法具有相同的名称

#### 方法重载

Java编程语言支持重载方法，Java可以区分具有不同方法签名的方法。这意味着如果类中的方法具有不同的参数列表，则它们可以具有相同的名称。（有一些资格性问题，后面讨论）

在区分方法时编译器不考虑返回类型return type，因此即使它们具有不同的返回类型，也不能声明具有相同签名的两个方法

### 为类提供构造函数

构造函数用于从类的蓝图构造对象。构造函数定义和方法定义很相似，除了构造函数
使用类的名字并且没有返回值。例如:

```java


public Bicycle(int startCadence, int startSpeed, int startGear) {
    gear = startGear;
    cadence = startCadence;
    speed = startSpeed;
}

```

一个类也可以包括多个构造函数，名称都是类名，但是参数的种类和数量不同。
例如，也可以创建另一个Bicycle的构造函数如下:

```java

public Bicycle() {
    gear = 1;
    cadence = 10;
    speed = 0;
}

```

可以不必类提供任何构造函数，但在执行此操作时必须小心。编译器自动为没有构造函数的任何类提供无参数的默认构造函数。此默认构造函数将调用超类superclass的无参数构造函数。在这种情况下，如果超类没有无参数构造函数，编译器会抱怨，因此您必须验证它是否存在。如果你的类没有显式的超类，那么它有一个隐式的超类Object，它有一个无参数的构造函数。**这部分说明还不太懂 下面给出原文链接**

> You don't have to provide any constructors for your class, but you must be careful when doing this. The compiler automatically provides a no-argument, default constructor for any class without constructors. This default constructor will call the no-argument constructor of the superclass. In this situation, the compiler will complain if the superclass doesn't have a no-argument constructor so you must verify that it does. If your class has no explicit superclass, then it has an implicit superclass of Object, which does have a no-argument constructor.

可以给构造函数constructor加访问符，以控制其他类对这个构造函数的访问权限。
如果另一个类不能调用该构造函数，那么他也不能直接创建该类对象。

### 将信息传递给方法method或构造函数constructor

形参Parameters是指方法定义时的一系列变量variables，实参是方法被调用时真正传入的值。
当调用一个方法的时候，使用的实参必须匹配定义的形参的类型和顺序。

> Parameters refers to the list of variables in a method declaration. Arguments are the actual values that are passed in when the method is invoked. When you invoke a method, the arguments used must match the declaration's parameters in type and order.

#### 任意数量的参数

可以使用动态参数传递任意数量的参数到方法里，例如下面的Point...

```java

public Polygon polygonFrom(Point... corners) {
    int numberOfSides = corners.length;
    double squareOfSide1, lengthOfSide1;
    squareOfSide1 = (corners[1].x - corners[0].x)
                     * (corners[1].x - corners[0].x) 
                     + (corners[1].y - corners[0].y)
                     * (corners[1].y - corners[0].y);
    lengthOfSide1 = Math.sqrt(squareOfSide1);

    // more method body code follows that creates and returns a 
    // polygon connecting the Points
}

```

#### 传递原始数据类型的参数

原始参数（如int或double）按值传递给方法。这意味着对参数值的任何更改都仅存在于方法的范围内。方法返回时，参数消失，对它们的任何更改都将丢失。

#### 传递引用数据类型的参数

引用数据类型参数（如对象）也按值传递给方法。这意味着当方法返回时，传入的引用仍然引用与以前相同的对象。但是，如果对象的字段的值具有适当的访问级别，则可以在该方法中更改它们的值。
这里和golang中传递slice同理。

## Objects

典型的Java程序会创建许多对象，通过调用方法进行交互。

### 创建对象Creating Objects

类class提供了对象的蓝本blueprint，可以从类创建对象。例子如下：

```java

Point originOnePoint originOne = new Point(23, 94); 
Rectangle rectOneRectangle rectOne = new Rectangle(originOne, 100, 200);

```

#### 定义一个引用对象Object的变量

下面是定义一个变量的通用形式，这种定义题型编译器complier你将会使用name去引用类型
为type的数据data。

```java

type name;

```

对于原始数据类型，上述声明也让编译器同时为name变量预留足够内存空间；
而对于引用变量，比如下面,如果仅仅这样进行声明，这个变量是不确定的直到一个对象真正被创建并且分配给它。简而言之，声明引用变量不会创建对象。如果要真正的创建对象，需要使用new操作符。

```java

Point originOne;

```

#### 实例化一个类class

new运算符通过为新对象分配内存并返回对该内存的引用来实例化一个类。 new运算符还调用对象构造函数。

> 短语“实例化一个类”("instantiating a class" )与“创建一个对象”("creating an object.")意思相同。创建对象时，您正在创建类的“实例”，因此“实例化”一个类。

new运算符返回的引用不一定非要分配给变量。它也可以直接用在表达式中。例如：

```java

int height = new Rectangle().height;

```

#### 初始化一个对象

所有类至少有一个构造函数。如果类没有显式声明任何构造函数，则Java编译器会自动提供一个无参构造函数，称为默认构造函数。此默认构造函数调用=父类的无参数构造函数，如果类没有其他父级，则调用Object构造函数。如果父级没有构造函数（Object确实有构造函数），编译器将拒绝该程序。

### 使用对象 Using Objects

#### 引用一个对象的字段fields

在自己的类内，可以使用简单的对象名，如下:

```java

System.out.println("Width and height are: " + width + ", " + height);

```
对象的类之外的代码使用对象的字段必须使用：对象引用或表达式+.+字段名，如下：

```java

System.out.println("Width of rectOne: "  + rectOne.width);
System.out.println("Height of rectOne: " + rectOne.height);

```

#### 调用一个对象的方法

请记住，在特定对象上调用方法与向该对象发送消息相同。

#### 垃圾回收器 The Garbage Collector

Java运行时环境The Java runtime environment在确定不再使用对象时删除对象。此过程称为垃圾回收garbage collection。

当没有对该对象的引用时，java会对该对象进行垃圾回收。当变量超出范围时，通常会删除变量中保存的引用。或者，您可以通过将变量设置为特殊值null来显式删除对象引用。请记住，程序可以对同一对象进行多次引用;在对象被垃圾回收处理之前，必须删除对该对象的所有引用。

Java运行时环境具有垃圾收集器garbage collector，可以定期释放不再引用的对象使用的内存。垃圾收集器在确定时间正确时自动完成其工作。

## 关于类的更多方面 More on Classes

### 从一个方法中返回一个值 Returning a Value from a Method

一个方法在符合下面的任一中情况时，返回到调用它的代码：

- 完成了方法里的所有语句statements;
- 碰到了return语句;
- 抛出了一个异常exception;

任何未声明为void的方法都必须包含带有相应返回值的return语句，如下所示：

```java

return returnValue;

```

#### 返回一个类Class或者接口Interface

当方法使用类名作为其返回类型时，返回对象的类型所属的类 必须是 规定的返回类型的所属类或者子类。假设现在有一个类层次结构，其中ImaginaryNumber是java.lang.Number的子类，而java.lang.Number又是Object的子类，如下图所示：

![The class hierarchy for ImaginaryNumber](https://docs.oracle.com/javase/tutorial/figures/java/classes-hierarchy.gif)

现在假设定义一个返回类型为Number的方法：

```java

public Number returnANumber() {
    ...
}

```

returnANumber方法可以返回ImaginaryNumber而不是Object。 ImaginaryNumber是Number的子类，满足Number类的条件但是，Object不一定是Number类的条件。

您还可以使用接口名称作为返回类型。在这种情况下，返回的对象必须实现指定的接口。

### 使用this关键字 Using the this Keyword

在实例方法或构造函数中，这是对当前对象的引用。您可以使用此方法从实例方法或构造函数中引用当前对象的任何成员。

#### Using this with a Field

使用this关键字的最常见原因是因为字段被方法或构造函数参数覆盖了。如下：

```java

public class Point {
    public int x = 0;
    public int y = 0;
        
    //constructor
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

```

#### Using this with a Constructor

在构造函数中，您还可以使用this关键字来调用同一个类中的另一个构造函数。这样做称为显式构造函数调用。如下：

```java

public class Rectangle {
    private int x, y;
    private int width, height;
        
    public Rectangle() {
        this(0, 0, 1, 1);
    }
    public Rectangle(int width, int height) {
        this(0, 0, width, height);
    }
    public Rectangle(int x, int y, int width, int height) {
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }
    ...
}

```

如果需要在构造函数中调用另一个构造函数，则该调用必须放在第一行。

### 控制对类成员的访问

访问级别修饰符确定其他类是否可以使用特定字段或调用特定方法。访问控制有两个级别：

- At the top level——pbuilc，or package-private (no explicit modifier).

- At the member level—public, private, protected, or package-private (no explicit modifier).

可以使用修饰符public声明一个类，在这种情况下，该类对于所有类都可见。如果一个类没有修饰符（默认，也称为包私有package-private），它只在自己的包中可见

在成员级别，您也可以使用public修饰符或不用修饰符（package-private），就像在类中使用一样，并且具有相同的含义。对于成员，还有两个额外的访问修饰符：private和protected。 private修饰符指定只能在其自己的类中访问该成员。 protected修饰符指定只能在其自己的包中访问该成员（与package-private一样），此外，还可以在另一个包中通过其类的子类访问该成员。

下表显示了每个修饰符允许的成员访问权限：

|Modifier|Class|Package|Subclass|World|
|:---:|:---:|:----:|:---:|:---:|
|public|Y|Y|Y|Y|
|protected|Y|Y|Y|N|
|no modifier|Y|Y|N|N|
|private|Y|N|N|N|

访问级别以两种方式产生影响。首先，当使用来自其他源的类（例如Java平台中的类）时，访问级别将确定当前类可以使用的被调用类的哪些成员。其次，当编写一个类时，需要确定每个成员变量和每个方法应具有的访问级别。

选择访问级别时的提示：

- 使用对特定成员有意义的最严格的访问级别。除非有充分的理由，否则优先使用private

- 避免除常量之外的public字段。 （本教程中的许多示例都使用public字段。这可能有助于简明地说明某些要点，但不建议用于生产代码。）public字段倾向于链接到特定实现，并限制更改代码的灵活性。

### 理解类成员 Understanding Class Members

在本节中，我们将讨论使用static关键字创建属于类的字段和方法，而不是类的实例。

#### 类变量 Class Variables

当从同一个类蓝图创建许多对象时，它们每个都有自己不同的实例变量instance variables副本。以Bicycle类为例，实例变量是 cadence, gear, speed。每个Bicycle对象都有自己的值，这些变量存储在不同的内存位置。

有时，如果想拥有所有对象共有的变量，这
可以通过static修饰符实现。在声明中具有static修饰符的字段称为静态字段static fields或类变量class variables。它们与类相关联，而不是与任何对象相关联。该类的每个实例共享一个类变量，该变量位于内存中的一个固定位置。任何对象都可以更改类变量的值，但也可以在不创建类实例的情况下操作类变量。

类变量由类名本身引用，如：

```java

Bicycle.numberOfBicycles

```

这种引用方式楚地表明它们是类变量class variables.

> note:也可以使用对象引用来引用静态字段，如下面所示，但是这是不被鼓励的，因为
这种方式没有清楚的表明引用的是类变量。

```java

myBike.numberOfBicycles

```

#### 类方法 Class Methods

和类变量一样，类方法也由static修饰符定义，应该使用类名调用，并且不必创建一个类的实例。

类方法也可以使用类的实例名调用，但是不鼓励这种方式，因为这种方式并没有清晰的表明正在调用类方法Class Methods

静态方法的常见用途是访问静态字段。

并非所有实例和类变量和方法的组合都是允许的：

- 实例方法可以直接访问实例变量和实例方法。
- 实例方法可以直接访问类变量和类方法。
- 类方法可以直接访问类变量和类方法。
- *类方法**不能**直接访问实例变量或实例方法,它们必须使用对象引用。此外，类方法不能使用this关键字，因为没有要引用的实例。？？？*

### 常量Constants

static修饰符与final修饰符结合使用，也用于定义常量。final修饰符表示此字段的值不能更改。下面定义了一个常量PI

```java

static final double PI = 3.141592653589793;

```

以这种方式定义的常量不能重新分配，如果程序尝试这样做，则会发生编译时错误。按照惯例，常量值的名称拼写为大写字母。如果名称由多个单词组成，则单词由下划线（_）分隔。

> note: 如果一个常量是基本类型或字符串类型，并且在编译时该值已知，则编译器会将代码中的常量名称替换为其值。这称为编译时常量compile-time constant。如果外部世界中常量的值发生变化（例如，如果立法规定pi实际上应该是3.975），则需要重新编译使用此常量的所有类，以获取当前的新值。


### 初始化字段 Initializing Fields

如下，是常用的初始化一个字段的方法：

```java


public class BedAndBreakfast {

    // initialize to 10
    public static int capacity = 10;

    // initialize to false
    private boolean full = false;
}

```

当初始化值可用并且初始化可以放在一行上时，这很有效。然而，这种形式的初始化由于其简单性而具有局限性。如果初始化需要一些逻辑（例如，错误处理或for循环来填充复杂数组），则简单的赋值是不合适的。

实例变量可以在构造函数中初始化，其中可以使用错误处理或其他逻辑。为了为类变量提供相同的功能，Java编程语言包括静态初始化块。

> note: 虽然这是最常见的做法，但没有必要在类定义的开头声明字段。只有在使用它们之前才需要声明和初始化它们。

#### *静态初始化块???*

静态初始化块是用大括号{}括起来的常规代码块，前面是static关键字。这是一个例子:

```java

static {
    // whatever code is needed for initialization goes here
}

```

一个类可以有任意数量的静态初始化块，它们可以出现在类体中的任何位置。运行时系统保证按照它们在源代码中出现的顺序调用静态初始化块。

还有静态块的替代方法 - 您可以编写私有静态方法，如下：

```java

class Whatever {
    public static varType myVar = initializeClassVariable(); 
    private static varType initializeClassVariable() {

        // initialization code goes here
    }
}

```

私有静态方法的优点是，如果需要重新初始化类变量，可以在以后重用它们

#### 初始化实例成员 Initializing Instance Members

通常，可以使用代码在构造函数中初始化实例变量。除此之外，还有两种方法可用:初始化块 initializer blocks和final方法

实例变量的初始化程序块看起来就像静态初始化程序块，但没有static关键字

```java

{
    // whatever code is needed for initialization goes here
}

```

Java编译器将初始化程序块复制到每个构造函数中。因此，该方法可用于在多个构造函数之间共享代码块。

无法在子类中重写final方法。这在接口和继承的课程中讨论。以下是使用final方法初始化实例变量的示例:

```java

class Whatever {
    private varType myVar = initializeInstanceVariable();

    protected final varType initializeInstanceVariable() {

        // initialization code goes here
    }
}

```

如果子类可能想要重用初始化方法，这尤其有用。该方法是最终的，因为在实例初始化期间调用非final方法可能会导致问题。

### 关于创建和使用类和对象的总结

## 嵌套类 Nested Classes

嵌套类的定义如下

```java


class OuterClass {
    ...
    class NestedClass {
        ...
    }
}

```

嵌套类分为两类：静态和非静态。声明为static的嵌套类称为静态嵌套类static nested classes。非静态嵌套类称为内部类inner classes。如下所示：

```java

class OuterClass {
    ...
    static class StaticNestedClass {
        ...
    }
    class InnerClass {
        ...
    }
}

```

嵌套类是其封闭类的成员。非静态嵌套类（内部类）可以访问封闭类的其他成员，即使它们被声明为私有。静态嵌套类无权访问封闭类的其他成员。作为OuterClass的成员，可以将嵌套类声明为private，public，protected或package private。 （回想一下，外部类只能声明为public或package private）。

### 为什么使用嵌套类

    - 它是一种对仅在一个地方使用的类进行逻辑分组的方法：如果一个类只对另一个类有用，那么将它嵌入该类并将两者保持在一起是合乎逻辑的。嵌套这样的“帮助类”"helper classes"使得它们的包更加简化。
    - 它增加了封装：考虑两个顶级类A和B，其中B需要访问A的私有成员。通过将类B隐藏在类A中，可以将A的成员声明为私有，并且B可以访问它们。另外，B本身也可以对外部世界不可见。
    - 更好的可读性和可维护性：在顶级类中嵌套小的类会使代码更接近于使用它的位置。

### 静态嵌套类 Static Nested Classes

与类方法和变量一样，静态嵌套类与其外部类相关联。和静态类方法一样，静态嵌套类不能直接引用其封闭类中定义的实例变量或方法：它只能通过对象引用来使用它们。
创建一个静态嵌套类的定义如下：

```java

OuterClass.StaticNestedClass nestedObject =
     new OuterClass.StaticNestedClass();

```

### 内部类 Inner Classes

与实例方法和变量一样，内部类与其封闭类its enclosing instance的实例相关联，并且可以直接访问该对象的方法和字段。此外，由于内部类与实例相关联，因此无法定义任何静态成员本身。

作为内部类的实例的对象存在于外部类的实例中。考虑以下类：

```java

class OuterClass {
    ...
    class InnerClass {
        ...
    }
}

```

InnerClass的实例只能存在于OuterClass的实例中，并且可以直接访问其封闭实例的方法和字段。
要实例化内部类，必须首先实例化外部类。然后，使用以下语法在外部对象中创建内部对象：

```java

OuterClass.InnerClass innerObject = outerObject.new InnerClass();

```

有两种特殊类型的内部类inner classes：局部类local classes，匿名类anonymous classes

### 屏蔽Shadowing

如果特定范围（例如内部类或方法定义）中的类型声明（例如成员变量或参数名称）与封闭范围the enclosing scope中的另一个声明具有相同的名称，则该声明将隐藏封闭范围中的声明。如下所示：

```java

public class ShadowTest {

    public int x = 0;

    class FirstLevel {

        public int x = 1;

        void methodInFirstLevel(int x) {

            //x = 23
            System.out.println("x = " + x);

            //this.x = 1
            System.out.println("this.x = " + this.x);

            //ShadowTest.this.x = 0
            System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
        }
    }

    public static void main(String... args) {
        ShadowTest st = new ShadowTest();
        ShadowTest.FirstLevel fl = st.new FirstLevel();
        fl.methodInFirstLevel(23);
    }
}

```

### 序列化 Serialization

强烈建议不要对内部类（包括局部类和匿名类）进行序列化。当Java编译器编译某些构造（如内部类）时，它会创建合成构造synthetic constructs;这些是类，方法，字段和其他在源代码中没有相应构造的构造。合成构造使Java编译器能够在不更改JVM的情况下实现新的Java语言功能。但是，合成构造在不同的Java编译器中的实现非常不同，这意味着.class文件也可以在不同的实现之间变化。因此，如果序列化内部类，然后使用其他JRE实现反序列化，则可能存在兼容性问题。
 See the section Implicit and Synthetic Parameters in the section Obtaining Names of Method Parameters for more information about the synthetic constructs generated when an inner class is compiled.

### 内部类例子

### 局部类 Local Classes

局部类是在块中定义的类，它是平衡大括号之间的一组零个或多个语句。

#### 定义局部类Declaring Local Classes

您可以在任何块内定义局部类。例如，可以在方法体，for循环或if子句中定义局部类。如下所示：

```java

public class LocalClassExample {
  
    static String regularExpression = "[^0-9]";
  
    public static void validatePhoneNumber(
        String phoneNumber1, String phoneNumber2) {
        final int numberLength = 10;
        // Valid in JDK 8 and later:
        // int numberLength = 10;
        class PhoneNumber {

            String formattedPhoneNumber = null;

            PhoneNumber(String phoneNumber){
                // numberLength = 7;
                String currentNumber = phoneNumber.replaceAll(
                  regularExpression, "");
                if (currentNumber.length() == numberLength)
                    formattedPhoneNumber = currentNumber;
                else
                    formattedPhoneNumber = null;
            }

            public String getNumber() {
                return formattedPhoneNumber;
            }

            // Valid in JDK 8 and later:

//            public void printOriginalNumbers() {
//                System.out.println("Original numbers are " + phoneNumber1 +
//                    " and " + phoneNumber2);
//            }
        }

        PhoneNumber myNumber1 = new PhoneNumber(phoneNumber1);
        PhoneNumber myNumber2 = new PhoneNumber(phoneNumber2);
        // Valid in JDK 8 and later:

//        myNumber1.printOriginalNumbers();

        if (myNumber1.getNumber() == null) 
            System.out.println("First number is invalid");
        else
            System.out.println("First number is " + myNumber1.getNumber());
        if (myNumber2.getNumber() == null)
            System.out.println("Second number is invalid");
        else
            System.out.println("Second number is " + myNumber2.getNumber());

    }

    public static void main(String... args) {
        validatePhoneNumber("123-456-7890", "456-7890");
    }
}


```

#### 获取封闭类的成员 Accessing Members of an Enclosing Class

局部类Local Class可以访问其封闭类its enclosing class的成员。

此外，局部类可以访问所在代码块的局部变量或参数。在Java SE 8之前，局部类只能访问声明为final的局部变量；之后，也可以访问没有final声明的局部变量，不过该变量的值必须在初始化之后不能再被改变，等同于final(effectively final).

#### 局部类与内部类相似Local Classes Are Similar To Inner Classes

局部类与内部类类似，因为它们无法定义或声明任何静态成员。

局部类是非静态的，因为它们可以访问封闭块的实例成员。因此，它们不能包含大多数类型的静态声明。

不能在代码块内声明一个接口;接口本质上是静态的。

局部类可以具有静态成员，前提是它们是常量变量。（常量变量是原始类型或类型String的变量，声明为final并使用编译时常量表达式初始化。编译时常量表达式通常是可在编译时计算的字符串或算术表达式.）如下，EnglishGoodbye.farewell是一个常量

```java

    public void sayGoodbyeInEnglish() {
        class EnglishGoodbye {
            public static final String farewell = "Bye bye";
            public void sayGoodbye() {
                System.out.println(farewell);
            }
        }
        EnglishGoodbye myEnglishGoodbye = new EnglishGoodbye();
        myEnglishGoodbye.sayGoodbye();
    }

```

### 匿名类 Anonymous Classes

匿名类可以使代码更简洁。它们能够同时声明和实例化一个类。它们就像局部类，除了它们没有名称。如果只需要使用局部类一次，请使用它们。

#### 定义匿名类 Declaring Anonymous Classes

虽然局部类是类声明，但匿名类是表达式，这意味着需要在另一个表达式中定义匿名类。

#### 匿名类的语法

如下，是一个匿名类的对象的实例化：

```java

        HelloWorld frenchGreeting = new HelloWorld() {
            String name = "tout le monde";
            public void greet() {
                greetSomeone("tout le monde");
            }
            public void greetSomeone(String someone) {
                name = someone;
                System.out.println("Salut " + name);
            }
        };

```

匿名类的实例化包含如下内容：

- new操作符

- 要实现的接口的名称或要扩展的类。在此示例中，匿名类正在实现接口HelloWorld。

- 包含构造函数参数的圆括号，就像普通的类实例创建表达式一样。注意：实现接口时，没有构造函数，因此您使用一对空括号，如上所示。

- a body, 是一个类声明体。更具体地说，in body，方法声明是允许的，但语句不是。A body, which is a class declaration body. More specifically, in the body, method declarations are allowed but statements are not.

#### 访问封闭范围的本地变量，以及声明和访问匿名类的成员

像局部类一样，匿名类可以捕获变量;它们对封闭范围的局部变量具有相同的访问权限：

- 匿名类可以访问其封闭类的成员。

- 匿名类无法访问其封闭范围中未声明为final或者等同于final(effectively)的局部变量。

- 与嵌套类一样，匿名类中的类型（例如变量）的声明会影响封闭范围中具有相同名称的任何其他声明。

匿名类对其成员也具有与局部类相同的限制：

- 不能在匿名类中声明静态初始化程序或成员接口

- 匿名类可以具有静态成员，前提是它们是常量变量

可以在匿名类中声明以下内容：

- 成员Fields

- 额外的方法（即使他们没有实现超类型的任何方法）

- 实例初始化

- 局部类Local classes

但是，不能在匿名类中声明构造函数。

#### 匿名类的例子

### 匿名函数表达式 Lambda Expressions

匿名类的一个问题是，如果匿名类的实现非常简单，例如只包含一个方法的接口，那么匿名类的语法可能看起来不实用且不清楚。在这些情况下，您通常会尝试将功能functionality作为参数传递给另一个方法，例如当有人单击按钮时应采取的操作。 Lambda表达式使您可以执行此操作，将功能functionality视为方法参数，或将代码视为数据。

#### Lambda表达式的理想用例

#### Lambda表达式的语法

lambda表达式包含以下内容：

- 括号中用逗号分隔的形式参数列表。可以省略lambda表达式中参数的数据类型。此外，如果只有一个参数，则可以省略括号。

- 箭头标记 ->

- 一个主体，由单个表达式或语句块组成。如果指定单个表达式，则Java运行时将计算表达式，然后返回其值。或者，您可以使用return语句。

- 在lambda表达式中，必须将语句括在大括号（{}）中。如果只是返回类型为void类型的方法调用，就不必使用括号，如下所示：

```java

email -> System.out.println(email)

```

#### 访问封闭范围的局部变量

像本地和匿名类一样，lambda表达式可以捕获变量;它们对封闭范围的局部变量具有相同的访问权限。但是，与本地和匿名类不同，lambda表达式没有任何屏蔽shadowing问题。 Lambda表达式是词法范围的。这意味着它们不会从超类型继承任何名称或引入新级别的范围(introduce a new level of scoping)。 lambda表达式中的声明与封闭环境中的声明一样被解释。

但是，与本地和匿名类一样，lambda表达式只能访问final或effectively final的封闭块的局部变量和参数。

#### 目标类型Target Typing

你如何确定lambda表达式的类型？回想一下上面的lambda表达式：

```java

p -> p.getGender() == Person.Sex.MALE
    && p.getAge() >= 18
    && p.getAge() <= 25

```

这个lambda表达式用于以下两种方法：

```java

public static void printPersons(List<Person> roster, CheckPerson tester)

public void printPersonsWithPredicate(List<Person> roster, Predicate<Person> tester)

```

当Java运行时调用方法printPersons时，它期望```CheckPerson```的数据类型，因此lambda表达式属于这种类型。但是，当Java运行时调用方法printPersonsWithPredicate时，它期望数据类型为```Predicate<Person>```，因此lambda表达式属于此类型。这些方法所期望的数据类型称为目标类型the target type。要确定lambda表达式的类型，Java编译器将使用发现lambda表达式的上下文或情境的目标类型。因此，您只能在Java编译器可以确定目标类型的情况下使用lambda表达式,包括以下情况：

- Variable declarations
- Assignments
- Return statements
- Array initializers
- Method or constructor arguments
- Lambda expression bodies
- Conditional expressions, ?:
- Cast expressions

### 方法引用 Method References

有时，lambda表达式只会调用现有方法。在这些情况下，通过名称引用现有方法通常更清楚。方法参考使您可以这样做;对于已经有名称的方法，它们是紧凑的，易于阅读的lambda表达式。

方法引用的分类如下：

| Kind | Example |
| ---- | ------- |
| Reference to a static method | ContainingClass::staticMethodName |
| Reference to an instance method of a particular object | containingObject::instanceMethodName |
| Reference to an instance method of an arbitrary object of a particular type | ContainingType::methodName |
| Reference to a constructor | ClassName::new |

#### 静态方法的引用 Reference to a static method

方法引用```Person :: compareByAge```是对静态方法的引用。

#### 引用特定对象的实例方法 Reference to an Instance Method of a Particular Object

如下所示

```java

class ComparisonProvider {
    public int compareByName(Person a, Person b) {
        return a.getName().compareTo(b.getName());
    }

    public int compareByAge(Person a, Person b) {
        return a.getBirthday().compareTo(b.getBirthday());
    }
}
ComparisonProvider myComparisonProvider = new ComparisonProvider();
Arrays.sort(rosterAsArray, myComparisonProvider::compareByName);

```

#### 对特定类型的任意对象的实例方法的引用 Reference to an Instance Method of an Arbitrary Object of a Particular Type

如下所示：

```java

String[] stringArray = { "Barbara", "James", "Mary", "John",
    "Patricia", "Robert", "Michael", "Linda" };
Arrays.sort(stringArray, String::compareToIgnoreCase);

```

#### 对构造函数的引用 Reference to a Constructor

可以使用名称new以与静态方法相同的方式引用构造函数。如下所示

```java

Set<Person> rosterSet = transferElements(roster, HashSet::new);

Set<Person> rosterSet = transferElements(roster, HashSet<Person>::new);

```

### 何时使用嵌套类Nested Classes，局部类Local Classes，匿名类Anonymous Classes和Lambda表达式

如嵌套类一节所述，嵌套类能够对仅在一个地方使用的类进行逻辑分组，增加封装的使用，并创建更易读和可维护的代码。局部类，匿名类和lambda表达式也具有这些优点;但是，它们旨在用于更具体的情况：

1. 局部类Local class：如果需要创建一个类的多个实例，访问其构造函数或引入新的命名类型（例如，需要稍后调用其他方法），请使用它。
2. 匿名类Anonymous class：如果需要声明字段或其他方法，请使用它。
3. Lambda表达式：

    - 如果要封装要传递给其他代码的单个行为单元，请使用它。例如，如果要在集合的每个元素上执行某个操作，在进程完成时或者在进程遇到错误时，您将使用lambda表达式。

    - 如果需要功能接口的简单实例并且不应用前述条件（例如，不需要构造函数，命名类型，字段或其他方法），请使用它。

4. 嵌套类Nested class：如果与本地类的要求类似，希望更广泛地使用该类型，并且不需要访问局部变量或方法参数，则需要使用它。

    - 如果需要访问封闭实例的非公共字段和方法，请使用非静态嵌套类（或内部类）。如果不需要此访问权限，请使用静态嵌套类。

## 枚举类型

枚举类型是一种特殊的数据类型，适用于变量的取值范围是一组预定义的常量。变量必须等于为其预定义的值之一。

编译器在创建枚举时会自动添加一些特殊方法。所有枚举都隐式扩展了java.lang.Enum。由于类只能扩展一个父级，因此Java语言不支持多状态继承，因此枚举不能扩展其他任何内容。