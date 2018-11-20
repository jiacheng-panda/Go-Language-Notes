# Notes Part 2: Conditional Statements, Loop and Function

# 5.1 if-else 结构

if 是用于测试某个条件（布尔型或逻辑型）的语句，如果该条件成立，则会执行 if 后由大括号括起来的代码块，否则就忽略该代码块继续执行后续的代码。

如果存在第二个分支，则可以在上面代码的基础上添加 else 关键字以及另一代码块，这个代码块中的代码只有在条件不满足时才会执行。if 和 else 后的两个代码块是相互独立的分支，只可能执行其中一个。

如果存在第三个分支，则可以使用下面这种三个独立分支的形式：

```go
if condition1 {
  // do something 
} else if condition2 {
  // do something else  
} else {
  // catch-all or default
}
```

else-if 分支的数量是没有限制的，但是为了代码的可读性，还是不要在 if 后面加入太多的 else-if 结构。如果你必须使用这种形式，则把尽可能先满足的条件放在前面。

即使当代码块之间只有一条语句时，大括号也不可被省略(尽管有些人并不赞成，但这还是符合了软件工程原则的主流做法)。

关键字 if 和 else 之后的左大括号 `{` 必须和关键字在同一行，如果你使用了 else-if 结构，则前段代码块的右大括号 `}` 必须和 else-if 关键字在同一行。这两条规则都是被编译器强制规定的。

要注意的是，在你使用 `gofmt` 格式化代码之后，每个分支内的代码都会缩进 4 个或 8 个空格，或者是 1 个 tab，并且右大括号与对应的 if 关键字垂直对齐。

在有些情况下，条件语句两侧的括号是可以被省略的；当条件比较复杂时，则可以使用括号让代码更易读。条件允许是符合条件，需使用 &&、|| 或 !，你可以使用括号来提升某个表达式的运算优先级，并提高代码的可读性。

当 if 结构内有 break、continue、goto 或者 return 语句时，Go 代码的常见写法是省略 else 部分（另见第 5.2 节）。无论满足哪个条件都会返回 x 或者 y 时，一般使用以下写法：

```go
if condition {
  return x
}
return y
```

这里举一些有用的例子：

1. 判断一个字符串是否为空：
  - `if str == "" { ... }`
  - `if len(str) == 0 {...}`  
2. 判断运行 Go 程序的操作系统类型，这可以通过常量 `runtime.GOOS` 来判断(第 2.2 节)。
  
    if runtime.GOOS == "windows"   {
      . ..
    } else { // Unix-like
      . ..
    }

  这段代码一般被放在 init() 函数中执行。这儿还有一段示例来演示如何根据操作系统来决定输入结束的提示：

    var prompt = "Enter a digit, e.g. 3 "+ "or %s to quit."
    
    func init() {
      if runtime.GOOS == "windows" {
        prompt = fmt.Sprintf(prompt, "Ctrl+Z, Enter")   
      } else { //Unix-like
        prompt = fmt.Sprintf(prompt, "Ctrl+D")
      }
    }

3. 函数 `Abs()` 用于返回一个整型数字的绝对值:

    func Abs(x int) int {
      if x < 0 {
        return -x
      }
      return x  
    }

4. `isGreater` 用于比较两个整型数字的大小:

    func isGreater(x, y int) bool {
      if x > y {
        return true 
      }
      return false
    }

在第四种情况中，if 可以包含一个初始化语句（如：给一个变量赋值）。这种写法具有固定的格式（在初始化语句后方必须加上分号）：

```go
if initialization; condition {
  // do something
}
```

例如:

```go
val := 10
if val > max {
  // do something
}
```

你也可以这样写:

```go
if val := 10; val > max {
  // do something
}
```

但要注意的是，使用简短方式 `:=` 声明的变量的作用域只存在于 if 结构中（在 if 结构的大括号之间，如果使用 if-else 结构则在 else 代码块中变量也会存在）。如果变量在 if 结构之前就已经存在，那么在 if 结构中，该变量原来的值会被隐藏。最简单的解决方案就是不要在初始化语句中声明变量（见 5.2 节的例 3 了解更多)。

示例 5.2 [ifelse.go](examples/chapter_5/ifelse.go)

```go
package main

import "fmt"

func main() {
  var first int = 10
  var cond int

  if first <= 0 {

    fmt.Printf("first is less than or equal to 0\n")
  } else if first > 0 && first < 5 {

    fmt.Printf("first is between 0 and 5\n")
  } else {

    fmt.Printf("first is 5 or greater\n")
  }
  if cond = 5; cond > 10 {

    fmt.Printf("cond is greater than 10\n")
  } else {

    fmt.Printf("cond is not greater than 10\n")
  }
}
```

输出：

  first is 5 or greater cond is not greater than 10

下面的代码片段展示了如何通过在初始化语句中获取函数 `process()` 的返回值，并在条件语句中作为判定条件来决定是否执行 if 结构中的代码：

```go
if value := process(data); value > max {
  ...
}
```

# 5.2 测试多返回值函数的错误

Go 语言的函数经常使用两个返回值来表示执行是否成功：返回某个值以及 true 表示成功；返回零值（或 nil）和 false 表示失败（第 4.4 节）。当不使用 true 或 false 的时候，也可以使用一个 error 类型的变量来代替作为第二个返回值：成功执行的话，error 的值为 nil，否则就会包含相应的错误信息（Go 语言中的错误类型为 error: `var err error`，我们将会在第 13 章进行更多地讨论）。这样一来，就很明显需要用一个 if 语句来测试执行结果；由于其符号的原因，这样的形式又称之为 comma,ok 模式（pattern）。

在第 4.7 节的程序 `string_conversion.go` 中，函数 `strconv.Atoi` 的作用是将一个字符串转换为一个整数。之前我们忽略了相关的错误检查：

```go
anInt, _ = strconv.Atoi(origStr)
```

如果 origStr 不能被转换为整数，anInt 的值会变成 0 而 `_` 无视了错误，程序会继续运行。

这样做是非常不好的：程序应该在最接近的位置检查所有相关的错误，至少需要暗示用户有错误发生并对函数进行返回，甚至中断程序。

我们在第二个版本中对代码进行了改进：


示例 1：

示例 5.3 [string_conversion2.go](examples/chapter_5/string_conversion2.go)

```go
package main

import (
  "fmt"
  "strconv"
)

func main() {
  var orig string = "ABC"
  // var an int
  var newS string
  // var err error

  fmt.Printf("The size of ints is: %d\n", strconv.IntSize)    
  // anInt, err = strconv.Atoi(origStr)
  an, err := strconv.Atoi(orig)
  if err != nil {
    fmt.Printf("orig %s is not an integer - exiting with error\n", orig)
    return
  } 
  fmt.Printf("The integer is %d\n", an)
  an = an + 5
  newS = strconv.Itoa(an)
  fmt.Printf("The new string is: %s\n", newS)
}
```

