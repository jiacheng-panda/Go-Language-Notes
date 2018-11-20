# Notes Part 3: Array, Slice and Map

# 7.1 数组 Array

## 7.1.1 概念
数组是具有相同 **唯一类型** 的一组已编号且长度固定的数据项序列（这是一种同构的数据结构）；这种类型可以是任意的原始类型例如整型、字符串或者自定义类型。数组长度必须是一个常量表达式，并且必须是一个非负整数。数组长度也是数组类型的一部分，所以[5]int和[10]int是属于不同类型的。数组的编译时值初始化是按照数组顺序完成的（如下）。

数组元素可以通过 **索引**（位置）来读取（或者修改），索引从 0 开始，第一个元素索引为 0，第二个索引为 1，以此类推。（数组以 0 开始在所有类 C 语言中是相似的）。元素的数目，也称为长度或者数组大小必须是固定的并且在声明该数组时就给出（编译时需要知道数组长度以便分配内存）；数组长度最大为 2Gb。

声明的格式是： 

```go
var identifier [len]type
```

例如： 

```go
var arr1 [5]int
```

每个元素是一个整型值，当声明数组时所有的元素都会被自动初始化为默认值 0。

arr1 的长度是 5，索引范围从 0 到 `len(arr1)-1`。

第一个元素是 `arr1[0]`，第三个元素是 `arr1[2]`；总体来说索引 i 代表的元素是 `arr1[i]`，最后一个元素是 `arr1[len(arr1)-1]`。

对索引项为 i 的数组元素赋值可以这么操作：`arr[i] = value`，所以数组是 **可变的**。

只有有效的索引可以被使用，当使用等于或者大于 `len(arr1)` 的索引时：如果编译器可以检测到，会给出索引超限的提示信息；如果检测不到的话编译会通过而运行时会 panic:（参考 [第 13 章](13.0.md)）

  runtime error: index out of range

由于索引的存在，遍历数组的方法自然就是使用 for 结构:

- 通过 for 初始化数组项
- 通过 for 打印数组元素
- 通过 for 依次处理元素

示例 7.1 [for_arrays.go](examples/chapter_7/for_arrays.go)

```go
package main
import "fmt"

func main() {
  var arr1 [5]int

  for i:=0; i < len(arr1); i++ {
    arr1[i] = i * 2
  }

  for i:=0; i < len(arr1); i++ {
    fmt.Printf("Array at index %d is %d\n", i, arr1[i])
  }
}
```

输出结果：

  Array at index 0 is 0
  Array at index 1 is 2
  Array at index 2 is 4
  Array at index 3 is 6
  Array at index 4 is 8

for 循环中的条件非常重要：`i < len(arr1)`，如果写成 `i <= len(arr1)` 的话会产生越界错误。

IDIOM:

```go
for i:=0; i < len(arr1); i++｛
  arr1[i] = ...
}
```

也可以使用 for-range 的生成方式：

IDIOM:

```go
for i,_:= range arr1 {
...
}
```

在这里i也是数组的索引。当然这两种 for 结构对于切片（slices）（参考 [第 7 章](07.2.md)）来说也同样适用。


Go 语言中的数组是一种 **值类型**（不像 C/C++ 中是指向首元素的指针），所以可以通过 `new()` 来创建： `var arr1 = new([5]int)`。

那么这种方式和 `var arr2 [5]int` 的区别是什么呢？arr1 的类型是 `*[5]int`，而 arr2的类型是 `[5]int`。

这样的结果就是当把一个数组赋值给另一个时，需要在做一次数组内存的拷贝操作。例如：

```go
arr2 := *arr1
arr2[2] = 100
```

这样两个数组就有了不同的值，在赋值后修改 arr2 不会对 arr1 生效。

所以在函数中数组作为参数传入时，如 `func1(arr2)`，会产生一次数组拷贝，func1 方法不会修改原始的数组 arr2。

