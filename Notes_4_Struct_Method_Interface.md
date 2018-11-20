# Notes Part 4: Struct, Method and Interface

# 10 结构（struct）与方法（method）

Go 通过类型别名（alias types）和结构体的形式支持用户自定义类型，或者叫定制类型。一个带属性的结构体试图表示一个现实世界中的实体。结构体是复合类型（composite types），当需要定义一个类型，它由一系列属性组成，每个属性都有自己的类型和值的时候，就应该使用结构体，它把数据聚集在一起。然后可以访问这些数据，就好像它是一个独立实体的一部分。结构体也是值类型，因此可以通过 **new** 函数来创建。

组成结构体类型的那些数据称为 **字段（fields）**。每个字段都有一个类型和一个名字；在一个结构体中，字段名字必须是唯一的。

结构体的概念在软件工程上旧的术语叫 ADT（抽象数据类型：Abstract Data Type），在一些老的编程语言中叫 **记录（Record）**，比如 Cobol，在 C 家族的编程语言中它也存在，并且名字也是 **struct**，在面向对象的编程语言中，跟一个无方法的轻量级类一样。不过因为 Go 语言中没有类的概念，因此在 Go 中结构体有着更为重要的地位。

# 10.1 结构体定义 Struct

结构体定义的一般方式如下：

```go
type identifier struct {
    field1 type1
    field2 type2
    ...
}
```

`type T struct {a, b int}` 也是合法的语法，它更适用于简单的结构体。

结构体里的字段都有 **名字**，像 field1、field2 等，如果字段在代码中从来也不会被用到，那么可以命名它为 **_**。

结构体的字段可以是任何类型，甚至是结构体本身（参考第 [10.5](10.5.md) 节），也可以是函数或者接口（参考第 11 章）。可以声明结构体类型的一个变量，然后像下面这样给它的字段赋值：

```go
var s T
s.a = 5
s.b = 8
```

数组可以看作是一种结构体类型，不过它使用下标而不是具名的字段。

**使用 new**

使用 **new** 函数给一个新的结构体变量分配内存，它返回指向已分配内存的指针：`var t *T = new(T)`，如果需要可以把这条语句放在不同的行（比如定义是包范围的，但是分配却没有必要在开始就做）。

```go
var t *T
t = new(T)
```

写这条语句的惯用方法是：`t := new(T)`，变量 `t` 是一个指向 `T`的指针，此时结构体字段的值是它们所属类型的零值。

声明 `var t T` 也会给 `t` 分配内存，并零值化内存，但是这个时候 `t` 是类型T。在这两种方式中，`t` 通常被称做类型 T 的一个实例（instance）或对象（object）。

示例 10.1 [structs_fields.go](examples/chapter_10/structs_fields.go) 给出了一个非常简单的例子：

```go
package main
import "fmt"

type struct1 struct {
    i1  int
    f1  float32
    str string
}

func main() {
    ms := new(struct1)
    ms.i1 = 10
    ms.f1 = 15.5
    ms.str= "Chris"

    fmt.Printf("The int is: %d\n", ms.i1)
    fmt.Printf("The float is: %f\n", ms.f1)
    fmt.Printf("The string is: %s\n", ms.str)
    fmt.Println(ms)
}
```

输出：

    The int is: 10
    The float is: 15.500000
    The string is: Chris
    &{10 15.5 Chris}

使用 `fmt.Println` 打印一个结构体的默认输出可以很好的显示它的内容，类似使用 **%v** 选项。

就像在面向对象语言所作的那样，可以使用点号符给字段赋值：`structname.fieldname = value`。

同样的，使用点号符可以获取结构体字段的值：`structname.fieldname`。

在 Go 语言中这叫 **选择器（selector）**。无论变量是一个结构体类型还是一个结构体类型指针，都使用同样的 **选择器符（selector-notation）** 来引用结构体的字段：

```go
type myStruct struct { i int }
var v myStruct    // v是结构体类型变量
var p *myStruct   // p是指向一个结构体类型变量的指针
v.i
p.i
```

初始化一个结构体实例（一个结构体字面量：struct-literal）的更简短和惯用的方式如下：

```go
    ms := &struct1{10, 15.5, "Chris"}
    // 此时ms的类型是 *struct1
```

或者：

```go
    var ms struct1
    ms = struct1{10, 15.5, "Chris"}
```

混合字面量语法（composite literal syntax）`&struct1{a, b, c}` 是一种简写，底层仍然会调用 `new ()`，这里值的顺序必须按照字段顺序来写。在下面的例子中能看到可以通过在值的前面放上字段名来初始化字段的方式。表达式 `new(Type)` 和 `&Type{}` 是等价的。

时间间隔（开始和结束时间以秒为单位）是使用结构体的一个典型例子：

```go
type Interval struct {
    start int
    end   int
}
```

初始化方式：

```go
intr := Interval{0, 3}            (A)
intr := Interval{end:5, start:1}  (B)
intr := Interval{end:5}           (C)
```

在（A）中，值必须以字段在结构体定义时的顺序给出，**&** 不是必须的。（B）显示了另一种方式，字段名加一个冒号放在值的前面，这种情况下值的顺序不必一致，并且某些字段还可以被忽略掉，就像（C）中那样。

结构体类型和字段的命名遵循可见性规则（第 [4.2](04.2.md) 节），一个导出的结构体类型中有些字段是导出的，另一些不是，这是可能的。

下图说明了结构体类型实例和一个指向它的指针的内存布局：

```go
type Point struct { x, y int }
```

类型 strcut1 在定义它的包 pack1 中必须是唯一的，它的完全类型名是：`pack1.struct1`。

下面的例子 [Listing 10.2—person.go](examples/chapter_10/person.go) 显示了一个结构体 Person，一个方法，方法有一个类型为 `*Person` 的参数（因此对象本身是可以被改变的），以及三种调用这个方法的不同方式：

```go
package main
import (
    "fmt"
    "strings"
)

type Person struct {
    firstName   string
    lastName    string
}

func upPerson(p *Person) {
    p.firstName = strings.ToUpper(p.firstName)
    p.lastName = strings.ToUpper(p.lastName)
}

func main() {
    // 1-struct as a value type:
    var pers1 Person
    pers1.firstName = "Chris"
    pers1.lastName = "Woodward"
    upPerson(&pers1)
    fmt.Printf("The name of the person is %s %s\n", pers1.firstName, pers1.lastName)

    // 2—struct as a pointer:
    pers2 := new(Person)
    pers2.firstName = "Chris"
    pers2.lastName = "Woodward"
    (*pers2).lastName = "Woodward"  // 这是合法的
    upPerson(pers2)
    fmt.Printf("The name of the person is %s %s\n", pers2.firstName, pers2.lastName)

    // 3—struct as a literal:
    pers3 := &Person{"Chris","Woodward"}
    upPerson(pers3)
    fmt.Printf("The name of the person is %s %s\n", pers3.firstName, pers3.lastName)
}
```

输出：

    The name of the person is CHRIS WOODWARD
    The name of the person is CHRIS WOODWARD
    The name of the person is CHRIS WOODWARD

在上面例子的第二种情况中，可以直接通过指针，像 `pers2.lastName="Woodward"` 这样给结构体字段赋值，没有像 C++ 中那样需要使用 `->` 操作符，Go 会自动做这样的转换。

注意也可以通过解指针的方式来设置值：`(*pers2).lastName = "Woodward"`

**结构体的内存布局**

Go 语言中，结构体和它所包含的数据在内存中是以连续块的形式存在的，即使结构体中嵌套有其他的结构体，这在性能上带来了很大的优势。不像 Java 中的引用类型，一个对象和它里面包含的对象可能会在不同的内存空间中，这点和 Go 语言中的指针很像。

**递归结构体**

结构体类型可以通过引用自身来定义。这在定义链表或二叉树的元素（通常叫节点）时特别有用，此时节点包含指向临近节点的链接（地址）。如下所示，链表中的 `su`，树中的 `ri` 和 `le` 分别是指向别的节点的指针。

Go 代码：

```go
type Node struct {
    data    float64
    su      *Node
}
```

链表中的第一个元素叫 `head`，它指向第二个元素；最后一个元素叫 `tail`，它没有后继元素，所以它的 `su` 为 nil 值。当然真实的链接会有很多数据节点，并且链表可以动态增长或收缩。

同样地可以定义一个双向链表，它有一个前趋节点 `pr` 和一个后继节点 `su`：

```go
type Node struct {
    pr      *Node
    data    float64
    su      *Node
}
```


二叉树中每个节点最多能链接至两个节点：左节点（le）和右节点（ri），这两个节点本身又可以有左右节点，依次类推。树的顶层节点叫根节点（**root**），底层没有子节点的节点叫叶子节点（**leaves**），叶子节点的 `le` 和 `ri` 指针为 nil 值。在 Go 中可以如下定义二叉树：

```go
type Tree strcut {
    le      *Tree
    data    float64
    ri      *Tree
}
```

**结构体转换**

Go 中的类型转换遵循严格的规则。当为结构体定义了一个 alias 类型时，此结构体类型和它的 alias 类型都有相同的底层类型，它们可以如示例 10.3 那样互相转换，同时需要注意其中非法赋值或转换引起的编译错误。

示例 10.3：

```go
package main
import "fmt"

type number struct {
    f float32
}

type nr number   // alias type

func main() {
    a := number{5.0}
    b := nr{5.0}
    // var i float32 = b   // compile-error: cannot use b (type nr) as type float32 in assignment
    // var i = float32(b)  // compile-error: cannot convert b (type nr) to type float32
    // var c number = b    // compile-error: cannot use b (type nr) as type number in assignment
    // needs a conversion:
    var c = number(b)
    fmt.Println(a, b, c)
}
```

输出：

    {5} {5} {5}


# 10.2 使用工厂方法创建结构体实例

## 10.2.1 结构体工厂

Go 语言不支持面向对象编程语言中那样的构造子方法，但是可以很容易的在 Go 中实现 “构造子工厂”方法。为了方便通常会为类型定义一个工厂，按惯例，工厂的名字以 new 或 New 开头。假设定义了如下的 File 结构体类型：

```go
type File struct {
    fd      int     // 文件描述符
    name    string  // 文件名
}
```

下面是这个结构体类型对应的工厂方法，它返回一个指向结构体实例的指针：

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }

    return &File{fd, name}
}
```

然后这样调用它：

```go
f := NewFile(10, "./test.txt")
```

在 Go 语言中常常像上面这样在工厂方法里使用初始化来简便的实现构造函数。

如果 `File` 是一个结构体类型，那么表达式 `new(File)` 和 `&File{}` 是等价的。

这可以和大多数面向对象编程语言中笨拙的初始化方式做个比较：`File f = new File(...)`。

我们可以说是工厂实例化了类型的一个对象，就像在基于类的OO语言中那样。

如果想知道结构体类型T的一个实例占用了多少内存，可以使用：`size := unsafe.Sizeof(T{})`。

**如何强制使用工厂方法**

通过应用可见性规则参考[4.2.1节](04.2.md)、[9.5 节](09.5.md)就可以禁止使用 new 函数，强制用户使用工厂方法，从而使类型变成私有的，就像在面向对象语言中那样。

```go
type matrix struct {
    ...
}

func NewMatrix(params) *matrix {
    m := new(matrix) // 初始化 m
    return m
}
```

在其他包里使用工厂方法：

```go
package main
import "matrix"
...
wrong := new(matrix.matrix)     // 编译失败（matrix 是私有的）
right := matrix.NewMatrix(...)  // 实例化 matrix 的唯一方式
```

## 10.2.2 map 和 struct vs new() 和 make()

new 和 make 这两个内置函数已经在第 [7.2.4](07.2.md) 节通过切片的例子说明过一次。

现在为止我们已经见到了可以使用 `make()` 的三种类型中的其中两个：

    slices  /  maps / channels（见第 14 章）

下面的例子说明了在映射上使用 new 和 make 的区别以及可能发生的错误：

示例 10.4 new_make.go（不能编译）

```go
package main

type Foo map[string]string
type Bar struct {
    thingOne string
    thingTwo int
}

func main() {
    // OK
    y := new(Bar)
    (*y).thingOne = "hello"
    (*y).thingTwo = 1

    // NOT OK
    z := make(Bar) // 编译错误：cannot make type Bar
    (*z).thingOne = "hello"
    (*z).thingTwo = 1

    // OK
    x := make(Foo)
    x["x"] = "goodbye"
    x["y"] = "world"

    // NOT OK
    u := new(Foo)
    (*u)["x"] = "goodbye" // 运行时错误!! panic: assignment to entry in nil map
    (*u)["y"] = "world"
}
```

试图 `make()` 一个结构体变量，会引发一个编译错误，这还不是太糟糕，但是 `new()` 一个映射并试图使用数据填充它，将会引发运行时错误！ 因为 `new(Foo)` 返回的是一个指向 `nil` 的指针，它尚未被分配内存。所以在使用 `map` 时要特别谨慎。

# 10.4 带标签的结构体

结构体中的字段除了有名字和类型外，还可以有一个可选的标签（tag）：它是一个附属于字段的字符串，可以是文档或其他的重要标记。标签的内容不可以在一般的编程中使用，只有包 `reflect` 能获取它。我们将在下一章（第 [11.10 节](11.10.md)）中深入的探讨 `reflect`包，它可以在运行时自省类型、属性和方法，比如：在一个变量上调用 `reflect.TypeOf()` 可以获取变量的正确类型，如果变量是一个结构体类型，就可以通过 Field 来索引结构体的字段，然后就可以使用 Tag 属性。

示例 10.7 [struct_tag.go](examples/chapter_10/struct_tag.go)：

```go
package main