这是测试 err 变量是否包含一个真正的错误（`if err != nil`）的习惯用法。如果确实存在错误，则会打印相应的错误信息然后通过 return 提前结束函数的执行。我们还可以使用携带返回值的 return 形式，例如 `return err`。这样一来，函数的调用者就可以检查函数执行过程中是否存在错误了。

**习惯用法**

```go
value, err := pack1.Function1(param1)
if err != nil {
  fmt.Printf("An error occured in pack1.Function1 with parameter %v", param1)
  return err
}
// 未发生错误，继续执行：
```

由于本例的函数调用者属于 main 函数，所以程序会直接停止运行。

如果我们想要在错误发生的同时终止程序的运行，我们可以使用 `os` 包的 `Exit` 函数：

**习惯用法**

```go
if err != nil {
  fmt.Printf("Program stopping with error %v", err)
  os.Exit(1)
}
```

（此处的退出代码 1 可以使用外部脚本获取到）

有时候，你会发现这种习惯用法被连续重复地使用在某段代码中。

当没有错误发生时，代码继续运行就是唯一要做的事情，所以 if 语句块后面不需要使用 else 分支。

示例 2：我们尝试通过 `os.Open` 方法打开一个名为 `name` 的只读文件：

```go
f, err := os.Open(name)
if err != nil {
  return err
}
doSomething(f) // 当没有错误发生时，文件对象被传入到某个函数中
doSomething
```

示例 3：可以将错误的获取放置在 if 语句的初始化部分：

**习惯用法**

```go
if err := file.Chmod(0664); err != nil {
  fmt.Println(err)
  return err
}
```

示例 4：或者将 ok-pattern 的获取放置在 if 语句的初始化部分，然后进行判断：

**习惯用法**

```go
if value, ok := readData(); ok {
…
}
```

**注意事项**

如果您像下面一样，没有为多返回值的函数准备足够的变量来存放结果：
  
```go
func mySqrt(f float64) (v float64, ok bool) {
  if f < 0 { return } // error case
  return math.Sqrt(f),true
}

func main() {
  t := mySqrt(25.0)
  fmt.Println(t)
}
```

您会得到一个编译错误：`multiple-value mySqrt() in single-value context`。

正确的做法是：

```go
t, ok := mySqrt(25.0)
if ok { fmt.Println(t) }
```

**注意事项 2**

当您将字符串转换为整数时，且确定转换一定能够成功时，可以将 `Atoi` 函数进行一层忽略错误的封装：

```go
func atoi (s string) (n int) {
  n, _ = strconv.Atoi(s)
  return
}
```

实际上，`fmt` 包（第 4.4.3 节）最简单的打印函数也有 2 个返回值：

```go
count, err := fmt.Println(x) // number of bytes printed, nil or 0, error
```

当打印到控制台时，可以将该函数返回的错误忽略；但当输出到文件流、网络流等具有不确定因素的输出对象时，应该始终检查是否有错误发生。


# 5.3 switch 结构

相比较 C 和 Java 等其它语言而言，Go 语言中的 switch 结构使用上更加灵活。它接受任意形式的表达式：

```go
switch var1 {
  case val1:
    ...
  case val2:
    ...
  default:
    ...
}
```

变量 var1 可以是任何类型，而 val1 和 val2 则可以是同类型的任意值。类型不被局限于常量或整数，但必须是相同的类型；或者最终结果为相同类型的表达式。前花括号 `{` 必须和 switch 关键字在同一行。

您可以同时测试多个可能符合条件的值，使用逗号分割它们，例如：`case val1, val2, val3`。

每一个 `case` 分支都是唯一的，从上至下逐一测试，直到匹配为止。（ Go 语言使用快速的查找算法来测试 switch 条件与 case 分支的匹配情况，直到算法匹配到某个 case 或者进入 default 条件为止。）

一旦成功地匹配到某个分支，在执行完相应代码后就会退出整个 switch 代码块，也就是说您不需要特别使用 `break` 语句来表示结束。

因此，程序也不会自动地去执行下一个分支的代码。如果在执行完每个分支的代码后，还希望继续执行后续分支的代码，可以使用 `fallthrough` 关键字来达到目的。

因此：

```go
switch i {
  case 0: // 空分支，只有当 i == 0 时才会进入分支
  case 1:
    f() // 当 i == 0 时函数不会被调用
}
```

并且：

```go
switch i {
  case 0: fallthrough
  case 1:
    f() // 当 i == 0 时函数也会被调用
}
```

在 `case ...:` 语句之后，您不需要使用花括号将多行语句括起来，但您可以在分支中进行任意形式的编码。当代码块只有一行时，可以直接放置在 `case` 语句之后。

您同样可以使用 `return` 语句来提前结束代码块的执行。当您在 switch 语句块中使用 `return` 语句，并且您的函数是有返回值的，您还需要在 switch 之后添加相应的 `return` 语句以确保函数始终会返回。

可选的 `default` 分支可以出现在任何顺序，但最好将它放在最后。它的作用类似与 `if-else` 语句中的 `else`，表示不符合任何已给出条件时，执行相关语句。

示例 5.4 [switch1.go](examples/chapter_5/switch1.go)：

```go
package main

import "fmt"

func main() {
  var num1 int = 100

  switch num1 {
  case 98, 99:
    fmt.Println("It's equal to 98")
  case 100: 
    fmt.Println("It's equal to 100")
  default:
    fmt.Println("It's not equal to 98 or 100")
  }
}

```

输出：

  It's equal to 100

在第 12.1 节，我们会使用 switch 语句判断从键盘输入的字符（详见第 12.2 节的 switch.go）。switch 语句的第二种形式是不提供任何被判断的值（实际上默认为判断是否为 true），然后在每个 case 分支中进行测试不同的条件。当任一分支的测试结果为 true 时，该分支的代码会被执行。这看起来非常像链式的 `if-else` 语句，但是在测试条件非常多的情况下，提供了可读性更好的书写方式。

```go
switch {
  case condition1:
    ...
  case condition2:
    ...
  default:
    ...
}
```

例如：

```go
switch {
  case i < 0:
    f1()
  case i == 0:
    f2()
  case i > 0:
    f3()
}
```

任何支持进行相等判断的类型都可以作为测试表达式的条件，包括 int、string、指针等。

示例 5.4 [switch2.go](examples/chapter_5/switch2.go)：

```go
package main

import "fmt"

func main() {
  var num1 int = 7

  switch {
      case num1 < 0:
        fmt.Println("Number is negative")
      case num1 > 0 && num1 < 10:
        fmt.Println("Number is between 0 and 10")
      default:
        fmt.Println("Number is 10 or greater")
  }
}
```

输出：

  Number is between 0 and 10

switch 语句的第三种形式是包含一个初始化语句：

```go
switch initialization {
  case val1:
    ...
  case val2:
    ...
  default:
    ...
}
```

这种形式可以非常优雅地进行条件判断：

```go
switch result := calculate(); {
  case result < 0:
    ...
  case result > 0:
    ...
  default:
    // 0
}
```

在下面这个代码片段中，变量 a 和 b 被平行初始化，然后作为判断条件：