如果你想修改原数组，那么 arr2 必须通过&操作符以引用方式传过来，例如 func1(&arr2），下面是一个例子

示例 7.2 [pointer_array.go](examples/chapter_7/pointer_array.go):

```go
package main
import "fmt"
func f(a [3]int) { fmt.Println(a) }
func fp(a *[3]int) { fmt.Println(a) }

func main() {
  var ar [3]int
  f(ar)   // passes a copy of ar
  fp(&ar) // passes a pointer to ar
}
```
    
输出结果：

  [0 0 0]
  &[0 0 0]

另一种方法就是生成数组切片并将其传递给函数（详见第 7.1.4 节）。


## 7.1.2 数组常量

如果数组值已经提前知道了，那么可以通过 **数组常量** 的方法来初始化数组，而不用依次使用 `[]=` 方法（所有的组成元素都有相同的常量语法）。

示例 7.3 [array_literals.go](examples/chapter_7/array_literals.go)

```go
package main
import "fmt"

func main() {
  // var arrAge = [5]int{18, 20, 15, 22, 16}
  // var arrLazy = [...]int{5, 6, 7, 8, 22}
  // var arrLazy = []int{5, 6, 7, 8, 22}
  var arrKeyValue = [5]string{3: "Chris", 4: "Ron"}
  // var arrKeyValue = []string{3: "Chris", 4: "Ron"}

  for i:=0; i < len(arrKeyValue); i++ {
    fmt.Printf("Person at %d is %s\n", i, arrKeyValue[i])
  }
}
```

第一种变化：

```go
var arrAge = [5]int{18, 20, 15, 22, 16}
```

注意 `[5]int` 可以从左边起开始忽略：`[10]int {1, 2, 3}` :这是一个有 10 个元素的数组，除了前三个元素外其他元素都为 0。

第二种变化：

```go
var arrLazy = [...]int{5, 6, 7, 8, 22}
```

`...` 可同样可以忽略，从技术上说它们其实变化成了切片。

第三种变化：`key: value syntax`

```go
var arrKeyValue = [5]string{3: "Chris", 4: "Ron"}
```

只有索引 3 和 4 被赋予实际的值，其他元素都被设置为空的字符串，所以输出结果为：

  Person at 0 is
  Person at 1 is
  Person at 2 is
  Person at 3 is Chris
  Person at 4 is Ron

在这里数组长度同样可以写成 `...` 或者直接忽略。

你可以取任意数组常量的地址来作为指向新实例的指针。

示例 7.4 [pointer_array2.go](examples/chapter_7/pointer_array2.go)

```go
package main
import "fmt"

func fp(a *[3]int) { fmt.Println(a) }

func main() {
  for i := 0; i < 3; i++ {
    fp(&[3]int{i, i * i, i * i * i})
  }
}
```

输出结果：
  
  &[0 0 0]
  &[1 1 1]
  &[2 4 8]

几何点（或者数学向量）是一个使用数组的经典例子。为了简化代码通常使用一个别名：

```go
type Vector3D [3]float32
var vec Vector3D
```

## 7.1.3 多维数组

数组通常是一维的，但是可以用来组装成多维数组，例如：`[3][5]int`，`[2][2][2]float64`。

内部数组总是长度相同的。Go 语言的多维数组是矩形式的（唯一的例外是切片的数组，参见第 7.2.5 节）。

示例 7.5 [multidim_array.go](examples/chapter_7/multidim_array.go)
    
```go
package main
const (
  WIDTH  = 1920
  HEIGHT = 1080
)

type pixel int
var screen [WIDTH][HEIGHT]pixel

func main() {
  for y := 0; y < HEIGHT; y++ {
    for x := 0; x < WIDTH; x++ {
      screen[x][y] = 0
    }
  }
}
```

## 7.1.4 将数组传递给函数

把一个大数组传递给函数会消耗很多内存。有两种方法可以避免这种现象：

- 传递数组的指针
- 使用数组的切片

接下来的例子阐明了第一种方法：

示例 7.6 [array_sum.go](examples/chapter_7/array_sum.go)
    
```go
package main
import "fmt"

func main() {
  array := [3]float64{7.0, 8.5, 9.1}
  x := Sum(&array) // Note the explicit address-of operator
  // to pass a pointer to the array
  fmt.Printf("The sum of the array is: %f", x)
}

func Sum(a *[3]float64) (sum float64) {
  for _, v := range a { // derefencing *a to get back to the array is not necessary!
    sum += v
  }
  return
}
```

输出结果：

  The sum of the array is: 24.600000

但这在 Go 中并不常用，通常使用切片。

# 7.2 切片 Slice

## 7.2.1 概念

切片（slice）是对数组一个连续片段的引用（该数组我们称之为相关数组，通常是匿名的），所以切片是一个引用类型（因此更类似于 C/C++ 中的数组类型，或者 Python 中的 list 类型）。这个片段可以是整个数组，或者是由起始和终止索引标识的一些项的子集。需要注意的是，终止索引标识的项不包括在切片内。切片提供了一个相关数组的动态窗口。

切片是可索引的，并且可以由 `len()` 函数获取长度。

给定项的切片索引可能比相关数组的相同元素的索引小。和数组不同的是，切片的长度可以在运行时修改，最小为 0 最大为相关数组的长度：切片是一个 **长度可变的数组**。

切片提供了计算容量的函数 `cap()` 可以测量切片最长可以达到多少：它等于切片的长度 + 数组除切片之外的长度。如果 s 是一个切片，`cap(s)` 就是从 `s[0]` 到数组末尾的数组长度。切片的长度永远不会超过它的容量，所以对于 切片 s 来说该不等式永远成立：`0 <= len(s) <= cap(s)`。

多个切片如果表示同一个数组的片段，它们可以共享数据；因此一个切片和相关数组的其他切片是共享存储的，相反，不同的数组总是代表不同的存储。数组实际上是切片的构建块。

**优点** 因为切片是引用，所以它们不需要使用额外的内存并且比使用数组更有效率，所以在 Go 代码中 切片比数组更常用。

声明切片的格式是： `var identifier []type`（不需要说明长度）。

一个切片在未初始化之前默认为 nil，长度为 0。

切片的初始化格式是：`var slice1 []type = arr1[start:end]`。

这表示 slice1 是由数组 arr1 从 start 索引到 `end-1` 索引之间的元素构成的子集（切分数组，start:end 被称为 slice 表达式）。所以 `slice1[0]` 就等于 `arr1[start]`。这可以在 arr1 被填充前就定义好。

如果某个人写：`var slice1 []type = arr1[:]` 那么 slice1 就等于完整的 arr1 数组（所以这种表示方式是 `arr1[0:len(arr1)]` 的一种缩写）。另外一种表述方式是：`slice1 = &arr1`。

`arr1[2:]` 和 `arr1[2:len(arr1)]` 相同，都包含了数组从第三个到最后的所有元素。

`arr1[:3]` 和 `arr1[0:3]` 相同，包含了从第一个到第三个元素（不包括第四个）。

如果你想去掉 slice1 的最后一个元素，只要 `slice1 = slice1[:len(slice1)-1]`。

一个由数字 1、2、3 组成的切片可以这么生成：`s := [3]int{1,2,3}[:]`(注: 应先用`s := [3]int{1, 2, 3}`生成数组, 再使用`s[:]`转成切片) 甚至更简单的 `s := []int{1,2,3}`。

`s2 := s[:]` 是用切片组成的切片，拥有相同的元素，但是仍然指向相同的相关数组。

一个切片 s 可以这样扩展到它的大小上限：`s = s[:cap(s)]`，如果再扩大的话就会导致运行时错误（参见第 7.7 节）。

对于每一个切片（包括 string），以下状态总是成立的：

  s == s[:i] + s[i:] // i是一个整数且: 0 <= i <= len(s)
  len(s) <= cap(s)

切片也可以用类似数组的方式初始化：`var x = []int{2, 3, 5, 7, 11}`。这样就创建了一个长度为 5 的数组并且创建了一个相关切片。

切片在内存中的组织方式实际上是一个有 3 个域的结构体：指向相关数组的指针，切片长度以及切片容量。下图给出了一个长度为 2，容量为 4 的切片y。

- `y[0] = 3` 且 `y[1] = 5`。
- 切片 `y[0:4]` 由 元素 3，5，7 和 11 组成。

如果 s2 是一个 slice，你可以将 s2 向后移动一位 `s2 = s2[1:]`，但是末尾没有移动。切片只能向后移动，`s2 = s2[-1:]` 会导致编译错误。切片不能被重新分片以获取数组的前一个元素。

**注意** 绝对不要用指针指向 slice。切片本身已经是一个引用类型，所以它本身就是一个指针!!

## 7.2.2 将切片传递给函数

如果你有一个函数需要对数组做操作，你可能总是需要把参数声明为切片。当你调用该函数时，把数组分片，创建为一个 切片引用并传递给该函数。这里有一个计算数组元素和的方法:

```go
func sum(a []int) int {
  s := 0
  for i := 0; i < len(a); i++ {
    s += a[i]
  }
  return s
}

func main() {
  var arr = [5]int{0, 1, 2, 3, 4}
  sum(arr[:])
}
```

## 7.2.3 用 make() 创建一个切片

当相关数组还没有定义时，我们可以使用 make() 函数来创建一个切片 同时创建好相关数组：`var slice1 []type = make([]type, len)`。

也可以简写为 `slice1 := make([]type, len)`，这里 `len` 是数组的长度并且也是 `slice` 的初始长度。

所以定义 `s2 := make([]int, 10)`，那么 `cap(s2) == len(s2) == 10`。

make 接受 2 个参数：元素的类型以及切片的元素个数。

如果你想创建一个 slice1，它不占用整个数组，而只是占用以 len 为个数个项，那么只要：`slice1 := make([]type, len, cap)`。

make 的使用方式是：`func make([]T, len, cap)`，其中 cap 是可选参数。

所以下面两种方法可以生成相同的切片:

```go
make([]int, 50, 100)
new([100]int)[0:50]
```

示例 7.8 [make_slice.go](examples/chapter_7/make_slice.go)

```go
package main
import "fmt"

func main() {
  var slice1 []int = make([]int, 10)
  // load the array/slice:
  for i := 0; i < len(slice1); i++ {
    slice1[i] = 5 * i
  }

  // print the slice:
  for i := 0; i < len(slice1); i++ {
    fmt.Printf("Slice at %d is %d\n", i, slice1[i])
  }
  fmt.Printf("\nThe length of slice1 is %d\n", len(slice1))
  fmt.Printf("The capacity of slice1 is %d\n", cap(slice1))
}
```

输出：  

  Slice at 0 is 0  
  Slice at 1 is 5  
  Slice at 2 is 10  
  Slice at 3 is 15  
  Slice at 4 is 20  
  Slice at 5 is 25  
  Slice at 6 is 30  
  Slice at 7 is 35  
  Slice at 8 is 40  
  Slice at 9 is 45  
  
  The length of slice1 is 10  
  The capacity of slice1 is 10  

因为字符串是纯粹不可变的字节数组，它们也可以被切分成 切片。

## 7.2.4 new() 和 make() 的区别

看起来二者没有什么区别，都在堆上分配内存，但是它们的行为不同，适用于不同的类型。

- new(T) 为每个新的类型T分配一片内存，初始化为 0 并且返回类型为\*T的内存地址：这种方法 **返回一个指向类型为 T，值为 0 的地址的指针**，它适用于值类型如数组和结构体（参见第 10 章）；它相当于 `&T{}`。
- make(T) **返回一个类型为 T 的初始值**，它只适用于3种内建的引用类型：切片、map 和 channel（参见第 8 章，第 13 章）。

换言之，new 函数分配内存，make 函数初始化。

下面的方法：

```go
var v []int = make([]int, 10, 50)
```

或者
  
```go
v := make([]int, 10, 50)
```

这样分配一个有 50 个 int 值的数组，并且创建了一个长度为 10，容量为 50 的 切片 v，该 切片 指向数组的前 10 个元素。

## 7.2.5 多维 切片

和数组一样，切片通常也是一维的，但是也可以由一维组合成高维。通过分片的分片（或者切片的数组），长度可以任意动态变化，所以 Go 语言的多维切片可以任意切分。而且，内层的切片必须单独分配（通过 make 函数）。

## 7.2.6 bytes 包

类型 `[]byte` 的切片十分常见，Go 语言有一个 bytes 包专门用来解决这种类型的操作方法。

bytes 包和字符串包十分类似（参见第 4.7 节）。而且它还包含一个十分有用的类型 Buffer:

```go
import "bytes"

type Buffer struct {
  ...
}
```

这是一个长度可变的 bytes 的 buffer，提供 Read 和 Write 方法，因为读写长度未知的 bytes 最好使用 buffer。

Buffer 可以这样定义：`var buffer bytes.Buffer`。

或者使用 new 获得一个指针：`var r *bytes.Buffer = new(bytes.Buffer)`。

或者通过函数：`func NewBuffer(buf []byte) *Buffer`，创建一个 Buffer 对象并且用 buf 初始化好；NewBuffer 最好用在从 buf 读取的时候使用。

**通过 buffer 串联字符串**

类似于 Java 的 StringBuilder 类。

在下面的代码段中，我们创建一个 buffer，通过 `buffer.WriteString(s)` 方法将字符串 s 追加到后面，最后再通过 `buffer.String()` 方法转换为 string：

```go
var buffer bytes.Buffer
for {
  if s, ok := getNextString(); ok { //method getNextString() not shown here
    buffer.WriteString(s)
  } else {
    break
  }
}
fmt.Print(buffer.String(), "\n")
```

这种实现方式比使用 `+=` 要更节省内存和 CPU，尤其是要串联的字符串数目特别多的时候。

# 7.3 For-range 结构

这种构建方法可以应用于数组和切片:

```go
for ix, value := range slice1 {
  ...
}
```

第一个返回值 ix 是数组或者切片的索引，第二个是在该索引位置的值；他们都是仅在 for 循环内部可见的局部变量。value 只是 slice1 某个索引位置的值的一个拷贝，不能用来修改 slice1 该索引位置的值。

示例 7.9 [slices_forrange.go](examples/chapter_7/slices_forrange.go)

```go
package main

import "fmt"

func main() {
  var slice1 []int = make([]int, 4)

  slice1[0] = 1
  slice1[1] = 2
  slice1[2] = 3
  slice1[3] = 4

  for ix, value := range slice1 {
    fmt.Printf("Slice at %d is: %d\n", ix, value)
  }
}
```

示例 7.10 [slices_forrange2.go](examples/chapter_7/slices_forrange2.go)

```go
package main
import "fmt"

func main() {
  seasons := []string{"Spring", "Summer", "Autumn", "Winter"}
  for ix, season := range seasons {
    fmt.Printf("Season %d is: %s\n", ix, season)
  }

  var season string
  for _, season = range seasons {
    fmt.Printf("%s\n", season)
  }
}
```

slices_forrange2.go 给出了一个关于字符串的例子， `_` 可以用于忽略索引。

如果你只需要索引，你可以忽略第二个变量，例如：

```go
for ix := range seasons {
  fmt.Printf("%d", ix)
}
// Output: 0 1 2 3
```

如果你需要修改 `seasons[ix]` 的值可以使用这个版本。

**多维切片下的 for-range：**

通过计算行数和矩阵值可以很方便的写出如（参考第 7.1.3 节）的 for 循环来，例如（参考第 7.5 节的例子 multidim_array.go）：

```go
for row := range screen {
  for column := range screen[row] {
    screen[row][column] = 1
  }
}
```

# 7.4 切片重组（reslice）

我们已经知道切片创建的时候通常比相关数组小，例如：

```go
slice1 := make([]type, start_length, capacity)
```

其中 `start_length` 作为切片初始长度而 `capacity` 作为相关数组的长度。

这么做的好处是我们的切片在达到容量上限后可以扩容。改变切片长度的过程称之为切片重组 **reslicing**，做法如下：`slice1 = slice1[0:end]`，其中 end 是新的末尾索引（即长度）。

将切片扩展 1 位可以这么做：

```go
sl = sl[0:len(sl)+1]
```

切片可以反复扩展直到占据整个相关数组。

示例 7.11 [reslicing.go](examples/chapter_7/reslicing.go)

```go
package main
import "fmt"

func main() {
  slice1 := make([]int, 0, 10)
  // load the slice, cap(slice1) is 10:
  for i := 0; i < cap(slice1); i++ {
    slice1 = slice1[0:i+1]
    slice1[i] = i
    fmt.Printf("The length of slice is %d\n", len(slice1))
  }

  // print the slice:
  for i := 0; i < len(slice1); i++ {
    fmt.Printf("Slice at %d is %d\n", i, slice1[i])
  }
}
```

输出结果：

  The length of slice is 1
  The length of slice is 2
  The length of slice is 3
  The length of slice is 4
  The length of slice is 5
  The length of slice is 6
  The length of slice is 7
  The length of slice is 8
  The length of slice is 9
  The length of slice is 10
  Slice at 0 is 0
  Slice at 1 is 1
  Slice at 2 is 2
  Slice at 3 is 3
  Slice at 4 is 4
  Slice at 5 is 5
  Slice at 6 is 6
  Slice at 7 is 7
  Slice at 8 is 8
  Slice at 9 is 9

另一个例子：

```go
var ar = [10]int{0,1,2,3,4,5,6,7,8,9}
var a = ar[5:7] // reference to subarray {5,6} - len(a) is 2 and cap(a) is 5
```

将 a 重新分片：

```go
a = a[0:4] // ref of subarray {5,6,7,8} - len(a) is now 4 but cap(a) is still 5
```

# 7.5 切片的复制与追加

如果想增加切片的容量，我们必须创建一个新的更大的切片并把原分片的内容都拷贝过来。下面的代码描述了从拷贝切片的 copy 函数和向切片追加新元素的 append 函数。

示例 7.12 [copy_append_slice.go](examples/chapter_7/copy_append_slice.go)

```go
package main
import "fmt"

func main() {
  sl_from := []int{1, 2, 3}
  sl_to := make([]int, 10)

  n := copy(sl_to, sl_from)
  fmt.Println(sl_to)
  fmt.Printf("Copied %d elements\n", n) // n == 3

  sl3 := []int{1, 2, 3}
  sl3 = append(sl3, 4, 5, 6)
  fmt.Println(sl3)
}
```

`func append(s[]T, x ...T) []T` 其中 append 方法将 0 个或多个具有相同类型 s 的元素追加到切片后面并且返回新的切片；追加的元素必须和原切片的元素同类型。如果 s 的容量不足以存储新增元素，append 会分配新的切片来保证已有切片元素和新增元素的存储。因此，返回的切片可能已经指向一个不同的相关数组了。append 方法总是返回成功，除非系统内存耗尽了。

如果你想将切片 y 追加到切片 x 后面，只要将第二个参数扩展成一个列表即可：`x = append(x, y...)`。

**注意**： append 在大多数情况下很好用，但是如果你想完全掌控整个追加过程，你可以实现一个这样的 AppendByte 方法：

```go
func AppendByte(slice []byte, data ...byte) []byte {
  m := len(slice)
  n := m + len(data)
  if n > cap(slice) { // if necessary, reallocate
    // allocate double what's needed, for future growth.
    newSlice := make([]byte, (n+1)*2)
    copy(newSlice, slice)
    slice = newSlice
  }
  slice = slice[0:n]
  copy(slice[m:n], data)
  return slice
}
```

`func copy(dst, src []T) int` copy 方法将类型为 T 的切片从源地址 src 拷贝到目标地址 dst，覆盖 dst 的相关元素，并且返回拷贝的元素个数。源地址和目标地址可能会有重叠。拷贝个数是 src 和 dst 的长度最小值。如果 src 是字符串那么元素类型就是 byte。如果你还想继续使用 src，在拷贝结束后执行 `src = dst`。

# 7.6 字符串、数组和切片的应用

## 7.6.1 从字符串生成字节切片

假设 s 是一个字符串（本质上是一个字节数组），那么就可以直接通过 `c := []byte(s)` 来获取一个字节的切片 c。另外，您还可以通过 copy 函数来达到相同的目的：`copy(dst []byte, src string)`。

同样的，还可以使用 for-range 来获得每个元素（Listing 7.13—for_string.go）：

```go
package main

import "fmt"

func main() {
    s := "\u00ff\u754c"
    for i, c := range s {
        fmt.Printf("%d:%c ", i, c)
    }
}
```

输出：

    0:ÿ 2:界

我们知道，Unicode 字符会占用 2 个字节，有些甚至需要 3 个或者 4 个字节来进行表示。如果发现错误的 UTF8 字符，则该字符会被设置为 U+FFFD 并且索引向前移动一个字节。和字符串转换一样，您同样可以使用 `c := []int32(s)` 语法，这样切片中的每个 int 都会包含对应的 Unicode 代码，因为字符串中的每次字符都会对应一个整数。类似的，您也可以将字符串转换为元素类型为 rune 的切片：`r := []rune(s)`。

可以通过代码 `len([]int32(s))` 来获得字符串中字符的数量，但使用 `utf8.RuneCountInString(s)` 效率会更高一点。(参考[count_characters.go](exercises/chapter_4/count_characters.go))

您还可以将一个字符串追加到某一个字符数组的尾部：

```go
var b []byte
var s string
b = append(b, s...)
```

## 7.6.2 获取字符串的某一部分

使用 `substr := str[start:end]` 可以从字符串 str 获取到从索引 start 开始到 `end-1` 位置的子字符串。同样的，`str[start:]` 则表示获取从 start 开始到 `len(str)-1` 位置的子字符串。而 `str[:end]` 表示获取从 0 开始到 `end-1` 的子字符串。

## 7.6.3 字符串和切片的内存结构

在内存中，一个字符串实际上是一个双字结构，即一个指向实际数据的指针和记录字符串长度的整数（见图 7.4）。因为指针对用户来说是完全不可见，因此我们可以依旧把字符串看做是一个值类型，也就是一个字符数组。


## 7.6.4 修改字符串中的某个字符

Go 语言中的字符串是不可变的，也就是说 `str[index]` 这样的表达式是不可以被放在等号左侧的。如果尝试运行 `str[i] = 'D'` 会得到错误：`cannot assign to str[i]`。

因此，您必须先将字符串转换成字节数组，然后再通过修改数组中的元素值来达到修改字符串的目的，最后将字节数组转换回字符串格式。

例如，将字符串 "hello" 转换为 "cello"：

```go
s := "hello"
c := []byte(s)
c[0] = 'c'
s2 := string(c) // s2 == "cello"
```

所以，您可以通过操作切片来完成对字符串的操作。

## 7.6.5 字节数组对比函数

下面的 `Compare` 函数会返回两个字节数组字典顺序的整数对比结果，即 `0 if a == b, -1 if a < b, 1 if a > b`。

```go
func Compare(a, b[]byte) int {
    for i:=0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    // 数组的长度可能不同
    switch {
    case len(a) < len(b):
        return -1
    case len(a) > len(b):
        return 1
    }
    return 0 // 数组相等
}
```

## 7.6.6 搜索及排序切片和数组

标准库提供了 `sort` 包来实现常见的搜索和排序操作。您可以使用 `sort` 包中的函数 `func Ints(a []int)` 来实现对 int 类型的切片排序。例如 `sort.Ints(arri)`，其中变量 arri 就是需要被升序排序的数组或切片。为了检查某个数组是否已经被排序，可以通过函数 `IntsAreSorted(a []int) bool` 来检查，如果返回 true 则表示已经被排序。

类似的，可以使用函数 `func Float64s(a []float64)` 来排序 float64 的元素，或使用函数 `func Strings(a []string)` 排序字符串元素。

想要在数组或切片中搜索一个元素，该数组或切片必须先被排序（因为标准库的搜索算法使用的是二分法）。然后，您就可以使用函数 `func SearchInts(a []int, n int) int` 进行搜索，并返回对应结果的索引值。

当然，还可以搜索 float64 和字符串：

```go
func SearchFloat64s(a []float64, x float64) int
func SearchStrings(a []string, x string) int
```

这就是如何使用 `sort` 包的方法，我们会在第 11.6 节对它的细节进行深入，并实现一个属于我们自己的版本。

## 7.6.7 append 函数常见操作

我们在第 7.5 节提到的 append 非常有用，它能够用于各种方面的操作：

1. 将切片 b 的元素追加到切片 a 之后：`a = append(a, b...)`
2. 复制切片 a 的元素到新的切片 b 上：

    ```go
    b = make([]T, len(a))
    copy(b, a)
    ```

3. 删除位于索引 i 的元素：`a = append(a[:i], a[i+1:]...)`
4. 切除切片 a 中从索引 i 至 j 位置的元素：`a = append(a[:i], a[j:]...)`
5. 为切片 a 扩展 j 个元素长度：`a = append(a, make([]T, j)...)`
6. 在索引 i 的位置插入元素 x：`a = append(a[:i], append([]T{x}, a[i:]...)...)`
7. 在索引 i 的位置插入长度为 j 的新切片：`a = append(a[:i], append(make([]T, j), a[i:]...)...)`
8. 在索引 i 的位置插入切片 b 的所有元素：`a = append(a[:i], append(b, a[i:]...)...)`
9. 取出位于切片 a 最末尾的元素 x：`x, a = a[len(a)-1], a[:len(a)-1]`
10. 将元素 x 追加到切片 a：`a = append(a, x)`

因此，您可以使用切片和 append 操作来表示任意可变长度的序列。

从数学的角度来看，切片相当于向量，如果需要的话可以定义一个向量作为切片的别名来进行操作。

## 7.6.8 切片和垃圾回收

切片的底层指向一个数组，该数组的实际容量可能要大于切片所定义的容量。只有在没有任何切片指向的时候，底层的数组内存才会被释放，这种特性有时会导致程序占用多余的内存。

**示例** 函数 `FindDigits` 将一个文件加载到内存，然后搜索其中所有的数字并返回一个切片。

```go
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}
```

这段代码可以顺利运行，但返回的 `[]byte` 指向的底层是整个文件的数据。只要该返回的切片不被释放，垃圾回收器就不能释放整个文件所占用的内存。换句话说，一点点有用的数据却占用了整个文件的内存。

想要避免这个问题，可以通过拷贝我们需要的部分到一个新的切片中：

```go
func FindDigits(filename string) []byte {
   b, _ := ioutil.ReadFile(filename)
   b = digitRegexp.Find(b)
   c := make([]byte, len(b))
   copy(c, b)
   return c
}
```
事实上，上面这段代码只能找到第一个匹配正则表达式的数字串。要想找到所有的数字，可以尝试下面这段代码：
```go
func FindFileDigits(filename string) []byte {
   fileBytes, _ := ioutil.ReadFile(filename)
   b := digitRegexp.FindAll(fileBytes, len(fileBytes))
   c := make([]byte, 0)
   for _, bytes := range b {
      c = append(c, bytes...)
   }
   return c
}
```

# 8.1 Map

## 8.1.1 概念

map 是一种特殊的数据结构：一种元素对（pair）的无序集合，pair 的一个元素是 key，对应的另一个元素是 value，所以这个结构也称为关联数组或字典。这是一种快速寻找值的理想结构：给定 key，对应的 value 可以迅速定位。

map 这种数据结构在其他编程语言中也称为字典（Python）、hash 和 HashTable 等。

map 是引用类型，可以使用如下声明：

```go
var map1 map[keytype]valuetype
var map1 map[string]int
```

（`[keytype]` 和 `valuetype` 之间允许有空格，但是 gofmt 移除了空格）

在声明的时候不需要知道 map 的长度，map 是可以动态增长的。

未初始化的 map 的值是 nil。

key 可以是任意可以用 == 或者 != 操作符比较的类型，比如 string、int、float。所以数组、切片和结构体不能作为 key (译者注：含有数组切片的结构体不能作为 key，只包含内建类型的 struct 是可以作为 key 的），但是指针和接口类型可以。如果要用结构体作为 key 可以提供 `Key()` 和 `Hash()` 方法，这样可以通过结构体的域计算出唯一的数字或者字符串的 key。

value 可以是任意类型的；通过使用空接口类型（详见第 11.9 节），我们可以存储任意值，但是使用这种类型作为值时需要先做一次类型断言（详见第 11.3 节）。

map 传递给函数的代价很小：在 32 位机器上占 4 个字节，64 位机器上占 8 个字节，无论实际上存储了多少数据。通过 key 在 map 中寻找值是很快的，比线性查找快得多，但是仍然比从数组和切片的索引中直接读取要慢 100 倍；所以如果你很在乎性能的话还是建议用切片来解决问题。

map 也可以用函数作为自己的值，这样就可以用来做分支结构（详见第 5 章）：key 用来选择要执行的函数。

如果 key1 是 map1 的key，那么 `map1[key1]` 就是对应 key1 的值，就如同数组索引符号一样（数组可以视为一种简单形式的 map，key 是从 0 开始的整数）。

key1 对应的值可以通过赋值符号来设置为 val1：`map1[key1] = val1`。

令 `v := map1[key1]` 可以将 key1 对应的值赋值给 v；如果 map 中没有 key1 存在，那么 v 将被赋值为 map1 的值类型的空值。

常用的 `len(map1)` 方法可以获得 map 中的 pair 数目，这个数目是可以伸缩的，因为 map-pairs 在运行时可以动态添加和删除。

示例 8.1 [make_maps.go](examples/chapter_8/make_maps.go)

```go
package main
import "fmt"

func main() {
  var mapLit map[string]int
  //var mapCreated map[string]float32
  var mapAssigned map[string]int

  mapLit = map[string]int{"one": 1, "two": 2}
  mapCreated := make(map[string]float32)
  mapAssigned = mapLit

  mapCreated["key1"] = 4.5
  mapCreated["key2"] = 3.14159
  mapAssigned["two"] = 3

  fmt.Printf("Map literal at \"one\" is: %d\n", mapLit["one"])
  fmt.Printf("Map created at \"key2\" is: %f\n", mapCreated["key2"])
  fmt.Printf("Map assigned at \"two\" is: %d\n", mapLit["two"])
  fmt.Printf("Map literal at \"ten\" is: %d\n", mapLit["ten"])
}
```

输出结果：

  Map literal at "one" is: 1
  Map created at "key2" is: 3.141590
  Map assigned at "two" is: 3
  Mpa literal at "ten" is: 0

mapLit 说明了 `map literals` 的使用方法： map 可以用 `{key1: val1, key2: val2}` 的描述方法来初始化，就像数组和结构体一样。

map 是 **引用类型** 的： 内存用 make 方法来分配。

map 的初始化：`var map1 = make(map[keytype]valuetype)`。

或者简写为：`map1 := make(map[keytype]valuetype)`。

上面例子中的 mapCreated 就是用这种方式创建的：`mapCreated := make(map[string]float32)`。

相当于：`mapCreated := map[string]float32{}`。

mapAssigned 也是 mapList 的引用，对 mapAssigned 的修改也会影响到 mapLit 的值。

**不要使用 new，永远用 make 来构造 map**

**注意** 如果你错误的使用 new() 分配了一个引用对象，你会获得一个空引用的指针，相当于声明了一个未初始化的变量并且取了它的地址：

```go
mapCreated := new(map[string]float32)
```

接下来当我们调用：`mapCreated["key1"] = 4.5` 的时候，编译器会报错：

  invalid operation: mapCreated["key1"] (index of type *map[string]float32).

为了说明值可以是任意类型的，这里给出了一个使用 `func() int` 作为值的 map：

示例 8.2 [map_func.go](examples/chapter_8/map_func.go)

```go
package main
import "fmt"

func main() {
  mf := map[int]func() int{
    1: func() int { return 10 },
    2: func() int { return 20 },
    5: func() int { return 50 },
  }
  fmt.Println(mf)
}
```

输出结果为：`map[1:0x10903be0 5:0x10903ba0 2:0x10903bc0]`: 整形都被映射到函数地址。

## 8.1.2 map 容量

和数组不同，map 可以根据新增的 key-value 对动态的伸缩，因此它不存在固定长度或者最大限制。但是你也可以选择标明 map 的初始容量 `capacity`，就像这样：`make(map[keytype]valuetype, cap)`。例如：

```go
map2 := make(map[string]float32, 100)
```

当 map 增长到容量上限的时候，如果再增加新的 key-value 对，map 的大小会自动加 1。所以出于性能的考虑，对于大的 map 或者会快速扩张的 map，即使只是大概知道容量，也最好先标明。

这里有一个 map 的具体例子，即将音阶和对应的音频映射起来：

```go
noteFrequency := map[string]float32 {
  "C0": 16.35, "D0": 18.35, "E0": 20.60, "F0": 21.83,
  "G0": 24.50, "A0": 27.50, "B0": 30.87, "A4": 440}
```

## 8.1.3 用切片作为 map 的值

既然一个 key 只能对应一个 value，而 value 又是一个原始类型，那么如果一个 key 要对应多个值怎么办？例如，当我们要处理unix机器上的所有进程，以父进程（pid 为整形）作为 key，所有的子进程（以所有子进程的 pid 组成的切片）作为 value。通过将 value 定义为 `[]int` 类型或者其他类型的切片，就可以优雅的解决这个问题。

这里有一些定义这种 map 的例子：

```go
mp1 := make(map[int][]int)
mp2 := make(map[int]*[]int)
```

# 8.2 测试键值对是否存在及删除元素

测试 map1 中是否存在 key1：

在例子 8.1 中，我们已经见过可以使用 `val1 = map1[key1]` 的方法获取 key1 对应的值 val1。如果 map 中不存在 key1，val1 就是一个值类型的空值。

这就会给我们带来困惑了：现在我们没法区分到底是 key1 不存在还是它对应的 value 就是空值。

为了解决这个问题，我们可以这么用：`val1, isPresent = map1[key1]`

isPresent 返回一个 bool 值：如果 key1 存在于 map1，val1 就是 key1 对应的 value 值，并且 isPresent为true；如果 key1 不存在，val1 就是一个空值，并且 isPresent 会返回 false。

如果你只是想判断某个 key 是否存在而不关心它对应的值到底是多少，你可以这么做：

```go
_, ok := map1[key1] // 如果key1存在则ok == true，否则ok为false
```

或者和 if 混合使用：

```go
if _, ok := map1[key1]; ok {
  // ...
}
```

从 map1 中删除 key1：

直接 `delete(map1, key1)` 就可以。

如果 key1 不存在，该操作不会产生错误。

# 8.3 for-range 的配套用法

可以使用 for 循环构造 map：

```go
for key, value := range map1 {
  ...
}
```

第一个返回值 key 是 map 中的 key 值，第二个返回值则是该 key 对应的 value 值；这两个都是仅 for 循环内部可见的局部变量。其中第一个返回值key值是一个可选元素。如果你只关心值，可以这么使用：

```go
for _, value := range map1 {
  ...
}
```

如果只想获取 key，你可以这么使用：

```go
for key := range map1 {
  fmt.Printf("key is: %d\n", key)
}
```

示例 8.5 [maps_forrange.go](examples/chapter_8/maps_forrange.go)：

```go
package main
import "fmt"

func main() {
  map1 := make(map[int]float32)
  map1[1] = 1.0
  map1[2] = 2.0
  map1[3] = 3.0
  map1[4] = 4.0
  for key, value := range map1 {
    fmt.Printf("key is: %d - value is: %f\n", key, value)
  }
}
```

输出结果：

  key is: 3 - value is: 3.000000
  key is: 1 - value is: 1.000000
  key is: 4 - value is: 4.000000
  key is: 2 - value is: 2.000000

注意 map 不是按照 key 的顺序排列的，也不是按照 value 的序排列的。

# 8.4 map 类型的切片

假设我们想获取一个 map 类型的切片，我们必须使用两次 `make()` 函数，第一次分配切片，第二次分配 切片中每个 map 元素（参见下面的例子 8.4）。

示例 8.4 [maps_forrange2.go](examples/chapter_8/maps_forrange2.go)：

```go
package main
import "fmt"

func main() {
  // Version A:
  items := make([]map[int]int, 5)
  for i:= range items {
    items[i] = make(map[int]int, 1)
    items[i][1] = 2
  }
  fmt.Printf("Version A: Value of items: %v\n", items)

  // Version B: NOT GOOD!
  items2 := make([]map[int]int, 5)
  for _, item := range items2 {
    item = make(map[int]int, 1) // item is only a copy of the slice element.
    item[1] = 2 // This 'item' will be lost on the next iteration.
  }
  fmt.Printf("Version B: Value of items: %v\n", items2)
}
```

输出结果：

  Version A: Value of items: [map[1:2] map[1:2] map[1:2] map[1:2] map[1:2]]
  Version B: Value of items: [map[] map[] map[] map[] map[]]

需要注意的是，应当像 A 版本那样通过索引使用切片的 map 元素。在 B 版本中获得的项只是 map 值的一个拷贝而已，所以真正的 map 元素没有得到初始化。


# 8.5 map 的排序

map 默认是无序的，不管是按照 key 还是按照 value 默认都不排序（详见第 8.3 节）。

如果你想为 map 排序，需要将 key（或者 value）拷贝到一个切片，再对切片排序（使用 sort 包，详见第 7.6.6 节），然后可以使用切片的 for-range 方法打印出所有的 key 和 value。

下面有一个示例：

示例 8.6 [sort_map.go](examples/chapter_8/sort_map.go)：

```go
// the telephone alphabet:
package main
import (
  "fmt"
  "sort"
)

var (
  barVal = map[string]int{"alpha": 34, "bravo": 56, "charlie": 23,
              "delta": 87, "echo": 56, "foxtrot": 12,
              "golf": 34, "hotel": 16, "indio": 87,
              "juliet": 65, "kili": 43, "lima": 98}
)

func main() {
  fmt.Println("unsorted:")
  for k, v := range barVal {
    fmt.Printf("Key: %v, Value: %v / ", k, v)
  }
  keys := make([]string, len(barVal))
  i := 0
  for k, _ := range barVal {
    keys[i] = k
    i++
  }
  sort.Strings(keys)
  fmt.Println()
  fmt.Println("sorted:")
  for _, k := range keys {
    fmt.Printf("Key: %v, Value: %v / ", k, barVal[k])
  }
}
```

输出结果：

  unsorted:
  Key: bravo, Value: 56 / Key: echo, Value: 56 / Key: indio, Value: 87 / Key: juliet, Value: 65 / Key: alpha, Value: 34 / Key: charlie, Value: 23 / Key: delta, Value: 87 / Key: foxtrot, Value: 12 / Key: golf, Value: 34 / Key: hotel, Value: 16 / Key: kili, Value: 43 / Key: lima, Value: 98 /
  sorted:
  Key: alpha, Value: 34 / Key: bravo, Value: 56 / Key: charlie, Value: 23 / Key: delta, Value: 87 / Key: echo, Value: 56 / Key: foxtrot, Value: 12 / Key: golf, Value: 34 / Key: hotel, Value: 16 / Key: indio, Value: 87 / Key: juliet, Value: 65 / Key: kili, Value: 43 / Key: lima, Value: 98 /

但是如果你想要一个排序的列表你最好使用结构体切片，这样会更有效：

```go
type name struct {
  key string
  value int
}
```
# 8.6 将 map 的键值对调

这里对调是指调换 key 和 value。如果 map 的值类型可以作为 key 且所有的 value 是唯一的，那么通过下面的方法可以简单的做到键值对调。

示例 8.7 [invert_map.go](examples/chapter_8/invert_map.go)：

```go
package main
import (
  "fmt"
)

var (
  barVal = map[string]int{"alpha": 34, "bravo": 56, "charlie": 23,
              "delta": 87, "echo": 56, "foxtrot": 12,
              "golf": 34, "hotel": 16, "indio": 87,
              "juliet": 65, "kili": 43, "lima": 98}
)

func main() {
  invMap := make(map[int]string, len(barVal))
  for k, v := range barVal {
    invMap[v] = k
  }
  fmt.Println("inverted:")
  for k, v := range invMap {
    fmt.Printf("Key: %v, Value: %v / ", k, v)
  }
}
```

输出结果：

  inverted:
  Key: 34, Value: golf / Key: 23, Value: charlie / Key: 16, Value: hotel / Key: 87, Value: delta / Key: 98, Value: lima / Key: 12, Value: foxtrot / Key: 43, Value: kili / Key: 56, Value: bravo / Key: 65, Value: juliet /

如果原始 value 值不唯一那么这么做肯定会出错；为了保证不出错，当遇到不唯一的 key 时应当立刻停止，这样可能会导致没有包含原 map 的所有键值对！一种解决方法就是仔细检查唯一性并且使用多值 map，比如使用 `map[int][]string` 类型。