import (
  "fmt"
  "reflect"
)

type TagType struct { // tags
  field1 bool   "An important answer"
  field2 string "The name of the thing"
  field3 int    "How much there are"
}

func main() {
  tt := TagType{true, "Barak Obama", 1}
  for i := 0; i < 3; i++ {
    refTag(tt, i)
  }
}

func refTag(tt TagType, ix int) {
  ttType := reflect.TypeOf(tt)
  ixField := ttType.Field(ix)
  fmt.Printf("%v\n", ixField.Tag)
}
```

输出：

    An important answer
    The name of the thing
    How much there are

# 10.5 匿名字段和内嵌结构体

## 10.5.1 定义

结构体可以包含一个或多个 **匿名（或内嵌）字段**，即这些字段没有显式的名字，只有字段的类型是必须的，此时类型就是字段的名字。匿名字段本身可以是一个结构体类型，即 **结构体可以包含内嵌结构体**。

可以粗略地将这个和面向对象语言中的继承概念相比较，随后将会看到它被用来模拟类似继承的行为。Go 语言中的继承是通过内嵌或组合来实现的，所以可以说，在 Go 语言中，相比较于继承，组合更受青睐。

考虑如下的程序：

示例 10.8 [structs_anonymous_fields.go](examples/chapter_10/structs_anonymous_fields.go)：

```go
package main

import "fmt"

type innerS struct {
  in1 int
  in2 int
}

type outerS struct {
  b    int
  c    float32
  int  // anonymous field
  innerS //anonymous field
}

func main() {
  outer := new(outerS)
  outer.b = 6
  outer.c = 7.5
  outer.int = 60
  outer.in1 = 5
  outer.in2 = 10

  fmt.Printf("outer.b is: %d\n", outer.b)
  fmt.Printf("outer.c is: %f\n", outer.c)
  fmt.Printf("outer.int is: %d\n", outer.int)
  fmt.Printf("outer.in1 is: %d\n", outer.in1)
  fmt.Printf("outer.in2 is: %d\n", outer.in2)

  // 使用结构体字面量
  outer2 := outerS{6, 7.5, 60, innerS{5, 10}}
  fmt.Println("outer2 is:", outer2)
}
```

输出：

    outer.b is: 6
    outer.c is: 7.500000
    outer.int is: 60
    outer.in1 is: 5
    outer.in2 is: 10
    outer2 is:{6 7.5 60 {5 10}}

通过类型 `outer.int` 的名字来获取存储在匿名字段中的数据，于是可以得出一个结论：在一个结构体中对于每一种数据类型只能有一个匿名字段。

## 10.5.2 内嵌结构体

同样地结构体也是一种数据类型，所以它也可以作为一个匿名字段来使用，如同上面例子中那样。外层结构体通过 `outer.in1` 直接进入内层结构体的字段，内嵌结构体甚至可以来自其他包。内层结构体被简单的插入或者内嵌进外层结构体。这个简单的“继承”机制提供了一种方式，使得可以从另外一个或一些类型继承部分或全部实现。

另外一个例子：

示例 10.9 [embedd_struct.go](examples/chapter_10/embedd_struct.go)：

```go
package main

import "fmt"

type A struct {
  ax, ay int
}

type B struct {
  A
  bx, by float32
}

func main() {
  b := B{A{1, 2}, 3.0, 4.0}
  fmt.Println(b.ax, b.ay, b.bx, b.by)
  fmt.Println(b.A)
}
```

输出：

    1 2 3 4
    {1 2}

## 10.5.3 命名冲突

当两个字段拥有相同的名字（可能是继承来的名字）时该怎么办呢？

1. 外层名字会覆盖内层名字（但是两者的内存空间都保留），这提供了一种重载字段或方法的方式；
2. 如果相同的名字在同一级别出现了两次，如果这个名字被程序使用了，将会引发一个错误（不使用没关系）。没有办法来解决这种问题引起的二义性，必须由程序员自己修正。

例子：

```go
type A struct {a int}
type B struct {a, b int}

type C struct {A; B}
var c C
```

规则 2：使用 `c.a` 是错误的，到底是 `c.A.a` 还是 `c.B.a` 呢？会导致编译器错误：**ambiguous DOT reference c.a disambiguate with either c.A.a or c.B.a**。

```go
type D struct {B; b float32}
var d D
```

规则1：使用 `d.b` 是没问题的：它是 float32，而不是 `B` 的 `b`。如果想要内层的 `b` 可以通过 `d.B.b` 得到。


# 10.6 方法

## 10.6.1 方法是什么

在 Go 语言中，结构体就像是类的一种简化形式，那么面向对象程序员可能会问：类的方法在哪里呢？在 Go 中有一个概念，它和方法有着同样的名字，并且大体上意思相同：Go 方法是作用在接收者（receiver）上的一个函数，接收者是某种类型的变量。因此方法是一种特殊类型的函数。

接收者类型可以是（几乎）任何类型，不仅仅是结构体类型：任何类型都可以有方法，甚至可以是函数类型，可以是 int、bool、string 或数组的别名类型。但是接收者不能是一个接口类型（参考 第 11 章），因为接口是一个抽象定义，但是方法却是具体实现；如果这样做会引发一个编译错误：**invalid receiver type…**。

最后接收者不能是一个指针类型，但是它可以是任何其他允许类型的指针。

一个类型加上它的方法等价于面向对象中的一个类。一个重要的区别是：在 Go 中，类型的代码和绑定在它上面的方法的代码可以不放置在一起，它们可以存在在不同的源文件，唯一的要求是：它们必须是同一个包的。

类型 T（或 \*T）上的所有方法的集合叫做类型 T（或 \*T）的方法集。

因为方法是函数，所以同样的，不允许方法重载，即对于一个类型只能有一个给定名称的方法。但是如果基于接收者类型，是有重载的：具有同样名字的方法可以在 2 个或多个不同的接收者类型上存在，比如在同一个包里这么做是允许的：

```go
func (a *denseMatrix) Add(b Matrix) Matrix
func (a *sparseMatrix) Add(b Matrix) Matrix
```

别名类型没有原始类型上已经定义过的方法。

定义方法的一般格式如下：

```go
func (recv receiver_type) methodName(parameter_list) (return_value_list) { ... }
```

在方法名之前，`func` 关键字之后的括号中指定 receiver。

如果 `recv` 是 receiver 的实例，Method1 是它的方法名，那么方法调用遵循传统的 `object.name` 选择器符号：**recv.Method1()**。

如果 `recv` 是一个指针，Go 会自动解引用。

如果方法不需要使用 `recv` 的值，可以用 **_** 替换它，比如：

```go
func (_ receiver_type) methodName(parameter_list) (return_value_list) { ... }
```

`recv` 就像是面向对象语言中的 `this` 或 `self`，但是 Go 中并没有这两个关键字。随个人喜好，你可以使用 `this` 或 `self` 作为 receiver 的名字。下面是一个结构体上的简单方法的例子：

示例 10.10 method .go：

```go
package main

import "fmt"

type TwoInts struct {
  a int
  b int
}

func main() {
  two1 := new(TwoInts)
  two1.a = 12
  two1.b = 10

  fmt.Printf("The sum is: %d\n", two1.AddThem())
  fmt.Printf("Add them to the param: %d\n", two1.AddToParam(20))

  two2 := TwoInts{3, 4}
  fmt.Printf("The sum is: %d\n", two2.AddThem())
}

func (tn *TwoInts) AddThem() int {
  return tn.a + tn.b
}

func (tn *TwoInts) AddToParam(param int) int {
  return tn.a + tn.b + param
}
```

输出：

    The sum is: 22
    Add them to the param: 42
    The sum is: 7

下面是非结构体类型上方法的例子：

示例 10.11 method2.go：

```go
package main

import "fmt"

type IntVector []int

func (v IntVector) Sum() (s int) {
  for _, x := range v {
    s += x
  }
  return
}

func main() {
  fmt.Println(IntVector{1, 2, 3}.Sum()) // 输出是6
}
```

类型和作用在它上面定义的方法必须在同一个包里定义，这就是为什么不能在 int、float 或类似这些的类型上定义方法。试图在 int 类型上定义方法会得到一个编译错误：

    cannot define new methods on non-local type int

比如想在 `time.Time` 上定义如下方法：

```go
func (t time.Time) first3Chars() string {
  return time.LocalTime().String()[0:3]
}
```

类型在其他的，或是非本地的包里定义，在它上面定义方法都会得到和上面同样的错误。

但是有一个间接的方式：可以先定义该类型（比如：int 或 float）的别名类型，然后再为别名类型定义方法。或者像下面这样将它作为匿名类型嵌入在一个新的结构体中。当然方法只在这个别名类型上有效。

示例 10.12 method_on_time.go：

```go
package main

import (
  "fmt"
  "time"
)

type myTime struct {
  time.Time //anonymous field
}

func (t myTime) first3Chars() string {
  return t.Time.String()[0:3]
}
func main() {
  m := myTime{time.Now()}
  // 调用匿名Time上的String方法
  fmt.Println("Full time now:", m.String())
  // 调用myTime.first3Chars
  fmt.Println("First 3 chars:", m.first3Chars())
}

/* Output:
Full time now: Mon Oct 24 15:34:54 Romance Daylight Time 2011
First 3 chars: Mon
*/
```

## 10.6.2 函数和方法的区别

函数将变量作为参数：**Function1(recv)**

方法在变量上被调用：**recv.Method1()**

在接收者是指针时，方法可以改变接收者的值（或状态），这点函数也可以做到（当参数作为指针传递，即通过引用调用时，函数也可以改变参数的状态）。

**不要忘记 Method1 后边的括号 ()，否则会引发编译器错误：`method recv.Method1 is not an expression, must be called`**

接收者必须有一个显式的名字，这个名字必须在方法中被使用。

**receiver_type** 叫做 **（接收者）基本类型**，这个类型必须在和方法同样的包中被声明。

在 Go 中，（接收者）类型关联的方法不写在类型结构里面，就像类那样；耦合更加宽松；类型和方法之间的关联由接收者来建立。

**方法没有和数据定义（结构体）混在一起：它们是正交的类型；表示（数据）和行为（方法）是独立的。**

## 10.6.3 指针或值作为接收者

鉴于性能的原因，`recv` 最常见的是一个指向 receiver_type 的指针（因为我们不想要一个实例的拷贝，如果按值调用的话就会是这样），特别是在 receiver 类型是结构体时，就更是如此了。

如果想要方法改变接收者的数据，就在接收者的指针类型上定义该方法。否则，就在普通的值类型上定义方法。

下面的例子 `pointer_value.go` 作了说明：`change()`接受一个指向 B 的指针，并改变它内部的成员；`write()` 通过拷贝接受 B 的值并只输出B的内容。注意 Go 为我们做了探测工作，我们自己并没有指出是否在指针上调用方法，Go 替我们做了这些事情。b1 是值而 b2 是指针，方法都支持运行了。

示例 10.13 pointer_value.go：

```go
package main

import (
  "fmt"
)

type B struct {
  thing int
}

func (b *B) change() { b.thing = 1 }

func (b B) write() string { return fmt.Sprint(b) }

func main() {
  var b1 B // b1是值
  b1.change()
  fmt.Println(b1.write())

  b2 := new(B) // b2是指针
  b2.change()
  fmt.Println(b2.write())
}