```go
switch a, b := x[i], y[j]; {
  case a < b: t = -1
  case a == b: t = 0
  case a > b: t = 1
}
```

switch 语句还可以被用于 type-switch（详见第 11.4 节）来判断某个 interface 变量中实际存储的变量类型。

# 5.4 for 结构

如果想要重复执行某些语句，Go 语言中您只有 for 结构可以使用。不要小看它，这个 for 结构比其它语言中的更为灵活。

**注意事项** 其它许多语言中也没有发现和 do while 完全对等的 for 结构，可能是因为这种需求并不是那么强烈。

## 5.4.1 基于计数器的迭代

文件 for1.go 中演示了最简单的基于计数器的迭代，基本形式为：

  for 初始化语句; 条件语句; 修饰语句 {}

示例 5.6 [for1.go](examples/chapter_5/for1.go)：

```go
package main

import "fmt"

func main() {
  for i := 0; i < 5; i++ {
    fmt.Printf("This is the %d iteration\n", i)
  }
}
```

输出：

  This is the 0 iteration
  This is the 1 iteration
  This is the 2 iteration
  This is the 3 iteration
  This is the 4 iteration

由花括号括起来的代码块会被重复执行已知次数，该次数是根据计数器（此例为 i）决定的。循环开始前，会执行且仅会执行一次初始化语句 `i := 0;`；这比在循环之前声明更为简短。紧接着的是条件语句 `i < 5;`，在每次循环开始前都会进行判断，一旦判断结果为 false，则退出循环体。最后一部分为修饰语句 `i++`，一般用于增加或减少计数器。

这三部分组成的循环的头部，它们之间使用分号 `;` 相隔，但并不需要括号 `()` 将它们括起来。例如：`for (i = 0; i < 10; i++) { }`，这是无效的代码！

同样的，左花括号 `{` 必须和 for 语句在同一行，计数器的生命周期在遇到右花括号 `}` 时便终止。一般习惯使用 i、j、z 或 ix 等较短的名称命名计数器。

特别注意，永远不要在循环体内修改计数器，这在任何语言中都是非常差的实践！

您还可以在循环中同时使用多个计数器：

```go
for i, j := 0, N; i < j; i, j = i+1, j-1 {}
```

这得益于 Go 语言具有的平行赋值的特性（可以查看第 7 章 string_reverse.go 中反转数组的示例）。

您可以将两个 for 循环嵌套起来：

```go
for i:=0; i<5; i++ {
  for j:=0; j<10; j++ {
    println(j)
  }
}
```

如果您使用 for 循环迭代一个 Unicode 编码的字符串，会发生什么？

示例 5.7 [for_string.go](examples/chapter_5/for_string.go)：

```go
package main

import "fmt"

func main() {
  str := "Go is a beautiful language!"
  fmt.Printf("The length of str is: %d\n", len(str))
  for ix :=0; ix < len(str); ix++ {
    fmt.Printf("Character on position %d is: %c \n", ix, str[ix])
  }
  str2 := "日本語"
  fmt.Printf("The length of str2 is: %d\n", len(str2))
  for ix :=0; ix < len(str2); ix++ {
    fmt.Printf("Character on position %d is: %c \n", ix, str2[ix])
  }
}
```

输出：

  The length of str is: 27
  Character on position 0 is: G 
  Character on position 1 is: o 
  Character on position 2 is:   
  Character on position 3 is: i 
  Character on position 4 is: s 
  Character on position 5 is:   
  Character on position 6 is: a 
  Character on position 7 is:   
  Character on position 8 is: b 
  Character on position 9 is: e 
  Character on position 10 is: a 
  Character on position 11 is: u 
  Character on position 12 is: t 
  Character on position 13 is: i 
  Character on position 14 is: f 
  Character on position 15 is: u 
  Character on position 16 is: l 
  Character on position 17 is:   
  Character on position 18 is: l 
  Character on position 19 is: a 
  Character on position 20 is: n 
  Character on position 21 is: g 
  Character on position 22 is: u 
  Character on position 23 is: a 
  Character on position 24 is: g 
  Character on position 25 is: e 
  Character on position 26 is: ! 
  The length of str2 is: 9
  Character on position 0 is: æ 
  Character on position 1 is:  
  Character on position 2 is: ¥ 
  Character on position 3 is: æ 
  Character on position 4 is:  
  Character on position 5 is: ¬ 
  Character on position 6 is: è 
  Character on position 7 is: ª 
  Character on position 8 is:  

如果我们打印 str 和 str2 的长度，会分别得到 27 和 9。

由此我们可以发现，ASCII 编码的字符占用 1 个字节，既每个索引都指向不同的字符，而非 ASCII 编码的字符（占有 2 到 4 个字节）不能单纯地使用索引来判断是否为同一个字符。我们会在第 5.4.4 节解决这个问题。


## 5.4.2 基于条件判断的迭代

for 结构的第二种形式是没有头部的条件判断迭代（类似其它语言中的 while 循环），基本形式为：`for 条件语句 {}`。

您也可以认为这是没有初始化语句和修饰语句的 for 结构，因此 `;;` 便是多余的了。

Listing 5.8 [for2.go](examples/chapter_5/for2.go)：

```go
package main

import "fmt"

func main() {
  var i int = 5

  for i >= 0 {
    i = i - 1
    fmt.Printf("The variable i is now: %d\n", i)
  }
}
```

输出：

  The variable i is now: 4
  The variable i is now: 3
  The variable i is now: 2
  The variable i is now: 1
  The variable i is now: 0
  The variable i is now: -1

## 5.4.3 无限循环

条件语句是可以被省略的，如 `i:=0; ; i++` 或 `for { }` 或 `for ;; { }`（`;;` 会在使用 gofmt 时被移除）：这些循环的本质就是无限循环。最后一个形式也可以被改写为 `for true { }`，但一般情况下都会直接写 `for { }`。

如果 for 循环的头部没有条件语句，那么就会认为条件永远为 true，因此循环体内必须有相关的条件判断以确保会在某个时刻退出循环。

想要直接退出循环体，可以使用 break 语句（第 5.5 节）或 return 语句直接返回（第 6.1 节）。

但这两者之间有所区别，break 只是退出当前的循环体，而 return 语句提前对函数进行返回，不会执行后续的代码。

无限循环的经典应用是服务器，用于不断等待和接受新的请求。

```go
for t, err = p.Token(); err == nil; t, err = p.Token() {
  ...
}
```

## 5.4.4 for-range 结构

这是 Go 特有的一种的迭代结构，您会发现它在许多情况下都非常有用。它可以迭代任何一个集合（包括数组和 map，详见第 7 和 8 章）。语法上很类似其它语言中 foreach 语句，但您依旧可以获得每次迭代所对应的索引。一般形式为：`for ix, val := range coll { }`。

要注意的是，`val` 始终为集合中对应索引的值拷贝，因此它一般只具有只读性质，对它所做的任何修改都不会影响到集合中原有的值（**译者注：如果 `val` 为指针，则会产生指针的拷贝，依旧可以修改集合中的原值**）。一个字符串是 Unicode 编码的字符（或称之为 `rune`）集合，因此您也可以用它迭代字符串：

