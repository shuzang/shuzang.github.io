---
title: Golang语法基础3-常量变量与基本数据类型
date: 2019-10-22
lastmod: 2020-07-02
tags: [Go语法]
categories: [Golang学习之路]
slug: Golang syntax 3-Constants, Variables and Elementary types
---

本文介绍 Go 中的常量、变量、基本数据类型和常用的类型转换。

<!--more-->

## 1. 常量

常量用于存储程序运行过程中不会改变的数据，试图改变会引发编译错误。Go 中常量使用关键字 `const` 定义，声明与初始化语句放在一起，格式如下

```go
const identifier [type] = value
```

常量的值必须在编译时能够确定，因此只能是基本数据类型和表达式。一个常量定义的例子如下

```go
const Pi float64 = 3.14159
```

另外，由于 Go 的一些特性，常量的定义有一些不同的形式

1. Go 支持根据值推断其类型，因此类型定义可省略

   ```go
   const Pi = 3.14159
   ```

2. Go 支持在同一行同时定义多个值，称为多重赋值

   ```go
   const Monday, Tuesday, Wednesday, Thursday, Friday, Saturday = 1, 2, 3, 4, 5, 6
   ```

3. Go 支持批量声明，这种定义形式叫做因式分解关键字

   ```go
   const (
   	Monday, Tuesday, Wednesday = 1, 2, 3
   	Thursday, Friday, Saturday = 4, 5, 6
   )
   ```

此外，Go 还提供关键字 `iota`，用作常量计数器，只能在常量定义时使用。`iota` 在 `const` 关键字出现时被重置为 0，每新增一行常量声明新增一个计数，能极大的简化定义。一个例子如下

```go
const a = iota // a=0 
const ( 
  b = iota     //b=0 
  c            //c=1   相当于c=iota
)
```

可以使用空白标识符跳过不想要的值

```go
const (
  a = iota // 0
  _
  c        // 2
)
```

同一行有多个变量不产生影响，中间有数值插队也不产生影响，因为 iota 的值是每新增一行声明增加 1

```go
const (
  a,b = iota,iota+1  // 0,1
  c,d                // 1,2
  e,f                // 2,3
)

const (
  a = iota  // 0
  b = 3.14
  c         // 2
)
```

## 2. 变量

变量使用 `var` 关键字声明，格式为

```go
var identifier type
```

当一个变量被声明后，系统会自动赋予它该类型的零值：int 为 0，float 为 0.0，bool 为 false，string 为空字符串，指针为 nil。

变量与常量的一个不同之处在于，一般情况下，声明与初始化是分离的，一个例子如下

```go
var a int
a = 15
```

但变量也可以将声明和初始化放到一起，就像常量一样，这也是最常使用的方式

```go
var a int = 15
```

最后，由于常量部分提到的三个 Go 的特性，变量的声明和使用也有一些不同的形式

1. 类型推断，从而可以省略类型定义

   ```go
   var a = 15
   ```

2. 多重赋值

   ```go
   var a, b, c = 5, 6, 7
   ```

3. 批量声明

   ```go
   var (
   	a = 15
   	b = false
   	str = "Go says hello to the world!"
   	numShips = 50
   	city string
   )
   ```

上面提到的所有方式可以用于全局变量，也可以用于局部变量，但还有一种更加简短的声明与定义方式，仅能用于局部变量（函数体内，包括 main 函数），这是我们使用非常多的一种写法

```go
a := 15
```

这里详细介绍一下常量部分提到过的空白标识符，空白标识符指的是下划线 `_`，也叫做匿名变量，只允许写入，任何类型都可以赋值给它，但无法使用它的值。我们在使用 iota 关键字时可以使用空白标识符跳过不想要的值，另外一种常见的使用场景是在多重赋值中抛弃不需要的变量

```go
_, b = 5, 7
```

匿名变量不会被分配内存，因此不占用内存空间，多次声明也不会引起冲突。

Go 还提供了一种非常友好的功能，如果想要交换两个变量的值，可以直接使用 `a, b = b, a` 这种形式，不需要再使用临时变量，为程序编写带来了极大的便利。

变量的命名规则最好遵循驼峰命名法，即首个单词小写，每个新单词的首字母大写，例如：`numShips`。但当全局变量需要对外部包可见时，首个单词首字母需要大写(可见性原则)。

变量作用域的规则同 C 语言相同，关于值类型和引用类型的理解也和 C 语言相同，Go 中引用类型包括指针、切片、映射和通道，值类型存储在栈中，引用类型存储在堆中，以便进行垃圾回收。

## 3. 基本数据类型