/* 输出：
{1}
{1}
*/
```

试着在 `write()` 中改变接收者b的值：将会看到它可以正常编译，但是开始的 b 没有被改变。

我们知道方法将指针作为接收者不是必须的，如下面的例子，我们只是需要 `Point3` 的值来做计算：

```go
type Point3 struct { x, y, z float64 }
// A method on Point3
func (p Point3) Abs() float64 {
    return math.Sqrt(p.x*p.x + p.y*p.y + p.z*p.z)
}
```

这样做稍微有点昂贵，因为 `Point3` 是作为值传递给方法的，因此传递的是它的拷贝，这在 Go 中是合法的。也可以在指向这个类型的指针上调用此方法（会自动解引用）。

假设 `p3` 定义为一个指针：`p3 := &Point{ 3, 4, 5}`。

可以使用 `p3.Abs()` 来替代 `(*p3).Abs()`。

像例子 10.10（method1.go）中接收者类型是 `*TwoInts` 的方法 `AddThem()`，它能在类型 `TwoInts` 的值上被调用，这是自动间接发生的。

因此 `two2.AddThem` 可以替代 `(&two2).AddThem()`。

在值和指针上调用方法：

可以有连接到类型的方法，也可以有连接到类型指针的方法。

但是这没关系：对于类型 T，如果在 \*T 上存在方法 `Meth()`，并且 `t` 是这个类型的变量，那么 `t.Meth()` 会被自动转换为 `(&t).Meth()`。

**指针方法和值方法都可以在指针或非指针上被调用**，如下面程序所示，类型 `List` 在值上有一个方法 `Len()`，在指针上有一个方法 `Append()`，但是可以看到两个方法都可以在两种类型的变量上被调用。

示例 10.14 methodset1.go：

```go
package main

import (
  "fmt"
)

type List []int

func (l List) Len() int        { return len(l) }
func (l *List) Append(val int) { *l = append(*l, val) }

func main() {
  // 值
  var lst List
  lst.Append(1)
  fmt.Printf("%v (len: %d)", lst, lst.Len()) // [1] (len: 1)

  // 指针
  plst := new(List)
  plst.Append(2)
  fmt.Printf("%v (len: %d)", plst, plst.Len()) // &[2] (len: 1)
}
```

## 10.6.4 方法和未导出字段

考虑 `person2.go` 中的 `person` 包：类型 `Person` 被明确的导出了，但是它的字段没有被导出。例如在 `use_person2.go` 中 `p.firstName` 就是错误的。该如何在另一个程序中修改或者只是读取一个 `Person` 的名字呢？

这可以通过面向对象语言一个众所周知的技术来完成：提供 getter 和 setter 方法。对于 setter 方法使用 Set 前缀，对于 getter 方法只使用成员名。

示例 10.15 person2.go：

```go
package person

type Person struct {
  firstName string
  lastName  string
}

func (p *Person) FirstName() string {
  return p.firstName
}

func (p *Person) SetFirstName(newName string) {
  p.firstName = newName
}
```

示例 10.16—use_person2.go：

```go
package main

import (
  "./person"
  "fmt"
)

func main() {
  p := new(person.Person)
  // p.firstName undefined
  // (cannot refer to unexported field or method firstName)
  // p.firstName = "Eric"
  p.SetFirstName("Eric")
  fmt.Println(p.FirstName()) // Output: Eric
}
```

**并发访问对象**

对象的字段（属性）不应该由 2 个或 2 个以上的不同线程在同一时间去改变。如果在程序发生这种情况，为了安全并发访问，可以使用包 `sync`（参考第 9.3 节）中的方法。在第 14.17 节中我们会通过 goroutines 和 channels 探索另一种方式。

## 10.6.5 内嵌类型的方法和继承

当一个匿名类型被内嵌在结构体中时，匿名类型的可见方法也同样被内嵌，这在效果上等同于外层类型 **继承** 了这些方法：**将父类型放在子类型中来实现亚型**。这个机制提供了一种简单的方式来模拟经典面向对象语言中的子类和继承相关的效果，也类似 Ruby 中的混入（mixin）。

下面是一个示例：假定有一个 `Engine` 接口类型，一个 `Car` 结构体类型，它包含一个 `Engine` 类型的匿名字段：

```go
type Engine interface {
  Start()
  Stop()
}

type Car struct {
  Engine
}
```

我们可以构建如下的代码：

```go
func (c *Car) GoToWorkIn() {
  // get in car
  c.Start()
  // drive to work
  c.Stop()
  // get out of car
}
```

下面是 `method3.go` 的完整例子，它展示了内嵌结构体上的方法可以直接在外层类型的实例上调用：

```go
package main

import (
  "fmt"
  "math"
)

type Point struct {
  x, y float64
}

func (p *Point) Abs() float64 {
  return math.Sqrt(p.x*p.x + p.y*p.y)
}

type NamedPoint struct {
  Point
  name string
}

func main() {
  n := &NamedPoint{Point{3, 4}, "Pythagoras"}
  fmt.Println(n.Abs()) // 打印5
}
```

内嵌将一个已存在类型的字段和方法注入到了另一个类型里：匿名字段上的方法“晋升”成为了外层类型的方法。当然类型可以有只作用于本身实例而不作用于内嵌“父”类型上的方法，

可以覆写方法（像字段一样）：和内嵌类型方法具有同样名字的外层类型的方法会覆写内嵌类型对应的方法。

在示例 10.18 method4.go 中添加：

```go
func (n *NamedPoint) Abs() float64 {
  return n.Point.Abs() * 100.
}
```

现在 `fmt.Println(n.Abs())` 会打印 `500`。

因为一个结构体可以嵌入多个匿名类型，所以实际上我们可以有一个简单版本的多重继承，就像：`type Child struct { Father; Mother}`。在第 10.6.7 节中会进一步讨论这个问题。

结构体内嵌和自己在同一个包中的结构体时，可以彼此访问对方所有的字段和方法。


## 10.6.6 如何在类型中嵌入功能

主要有两种方法来实现在类型中嵌入功能：

A：聚合（或组合）：包含一个所需功能类型的具名字段。

B：内嵌：内嵌（匿名地）所需功能类型，像前一节 10.6.5 所演示的那样。

为了使这些概念具体化，假设有一个 `Customer` 类型，我们想让它通过 `Log` 类型来包含日志功能，`Log` 类型只是简单地包含一个累积的消息（当然它可以是复杂的）。如果想让特定类型都具备日志功能，你可以实现一个这样的 `Log` 类型，然后将它作为特定类型的一个字段，并提供 `Log()`，它返回这个日志的引用。

方式 A 可以通过如下方法实现（使用了第 10.7 节中的 `String()` 功能）：

示例 10.19 embed_func1.go：

```go
package main

import (
  "fmt"
)

type Log struct {
  msg string
}

type Customer struct {
  Name string
  log  *Log
}

func main() {
  c := new(Customer)
  c.Name = "Barak Obama"
  c.log = new(Log)
  c.log.msg = "1 - Yes we can!"
  // shorter
  c = &Customer{"Barak Obama", &Log{"1 - Yes we can!"}}
  // fmt.Println(c) &{Barak Obama 1 - Yes we can!}
  c.Log().Add("2 - After me the world will be a better place!")
  //fmt.Println(c.log)
  fmt.Println(c.Log())

}

func (l *Log) Add(s string) {
  l.msg += "\n" + s
}

func (l *Log) String() string {
  return l.msg
}

func (c *Customer) Log() *Log {
  return c.log
}
```

输出：

    1 - Yes we can!
    2 - After me the world will be a better place!

相对的方式 B 可能会像这样：

```go
package main

import (
  "fmt"
)

type Log struct {
  msg string
}

type Customer struct {
  Name string
  Log
}

func main() {
  c := &Customer{"Barak Obama", Log{"1 - Yes we can!"}}
  c.Add("2 - After me the world will be a better place!")
  fmt.Println(c)

}

func (l *Log) Add(s string) {
  l.msg += "\n" + s
}

func (l *Log) String() string {
  return l.msg
}

func (c *Customer) String() string {
  return c.Name + "\nLog:" + fmt.Sprintln(c.Log)
}
```

输出：

    Barak Obama
    Log:{1 - Yes we can!
    2 - After me the world will be a better place!}

内嵌的类型不需要指针，`Customer` 也不需要 `Add` 方法，它使用 `Log` 的 `Add` 方法，`Customer` 有自己的 `String` 方法，并且在它里面调用了 `Log` 的 `String` 方法。

如果内嵌类型嵌入了其他类型，也是可以的，那些类型的方法可以直接在外层类型中使用。

因此一个好的策略是创建一些小的、可复用的类型作为一个工具箱，用于组成域类型。

## 10.6.7 多重继承

多重继承指的是类型获得多个父类型行为的能力，它在传统的面向对象语言中通常是不被实现的（C++ 和 Python 例外）。因为在类继承层次中，多重继承会给编译器引入额外的复杂度。但是在 Go 语言中，通过在类型中嵌入所有必要的父类型，可以很简单的实现多重继承。

作为一个例子，假设有一个类型 `CameraPhone`，通过它可以 `Call()`，也可以 `TakeAPicture()`，但是第一个方法属于类型 `Phone`，第二个方法属于类型 `Camera`。

只要嵌入这两个类型就可以解决这个问题，如下所示：

```go
package main

import (
  "fmt"
)

type Camera struct{}

func (c *Camera) TakeAPicture() string {
  return "Click"
}

type Phone struct{}

func (p *Phone) Call() string {
  return "Ring Ring"
}

type CameraPhone struct {
  Camera
  Phone
}

func main() {
  cp := new(CameraPhone)
  fmt.Println("Our new CameraPhone exhibits multiple behaviors...")
  fmt.Println("It exhibits behavior of a Camera: ", cp.TakeAPicture())
  fmt.Println("It works like a Phone too: ", cp.Call())
}
```

输出：

    Our new CameraPhone exhibits multiple behaviors...
    It exhibits behavior of a Camera: Click
    It works like a Phone too: Ring Ring


## 10.6.8 通用方法和方法命名

在编程中一些基本操作会一遍又一遍的出现，比如打开（Open）、关闭（Close）、读（Read）、写（Write）、排序（Sort）等等，并且它们都有一个大致的意思：打开（Open）可以作用于一个文件、一个网络连接、一个数据库连接等等。具体的实现可能千差万别，但是基本的概念是一致的。在 Go 语言中，通过使用接口（参考 第 11 章），标准库广泛的应用了这些规则，在标准库中这些通用方法都有一致的名字，比如 `Open()`、`Read()`、`Write()`等。想写规范的 Go 程序，就应该遵守这些约定，给方法合适的名字和签名，就像那些通用方法那样。这样做会使 Go 开发的软件更加具有一致性和可读性。比如：如果需要一个 convert-to-string 方法，应该命名为 `String()`，而不是 `ToString()`（参考第 10.7 节）。

## 10.6.9 和其他面向对象语言比较 Go 的类型和方法

在如 C++、Java、C# 和 Ruby 这样的面向对象语言中，方法在类的上下文中被定义和继承：在一个对象上调用方法时，运行时会检测类以及它的超类中是否有此方法的定义，如果没有会导致异常发生。

在 Go 语言中，这样的继承层次是完全没必要的：如果方法在此类型定义了，就可以调用它，和其他类型上是否存在这个方法没有关系。在这个意义上，Go 具有更大的灵活性。

Go 不需要一个显式的类定义，如同 Java、C++、C# 等那样，相反地，“类”是通过提供一组作用于一个共同类型的方法集来隐式定义的。类型可以是结构体或者任何用户自定义类型。

比如：我们想定义自己的 `Integer` 类型，并添加一些类似转换成字符串的方法，在 Go 中可以如下定义：

```go
type Integer int
func (i *Integer) String() string {
    return strconv.Itoa(int(*i))
}
```

在 Java 或 C# 中，这个方法需要和类 `Integer` 的定义放在一起，在 Ruby 中可以直接在基本类型 int 上定义这个方法。

**总结**

在 Go 中，类型就是类（数据和关联的方法）。Go 不知道类似面向对象语言的类继承的概念。继承有两个好处：代码复用和多态。

在 Go 中，代码复用通过组合和委托实现，多态通过接口的使用来实现：有时这也叫 **组件编程（Component Programming）**。

许多开发者说相比于类继承，Go 的接口提供了更强大、却更简单的多态行为。


# 10.7 类型的 String() 方法和格式化描述符

当定义了一个有很多方法的类型时，十之八九你会使用 `String()` 方法来定制类型的字符串形式的输出，换句话说：一种可阅读性和打印性的输出。如果类型定义了 `String()` 方法，它会被用在 `fmt.Printf()` 中生成默认的输出：等同于使用格式化描述符 `%v` 产生的输出。还有 `fmt.Print()` 和 `fmt.Println()` 也会自动使用 `String()` 方法。

我们使用第 10.4 节中程序的类型来进行测试：

示例 10.22 method_string.go：

```go
package main

import (
  "fmt"
  "strconv"
)

type TwoInts struct {
  a int
  b int
}

func main() {
  two1 := new(TwoInts)
  two1.a = 12
  two1.b = 10
  fmt.Printf("two1 is: %v\n", two1)
  fmt.Println("two1 is:", two1)
  fmt.Printf("two1 is: %T\n", two1)
  fmt.Printf("two1 is: %#v\n", two1)
}

func (tn *TwoInts) String() string {
  return "(" + strconv.Itoa(tn.a) + "/" + strconv.Itoa(tn.b) + ")"
}
```

输出：

    two1 is: (12/10)
    two1 is: (12/10)
    two1 is: *main.TwoInts
    two1 is: &main.TwoInts{a:12, b:10}

当你广泛使用一个自定义类型时，最好为它定义 `String()`方法。从上面的例子也可以看到，格式化描述符 `%T` 会给出类型的完全规格，`%#v` 会给出实例的完整输出，包括它的字段（在程序自动生成 `Go` 代码时也很有用）。