```go
for pos, char := range str {
...
}
```

每个 rune 字符和索引在 for-range 循环中是一一对应的。它能够自动根据 UTF-8 规则识别 Unicode 编码的字符。

示例 5.9 [range_string.go](examples/chapter_5/range_string.go)：

```go
package main

import "fmt"

func main() {
  str := "Go is a beautiful language!"
  fmt.Printf("The length of str is: %d\n", len(str))
  for pos, char := range str {
    fmt.Printf("Character on position %d is: %c \n", pos, char)
  }
  fmt.Println()
  str2 := "Chinese: 日本語"
  fmt.Printf("The length of str2 is: %d\n", len(str2))
  for pos, char := range str2 {
      fmt.Printf("character %c starts at byte position %d\n", char, pos)
  }
  fmt.Println()
  fmt.Println("index int(rune) rune    char bytes")
  for index, rune := range str2 {
      fmt.Printf("%-2d      %d      %U '%c' % X\n", index, rune, rune, rune, []byte(string(rune)))
  }
}
```

输出：

```
The length of str is: 27
Character on position 0 is: G 
Character on position 1 is: o 
Character on position 2 is:   
Character on position 3 is: i 
Character on position 4 is: s 
Character on position 5 is:   
Character on position 6 is: a 
Character on position 7 is:   
Character on position 8 is: b 
Character on position 9 is: e 
Character on position 10 is: a 
Character on position 11 is: u 
Character on position 12 is: t 
Character on position 13 is: i 
Character on position 14 is: f 
Character on position 15 is: u 
Character on position 16 is: l 
Character on position 17 is:   
Character on position 18 is: l 
Character on position 19 is: a 
Character on position 20 is: n 
Character on position 21 is: g 
Character on position 22 is: u 
Character on position 23 is: a 
Character on position 24 is: g 
Character on position 25 is: e 
Character on position 26 is: ! 

The length of str2 is: 18
character C starts at byte position 0
character h starts at byte position 1
character i starts at byte position 2
character n starts at byte position 3
character e starts at byte position 4
character s starts at byte position 5
character e starts at byte position 6
character : starts at byte position 7
character   starts at byte position 8
character 日 starts at byte position 9
character 本 starts at byte position 12
character 語 starts at byte position 15

index int(rune) rune    char bytes
0       67      U+0043 'C' 43
1       104      U+0068 'h' 68
2       105      U+0069 'i' 69
3       110      U+006E 'n' 6E
4       101      U+0065 'e' 65
5       115      U+0073 's' 73
6       101      U+0065 'e' 65
7       58      U+003A ':' 3A
8       32      U+0020 ' ' 20
9       26085      U+65E5 '日' E6 97 A5
12      26412      U+672C '本' E6 9C AC
15      35486      U+8A9E '語' E8 AA 9E
```

请将输出结果和 Listing 5.7（for_string.go）进行对比。

我们可以看到，常用英文字符使用 1 个字节表示，而汉字（**译者注：严格来说，“Chinese: 日本語”的Chinese应该是Japanese**）使用 3 个字符表示。

# 5.5 Break 与 continue

您可以使用 break 语句重写 for2.go 的代码：

示例 5.10 [for3.go](examples/chapter_5/for3.go)：

```go
for {
  i = i - 1
  fmt.Printf("The variable i is now: %d\n", i)
  if i < 0 {
    break
  }
}
```

因此每次迭代都会对条件进行检查（i < 0），以此判断是否需要停止循环。如果退出条件满足，则使用 break 语句退出循环。

一个 break 的作用范围为该语句出现后的最内部的结构，它可以被用于任何形式的 for 循环（计数器、条件判断等）。但在 switch 或 select 语句中（详见第 13 章），break 语句的作用结果是跳过整个代码块，执行后续的代码。

下面的示例中包含了嵌套的循环体（for4.go），break 只会退出最内层的循环：

示例 5.11 [for4.go](examples/chapter_5/for4.go)：

```go
package main

func main() {
  for i:=0; i<3; i++ {
    for j:=0; j<10; j++ {
      if j>5 {
          break   
      }
      print(j)
    }
    print("  ")
  }
}
```

输出：

  012345 012345 012345

关键字 continue 忽略剩余的循环体而直接进入下一次循环的过程，但不是无条件执行下一次循环，执行之前依旧需要满足循环的判断条件。

示例 5.12 [for5.go](examples/chapter_5/for5.go)：

```go
package main

func main() {
  for i := 0; i < 10; i++ {
    if i == 5 {
      continue
    }
    print(i)
    print(" ")
  }
}
```

输出：

```
0 1 2 3 4 6 7 8 9
```

显然，5 被跳过了。

另外，关键字 continue 只能被用于 for 循环中。

# 5.6 标签与 goto

for、switch 或 select 语句都可以配合标签（label）形式的标识符使用，即某一行第一个以冒号（`:`）结尾的单词（gofmt 会将后续代码自动移至下一行）。

示例 5.13 [for6.go](examples/chapter_5/for6.go)：

（标签的名称是大小写敏感的，为了提升可读性，一般建议使用全部大写字母）

```go
package main

import "fmt"

func main() {

LABEL1:
  for i := 0; i <= 5; i++ {
    for j := 0; j <= 5; j++ {
      if j == 4 {
        continue LABEL1
      }
      fmt.Printf("i is: %d, and j is: %d\n", i, j)
    }
  }

}
```

本例中，continue 语句指向 LABEL1，当执行到该语句的时候，就会跳转到 LABEL1 标签的位置。

您可以看到当 j==4 和 j==5 的时候，没有任何输出：标签的作用对象为外部循环，因此 i 会直接变成下一个循环的值，而此时 j 的值就被重设为 0，即它的初始值。如果将 continue 改为 break，则不会只退出内层循环，而是直接退出外层循环了。另外，还可以使用 goto 语句和标签配合使用来模拟循环。

示例 5.14 [goto.go](examples/chapter_5/goto.go)：

```go
package main

func main() {
  i:=0
  HERE:
    print(i)
    i++
    if i==5 {
      return
    }
    goto HERE
}
```

上面的代码会输出 `01234`。

使用逆向的 goto 会很快导致意大利面条式的代码，所以不应当使用而选择更好的替代方案。

**特别注意** 使用标签和 goto 语句是不被鼓励的：它们会很快导致非常糟糕的程序设计，而且总有更加可读的替代方案来实现相同的需求。

一个建议使用 goto 语句的示例会在第 15.1 章的 simple_tcp_server.go 中出现：示例中在发生读取错误时，使用 goto 来跳出无限读取循环并关闭相应的客户端链接。

定义但未使用标签会导致编译错误：`label … defined and not used`。

如果您必须使用 goto，应当只使用正序的标签（标签位于 goto 语句之后），但注意标签和 goto 语句之间不能出现定义新变量的语句，否则会导致编译失败。

示例 5.15 [goto2.go](examples/chapter_5/got2o.go)：