Go 拥有 4 大类共7种基本数据类型


- 布尔类型 bool
- 数字类型：
  - 整型 int，根据位数的不同包括 int8, int16, int32, int64 四种以及相对应的 uint
  - 浮点型 float，包括 float32 和 float64 两种
  - 复数 complex，包括 complex32 和 complex64 两种，复数类型并不常用
- 字符类型：
  - byte，uint8 的别名，完全等同
  - rune，int32 是别名，完全等同
- 字符串类型 string

### 3.1 布尔类型

使用`bool`关键字声明，值只可以是常量 true 或 false

```go
var b bool = true
```

两个**类型相同**的值可以使用关系运算符来获得一个布尔类型的值。布尔类型的值之间也可以使用逻辑运算符来产生另一个布尔值，运算规则与其它语言相同。

类型相同是一个很严格的规定，涉及到了 Go 的比较规则，我们将 Go 中不可比较类型总结如下，除此之外，其它类型都是可比较的

1. 切片类型
2. 映射类型
3. 函数类型
4. 任何字段为不可比较类型的结构体类型，以及任何元素类型为不可比较类型的数组类型

对于接口而言，情况更加复杂一点，如果值的类型是接口，那么它们必须实现了相同的接口。如果条件不满足，则必须事先进行类型转换才可以比较。或者直接一点，

布尔值（以及任何结果为布尔值的表达式）最常用在流程控制的的条件语句中，如：if、for 和 switch 结构。

另外值得注意的一点是，Go 中的布尔值并不等于数字 1 和 0，因此不能直接进行运算。

### 3.2 数字类型

Go 中数字类型分为三种，整型、浮点型和复数类型。

#### 整型

整型提供有符号和无符号两种，每一种又分别提供对应 8、16、32、64bit 大小的四种类型，总计八种，列表如下

| 整型                                                         | 无符号整型                                |
| ------------------------------------------------------------ | ----------------------------------------- |
| int8（-128 -> 127）                                          | uint8（0 -> 255）                         |
| int16（-32768 -> 32767）                                     | uint16（0 -> 65,535）                     |
| int32（-2,147,483,648 -> 2,147,483,647）                     | uint32（0 -> 4,294,967,295）              |
| int64（-9,223,372,036,854,775,808 -> 9,223,372,036,854,775,807） | uint64（0 -> 18,446,744,073,709,551,615） |

除此之外还提供两种不带位数的类型声明：int 和 uint。这两种类型的大小取决于所运行的平台处理器支持的字长，例如，在 32 位操作系统上，使用 32 位（4 个字节），在 64 位操作系统上，使用 64 位（8 个字节）。

尽管 int 有可能是 32 位，但在需要时 int 和 int32 之间也必须显式进行类型转换。

最后还有一种无符号整型 uintptr，它没有指定具体的 bit 大小但被设定为足够容纳一个指针。uintptr 类型只有在底层编程时才需要，特别是 Go 语言和 C 语言函数库或操作系统接口相交互的地方，一般用于指针计算。

int型是计算最快的类型，也是最常使用的类型。

#### 浮点型

Go 语言中没有 float 类型，没有 double 类型，只有 float32 和 float64。它们的算术规范由IEEE-754标准定义，该标准被所有现代的 CPU 支持。

float32 精确到小数点后 7 位，float64 精确到小数点后 15 位。通常应该优先使用 float64 类型，因为 `math` 包中所有有关数学运算的函数都会要求接收这个类型。

#### 复数类型

Go语言拥有两种复数类型，分别是 complex64（32 位实数和虚数）和 complex128（64 位实数和虚数）。

```go
var c1 complex64 = 5 + 10i
```

内置的 complex 函数用于构建复数，内置的 real 和 imag 函数分别返回复数的实部和虚部：

```go
var cl complex128 = complex(1, 2) // 1+2i
fmt.Println(real(cl))           // "1"
fmt.Println(imag(cl))           // "2"
```

`cmath` 包中包含了一些操作复数的公共方法。如果对内存的要求不是特别高，最好使用 complex128 作为计算类型，因为相关函数都使用这个类型的参数。

### 3.3 字符类型

Go语言的字符有两种：

- byte 型，代表了 ASCII 码的一个字符。
- rune 类型，代表一个 UTF-8 字符，当需要处理中文、日文或者其他复合字符时，则需要用到 rune 类型。

严格来说，字符只是整数的特殊用例。`byte` 类型是 `uint8` 的别名，刚好一个字节，足以表示传统 ASCII 编码的字符。例如：`var ch byte = 'A'`；`rune`是`int32`的别名，四个字节，足以表示最长的UTF-8字符。