**备注**

不要在 `String()` 方法里面调用涉及 `String()` 方法的方法，它会导致意料之外的错误，比如下面的例子，它导致了一个无限递归调用（`TT.String()` 调用 `fmt.Sprintf`，而 `fmt.Sprintf` 又会反过来调用 `TT.String()`...），很快就会导致内存溢出：

```go
type TT float64

func (t TT) String() string {
    return fmt.Sprintf("%v", t)
}
t.String()
```


实现栈（stack）数据结构：

它的格子包含数据，比如整数 i、j、k 和 l 等等，格子从底部（索引 0）至顶部（索引 n）来索引。这个例子中假定 `n=3`，那么一共有 4 个格子。

一个新栈中所有格子的值都是 0。

push 将一个新值放到栈的最顶部一个非空（非零）的格子中。

pop 获取栈的最顶部一个非空（非零）的格子的值。现在可以理解为什么栈是一个后进先出（LIFO）的结构了吧。

为栈定义一 `Stack` 类型，并为它定义一个 `Push` 和 `Pop` 方法，再为它定义 `String()` 方法（用于调试）它输出栈的内容，比如：`[0:i] [1:j] [2:k] [3:l]`。

1）stack_arr.go：使用长度为 4 的 int 数组作为底层数据结构。

2）stack_struct.go：使用包含一个索引和一个 int 数组的结构体作为底层数据结构，索引表示第一个空闲的位置。

3）使用常量 LIMIT 代替上面表示元素个数的 4 重新实现上面的 1）和 2），使它们更具有一般性。


# 10.8 垃圾回收和 SetFinalizer

Go 开发者不需要写代码来释放程序中不再使用的变量和结构占用的内存，在 Go 运行时中有一个独立的进程，即垃圾收集器（GC），会处理这些事情，它搜索不再使用的变量然后释放它们的内存。可以通过 `runtime` 包访问 GC 进程。

通过调用 `runtime.GC()` 函数可以显式的触发 GC，但这只在某些罕见的场景下才有用，比如当内存资源不足时调用 `runtime.GC()`，它会在此函数执行的点上立即释放一大片内存，此时程序可能会有短时的性能下降（因为 `GC` 进程在执行）。

如果想知道当前的内存状态，可以使用：

```go
// fmt.Printf("%d\n", runtime.MemStats.Alloc/1024)
// 此处代码在 Go 1.5.1下不再有效，更正为
var m runtime.MemStats
runtime.ReadMemStats(&m)
fmt.Printf("%d Kb\n", m.Alloc / 1024)
```

上面的程序会给出已分配内存的总量，单位是 Kb。

如果需要在一个对象 obj 被从内存移除前执行一些特殊操作，比如写到日志文件中，可以通过如下方式调用函数来实现：

```go
runtime.SetFinalizer(obj, func(obj *typeObj))
```

`func(obj *typeObj)` 需要一个 `typeObj` 类型的指针参数 `obj`，特殊操作会在它上面执行。`func` 也可以是一个匿名函数。

在对象被 GC 进程选中并从内存中移除以前，`SetFinalizer` 都不会执行，即使程序正常结束或者发生错误。


# 11.1 接口是什么

Go 语言不是一种 *“传统”* 的面向对象编程语言：它里面没有类和继承的概念。

但是 Go 语言里有非常灵活的 **接口** 概念，通过它可以实现很多面向对象的特性。接口提供了一种方式来 **说明** 对象的行为：如果谁能搞定这件事，它就可以用在这儿。

接口定义了一组方法（方法集），但是这些方法不包含（实现）代码：它们没有被实现（它们是抽象的）。接口里也不能包含变量。

通过如下格式定义接口：

```go
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
```

上面的 `Namer` 是一个 **接口类型**。

（按照约定，只包含一个方法的）接口的名字由方法名加 `[e]r` 后缀组成，例如 `Printer`、`Reader`、`Writer`、`Logger`、`Converter` 等等。还有一些不常用的方式（当后缀 `er` 不合适时），比如 `Recoverable`，此时接口名以 `able` 结尾，或者以 `I` 开头（像 `.NET` 或 `Java` 中那样）。

Go 语言中的接口都很简短，通常它们会包含 0 个、最多 3 个方法。

不像大多数面向对象编程语言，在 Go 语言中接口可以有值，一个接口类型的变量或一个 **接口值** ：`var ai Namer`，`ai` 是一个多字（multiword）数据结构，它的值是 `nil`。它本质上是一个指针，虽然不完全是一回事。指向接口值的指针是非法的，它们不仅一点用也没有，还会导致代码错误。

此处的方法指针表是通过运行时反射能力构建的。

类型（比如结构体）实现接口方法集中的方法，每一个方法的实现说明了此方法是如何作用于该类型的：**即实现接口**，同时方法集也构成了该类型的接口。实现了 `Namer` 接口类型的变量可以赋值给 `ai` （接收者值），此时方法表中的指针会指向被实现的接口方法。当然如果另一个类型（也实现了该接口）的变量被赋值给 `ai`，这二者（译者注：指针和方法实现）也会随之改变。

**类型不需要显式声明它实现了某个接口：接口被隐式地实现。多个类型可以实现同一个接口**。

**实现某个接口的类型（除了实现接口方法外）可以有其他的方法**。

**一个类型可以实现多个接口**。

**接口类型可以包含一个实例的引用， 该实例的类型实现了此接口（接口是动态类型）**。

即使接口在类型之后才定义，二者处于不同的包中，被单独编译：只要类型实现了接口中的方法，它就实现了此接口。

所有这些特性使得接口具有很大的灵活性。

第一个例子：

示例 11.1 [interfaces.go](examples/chapter_11/interfaces.go)：

```go
package main

import "fmt"

type Shaper interface {
  Area() float32
}

type Square struct {
  side float32
}

func (sq *Square) Area() float32 {
  return sq.side * sq.side
}

func main() {
  sq1 := new(Square)
  sq1.side = 5

  var areaIntf Shaper
  areaIntf = sq1
  // shorter,without separate declaration:
  // areaIntf := Shaper(sq1)
  // or even:
  // areaIntf := sq1
  fmt.Printf("The square has area: %f\n", areaIntf.Area())
}
```

输出：

    The square has area: 25.000000

上面的程序定义了一个结构体 `Square` 和一个接口 `Shaper`，接口有一个方法 `Area()`。

在 `main()` 方法中创建了一个 `Square` 的实例。在主程序外边定义了一个接收者类型是 `Square` 方法的 `Area()`，用来计算正方形的面积：结构体 `Square` 实现了接口 `Shaper` 。

所以可以将一个 `Square` 类型的变量赋值给一个接口类型的变量：`areaIntf = sq1` 。

现在接口变量包含一个指向 `Square` 变量的引用，通过它可以调用 `Square` 上的方法 `Area()`。当然也可以直接在 `Square` 的实例上调用此方法，但是在接口实例上调用此方法更令人兴奋，它使此方法更具有一般性。接口变量里包含了接收者实例的值和指向对应方法表的指针。

这是 **多态** 的 Go 版本，多态是面向对象编程中一个广为人知的概念：根据当前的类型选择正确的方法，或者说：同一种类型在不同的实例上似乎表现出不同的行为。

如果 `Square` 没有实现 `Area()` 方法，编译器将会给出清晰的错误信息：

    cannot use sq1 (type *Square) as type Shaper in assignment:
    *Square does not implement Shaper (missing Area method)

如果 `Shaper` 有另外一个方法 `Perimeter()`，但是`Square` 没有实现它，即使没有人在 `Square` 实例上调用这个方法，编译器也会给出上面同样的错误。

扩展一下上面的例子，类型 `Rectangle` 也实现了 `Shaper` 接口。接着创建一个 `Shaper` 类型的数组，迭代它的每一个元素并在上面调用 `Area()` 方法，以此来展示多态行为：

示例 11.2 [interfaces_poly.go](examples/chapter_11/interfaces_poly.go)：

```go
package main

import "fmt"

type Shaper interface {
  Area() float32
}

type Square struct {
  side float32
}

func (sq *Square) Area() float32 {
  return sq.side * sq.side
}

type Rectangle struct {
  length, width float32
}

func (r Rectangle) Area() float32 {
  return r.length * r.width
}

func main() {

  r := Rectangle{5, 3} // Area() of Rectangle needs a value
  q := &Square{5}      // Area() of Square needs a pointer
  // shapes := []Shaper{Shaper(r), Shaper(q)}
  // or shorter
  shapes := []Shaper{r, q}
  fmt.Println("Looping through shapes for area ...")
  for n, _ := range shapes {
    fmt.Println("Shape details: ", shapes[n])
    fmt.Println("Area of this shape is: ", shapes[n].Area())
  }
}
```

输出：

    Looping through shapes for area ...
    Shape details:  {5 3}
    Area of this shape is:  15
    Shape details:  &{5}
    Area of this shape is:  25

在调用 `shapes[n].Area() ` 这个时，只知道 `shapes[n]` 是一个 `Shaper` 对象，最后它摇身一变成为了一个 `Square` 或 `Rectangle` 对象，并且表现出了相对应的行为。

也许从现在开始你将看到通过接口如何产生 **更干净**、**更简单** 及 **更具有扩展性** 的代码。在 11.12.3 中将看到在开发中为类型添加新的接口是多么的容易。

下面是一个更具体的例子：有两个类型 `stockPosition` 和 `car`，它们都有一个 `getValue()` 方法，我们可以定义一个具有此方法的接口 `valuable`。接着定义一个使用 `valuable` 类型作为参数的函数 `showValue()`，所有实现了 `valuable` 接口的类型都可以用这个函数。

示例 11.3 [valuable.go](examples/chapter_11/valuable.go)：

```go
package main

import "fmt"

type stockPosition struct {
  ticker     string
  sharePrice float32
  count      float32
}

/* method to determine the value of a stock position */
func (s stockPosition) getValue() float32 {
  return s.sharePrice * s.count
}

type car struct {
  make  string
  model string
  price float32
}

/* method to determine the value of a car */
func (c car) getValue() float32 {
  return c.price
}

/* contract that defines different things that have value */
type valuable interface {
  getValue() float32
}

func showValue(asset valuable) {
  fmt.Printf("Value of the asset is %f\n", asset.getValue())
}

func main() {
  var o valuable = stockPosition{"GOOG", 577.20, 4}
  showValue(o)
  o = car{"BMW", "M3", 66500}
  showValue(o)
}
```

输出：

    Value of the asset is 2308.800049
    Value of the asset is 66500.000000

**一个标准库的例子**

`io` 包里有一个接口类型 `Reader`:

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

定义变量 `r`：` var r io.Reader`

那么就可以写如下的代码：

```go
  var r io.Reader
  r = os.Stdin    // see 12.1
  r = bufio.NewReader(r)
  r = new(bytes.Buffer)
  f,_ := os.Open("test.txt")
  r = bufio.NewReader(f)
```

上面 `r` 右边的类型都实现了 `Read()` 方法，并且有相同的方法签名，`r` 的静态类型是 `io.Reader`。

**备注**

有的时候，也会以一种稍微不同的方式来使用接口这个词：从某个类型的角度来看，它的接口指的是：它的所有导出方法，只不过没有显式地为这些导出方法额外定一个接口而已。


# 11.2 接口嵌套接口

一个接口可以包含一个或多个其他的接口，这相当于直接将这些内嵌接口的方法列举在外层接口中一样。

比如接口 `File` 包含了 `ReadWrite` 和 `Lock` 的所有方法，它还额外有一个 `Close()` 方法。

```go
type ReadWrite interface {
    Read(b Buffer) bool
    Write(b Buffer) bool
}

type Lock interface {
    Lock()
    Unlock()
}

type File interface {
    ReadWrite
    Lock
    Close()
}
```

# 11.3 类型断言：如何检测和转换接口变量的类型

一个接口类型的变量 `varI` 中可以包含任何类型的值，必须有一种方式来检测它的 **动态** 类型，即运行时在变量中存储的值的实际类型。在执行过程中动态类型可能会有所不同，但是它总是可以分配给接口变量本身的类型。通常我们可以使用 **类型断言** 来测试在某个时刻 `varI` 是否包含类型 `T` 的值：

```go
v := varI.(T)       // unchecked type assertion
```

**varI 必须是一个接口变量**，否则编译器会报错：`invalid type assertion: varI.(T) (non-interface type (type of varI) on left)` 。

类型断言可能是无效的，虽然编译器会尽力检查转换是否有效，但是它不可能预见所有的可能性。如果转换在程序运行时失败会导致错误发生。更安全的方式是使用以下形式来进行类型断言：