```go
// compile error goto2.go:8: goto TARGET jumps over declaration of b at goto2.go:8
package main

import "fmt"

func main() {
    a := 1
    goto TARGET // compile error
    b := 9
  TARGET:  
    b += a
    fmt.Printf("a is %v *** b is %v", a, b)
}
```

# 6.1 函数

每一个程序都包含很多的函数：函数是基本的代码块。

Go是编译型语言，所以函数编写的顺序是无关紧要的；鉴于可读性的需求，最好把 `main()` 函数写在文件的前面，其他函数按照一定逻辑顺序进行编写（例如函数被调用的顺序）。

编写多个函数的主要目的是将一个需要很多行代码的复杂问题分解为一系列简单的任务（那就是函数）来解决。而且，同一个任务（函数）可以被调用多次，有助于代码重用。

当函数执行到代码块最后一行（`}` 之前）或者 `return` 语句的时候会退出，其中 `return` 语句可以带有零个或多个参数；这些参数将作为返回值（参考 [第 6.2 节](06.2.md)）供调用者使用。简单的 `return ` 语句也可以用来结束 for 死循环，或者结束一个协程（goroutine）。

Go 里面有三种类型的函数：  

- 普通的带有名字的函数
- 匿名函数或者lambda函数
- 方法（Methods）

除了main()、init()函数外，其它所有类型的函数都可以有参数与返回值。函数参数、返回值以及它们的类型被统称为函数签名。
   
函数被调用的基本格式如下：

```go
pack1.Function(arg1, arg2, …, argn)
```

`Function` 是 `pack1` 包里面的一个函数，括号里的是被调用函数的实参（argument）：这些值被传递给被调用函数的*形参*（parameter）。函数被调用的时候，这些实参将被复制（简单而言）然后传递给被调用函数。函数一般是在其他函数里面被调用的，这个其他函数被称为调用函数（calling function）。函数能多次调用其他函数，这些被调用函数按顺序（简单而言）执行，理论上，函数调用其他函数的次数是无穷的（直到函数调用栈被耗尽）。

一个简单的函数调用其他函数的例子：

示例 6.1 [greeting.go](examples/chapter_6/greeting.go)

```go
package main

func main() {
    println("In main before calling greeting")
    greeting()
    println("In main after calling greeting")
}

func greeting() {
    println("In greeting: Hi!!!!!")
}
```
    
代码输出：

    In main before calling greeting
    In greeting: Hi!!!!!
    In main after calling greeting
    
函数可以将其他函数调用作为它的参数，只要这个被调用函数的返回值个数、返回值类型和返回值的顺序与调用函数所需求的实参是一致的，例如：

假设 f1 需要 3 个参数 `f1(a, b, c int)`，同时 f2 返回 3 个参数 `f2(a, b int) (int, int, int)`，就可以这样调用 f1：`f1(f2(a, b))`。

函数重载（function overloading）指的是可以编写多个同名函数，只要它们拥有不同的形参与/或者不同的返回值，在 Go 里面函数重载是不被允许的。这将导致一个编译错误：

    funcName redeclared in this book, previous declaration at lineno
    
Go 语言不支持这项特性的主要原因是函数重载需要进行多余的类型匹配影响性能；没有重载意味着只是一个简单的函数调度。所以你需要给不同的函数使用不同的名字，我们通常会根据函数的特征对函数进行命名。

如果需要申明一个在外部定义的函数，你只需要给出函数名与函数签名，不需要给出函数体：

```go
func flushICache(begin, end uintptr) // implemented externally
```

**函数也可以以申明的方式被使用，作为一个函数类型**，就像：

```go
type binOp func(int, int) int
```

在这里，不需要函数体 `{}`。

函数是一等值（first-class value）：它们可以赋值给变量，就像 `add := binOp` 一样。

这个变量知道自己指向的函数的签名，所以给它赋一个具有不同签名的函数值是不可能的。

函数值（functions value）之间可以相互比较：如果它们引用的是相同的函数或者都是 nil 的话，则认为它们是相同的函数。函数不能在其它函数里面声明（不能嵌套），不过我们可以通过使用匿名函数来破除这个限制。

目前 Go 没有泛型（generic）的概念，也就是说它不支持那种支持多种类型的函数。不过在大部分情况下可以通过接口（interface），特别是空接口与类型选择（type switch）与/或者通过使用反射来实现相似的功能。使用这些技术将导致代码更为复杂、性能更为低下，所以在非常注意性能的的场合，最好是为每一个类型单独创建一个函数，而且代码可读性更强。

# 6.2 函数参数与返回值

函数能够接收参数供自己使用，也可以返回零个或多个值（我们通常把返回多个值称为返回一组值）。相比与 C、C++、Java 和 C#，多值返回是 Go 的一大特性，为我们判断一个函数是否正常执行（参考 [第 5.2 节](05.2.md)）提供了方便。

我们通过 `return` 关键字返回一组值。事实上，任何一个有返回值（单个或多个）的函数都必须以 `return ` 或 `panic`（参考 [第 13 章](13.0.md)）结尾。

在函数块里面，`return` 之后的语句都不会执行。如果一个函数需要返回值，那么这个函数里面的每一个代码分支（code-path）都要有 `return` 语句。

函数定义时，它的形参一般是有名字的，不过我们也可以定义没有形参名的函数，只有相应的形参类型，就像这样：`func f(int, int, float64)`。

没有参数的函数通常被称为 **niladic** 函数（niladic function），就像 `main.main()`。

## 6.2.1 按值传递（call by value） 按引用传递（call by reference）

Go 默认使用按值传递来传递参数，也就是传递参数的副本。函数接收参数副本之后，在使用变量的过程中可能对副本的值进行更改，但不会影响到原来的变量，比如 `Function(arg1)`。

如果你希望函数可以直接修改参数的值，而不是对参数的副本进行操作，你需要将参数的地址（变量名前面添加&符号，比如 &variable）传递给函数，这就是按引用传递，比如 `Function(&arg1)`，此时传递给函数的是一个指针。如果传递给函数的是一个指针，指针的值（一个地址）会被复制，但指针的值所指向的地址上的值不会被复制；我们可以通过这个指针的值来修改这个值所指向的地址上的值。（**译者注：指针也是变量类型，有自己的地址和值，通常指针的值指向一个变量的地址。所以，按引用传递也是按值传递。**）

几乎在任何情况下，传递指针（一个32位或者64位的值）的消耗都比传递副本来得少。

在函数调用时，像切片（slice）、字典（map）、接口（interface）、通道（channel）这样的引用类型都是默认使用引用传递（即使没有显式的指出指针）。

有些函数只是完成一个任务，并没有返回值。我们仅仅是利用了这种函数的副作用，就像输出文本到终端，发送一个邮件或者是记录一个错误等。

但是绝大部分的函数还是带有返回值的。

如下，simple_function.go 里的 `MultiPly3Nums` 函数带有三个形参，分别是 `a`、`b`、`c`，还有一个 `int` 类型的返回值（被注释的代码具有和未注释部分同样的功能，只是多引入了一个本地变量）：