另一方面，由于字符只是整数的别名，因此其零值也是 0。

字符使用单引号括起来。

包 `unicode` 包含了一些针对测试字符的非常有用的函数（其中 `ch` 代表字符）：

- 判断是否为字母：`unicode.IsLetter(ch)`
- 判断是否为数字：`unicode.IsDigit(ch)`
- 判断是否为空白符号：`unicode.IsSpace(ch)`

这些函数返回一个布尔值。包 `utf8` 拥有更多与 rune 相关的函数。

### 3.4 字符串类型

字符串底层约定是字节的一个序列，编码方式建议是 UTF-8，但不是必须遵守，通常是ASCII。因此取字符串单个字符的类型通常是 byte，只有遇到中文等语言是才是 rune。另外，使用 for-range 结构遍历时字符串的单个字符类型是rune

字符串是值类型，且值不可变，即创建一个字符串后无法再次修改它的内容

Go 支持以下 2 种形式的字符串：

- 解释字符串：该类字符串使用双引号括起来，其中的转义字符将被替换，这些转义字符包括：

    - `\n`：换行符
    - `\r`：回车符
    - `\t`：tab 键
    - `\u` 或 `\U`：Unicode 字符
    - `\\`：反斜杠自身
    
- 非解释字符串：该类字符串使用反引号括起来，当使用多行字符串时使用这种形式。

    ```go
    a := `abc
    def`
    fmt.Println(a)
    // Output:
    abc
    def
    ```

`string` 类型的零值为长度为零的字符串，即空字符串 `""`。

Go 中的字符串是根据长度限定的，而非特殊字符`\0`，其长度可以使用内置函数`len()`来获取，长度的基本含义是字符串在内存中所占字节的个数，所以下面的例子虽然是两个中文，但长度是6

```go
a := "中国"
fmt.Println(len(a)) // 6
```

可以将字符串看作数组而索引其内的单个字符，如第i个字节表示为`str[i-1]`

使用拼接符`+`可以拼接两个字符串，以下是一个多行字符串拼接的例子

```go
str := "Beginning of the string " +
	"second part of the string"
```

`+`必须放在第一行末尾，因为编译器会在行尾自动补全分号。当然，`+=`一样可用于字符串

```go
s := "hel" + "lo,"
s += "world!"
fmt.Println(s) //输出 “hello, world!”
```

在循环中使用`+`拼接字符串并不是最高效的做法，更好的办法是使用`string.join()`，或者使用字节缓冲`bytes.Buffer`

### 3.5 类型别名

使用某个类型时可以给它起个别名在程序中使用，用于简化名称或解决名称冲突

在 `type TZ int` 中，TZ 就是 int 类型的新名称（用于表示程序中的时区），然后就可以使用 TZ 来操作 int 类型的数据。

```go
package main
import "fmt"

type TZ int

func main() {
	var a, b TZ = 3, 4
	c := a + b
	fmt.Printf("c has the value: %d", c) // 输出：c has the value: 7
}
```

实际上，类型别名得到的新类型并非和原类型完全相同，新类型不会拥有原类型所附带的方法；TZ 可以自定义一个方法用来输出更加人性化的时区信息。

## 4. 类型转换

Go语言中类型转换是可行的，但是不存在隐式类型转换，所有的转换都必须显式说明。类型转换的基本格式为

```go
valueOfTypeB = typeB(valueOfTypeA)
```

两条转换原则如下

1. 只有相同底层类型的变量间可以进行相互转换（如int16和int32），不同底层类型的变量相互转换会引发编译错误
2. 类型转换只有从取值范围小的类型转换到取值范围大的类型才能成功，反过来会发生精度丢失（截断）

```go
var a,b int16
var c int32
A := int32(a) // 标准转换
B := bool(b)  //类型不匹配，引发编译错误
C := int16(c)  //取值范围变小，精度丢失
```

浮点型可以转换为整型，转换时会将小数部分去掉，只保留整数部分

```go
a := 12.54
fmt.Println(int(a)) //输出12
```

精度丢失可以使用专门的函数保证安全，如int型到int8

```go
func Uint8FromInt(n int) (uint8, error) {
	if 0 <= n && n <= math.MaxUint8 { // conversion is safe
		return uint8(n), nil
	}
	return 0, fmt.Errorf("%d is out of the uint8 range", n)
}
```

其它的类型转换则需要使用一些库函数

### 4.1 bool与string

Go语言中bool类型值与数字1和0不等同，因此不能和数字类型相互转换（可以简单的使用if-else结构完成这一功能）。但借助strconv包，可以和string类型转换。