```go
if v, ok := varI.(T); ok {  // checked type assertion
    Process(v)
    return
}
// varI is not of type T
```

如果转换合法，`v` 是 `varI` 转换到类型 `T` 的值，`ok` 会是 `true`；否则 `v` 是类型 `T` 的零值，`ok` 是 `false`，也没有运行时错误发生。

**应该总是使用上面的方式来进行类型断言**。

多数情况下，我们可能只是想在 `if` 中测试一下 `ok` 的值，此时使用以下的方法会是最方便的：

```go
if _, ok := varI.(T); ok {
    // ...
}
```

示例 11.4 [type_interfaces.go](examples/chapter_11/type_interfaces.go)：

```go
package main

import (
  "fmt"
  "math"
)

type Square struct {
  side float32
}

type Circle struct {
  radius float32
}

type Shaper interface {
  Area() float32
}

func main() {
  var areaIntf Shaper
  sq1 := new(Square)
  sq1.side = 5

  areaIntf = sq1
  // Is Square the type of areaIntf?
  if t, ok := areaIntf.(*Square); ok {
    fmt.Printf("The type of areaIntf is: %T\n", t)
  }
  if u, ok := areaIntf.(*Circle); ok {
    fmt.Printf("The type of areaIntf is: %T\n", u)
  } else {
    fmt.Println("areaIntf does not contain a variable of type Circle")
  }
}

func (sq *Square) Area() float32 {
  return sq.side * sq.side
}

func (ci *Circle) Area() float32 {
  return ci.radius * ci.radius * math.Pi
}
```

输出：

    The type of areaIntf is: *main.Square
    areaIntf does not contain a variable of type Circle

程序中定义了一个新类型 `Circle`，它也实现了 `Shaper` 接口。 `if t, ok := areaIntf.(*Square); ok ` 测试 `areaIntf` 里是否有一个包含 `*Square` 类型的变量，结果是确定的；然后我们测试它是否包含一个 `*Circle` 类型的变量，结果是否定的。

**备注**

如果忽略 `areaIntf.(*Square)` 中的 `*` 号，会导致编译错误：`impossible type assertion: Square does not implement Shaper (Area method has pointer receiver)`。

# 11.4 类型判断：type-switch

接口变量的类型也可以使用一种特殊形式的 `switch` 来检测：**type-switch** （下面是示例 11.4 的第二部分）：

```go
switch t := areaIntf.(type) {
case *Square:
  fmt.Printf("Type Square %T with value %v\n", t, t)
case *Circle:
  fmt.Printf("Type Circle %T with value %v\n", t, t)
case nil:
  fmt.Printf("nil value: nothing to check?\n")
default:
  fmt.Printf("Unexpected type %T\n", t)
}
```

输出：

    Type Square *main.Square with value &{5}

变量 `t` 得到了 `areaIntf` 的值和类型， 所有 `case` 语句中列举的类型（`nil` 除外）都必须实现对应的接口（在上例中即 `Shaper`），如果被检测类型没有在 `case` 语句列举的类型中，就会执行 `default` 语句。

可以用 `type-switch` 进行运行时类型分析，但是在 `type-switch` 不允许有 `fallthrough` 。

如果仅仅是测试变量的类型，不用它的值，那么就可以不需要赋值语句，比如：

```go
switch areaIntf.(type) {
case *Square:
  // TODO
case *Circle:
  // TODO
...
default:
  // TODO
}
```

下面的代码片段展示了一个类型分类函数，它有一个可变长度参数，可以是任意类型的数组，它会根据数组元素的实际类型执行不同的动作：

```go
func classifier(items ...interface{}) {
  for i, x := range items {
    switch x.(type) {
    case bool:
      fmt.Printf("Param #%d is a bool\n", i)
    case float64:
      fmt.Printf("Param #%d is a float64\n", i)
    case int, int64:
      fmt.Printf("Param #%d is a int\n", i)
    case nil:
      fmt.Printf("Param #%d is a nil\n", i)
    case string:
      fmt.Printf("Param #%d is a string\n", i)
    default:
      fmt.Printf("Param #%d is unknown\n", i)
    }
  }
}
```

可以这样调用此方法：`classifier(13, -14.3, "BELGIUM", complex(1, 2), nil, false)` 。

在处理来自于外部的、类型未知的数据时，比如解析诸如 JSON 或 XML 编码的数据，类型测试和转换会非常有用。

在示例 12.17（xml.go）中解析 XML 文档时，我们就会用到 `type-switch`。

# 11.5 测试一个值是否实现了某个接口

这是 11.3 类型断言中的一个特例：假定 `v` 是一个值，然后我们想测试它是否实现了 `Stringer` 接口，可以这样做：

```go
type Stringer interface {
    String() string
}

if sv, ok := v.(Stringer); ok {
    fmt.Printf("v implements String(): %s\n", sv.String()) // note: sv, not v
}
```

`Print` 函数就是如此检测类型是否可以打印自身的。

接口是一种契约，实现类型必须满足它，它描述了类型的行为，规定类型可以做什么。接口彻底将类型能做什么，以及如何做分离开来，使得相同接口的变量在不同的时刻表现出不同的行为，这就是多态的本质。

编写参数是接口变量的函数，这使得它们更具有一般性。

**使用接口使代码更具有普适性。**

标准库里到处都使用了这个原则，如果对接口概念没有良好的把握，是不可能理解它是如何构建的。

# 11.6 使用方法集与接口

在第 10.6.3 节及例子 methodset1.go 中我们看到，作用于变量上的方法实际上是不区分变量到底是指针还是值的。当碰到接口类型值时，这会变得有点复杂，原因是接口变量中存储的具体值是不可寻址的，幸运的是，如果使用不当编译器会给出错误。考虑下面的程序：

示例 11.5 [methodset2.go](examples/chapter_11/methodset2.go)：

```go
package main

import (
  "fmt"
)

type List []int

func (l List) Len() int {
  return len(l)
}

func (l *List) Append(val int) {
  *l = append(*l, val)
}

type Appender interface {
  Append(int)
}

func CountInto(a Appender, start, end int) {
  for i := start; i <= end; i++ {
    a.Append(i)
  }
}

type Lener interface {
  Len() int
}

func LongEnough(l Lener) bool {
  return l.Len()*10 > 42
}

func main() {
  // A bare value
  var lst List
  // compiler error:
  // cannot use lst (type List) as type Appender in argument to CountInto:
  //       List does not implement Appender (Append method has pointer receiver)
  // CountInto(lst, 1, 10)
  if LongEnough(lst) { // VALID:Identical receiver type
    fmt.Printf("- lst is long enough\n")
  }

  // A pointer value
  plst := new(List)
  CountInto(plst, 1, 10) //VALID:Identical receiver type
  if LongEnough(plst) {
    // VALID: a *List can be dereferenced for the receiver
    fmt.Printf("- plst is long enough\n")
  }
}
```

**讨论**

在 `lst` 上调用 `CountInto` 时会导致一个编译器错误，因为 `CountInto` 需要一个 `Appender`，而它的方法 `Append` 只定义在指针上。 在 `lst` 上调用 `LongEnough` 是可以的，因为 `Len` 定义在值上。

在 `plst` 上调用 `CountInto` 是可以的，因为 `CountInto` 需要一个 `Appender`，并且它的方法 `Append` 定义在指针上。 在 `plst` 上调用 `LongEnough` 也是可以的，因为指针会被自动解引用。

**总结**

在接口上调用方法时，必须有和方法定义时相同的接收者类型或者是可以从具体类型 `P` 直接可以辨识的：

- 指针方法可以通过指针调用
- 值方法可以通过值调用
- 接收者是值的方法可以通过指针调用，因为指针会首先被解引用
- 接收者是指针的方法不可以通过值调用，因为存储在接口中的值没有地址

将一个值赋值给一个接口时，编译器会确保所有可能的接口方法都可以在此值上被调用，因此不正确的赋值在编译期就会失败。

**译注**

Go 语言规范定义了接口方法集的调用规则：

- 类型 \*T 的可调用方法集包含接受者为 \*T 或 T 的所有方法集
- 类型 T 的可调用方法集包含接受者为 T 的所有方法
- 类型 T 的可调用方法集不包含接受者为 \*T 的方法

# 11.7 第一个例子：使用 Sorter 接口排序

一个很好的例子是来自标准库的 `sort` 包，要对一组数字或字符串排序，只需要实现三个方法：反映元素个数的 `Len()`方法、比较第 `i` 和 `j` 个元素的 `Less(i, j)` 方法以及交换第 `i` 和 `j` 个元素的 `Swap(i, j)` 方法。

排序函数的算法只会使用到这三个方法（可以使用任何排序算法来实现，此处我们使用冒泡排序）：

```go
func Sort(data Sorter) {
    for pass := 1; pass < data.Len(); pass++ {
        for i := 0;i < data.Len() - pass; i++ {
            if data.Less(i+1, i) {
                data.Swap(i, i + 1)
            }
        }
    }
}
```

`Sort` 函数接收一个接口类型的参数：`Sorter` ，它声明了这些方法：

```go
type Sorter interface {
    Len() int
    Less(i, j int) bool
    Swap(i, j int)
}
```

参数中的 `int` 是待排序序列长度的类型，而不是说要排序的对象一定要是一组 `int`。`i` 和 `j` 表示元素的整型索引，长度也是整型的。

现在如果我们想对一个 `int` 数组进行排序，所有必须做的事情就是：为数组定一个类型并在它上面实现 `Sorter` 接口的方法：

```go
type IntArray []int
func (p IntArray) Len() int           { return len(p) }
func (p IntArray) Less(i, j int) bool { return p[i] < p[j] }
func (p IntArray) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }
```

下面是调用排序函数的一个具体例子：

```go
data := []int{74, 59, 238, -784, 9845, 959, 905, 0, 0, 42, 7586, -5467984, 7586}
a := sort.IntArray(data) //conversion to type IntArray from package sort
sort.Sort(a)
```

完整的、可运行的代码可以在 `sort.go` 和 `sortmain.go` 里找到。

同样的原理，排序函数可以用于一个浮点型数组，一个字符串数组，或者一个表示每周各天的结构体 `dayArray`。

示例 11.6 [sort.go](examples/chapter_11/sort/sort.go)：

```go
package sort

type Sorter interface {
  Len() int
  Less(i, j int) bool
  Swap(i, j int)
}

func Sort(data Sorter) {
  for pass := 1; pass < data.Len(); pass++ {
    for i := 0; i < data.Len()-pass; i++ {
      if data.Less(i+1, i) {
        data.Swap(i, i+1)
      }
    }
  }
}

func IsSorted(data Sorter) bool {
  n := data.Len()
  for i := n - 1; i > 0; i-- {
    if data.Less(i, i-1) {
      return false
    }
  }
  return true
}

// Convenience types for common cases
type IntArray []int

func (p IntArray) Len() int           { return len(p) }
func (p IntArray) Less(i, j int) bool { return p[i] < p[j] }
func (p IntArray) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

type StringArray []string

func (p StringArray) Len() int           { return len(p) }
func (p StringArray) Less(i, j int) bool { return p[i] < p[j] }
func (p StringArray) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

// Convenience wrappers for common cases
func SortInts(a []int)       { Sort(IntArray(a)) }
func SortStrings(a []string) { Sort(StringArray(a)) }

func IntsAreSorted(a []int) bool       { return IsSorted(IntArray(a)) }
func StringsAreSorted(a []string) bool { return IsSorted(StringArray(a)) }
```

示例 11.7 [sortmain.go](examples/chapter_11/sortmain.go)：

```go
package main

import (
  "./sort"
  "fmt"
)

func ints() {
  data := []int{74, 59, 238, -784, 9845, 959, 905, 0, 0, 42, 7586, -5467984, 7586}
  a := sort.IntArray(data) //conversion to type IntArray
  sort.Sort(a)
  if !sort.IsSorted(a) {
    panic("fails")
  }
  fmt.Printf("The sorted array is: %v\n", a)
}

func strings() {
  data := []string{"monday", "friday", "tuesday", "wednesday", "sunday", "thursday", "", "saturday"}
  a := sort.StringArray(data)
  sort.Sort(a)
  if !sort.IsSorted(a) {
    panic("fail")
  }
  fmt.Printf("The sorted array is: %v\n", a)
}

type day struct {
  num       int
  shortName string
  longName  string
}

type dayArray struct {
  data []*day
}

func (p *dayArray) Len() int           { return len(p.data) }
func (p *dayArray) Less(i, j int) bool { return p.data[i].num < p.data[j].num }
func (p *dayArray) Swap(i, j int)      { p.data[i], p.data[j] = p.data[j], p.data[i] }

func days() {
  Sunday    := day{0, "SUN", "Sunday"}
  Monday    := day{1, "MON", "Monday"}
  Tuesday   := day{2, "TUE", "Tuesday"}
  Wednesday := day{3, "WED", "Wednesday"}
  Thursday  := day{4, "THU", "Thursday"}
  Friday    := day{5, "FRI", "Friday"}
  Saturday  := day{6, "SAT", "Saturday"}
  data := []*day{&Tuesday, &Thursday, &Wednesday, &Sunday, &Monday, &Friday, &Saturday}
  a := dayArray{data}
  sort.Sort(&a)
  if !sort.IsSorted(&a) {
    panic("fail")
  }
  for _, d := range data {
    fmt.Printf("%s ", d.longName)
  }
  fmt.Printf("\n")
}

func main() {
  ints()
  strings()
  days()
}
```