示例 6.2 [simple_function.go](examples/chapter_6/simple_function.go)

```go
package main

import "fmt"

func main() {
    fmt.Printf("Multiply 2 * 5 * 6 = %d\n", MultiPly3Nums(2, 5, 6))
    // var i1 int = MultiPly3Nums(2, 5, 6)
    // fmt.Printf("MultiPly 2 * 5 * 6 = %d\n", i1)
}

func MultiPly3Nums(a int, b int, c int) int {
    // var product int = a * b * c
    // return product
    return a * b * c
}
```
    
输出显示：

    Multiply 2 * 5 * 6 = 60
    
如果一个函数需要返回四到五个值，我们可以传递一个切片给函数（如果返回值具有相同类型）或者是传递一个结构体（如果返回值具有不同的类型）。因为传递一个指针允许直接修改变量的值，消耗也更少。

## 6.2.2 命名的返回值（named return variables）

如下，multiple_return.go 里的函数带有一个 `int` 参数，返回两个 `int` 值；其中一个函数的返回值在函数调用时就已经被赋予了一个初始零值。

`getX2AndX3` 与 `getX2AndX3_2` 两个函数演示了如何使用非命名返回值与命名返回值的特性。当需要返回多个非命名返回值时，需要使用 `()` 把它们括起来，比如 `(int, int)`。

命名返回值作为结果形参（result parameters）被初始化为相应类型的零值，当需要返回的时候，我们只需要一条简单的不带参数的return语句。需要注意的是，即使只有一个命名返回值，也需要使用 `()` 括起来（参考 [第 6.6 节](06.6.md)的 fibonacci.go 函数）。

示例 6.3 [multiple_return.go](examples/chapter_6/multiple_return.go)

```go
package main

import "fmt"

var num int = 10
var numx2, numx3 int

func main() {
    numx2, numx3 = getX2AndX3(num)
    PrintValues()
    numx2, numx3 = getX2AndX3_2(num)
    PrintValues()
}

func PrintValues() {
    fmt.Printf("num = %d, 2x num = %d, 3x num = %d\n", num, numx2, numx3)
}

func getX2AndX3(input int) (int, int) {
    return 2 * input, 3 * input
}

func getX2AndX3_2(input int) (x2 int, x3 int) {
    x2 = 2 * input
    x3 = 3 * input
    // return x2, x3
    return
}
```
    
输出结果：
    
    num = 10, 2x num = 20, 3x num = 30    
    num = 10, 2x num = 20, 3x num = 30 

警告：

- return 或 return var 都是可以的。
- 不过 `return var = expression`（表达式） 会引发一个编译错误：`syntax error: unexpected =, expecting semicolon or newline or }`。
        
即使函数使用了命名返回值，你依旧可以无视它而返回明确的值。        
            
任何一个非命名返回值（使用非命名返回值是很糟的编程习惯）在 `return` 语句里面都要明确指出包含返回值的变量或是一个可计算的值（就像上面警告所指出的那样）。

**尽量使用命名返回值：会使代码更清晰、更简短，同时更加容易读懂。**

## 6.2.3 空白符（blank identifier）

空白符用来匹配一些不需要的值，然后丢弃掉，下面的 blank_identifier.go 就是很好的例子。

`ThreeValues` 是拥有三个返回值的不需要任何参数的函数，在下面的例子中，我们将第一个与第三个返回值赋给了 `i1` 与 `f1`。第二个返回值赋给了空白符 `_`，然后自动丢弃掉。

示例 6.4 [blank_identifier.go](examples/chapter_6/blank_identifier.go)

```go
package main

import "fmt"

func main() {
    var i1 int
    var f1 float32
    i1, _, f1 = ThreeValues()
    fmt.Printf("The int: %d, the float: %f \n", i1, f1)
}

func ThreeValues() (int, int, float32) {
    return 5, 6, 7.5
}
```
    
输出结果：

    The int: 5, the float: 7.500000
    
另外一个示例，函数接收两个参数，比较它们的大小，然后按小-大的顺序返回这两个数，示例代码为minmax.go。

示例 6.5 [minmax.go](examples/chapter_6/minmax.go)

```go
package main

import "fmt"

func main() {
    var min, max int
    min, max = MinMax(78, 65)
    fmt.Printf("Minmium is: %d, Maximum is: %d\n", min, max)
}

func MinMax(a int, b int) (min int, max int) {
    if a < b {
        min = a
        max = b
    } else { // a = b or a < b
        min = b
        max = a
    }
    return
}
```
    
输出结果：

    Minimum is: 65, Maximum is 78
    
## 6.2.4 改变外部变量（outside variable）

传递指针给函数不但可以节省内存（因为没有复制变量的值），而且赋予了函数直接修改外部变量的能力，所以被修改的变量不再需要使用 `return` 返回。如下的例子，`reply` 是一个指向 `int` 变量的指针，通过这个指针，我们在函数内修改了这个 `int` 变量的数值。

示例 6.6 [side_effect.go](examples/chapter_6/side_effect.go)

```go
package main

import (
    "fmt"
)

// this function changes reply:
func Multiply(a, b int, reply *int) {
    *reply = a * b
}

func main() {
    n := 0
    reply := &n
    Multiply(10, 5, reply)
    fmt.Println("Multiply:", *reply) // Multiply: 50
}
```
    
这仅仅是个指导性的例子，当需要在函数内改变一个占用内存比较大的变量时，性能优势就更加明显了。然而，如果不小心使用的话，传递一个指针很容易引发一些不确定的事，所以，我们要十分小心那些可以改变外部变量的函数，在必要时，需要添加注释以便其他人能够更加清楚的知道函数里面到底发生了什么。

# 6.3 传递变长参数

如果函数的最后一个参数是采用 `...type` 的形式，那么这个函数就可以处理一个变长的参数，这个长度可以为 0，这样的函数称为变参函数。

```go
func myFunc(a, b, arg ...int) {}
```

这个函数接受一个类似某个类型的 slice 的参数，该参数可以通过第 5.4.4 节中提到的 for 循环结构迭代。

示例函数和调用：

```go
func Greeting(prefix string, who ...string)
Greeting("hello:", "Joe", "Anna", "Eileen")
```

在 Greeting 函数中，变量 `who` 的值为 `[]string{"Joe", "Anna", "Eileen"}`。

如果参数被存储在一个 slice 类型的变量 `slice` 中，则可以通过 `slice...` 的形式来传递参数，调用变参函数。

示例 6.7 [varnumpar.go](examples/chapter_6/varnumpar.go)

```go
package main

import "fmt"

func main() {
  x := min(1, 3, 2, 0)
  fmt.Printf("The minimum is: %d\n", x)
  slice := []int{7,9,3,5,1}
  x = min(slice...)
  fmt.Printf("The minimum in the slice is: %d", x)
}

func min(s ...int) int {
  if len(s)==0 {
    return 0
  }
  min := s[0]
  for _, v := range s {
    if v < min {
      min = v
    }
  }
  return min
}
```