```go
//string->bool
b, err := strconv.ParseBool("true")
//bool->string
s := strconv.FormatBool(true)
```

两个函数的原型如下

```go
//ParseBool返回字符串代表的bool值，接受 1, t, T, TRUE, true, True, 0, f, F, FALSE, false, False作为传入参数，其他参数均返回error
func ParseBool(str string) (bool, error) {
	switch str {
	case "1", "t", "T", "true", "TRUE", "True":
		return true, nil
	case "0", "f", "F", "false", "FALSE", "False":
		return false, nil
	}
	return false, syntaxError("ParseBool", str)
}

// FormatBool returns "true" or "false" according to the value of b.
func FormatBool(b bool) string {
	if b {
		return "true"
	}
	return "false"
}
```

### 4.2 int/float与string

与字符串相关的类型转换都是通过strconv包实现的。最常用的是Atoi(string to int)和Itoa(int to string)函数

```go
i, err := strconv.Atoi("-42")
s := strconv.Itoa(-42)
```

函数原型如下

```go
func Atoi(s string) (int, error)
func Itoa(i int) string
```

#### 字符串->数字类型

ParseFloat, ParseInt, 和ParseUint可以将字符串转化为对应的值

```go
f, err := strconv.ParseFloat("3.1415", 64)
i, err := strconv.ParseInt("-42", 10, 64)
u, err := strconv.ParseUint("42", 10, 64)
```

浮点型函数原型如下

```go
func ParseFloat(s string, bitSize int) (float64, error)
```

bitSize指定了返回值的类型，当bitSize=32，返回float32类型（结果仍是float64，但会转换为float32)；当bitSize=64，返回float64类型。只有这两种情况。

整型函数原型如下

```go
func ParseInt(s string, base int, bitSize int) (i int64, err error)
func ParseUint(s string, base int, bitSize int) (uint64, error)
```

如果base为0那么实际上base由string的前缀指定，`0x`意味着base=16，`0`意味着base=8，否则base=10，本质上是进制的前缀。若base等于1，小于0或超过36，返回一个error。

bitSize仍然指定返回值类型. 其值为 0, 8, 16, 32, 和 64 分别对应 int, int8, int16, int32和int64. bitSize值小于0或大于64，返回一个error。

#### 数字类型->字符串

FormatFloat, FormatInt, 和FormatUint可以将值转换为字符串

```go
s := strconv.FormatFloat(3.1415, 'E', -1, 64)
s := strconv.FormatInt(-42, 16)
s := strconv.FormatUint(42, 16)
```

浮点型的函数原型如下

```go
func FormatFloat(f float64, fmt byte, prec, bitSize int) string
```

bitSize表示f的来源类型（32：float32、64：float64），会据此进行舍入。

fmt表示格式：'b' (-ddddp±ddd, 二进制指数), 'e' (-d.dddde±dd,十进制指数), 'E' (-d.ddddE±dd, 十进制指数), 'f' (-ddd.dddd, 没有指数), 'g' (指数很大时用‘e', 否则用 'f' ),  'G' (指数很大时用‘E', 否则用 'f' ).

精度prec控制数字的个数 (排除指数)。对'e', 'E', 和 'f' ，表示小数点后的数字位数. 对 'g' 和 'G' 是有效数字位数 (trailing zeros are removed)。prec等于-1时则使用最少数量但又必须的数字来表示f。

整型的函数原型如下

```go
func FormatInt(i int64, base int) string
func FormatUint(i uint64, base int) string
```

返回base指定进制的整数i的字符串形式，2 <= base <= 36。使用小写字母 'a' 到 'z' 表示大于10的数字。

忽略可能出现的转换错误，可以给出如下例子：

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	var orig string = "666"
	var an int
	var newS string  

	an, _ = strconv.Atoi(orig)
	fmt.Printf("The integer is: %d\n", an) 
	an = an + 5
	newS = strconv.Itoa(an)
	fmt.Printf("The new string is: %s\n", newS)
}
//输出：
The integer is: 666
The new string is: 671
```

### 4.3 []rune与string

用 for range 遍历字符串可以返回每个字符，返回的字符类型是 rune。但 []rune 类型也经常需要转换为 string，和单个 rune 类型相似，都可以直接进行显示类型转换，如下：

```go
a := []rune{'a', 'b', 'c'}
b := 'g'
c := string(a)
d := string(b)
fmt.Println(reflect.TypeOf(a), reflect.TypeOf(b), reflect.TypeOf(c), reflect.TypeOf(d))
fmt.Println(a, b, c, d)
//Output
[]int32 int32 string string
[97 98 99] 103 abc g
```