输出：

    The sorted array is: [-5467984 -784 0 0 42 59 74 238 905 959 7586 7586 9845]
    The sorted array is: [ friday monday saturday sunday thursday tuesday wednesday]
    Sunday Monday Tuesday Wednesday Thursday Friday Saturday 

**备注**：

`panic("fail")` 用于停止处于在非正常情况下的程序（详细请参考 第13章），当然也可以先打印一条信息，然后调用 `os.Exit(1)` 来停止程序。

上面的例子帮助我们进一步了解了接口的意义和使用方式。对于基本类型的排序，标准库已经提供了相关的排序函数，所以不需要我们再重复造轮子了。对于一般性的排序，`sort` 包定义了一个接口：

```go
type Interface interface {
  Len() int
  Less(i, j int) bool
  Swap(i, j int)
}
```

这个接口总结了需要用于排序的抽象方法，函数 `Sort(data Interface)` 用来对此类对象进行排序，可以用它们来实现对其他类型的数据（非基本类型）进行排序。在上面的例子中，我们也是这么做的，不仅可以对 `int` 和 `string` 序列进行排序，也可以对用户自定义类型 `dayArray` 进行排序。

# 11.8 第二个例子：读和写

读和写是软件中很普遍的行为，提起它们会立即想到读写文件、缓存（比如字节或字符串切片）、标准输入输出、标准错误以及网络连接、管道等等，或者读写我们的自定义类型。为了让代码尽可能通用，Go 采取了一致的方式来读写数据。

`io` 包提供了用于读和写的接口 `io.Reader` 和 `io.Writer`：

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

只要类型实现了读写接口，提供 `Read()` 和 `Write` 方法，就可以从它读取数据，或向它写入数据。一个对象要是可读的，它必须实现 `io.Reader` 接口，这个接口只有一个签名是 `Read(p []byte) (n int, err error)` 的方法，它从调用它的对象上读取数据，并把读到的数据放入参数中的字节切片中，然后返回读取的字节数和一个 `error` 对象，如果没有错误发生返回 `nil`，如果已经到达输入的尾端，会返回 `io.EOF("EOF")`，如果读取的过程中发生了错误，就会返回具体的错误信息。类似地，一个对象要是可写的，它必须实现 `io.Writer` 接口，这个接口也只有一个签名是 `Write(p []byte) (n int, err error)` 的方法，它将指定字节切片中的数据写入调用它的对象里，然后返回实际写入的字节数和一个 `error` 对象（如果没有错误发生就是 `nil`）。

`io` 包里的 `Readers` 和 `Writers` 都是不带缓冲的，`bufio` 包里提供了对应的带缓冲的操作，在读写 `UTF-8` 编码的文本文件时它们尤其有用。在 第12章 我们会看到很多在实战中使用它们的例子。

在实际编程中尽可能的使用这些接口，会使程序变得更通用，可以在任何实现了这些接口的类型上使用读写方法。

例如一个 `JPEG` 图形解码器，通过一个 `Reader` 参数，它可以解码来自磁盘、网络连接或以 `gzip` 压缩的 `HTTP` 流中的 `JPEG` 图形数据，或者其他任何实现了 `Reader` 接口的对象。 

# 11.9 空接口

## 11.9.1 概念

**空接口或者最小接口** 不包含任何方法，它对实现不做任何要求：

```go
type Any interface {}
```

任何其他类型都实现了空接口（它不仅仅像 `Java/C#` 中 `Object` 引用类型），`any` 或 `Any` 是空接口一个很好的别名或缩写。

空接口类似 `Java/C#` 中所有类的基类： `Object` 类，二者的目标也很相近。

可以给一个空接口类型的变量 `var val interface {}` 赋任何类型的值。

示例 11.8 [empty_interface.go](examples/chapter_11/empty_interface.go)：

```go
package main
import "fmt"

var i = 5
var str = "ABC"

type Person struct {
  name string
  age  int
}

type Any interface{}

func main() {
  var val Any
  val = 5
  fmt.Printf("val has the value: %v\n", val)
  val = str
  fmt.Printf("val has the value: %v\n", val)
  pers1 := new(Person)
  pers1.name = "Rob Pike"
  pers1.age = 55
  val = pers1
  fmt.Printf("val has the value: %v\n", val)
  switch t := val.(type) {
  case int:
    fmt.Printf("Type int %T\n", t)
  case string:
    fmt.Printf("Type string %T\n", t)
  case bool:
    fmt.Printf("Type boolean %T\n", t)
  case *Person:
    fmt.Printf("Type pointer to Person %T\n", t)
  default:
    fmt.Printf("Unexpected type %T", t)
  }
}
```

输出：

    val has the value: 5
    val has the value: ABC
    val has the value: &{Rob Pike 55}
    Type pointer to Person *main.Person

在上面的例子中，接口变量 `val` 被依次赋予一个 `int`，`string` 和 `Person` 实例的值，然后使用 `type-switch` 来测试它的实际类型。每个 `interface {}` 变量在内存中占据两个字长：一个用来存储它包含的类型，另一个用来存储它包含的数据或者指向数据的指针。

示例 [emptyint_switch.go](examples/chapter_11/emptyint_switch.go) 说明了空接口在 `type-switch` 中联合 `lambda` 函数的用法：

```go
package main

import "fmt"

type specialString string

var whatIsThis specialString = "hello"

func TypeSwitch() {
  testFunc := func(any interface{}) {
    switch v := any.(type) {
    case bool:
      fmt.Printf("any %v is a bool type", v)
    case int:
      fmt.Printf("any %v is an int type", v)
    case float32:
      fmt.Printf("any %v is a float32 type", v)
    case string:
      fmt.Printf("any %v is a string type", v)
    case specialString:
      fmt.Printf("any %v is a special String!", v)
    default:
      fmt.Println("unknown type!")
    }
  }
  testFunc(whatIsThis)
}

func main() {
  TypeSwitch()
}
```

输出：

    any hello is a special String!


## 11.9.2 构建通用类型或包含不同类型变量的数组

在 7.6.6 中我们看到了能被搜索和排序的 `int` 数组、`float` 数组以及 `string` 数组，那么对于其他类型的数组呢，是不是我们必须得自己编程实现它们？

现在我们知道该怎么做了，就是通过使用空接口。让我们给空接口定一个别名类型 `Element`：`type Element interface{}`

然后定义一个容器类型的结构体 `Vector`，它包含一个 `Element` 类型元素的切片：

```go
type Vector struct {
  a []Element
}
```

`Vector` 里能放任何类型的变量，因为任何类型都实现了空接口，实际上 `Vector` 里放的每个元素可以是不同类型的变量。我们为它定义一个 `At()` 方法用于返回第 `i` 个元素：

```go
func (p *Vector) At(i int) Element {
  return p.a[i]
}
```

再定一个 `Set()` 方法用于设置第 `i` 个元素的值：

```go
func (p *Vector) Set(i int, e Element) {
  p.a[i] = e
}
```

`Vector` 中存储的所有元素都是 `Element` 类型，要得到它们的原始类型（unboxing：拆箱）需要用到类型断言。TODO：The compiler rejects assertions guaranteed to fail，类型断言总是在运行时才执行，因此它会产生运行时错误。


## 11.9.3 复制数据切片至空接口切片

假设你有一个 `myType` 类型的数据切片，你想将切片中的数据复制到一个空接口切片中，类似：

```go
var dataSlice []myType = FuncReturnSlice()
var interfaceSlice []interface{} = dataSlice
```

可惜不能这么做，编译时会出错：`cannot use dataSlice (type []myType) as type []interface { } in assignment`。