输出：

  The minimum is: 0
  The minimum in the slice is: 1


一个接受变长参数的函数可以将这个参数作为其它函数的参数进行传递：

```go
func F1(s ...string) {
  F2(s...)
  F3(s)
}

func F2(s ...string) { }
func F3(s []string) { }
```

变长参数可以作为对应类型的 slice 进行二次传递。

但是如果变长参数的类型并不是都相同的呢？使用 5 个参数来进行传递并不是很明智的选择，有 2 种方案可以解决这个问题：

1. 使用结构（详见第 10 章）：

  定义一个结构类型，假设它叫 `Options`，用以存储所有可能的参数：

  ```go
  type Options struct {
    par1 type1,
    par2 type2,
    ...
  }
  ```

  函数 F1 可以使用正常的参数 a 和 b，以及一个没有任何初始化的 Options 结构： `F1(a, b, Options {})`。如果需要对选项进行初始化，则可以使用 `F1(a, b, Options {par1:val1, par2:val2})`。


# 6.4 defer 和追踪

关键字 defer 允许我们推迟到函数返回之前（或任意位置执行 `return` 语句之后）一刻才执行某个语句或函数（为什么要在返回之后才执行这些语句？因为 `return` 语句同样可以包含一些操作，而不是单纯地返回某个值）。

关键字 defer 的用法类似于面向对象编程语言 Java 和 C# 的 `finally` 语句块，它一般用于释放某些已分配的资源。

示例 6.8 [defer.go](examples/chapter_6/defer.go)：

```go
package main
import "fmt"

func main() {
  function1()
}

func function1() {
  fmt.Printf("In function1 at the top\n")
  defer function2()
  fmt.Printf("In function1 at the bottom!\n")
}

func function2() {
  fmt.Printf("Function2: Deferred until the end of the calling function!")
}
```

输出：

```
In Function1 at the top
In Function1 at the bottom!
Function2: Deferred until the end of the calling function!
```

请将 defer 关键字去掉并对比输出结果。

使用 defer 的语句同样可以接受参数，下面这个例子就会在执行 defer 语句时打印 `0`：

```go
func a() {
  i := 0
  defer fmt.Println(i)
  i++
  return
}
```

当有多个 defer 行为被注册时，它们会以逆序执行（类似栈，即后进先出）：

```go
func f() {
  for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
  }
}
```

上面的代码将会输出：`4 3 2 1 0`。

关键字 defer 允许我们进行一些函数执行完成后的收尾工作，例如：

1. 关闭文件流 （详见 [第 12.2 节](12.2.md)）

```go
// open a file  
defer file.Close()
```

2. 解锁一个加锁的资源 （详见 [第 9.3 节](09.3.md)）

```go
mu.Lock()  
defer mu.Unlock() 
```

3. 打印最终报告

```go
printHeader()  
defer printFooter()
```

4. 关闭数据库链接

```go
// open a database connection  
defer disconnectFromDB()
```

合理使用 defer 语句能够使得代码更加简洁。

以下代码模拟了上面描述的第 4 种情况：

```go
package main

import "fmt"

func main() {
  doDBOperations()
}

func connectToDB() {
  fmt.Println("ok, connected to db")
}

func disconnectFromDB() {
  fmt.Println("ok, disconnected from db")
}

func doDBOperations() {
  connectToDB()
  fmt.Println("Defering the database disconnect.")
  defer disconnectFromDB() //function called here with defer
  fmt.Println("Doing some DB operations ...")
  fmt.Println("Oops! some crash or network error ...")
  fmt.Println("Returning from function here!")
  return //terminate the program
  // deferred function executed here just before actually returning, even if
  // there is a return or abnormal termination before
}
```

输出：

```
ok, connected to db
Defering the database disconnect.
Doing some DB operations ...
Oops! some crash or network error ...
Returning from function here!
ok, disconnected from db
```

**使用 defer 语句实现代码追踪**

一个基础但十分实用的实现代码执行追踪的方案就是在进入和离开某个函数打印相关的消息，即可以提炼为下面两个函数：

```go
func trace(s string) { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }
```

以下代码展示了何时调用这两个函数：

示例 6.11 [defer_tracing2.go](examples/chapter_6/defer_tracing2.go)）：

```go
package main

import "fmt"

func trace(s string) string {
  fmt.Println("entering:", s)
  return s
}

func un(s string) {
  fmt.Println("leaving:", s)
}

func a() {
  defer un(trace("a"))
  fmt.Println("in a")
}

func b() {
  defer un(trace("b"))
  fmt.Println("in b")
  a()
}

func main() {
  b()
}
```

**使用 defer 语句来记录函数的参数与返回值**

下面的代码展示了另一种在调试时使用 defer 语句的手法（示例 6.12 [defer_logvalues.go](examples/chapter_6/defer_logvalues.go)）：

```go
package main

import (
  "io"
  "log"
)

func func1(s string) (n int, err error) {
  defer func() {
    log.Printf("func1(%q) = %d, %v", s, n, err)
  }()
  return 7, io.EOF
}

func main() {
  func1("Go")
}

```

输出：

  Output: 2011/10/04 10:46:11 func1("Go") = 7, EOF

# 6.5 内置函数

Go 语言拥有一些不需要进行导入操作就可以使用的内置函数。它们有时可以针对不同的类型进行操作，例如：len、cap 和 append，或必须用于系统级的操作，例如：panic。因此，它们需要直接获得编译器的支持。

以下是一个简单的列表，我们会在后面的章节中对它们进行逐个深入的讲解。

|名称|说明|
|---|---|
|close|用于管道通信|
|len、cap|len 用于返回某个类型的长度或数量（字符串、数组、切片、map 和管道）；cap 是容量的意思，用于返回某个类型的最大容量（只能用于切片和 map）|
|new、make|new 和 make 均是用于分配内存：new 用于值类型和用户定义的类型，如自定义结构，make 用于内置引用类型（切片、map 和管道）。它们的用法就像是函数，但是将类型作为参数：new(type)、make(type)。new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针（详见第 10.1 节）。它也可以被用于基本类型：`v := new(int)`。make(T) 返回类型 T 的初始化之后的值，因此它比 new 进行更多的工作（详见第 7.2.3/4 节、第 8.1.1 节和第 14.2.1 节）**new() 是一个函数，不要忘记它的括号**|
|copy、append|用于复制和连接切片|
|panic、recover|两者均用于错误处理机制|
|print、println|底层打印函数（详见第 4.2 节），在部署环境中建议使用 fmt 包|
|complex、real imag|用于创建和操作复数（详见第 4.5.2.2 节）|


# 6.7 将函数作为参数

函数可以作为其它函数的参数进行传递，然后在其它函数内调用执行，一般称之为回调。下面是一个将函数作为参数的简单例子（function_parameter.go）：

```go
package main

import (
  "fmt"
)

func main() {
  callback(1, Add)
}

func Add(a, b int) {
  fmt.Printf("The sum of %d and %d is: %d\n", a, b, a+b)
}

func callback(y int, f func(int, int)) {
  f(y, 2) // this becomes Add(1, 2)
}
```