原因是它们俩在内存中的布局是不一样的（参考 [官方说明](http://golang.org/doc/go_spec.html)）。

必须使用 `for-range` 语句来一个一个显式地复制：

```go
var dataSlice []myType = FuncReturnSlice()
var interfaceSlice []interface{} = make([]interface{}, len(dataSlice))
for i, d := range dataSlice {
    interfaceSlice[i] = d
}
```

## 11.9.4 通用类型的节点数据结构

在10.1中我们遇到了诸如列表和树这样的数据结构，在它们的定义中使用了一种叫节点的递归结构体类型，节点包含一个某种类型的数据字段。现在可以使用空接口作为数据字段的类型，这样我们就能写出通用的代码。下面是实现一个二叉树的部分代码：通用定义、用于创建空节点的 `NewNode` 方法，及设置数据的 `SetData` 方法。

示例 11.10 [node_structures.go](examples/chapter_11/node_structures.go)：

```go
package main

import "fmt"

type Node struct {
  le   *Node
  data interface{}
  ri   *Node
}

func NewNode(left, right *Node) *Node {
  return &Node{left, nil, right}
}

func (n *Node) SetData(data interface{}) {
  n.data = data
}

func main() {
  root := NewNode(nil, nil)
  root.SetData("root node")
  // make child (leaf) nodes:
  a := NewNode(nil, nil)
  a.SetData("left node")
  b := NewNode(nil, nil)
  b.SetData("right node")
  root.le = a
  root.ri = b
  fmt.Printf("%v\n", root) // Output: &{0x125275f0 root node 0x125275e0}
}
```

## 11.9.5 接口到接口

一个接口的值可以赋值给另一个接口变量，只要底层类型实现了必要的方法。这个转换是在运行时进行检查的，转换失败会导致一个运行时错误：这是 `Go` 语言动态的一面，可以拿它和 `Ruby` 和 `Python` 这些动态语言相比较。

假定：

```go
var ai AbsInterface // declares method Abs()
type SqrInterface interface {
    Sqr() float
}
var si SqrInterface
pp := new(Point) // say *Point implements Abs, Sqr
var empty interface{}
```

那么下面的语句和类型断言是合法的：

```go
empty = pp                // everything satisfies empty
ai = empty.(AbsInterface) // underlying value pp implements Abs()
// (runtime failure otherwise)
si = ai.(SqrInterface) // *Point has Sqr() even though AbsInterface doesn’t
empty = si             // *Point implements empty set
// Note: statically checkable so type assertion not necessary.
```

下面是函数调用的一个例子：

```go
type myPrintInterface interface {
  print()
}

func f3(x myInterface) {
  x.(myPrintInterface).print() // type assertion to myPrintInterface
}
```

`x` 转换为 `myPrintInterface` 类型是完全动态的：只要 `x` 的底层类型（动态类型）定义了 `print` 方法这个调用就可以正常运行。


# 11.10 反射包

## 11.10.1 方法和类型的反射

在 10.4 节我们看到可以通过反射来分析一个结构体。本节我们进一步探讨强大的反射功能。反射是用程序检查其所拥有的结构，尤其是类型的一种能力；这是元编程的一种形式。反射可以在运行时检查类型和变量，例如它的大小、方法和 `动态`
的调用这些方法。这对于没有源代码的包尤其有用。这是一个强大的工具，除非真得有必要，否则应当避免使用或小心使用。

变量的最基本信息就是类型和值：反射包的 `Type` 用来表示一个 Go 类型，反射包的 `Value` 为 Go 值提供了反射接口。

两个简单的函数，`reflect.TypeOf` 和 `reflect.ValueOf`，返回被检查对象的类型和值。例如，x 被定义为：`var x float64 = 3.4`，那么 `reflect.TypeOf(x)` 返回 `float64`，`reflect.ValueOf(x)` 返回 `<float64 Value>`

实际上，反射是通过检查一个接口的值，变量首先被转换成空接口。这从下面两个函数签名能够很明显的看出来：

```go
func TypeOf(i interface{}) Type
func ValueOf(i interface{}) Value
```

接口的值包含一个 type 和 value。

反射可以从接口值反射到对象，也可以从对象反射回接口值。

reflect.Type 和 reflect.Value 都有许多方法用于检查和操作它们。一个重要的例子是 Value 有一个 Type 方法返回 reflect.Value 的 Type。另一个是 Type 和 Value 都有 Kind 方法返回一个常量来表示类型：Uint、Float64、Slice 等等。同样 Value 有叫做 Int 和 Float 的方法可以获取存储在内部的值（跟 int64 和 float64 一样）

```go
const (
  Invalid Kind = iota
  Bool
  Int
  Int8
  Int16
  Int32
  Int64
  Uint
  Uint8
  Uint16
  Uint32
  Uint64
  Uintptr
  Float32
  Float64
  Complex64
  Complex128
  Array
  Chan
  Func
  Interface
  Map
  Ptr
  Slice
  String
  Struct
  UnsafePointer
)
```

对于 float64 类型的变量 x，如果 `v:=reflect.ValueOf(x)`，那么 `v.Kind()` 返回 `reflect.Float64` ，所以下面的表达式是 `true`
`v.Kind() == reflect.Float64`

Kind 总是返回底层类型：

```go
type MyInt int
var m MyInt = 5
v := reflect.ValueOf(m)
```

方法 `v.Kind()` 返回 `reflect.Int`。

变量 v 的 `Interface()` 方法可以得到还原（接口）值，所以可以这样打印 v 的值：`fmt.Println(v.Interface())`


尝试运行下面的代码：

示例 11.11 [reflect1.go](examples/chapter_11/reflect1.go)：

```go
// blog: Laws of Reflection
package main

import (
  "fmt"
  "reflect"
)

func main() {
  var x float64 = 3.4
  fmt.Println("type:", reflect.TypeOf(x))
  v := reflect.ValueOf(x)
  fmt.Println("value:", v)
  fmt.Println("type:", v.Type())
  fmt.Println("kind:", v.Kind())
  fmt.Println("value:", v.Float())
  fmt.Println(v.Interface())
  fmt.Printf("value is %5.2e\n", v.Interface())
  y := v.Interface().(float64)
  fmt.Println(y)
}
```

输出：

```
type: float64
value: 3.4
type: float64
kind: float64
value: 3.4
3.4
value is 3.40e+00
3.4
```

x 是一个 float64 类型的值，`reflect.ValueOf(x).Float()` 返回这个 float64 类型的实际值；同样的适用于 `Int(), Bool(), Complex(), String()`

## 11.10.2 通过反射修改(设置)值

继续前面的例子（参阅 11.9 [reflect2.go](examples/chapter_11/reflect2.go)），假设我们要把 x 的值改为 3.1415。Value 有一些方法可以完成这个任务，但是必须小心使用：`v.SetFloat(3.1415)`。

这将产生一个错误：`reflect.Value.SetFloat using unaddressable value`。

为什么会这样呢？问题的原因是 v 不是可设置的（这里并不是说值不可寻址）。是否可设置是 Value 的一个属性，并且不是所有的反射值都有这个属性：可以使用 `CanSet()` 方法测试是否可设置。

在例子中我们看到 `v.CanSet()` 返回 false： `settability of v: false`

当 `v := reflect.ValueOf(x)` 函数通过传递一个 x 拷贝创建了 v，那么 v 的改变并不能更改原始的 x。要想 v 的更改能作用到 x，那就必须传递 x 的地址 `v = reflect.ValueOf(&x)`。

通过 Type() 我们看到 v 现在的类型是 `*float64` 并且仍然是不可设置的。

要想让其可设置我们需要使用 `Elem()` 函数，这间接的使用指针：`v = v.Elem()`

现在 `v.CanSet()` 返回 true 并且 `v.SetFloat(3.1415)` 设置成功了！


示例 11.12 [reflect2.go](examples/chapter_11/reflect2.go)：

```go
package main

import (
  "fmt"
  "reflect"
)

func main() {
  var x float64 = 3.4
  v := reflect.ValueOf(x)
  // setting a value:
  // v.SetFloat(3.1415) // Error: will panic: reflect.Value.SetFloat using unaddressable value
  fmt.Println("settability of v:", v.CanSet())
  v = reflect.ValueOf(&x) // Note: take the address of x.
  fmt.Println("type of v:", v.Type())
  fmt.Println("settability of v:", v.CanSet())
  v = v.Elem()
  fmt.Println("The Elem of v is: ", v)
  fmt.Println("settability of v:", v.CanSet())
  v.SetFloat(3.1415) // this works!
  fmt.Println(v.Interface())
  fmt.Println(v)
}
```

输出：

```
settability of v: false
type of v: *float64
settability of v: false
The Elem of v is:  <float64 Value>
settability of v: true
3.1415
<float64 Value>
```

反射中有些内容是需要用地址去改变它的状态的。

## 11.10.3 反射结构

有些时候需要反射一个结构类型。`NumField()` 方法返回结构内的字段数量；通过一个 for 循环用索引取得每个字段的值 `Field(i)`。

我们同样能够调用签名在结构上的方法，例如，使用索引 n 来调用：`Method(n).Call(nil)`。

示例 11.13 [reflect_struct.go](examples/chapter_11/reflect_struct.go)：

```go
package main

import (
  "fmt"
  "reflect"
)

type NotknownType struct {
  s1, s2, s3 string
}

func (n NotknownType) String() string {
  return n.s1 + " - " + n.s2 + " - " + n.s3
}

// variable to investigate:
var secret interface{} = NotknownType{"Ada", "Go", "Oberon"}

func main() {
  value := reflect.ValueOf(secret) // <main.NotknownType Value>
  typ := reflect.TypeOf(secret)    // main.NotknownType
  // alternative:
  //typ := value.Type()  // main.NotknownType
  fmt.Println(typ)
  knd := value.Kind() // struct
  fmt.Println(knd)

  // iterate through the fields of the struct:
  for i := 0; i < value.NumField(); i++ {
    fmt.Printf("Field %d: %v\n", i, value.Field(i))
    // error: panic: reflect.Value.SetString using value obtained using unexported field
    //value.Field(i).SetString("C#")
  }

  // call the first method, which is String():
  results := value.Method(0).Call(nil)
  fmt.Println(results) // [Ada - Go - Oberon]
}
```

输出：

```
main.NotknownType
struct
Field 0: Ada
Field 1: Go
Field 2: Oberon
[Ada - Go - Oberon]
```

但是如果尝试更改一个值，会得到一个错误：

```
panic: reflect.Value.SetString using value obtained using unexported field
```

这是因为结构中只有被导出字段（首字母大写）才是可设置的；来看下面的例子：

示例 11.14 [reflect_struct2.go](examples/chapter_11/reflect_struct2.go)：

```go
package main

import (
  "fmt"
  "reflect"
)

type T struct {
  A int
  B string
}

func main() {
  t := T{23, "skidoo"}
  s := reflect.ValueOf(&t).Elem()
  typeOfT := s.Type()
  for i := 0; i < s.NumField(); i++ {
    f := s.Field(i)
    fmt.Printf("%d: %s %s = %v\n", i,
      typeOfT.Field(i).Name, f.Type(), f.Interface())
  }
  s.Field(0).SetInt(77)
  s.Field(1).SetString("Sunset Strip")
  fmt.Println("t is now", t)
}
```

输出：

```
0: A int = 23
1: B string = skidoo
t is now {77 Sunset Strip}
```
# 11.11 Printf 和反射

在 Go 语言的标准库中，前几节所述的反射的功能被大量地使用。举个例子，fmt 包中的 Printf（以及其他格式化输出函数）都会使用反射来分析它的 `...` 参数。

Printf 的函数声明为：

```go
func Printf(format string, args ... interface{}) (n int, err error)
```

Printf 中的 `...` 参数为空接口类型。Printf 使用反射包来解析这个参数列表。所以，Printf 能够知道它每个参数的类型。因此格式化字符串中只有%d而没有 %u 和 %ld，因为它知道这个参数是 unsigned 还是 long。这也是为什么 Print 和 Println 在没有格式字符串的情况下还能如此漂亮地输出。

为了让大家更加具体地了解 Printf 中的反射，我们实现了一个简单的通用输出函数。其中使用了 type-switch 来推导参数类型，并根据类型来输出每个参数的值

示例 11.15 [print.go](examples/chapter_11/print.go)：

```go
package main

import (
  "os"
  "strconv"
)

type Stringer interface {
  String() string
}

type Celsius float64

func (c Celsius) String() string {
  return strconv.FormatFloat(float64(c),'f', 1, 64) + " °C"
}

type Day int

var dayName = []string{"Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"}

func (day Day) String() string {
  return dayName[day]
}

func print(args ...interface{}) {
  for i, arg := range args {
    if i > 0 {os.Stdout.WriteString(" ")}
    switch a := arg.(type) { // type switch
      case Stringer:  os.Stdout.WriteString(a.String())
      case int:   os.Stdout.WriteString(strconv.Itoa(a))
      case string:  os.Stdout.WriteString(a)
      // more types
      default:    os.Stdout.WriteString("???")
    }
  }
}

func main() {
  print(Day(1), "was", Celsius(18.36))  // Tuesday was 18.4 °C
}
```

在 12.8 节中我们将阐释 `fmt.Fprintf()` 是怎么运用同样的反射原则的。

# 11.12 接口与动态类型

## 11.12.1 Go 的动态类型

在经典的面向对象语言（像 C++，Java 和 C#）中数据和方法被封装为 `类` 的概念：类包含它们两者，并且不能剥离。

Go 没有类：数据（结构体或更一般的类型）和方法是一种松耦合的正交关系。

Go 中的接口跟 Java/C# 类似：都是必须提供一个指定方法集的实现。但是更加灵活通用：任何提供了接口方法实现代码的类型都隐式地实现了该接口，而不用显式地声明。

和其它语言相比，Go 是唯一结合了接口值，静态类型检查（是否该类型实现了某个接口），运行时动态转换的语言，并且不需要显式地声明类型是否满足某个接口。该特性允许我们在不改变已有的代码的情况下定义和使用新接口。

接收一个（或多个）接口类型作为参数的函数，其实参可以是任何实现了该接口的类型。 `实现了某个接口的类型可以被传给任何以此接口为参数的函数` 。

类似于 Python 和 Ruby 这类动态语言中的 `动态类型（duck typing）`；这意味着对象可以根据提供的方法被处理（例如，作为参数传递给函数），而忽略它们的实际类型：它们能做什么比它们是什么更重要。

这在程序 duck_dance.go 中得以阐明，函数 DuckDance 接受一个 IDuck 接口类型变量。仅当 DuckDance 被实现了 IDuck 接口的类型调用时程序才能编译通过。

示例 11.16 [duck_dance.go](examples/chapter_11/duck_dance.go)：

```go
package main

import "fmt"

type IDuck interface {
  Quack()
  Walk()
}

func DuckDance(duck IDuck) {
  for i := 1; i <= 3; i++ {
    duck.Quack()
    duck.Walk()
  }
}

type Bird struct {
  // ...
}

func (b *Bird) Quack() {
  fmt.Println("I am quacking!")
}

func (b *Bird) Walk()  {
  fmt.Println("I am walking!")
}

func main() {
  b := new(Bird)
  DuckDance(b)
}
```

输出：

```
I am quacking!
I am walking!
I am quacking!
I am walking!
I am quacking!
I am walking!
```

如果 `Bird` 没有实现 `Walk()`（把它注释掉），会得到一个编译错误：

```
cannot use b (type *Bird) as type IDuck in function argument:
*Bird does not implement IDuck (missing Walk method)
```

如果对 `cat` 调用函数 `DuckDance()`，Go 会提示编译错误，但是 Python 和 Ruby 会以运行时错误结束。

## 11.12.2 动态方法调用

像 Python，Ruby 这类语言，动态类型是延迟绑定的（在运行时进行）：方法只是用参数和变量简单地调用，然后在运行时才解析（它们很可能有像 `responds_to` 这样的方法来检查对象是否可以响应某个方法，但是这也意味着更大的编码量和更多的测试工作）

Go 的实现与此相反，通常需要编译器静态检查的支持：当变量被赋值给一个接口类型的变量时，编译器会检查其是否实现了该接口的所有函数。如果方法调用作用于像 `interface{}` 这样的“泛型”上，你可以通过类型断言（参见 11.3 节）来检查变量是否实现了相应接口。

例如，你用不同的类型表示 XML 输出流中的不同实体。然后我们为 XML 定义一个如下的“写”接口（甚至可以把它定义为私有接口）：

```go
type xmlWriter interface {
  WriteXML(w io.Writer) error
}
```

现在我们可以实现适用于该流类型的任何变量的 `StreamXML` 函数，并用类型断言检查传入的变量是否实现了该接口；如果没有，我们就调用内建的 `encodeToXML` 来完成相应工作：

```go
// Exported XML streaming function.
func StreamXML(v interface{}, w io.Writer) error {
  if xw, ok := v.(xmlWriter); ok {
    // It’s an  xmlWriter, use method of asserted type.
    return xw.WriteXML(w)
  }
  // No implementation, so we have to use our own function (with perhaps reflection):
  return encodeToXML(v, w)
}

// Internal XML encoding function.
func encodeToXML(v interface{}, w io.Writer) error {
  // ...
}
```

Go 在这里用了和 `gob` 相同的机制：定义了两个接口 `GobEncoder` 和 `GobDecoder`。这样就允许类型自己实现从流编解码的具体方式；如果没有实现就使用标准的反射方式。

因此 Go 提供了动态语言的优点，却没有其他动态语言在运行时可能发生错误的缺点。

对于动态语言非常重要的单元测试来说，这样即可以减少单元测试的部分需求，又可以发挥相当大的作用。

Go 的接口提高了代码的分离度，改善了代码的复用性，使得代码开发过程中的设计模式更容易实现。用 Go 接口还能实现 `依赖注入模式`。

## 11.12.3 接口的提取

`提取接口` 是非常有用的设计模式，可以减少需要的类型和方法数量，而且不需要像传统的基于类的面向对象语言那样维护整个的类层次结构。

Go 接口可以让开发者找出自己写的程序中的类型。假设有一些拥有共同行为的对象，并且开发者想要抽象出这些行为，这时就可以创建一个接口来使用。
我们来扩展 11.1 节的示例 11.2 interfaces_poly.go，假设我们需要一个新的接口 `TopologicalGenus`，用来给 shape 排序（这里简单地实现为返回 int）。我们需要做的是给想要满足接口的类型实现 `Rank()` 方法：

示例 11.17 [multi_interfaces_poly.go](examples/chapter_11/multi_interfaces_poly.go)：

```go
//multi_interfaces_poly.go
package main

import "fmt"

type Shaper interface {
  Area() float32
}

type TopologicalGenus interface {
  Rank() int
}

type Square struct {
  side float32
}

func (sq *Square) Area() float32 {
  return sq.side * sq.side
}

func (sq *Square) Rank() int {
  return 1
}

type Rectangle struct {
  length, width float32
}

func (r Rectangle) Area() float32 {
  return r.length * r.width
}

func (r Rectangle) Rank() int {
  return 2
}

func main() {
  r := Rectangle{5, 3} // Area() of Rectangle needs a value
  q := &Square{5}      // Area() of Square needs a pointer
  shapes := []Shaper{r, q}
  fmt.Println("Looping through shapes for area ...")
  for n, _ := range shapes {
    fmt.Println("Shape details: ", shapes[n])
    fmt.Println("Area of this shape is: ", shapes[n].Area())
  }
  topgen := []TopologicalGenus{r, q}
  fmt.Println("Looping through topgen for rank ...")
  for n, _ := range topgen {
    fmt.Println("Shape details: ", topgen[n])
    fmt.Println("Topological Genus of this shape is: ", topgen[n].Rank())
  }
}
```

输出：

```
Looping through shapes for area ...
Shape details:  {5 3}
Area of this shape is:  15
Shape details:  &{5}
Area of this shape is:  25
Looping through topgen for rank ...
Shape details:  {5 3}
Topological Genus of this shape is:  2
Shape details:  &{5}
Topological Genus of this shape is:  1
```

所以你不用提前设计出所有的接口；`整个设计可以持续演进，而不用废弃之前的决定`。类型要实现某个接口，它本身不用改变，你只需要在这个类型上实现新的方法。

## 11.12.4 显式地指明类型实现了某个接口

如果你希望满足某个接口的类型显式地声明它们实现了这个接口，你可以向接口的方法集中添加一个具有描述性名字的方法。例如：

```go
type Fooer interface {
  Foo()
  ImplementsFooer()
}
```

类型 Bar 必须实现 `ImplementsFooer` 方法来满足 `Fooer` 接口，以清楚地记录这个事实。

```go
type Bar struct{}
func (b Bar) ImplementsFooer() {}
func (b Bar) Foo() {}
```

大部分代码并不使用这样的约束，因为它限制了接口的实用性。

但是有些时候，这样的约束在大量相似的接口中被用来解决歧义。

## 11.12.5 空接口和函数重载

在 6.1 节中, 我们看到函数重载是不被允许的。在 Go 语言中函数重载可以用可变参数 `...T` 作为函数最后一个参数来实现（参见 6.3 节）。如果我们把 T 换为空接口，那么可以知道任何类型的变量都是满足 T (空接口）类型的，这样就允许我们传递任何数量任何类型的参数给函数，即重载的实际含义。

函数 `fmt.Printf` 就是这样做的：

```go
fmt.Printf(format string, a ...interface{}) (n int, errno error)
```

这个函数通过枚举 `slice` 类型的实参动态确定所有参数的类型。并查看每个类型是否实现了 `String()` 方法，如果是就用于产生输出信息。我们可以回到 11.10 节查看这些细节。

## 11.12.6 接口的继承

当一个类型包含（内嵌）另一个类型（实现了一个或多个接口）的指针时，这个类型就可以使用（另一个类型）所有的接口方法。

例如：

```go
type Task struct {
  Command string
  *log.Logger
}
```

这个类型的工厂方法像这样：

```go
func NewTask(command string, logger *log.Logger) *Task {
  return &Task{command, logger}
}
```

当 `log.Logger` 实现了 `Log()` 方法后，Task 的实例 task 就可以调用该方法：

```go
task.Log()
```

类型可以通过继承多个接口来提供像 `多重继承` 一样的特性：

```go
type ReaderWriter struct {
  *io.Reader
  *io.Writer
}
```

上面概述的原理被应用于整个 Go 包，多态用得越多，代码就相对越少（参见 12.8 节）。这被认为是 Go 编程中的重要的最佳实践。

有用的接口可以在开发的过程中被归纳出来。添加新接口非常容易，因为已有的类型不用变动（仅仅需要实现新接口的方法）。已有的函数可以扩展为使用接口类型的约束性参数：通常只有函数签名需要改变。对比基于类的 OO 类型的语言在这种情况下则需要适应整个类层次结构的变化。

我们开发了一些栈结构类型。但是它们被限制为某种固定的内建类型。现在用一个元素类型是 interface{}（空接口）的切片开发一个通用的栈类型。

实现下面的栈方法：

```go
Len() int
IsEmpty() bool
Push(x interface{})
Pop() (x interface{}, error)
```

`Pop()` 改变栈并返回最顶部的元素；`Top()` 只返回最顶部元素。

在主程序中构建一个充满不同类型元素的栈，然后弹出并打印所有元素的值。

# 总结：Go 中的面向对象

我们总结一下前面看到的：Go 没有类，而是松耦合的类型、方法对接口的实现。

OO 语言最重要的三个方面分别是：封装，继承和多态，在 Go 中它们是怎样表现的呢？

- 封装（数据隐藏）：和别的 OO 语言有 4 个或更多的访问层次相比，Go 把它简化为了 2 层（参见 4.2 节的可见性规则）:

  1）包范围内的：通过标识符首字母小写，`对象` 只在它所在的包内可见

  2）可导出的：通过标识符首字母大写，`对象` 对所在包以外也可见

类型只拥有自己所在包中定义的方法。

- 继承：用组合实现：内嵌一个（或多个）包含想要的行为（字段和方法）的类型；多重继承可以通过内嵌多个类型实现
- 多态：用接口实现：某个类型的实例可以赋给它所实现的任意接口类型的变量。类型和接口是松耦合的，并且多重继承可以通过实现多个接口实现。Go 接口不是 Java 和 C# 接口的变体，而且接口间是不相关的，并且是大规模编程和可适应的演进型设计的关键。

# 11.14 结构体、集合和高阶函数

通常你在应用中定义了一个结构体，那么你也可能需要这个结构体的（指针）对象集合，比如：

```go
type Any interface{}
type Car struct {
  Model        string
  Manufacturer string
  BuildYear    int
  // ...
}

type Cars []*Car
```

在定义所需功能时我们可以利用函数可以作为（其它函数的）参数的事实来使用高阶函数，例如：

1）定义一个通用的 `Process()` 函数，它接收一个作用于每一辆 car 的 f 函数作参数：

```go
// Process all cars with the given function f:
func (cs Cars) Process(f func(car *Car)) {
  for _, c := range cs {
    f(c)
  }
}
```

2）在上面的基础上，实现一个查找函数来获取子集合，并在 `Process()` 中传入一个闭包执行（这样就可以访问局部切片 `cars`）：

```go
// Find all cars matching a given criteria.
func (cs Cars) FindAll(f func(car *Car) bool) Cars {

  cars := make([]*Car, 0)
  cs.Process(func(c *Car) {
    if f(c) {
      cars = append(cars, c)
    }
  })
  return cars
}
```

3）实现 Map 功能，产出除 car 对象以外的东西：

```go
// Process cars and create new data.
func (cs Cars) Map(f func(car *Car) Any) []Any {
  result := make([]Any, 0)
  ix := 0
  cs.Process(func(c *Car) {
    result[ix] = f(c)
    ix++
  })
  return result
}
```

现在我们可以定义下面这样的具体查询：

```go
allNewBMWs := allCars.FindAll(func(car *Car) bool {
  return (car.Manufacturer == "BMW") && (car.BuildYear > 2010)
})
```

4）我们也可以根据入参返回不同的函数。也许我们想根据不同的厂商添加汽车到不同的集合，但是这可能会是多变的。所以我们可以定义一个函数来产生特定的添加函数和 map 集：

```go
func MakeSortedAppender(manufacturers[]string)(func(car*Car),map[string]Cars) {
  // Prepare maps of sorted cars.
  sortedCars := make(map[string]Cars)
  for _, m := range manufacturers {
    sortedCars[m] = make([]*Car, 0)
  }
  sortedCars["Default"] = make([]*Car, 0)
  // Prepare appender function:
  appender := func(c *Car) {
    if _, ok := sortedCars[c.Manufacturer]; ok {
      sortedCars[c.Manufacturer] = append(sortedCars[c.Manufacturer], c)
    } else {
      sortedCars["Default"] = append(sortedCars["Default"], c)
    }

  }
  return appender, sortedCars
}
```

现在我们可以用它把汽车分类为独立的集合，像这样：

```go
manufacturers := []string{"Ford", "Aston Martin", "Land Rover", "BMW", "Jaguar"}
sortedAppender, sortedCars := MakeSortedAppender(manufacturers)
allUnsortedCars.Process(sortedAppender)
BMWCount := len(sortedCars["BMW"])
```

我们让这些代码在下面的程序 cars.go 中执行：

示例 11.18 [cars.go](examples/chapter_11/cars.go)：

```go
// cars.go
package main

import (
  "fmt"
)

type Any interface{}
type Car struct {
  Model        string
  Manufacturer string
  BuildYear    int
  // ...
}
type Cars []*Car

func main() {
  // make some cars:
  ford := &Car{"Fiesta", "Ford", 2008}
  bmw := &Car{"XL 450", "BMW", 2011}
  merc := &Car{"D600", "Mercedes", 2009}
  bmw2 := &Car{"X 800", "BMW", 2008}
  // query:
  allCars := Cars([]*Car{ford, bmw, merc, bmw2})
  allNewBMWs := allCars.FindAll(func(car *Car) bool {
    return (car.Manufacturer == "BMW") && (car.BuildYear > 2010)
  })
  fmt.Println("AllCars: ", allCars)
  fmt.Println("New BMWs: ", allNewBMWs)
  //
  manufacturers := []string{"Ford", "Aston Martin", "Land Rover", "BMW", "Jaguar"}
  sortedAppender, sortedCars := MakeSortedAppender(manufacturers)
  allCars.Process(sortedAppender)
  fmt.Println("Map sortedCars: ", sortedCars)
  BMWCount := len(sortedCars["BMW"])
  fmt.Println("We have ", BMWCount, " BMWs")
}

// Process all cars with the given function f:
func (cs Cars) Process(f func(car *Car)) {
  for _, c := range cs {
    f(c)
  }
}

// Find all cars matching a given criteria.
func (cs Cars) FindAll(f func(car *Car) bool) Cars {
  cars := make([]*Car, 0)

  cs.Process(func(c *Car) {
    if f(c) {
      cars = append(cars, c)
    }
  })
  return cars
}

// Process cars and create new data.
func (cs Cars) Map(f func(car *Car) Any) []Any {
  result := make([]Any, len(cs))
  ix := 0
  cs.Process(func(c *Car) {
    result[ix] = f(c)
    ix++
  })
  return result
}

func MakeSortedAppender(manufacturers []string) (func(car *Car), map[string]Cars) {
  // Prepare maps of sorted cars.
  sortedCars := make(map[string]Cars)

  for _, m := range manufacturers {
    sortedCars[m] = make([]*Car, 0)
  }
  sortedCars["Default"] = make([]*Car, 0)

  // Prepare appender function:
  appender := func(c *Car) {
    if _, ok := sortedCars[c.Manufacturer]; ok {
      sortedCars[c.Manufacturer] = append(sortedCars[c.Manufacturer], c)
    } else {
      sortedCars["Default"] = append(sortedCars["Default"], c)
    }
  }
  return appender, sortedCars
}
```

输出：

```
AllCars:  [0xf8400038a0 0xf840003bd0 0xf840003ba0 0xf840003b70]
New BMWs:  [0xf840003bd0]
Map sortedCars:  map[Default:[0xf840003ba0] Jaguar:[] Land Rover:[] BMW:[0xf840003bd0 0xf840003b70] Aston Martin:[] Ford:[0xf8400038a0]]
We have  2  BMWs
```