输出：

  The sum of 1 and 2 is: 3

将函数作为参数的最好的例子是函数 `strings.IndexFunc()`：

该函数的签名是 `func IndexFunc(s string, f func(c int) bool) int`，它的返回值是在函数 `f(c)` 返回 true、-1 或从未返回时的索引值。

例如 `strings.IndexFunc(line, unicode.IsSpace)` 就会返回 `line` 中第一个空白字符的索引值。当然，您也可以书写自己的函数：

```go
func IsAscii(c int) bool {
  if c > 255 {
    return false
  }
  return true
}
```

# 6.8 闭包

当我们不希望给函数起名字的时候，可以使用匿名函数，例如：`func(x, y int) int { return x + y }`。

这样的一个函数不能够独立存在（编译器会返回错误：`non-declaration statement
outside function body`），但可以被赋值于某个变量，即保存函数的地址到变量中：`fplus := func(x, y int) int { return x + y }`，然后通过变量名对函数进行调用：`fplus(3,4)`。

当然，您也可以直接对匿名函数进行调用：`func(x, y int) int { return x + y } (3, 4)`。

下面是一个计算从 1 到 1 百万整数的总和的匿名函数：

```go
func() {
  sum := 0
  for i := 1; i <= 1e6; i++ {
    sum += i
  }
}()
```

表示参数列表的第一对括号必须紧挨着关键字 `func`，因为匿名函数没有名称。花括号 `{}` 涵盖着函数体，最后的一对括号表示对该匿名函数的调用。

下面的例子展示了如何将匿名函数赋值给变量并对其进行调用（function_literal.go）：

```go
package main

import "fmt"

func main() {
  f()
}
func f() {
  for i := 0; i < 4; i++ {
    g := func(i int) { fmt.Printf("%d ", i) } //此例子中只是为了演示匿名函数可分配不同的内存地址，在现实开发中，不应该把该部分信息放置到循环中。
    g(i)
    fmt.Printf(" - g is of type %T and has value %v\n", g, g)
  }
}
```

输出：

```
0 - g is of type func(int) and has value 0x681a80
1 - g is of type func(int) and has value 0x681b00
2 - g is of type func(int) and has value 0x681ac0
3 - g is of type func(int) and has value 0x681400
```

我们可以看到变量 `g` 代表的是 `func(int)`，变量的值是一个内存地址。

所以我们实际上拥有的是一个函数值：匿名函数可以被赋值给变量并作为值使用。

匿名函数像所有函数一样可以接受或不接受参数。下面的例子展示了如何传递参数到匿名函数中：

```go
func (u string) {
  fmt.Println(u)
  …
}(v)
```

匿名函数同样被称之为闭包（函数式语言的术语）：它们被允许调用定义在其它环境下的变量。闭包可使得某个函数捕捉到一些外部状态，例如：函数被创建时的状态。另一种表示方式为：一个闭包继承了函数所声明时的作用域。这种状态（作用域内的变量）都被共享到闭包的环境中，因此这些变量可以在闭包中被操作，直到被销毁，详见第 6.9 节中的示例。闭包经常被用作包装函数：它们会预先定义好 1 个或多个参数以用于包装，详见下一节中的示例。另一个不错的应用就是使用闭包来完成更加简洁的错误检查（详见第 16.10.2 节）。


# 6.9 应用闭包：将函数作为返回值

在程序 `function_return.go` 中我们将会看到函数 Add2 和 Adder 均会返回签名为 `func(b int) int` 的函数：

```go
func Add2() (func(b int) int)
func Adder(a int) (func(b int) int)
```

函数 Add2 不接受任何参数，但函数 Adder 接受一个 int 类型的整数作为参数。

我们也可以将 Adder 返回的函数存到变量中（function_return.go）。

```go
package main

import "fmt"

func main() {
  // make an Add2 function, give it a name p2, and call it:
  p2 := Add2()
  fmt.Printf("Call Add2 for 3 gives: %v\n", p2(3))
  // make a special Adder function, a gets value 2:
  TwoAdder := Adder(2)
  fmt.Printf("The result is: %v\n", TwoAdder(3))
}

func Add2() func(b int) int {
  return func(b int) int {
    return b + 2
  }
}

func Adder(a int) func(b int) int {
  return func(b int) int {
    return a + b
  }
}
```

输出：

```
Call Add2 for 3 gives: 5
The result is: 5
```

下例为一个略微不同的实现（function_closure.go）：

```go
package main

import "fmt"

func main() {
  var f = Adder()
  fmt.Print(f(1), " - ")
  fmt.Print(f(20), " - ")
  fmt.Print(f(300))
}

func Adder() func(int) int {
  var x int
  return func(delta int) int {
    x += delta
    return x
  }
}
```

函数 Adder() 现在被赋值到变量 f 中（类型为 `func(int) int`）。

输出：

  1 - 21 - 321

三次调用函数 f 的过程中函数 Adder() 中变量 delta 的值分别为：1、20 和 300。

我们可以看到，在多次调用中，变量 x 的值是被保留的，即 `0 + 1 = 1`，然后 `1 + 20 = 21`，最后 `21 + 300 = 321`：闭包函数保存并积累其中的变量的值，不管外部函数退出与否，它都能够继续操作外部函数中的局部变量。

这些局部变量同样可以是参数，例如之前例子中的 `Adder(as int)`。

这些例子清楚地展示了如何在 Go 语言中使用闭包。

在闭包中使用到的变量可以是在闭包函数体内声明的，也可以是在外部函数声明的：

```go
var g int
go func(i int) {
  s := 0
  for j := 0; j < i; j++ { s += j }
  g = s
}(1000) // Passes argument 1000 to the function literal.
```

这样闭包函数就能够被应用到整个集合的元素上，并修改它们的值。然后这些变量就可以用于表示或计算全局或平均值。

一个返回值为另一个函数的函数可以被称之为工厂函数，这在您需要创建一系列相似的函数的时候非常有用：书写一个工厂函数而不是针对每种情况都书写一个函数。下面的函数演示了如何动态返回追加后缀的函数：

```go
func MakeAddSuffix(suffix string) func(string) string {
  return func(name string) string {
    if !strings.HasSuffix(name, suffix) {
      return name + suffix
    }
    return name
  }
}
```

现在，我们可以生成如下函数：

```go
addBmp := MakeAddSuffix(".bmp")
addJpeg := MakeAddSuffix(".jpeg")
```

然后调用它们：

```go
addBmp("file") // returns: file.bmp
addJpeg("file") // returns: file.jpeg
```

可以返回其它函数的函数和接受其它函数作为参数的函数均被称之为高阶函数，是函数式语言的特点。我们已经在第 6.7 中得知函数也是一种值，因此很显然 Go 语言具有一些函数式语言的特性。闭包在 Go 语言中非常常见，常用于 goroutine 和管道操作（详见第 14.8-14.9 节）。在第 11.14 节的程序中，我们将会看到 Go 语言中的函数在处理混合对象时的强大能力。
