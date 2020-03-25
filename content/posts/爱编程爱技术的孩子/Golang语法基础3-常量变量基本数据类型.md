---
title: Golang语法基础3-常量变量与基本数据类型
date: 2019-10-22
url: "Golang syntax 3-Constants, Variables and Elementary types"
tags: [Golang]
categories: [爱编程爱技术的孩子]
---

## 常量

常量使用关键字`const`定义，用于存储程序运行过程中不会改变的数据，试图改变会引发编译错误。Fundamentals of golang syntax 3 - constant variables and basic data types

常量只能定义为基本数据类型和表达式，表达式的值必须能在编译时确定。

常量的定义格式为`const identifier [type] = value`，声明和初始化语句放在一起，方括号代表类型定义可以省略，因为编译器可以根据常量的值来推断其类型。例如

```go
const Pi float64 = 3.14159
const Pi = 3.14159
```

常量可以批量声明，示例如下

```go
const Monday, Tuesday, Wednesday, Thursday, Friday, Saturday = 1, 2, 3, 4, 5, 6
const (
	Monday, Tuesday, Wednesday = 1, 2, 3
	Thursday, Friday, Saturday = 4, 5, 6
)
```

此外，使用`iota`关键字可以按固定规则初始化一组常量而不必每行都写一遍初始化语句。下面是一个最简单的例子，第一行的`Sunday`将会被iota置为0，其后的变量值依次加1

```go
const (
	Sunday    = iota //0
	Monday           //1
	Tuesday          //2
	Wednesday        //3
	Thursday         //4
	Friday           //5
	Saturday         //6
)
```

## 变量

使用`var`关键字声明变量。一般形式为`var identifier type`，例如

```go
var a int
```

变量也可以批量声明，称作因式分解关键字

```go
var (
	a   int
	b   bool
	str string
)
```

当一个变量被声明后，系统自动赋予它该类型的零值：int 为 0，float 为 0.0，bool 为 false，string 为空字符串，指针为 nil。

变量的声明和初始化(赋值)可以分离或结合，有以下几种情况

```go
// 声明与初始化分离
var a int
a = 15
// 声明与初始化合并
var a int = 15
// 根据初始化的变量值自动推导变量类型
var a = 15
// 函数体内的简短声明写法
a := 15
```

最后一种只能用于函数体内（main函数同样可以），前三种则任意，但一般用于全局变量，一个全局变量的批量声明如下。

```go
var (
	a = 15
	b = false
	str = "Go says hello to the world!"
	numShips = 50
	city string
)
```

同类型的多个变量可以将声明和初始化放在同一行，在同一行对多个变量初始化称作多重赋值。这一点在常量部分已经见过示例，如下

```go
//声明和初始化分离
var a, b, c int
a, b, c = 5, 6, 7
//声明和初始化合并
a, b, c := 5, 6, 7
```

而且，如果想要交换两个变量的值，可以直接使用`a, b = b, a`，为程序编写带来了极大的便利。

多重赋值中，可以使用空白标识符`_`抛弃不需要的变量值

```go
_, b = 5, 7

```

`_`也称作匿名变量，是一个只写的变量，任意类型都可以赋值给它，但我们无法使用它的值。匿名变量不会被分配内存，因此不占内存空间，多次声明也不会引起冲突。

变量的命名规则遵循驼峰命名法，即首个单词小写，每个新单词的首字母大写，例如：`numShips`。但当全局变量需要对外部包可见时，首个单词首字母同样大写(可见性原则)。

变量作用域的规则同C相同，关于值类型和引用类型的理解也和C相同，Go中引用类型包括指针、slices、maps和channel，值类型存储在栈中，引用类型存储在堆中，以便进行垃圾回收。

本文介绍Golang基本数据类型和类型转换

## 基本数据类型

go语言拥有的基本数据类型包括


- 布尔类型bool
- 数字类型：
  - 整型int，根据位数的不同包括int8, int16, int32, int64四种以及相对应的uint
  - 浮点型float，float32和float64两种
  - 复数complex，complex32和complex64两种，复数类型并不常用
- 字符类型：
  - byte，uint8的别名，完全等同
  - rune，int32是别名，完全等同
- 字符串类型string

### 1. 布尔类型

使用`bool`关键字声明，值只可以是常量 true 或 false

```go
var b bool = true
```

两个**类型相同**的值可以使用关系运算符来获得一个布尔类型的值。布尔类型的值之间也可以使用逻辑运算符来产生另一个布尔值，运算规则与其它语言相同。

类型相同是一个很严格的规定，如果值的类型是接口，那么它们必须实现了相同的接口。如果条件不满足，则必须事先进行类型转换才可以比较。

布尔值（以及任何结果为布尔值的表达式）最常用在流程控制的的条件语句中，如：if、for 和 switch 结构。

Go种的布尔值并不等于数字1和0，因此不能直接进行运算。

### 2. 数字类型

数字类型分为三种，除了常规的整型和浮点型外，还包括复数类型。

#### 1）整型

整型提供有符号和无符号两种，每一种又分别提供对应8、16、32、64bit大小的四种类型，总计八种，列表如下

| 整型                                                         | 无符号整型                                |
| ------------------------------------------------------------ | ----------------------------------------- |
| int8（-128 -> 127）                                          | uint8（0 -> 255）                         |
| int16（-32768 -> 32767）                                     | uint16（0 -> 65,535）                     |
| int32（-2,147,483,648 -> 2,147,483,647）                     | uint32（0 -> 4,294,967,295）              |
| int64（-9,223,372,036,854,775,808 -> 9,223,372,036,854,775,807） | uint64（0 -> 18,446,744,073,709,551,615） |

除此之外还提供两种不带位数的类型声明：int和uint。这两种类型的大小取决于所运行的平台处理器支持的字长，例如，在 32 位操作系统上，使用 32 位（4 个字节），在 64 位操作系统上，使用 64 位（8 个字节）。

尽管int有可能是32位，但在需要时int和int32之间也必须显式进行类型转换。

最后还有一种无符号整型uintptr，它没有指定具体的 bit 大小但被设定为足够容纳一个指针。uintptr 类型只有在底层编程时才需要，特别是Go语言和C语言函数库或操作系统接口相交互的地方。

int型是计算最快的类型，也是最常使用的类型。

#### 2）浮点型

Go 语言中没有 float 类型，没有double类型，只有 float32 和 float64。它们的算术规范由IEEE-754标准定义，该标准被所有现代的 CPU 支持。

float32 精确到小数点后 7 位，float64 精确到小数点后 15 位。通常应该优先使用 float64 类型，因为 `math` 包中所有有关数学运算的函数都会要求接收这个类型。

#### 3）复数

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

### 3. 字符类型

Go语言的字符有两种：

- byte 型，代表了 ASCII 码的一个字符。
- rune 类型，代表一个 UTF-8 字符，当需要处理中文、日文或者其他复合字符时，则需要用到 rune 类型。

严格来说，字符只是整数的特殊用例。`byte` 类型是 `uint8` 的别名，刚好一个字节，足以表示传统 ASCII 编码的字符。例如：`var ch byte = 'A'`；`rune`是`int32`的别名，四个字节，足以表示最长的UTF-8字符。

字符使用单引号括起来。

包 `unicode` 包含了一些针对测试字符的非常有用的函数（其中 `ch` 代表字符）：

- 判断是否为字母：`unicode.IsLetter(ch)`
- 判断是否为数字：`unicode.IsDigit(ch)`
- 判断是否为空白符号：`unicode.IsSpace(ch)`

这些函数返回一个布尔值。包 `utf8` 拥有更多与 rune 相关的函数。

### 4. 字符串类型

字符串是 UTF-8 字符的一个序列，这也意味着使用for-range结构遍历时字符串的单个字符类型是rune

字符串是值类型，且值不可变，即创建一个字符串后无法再次修改它的内容

Go 支持以下 2 种形式的字符串：

- 解释字符串：该类字符串使用双引号括起来，其中的转义字符将被替换，这些转义字符包括：

    - `\n`：换行符
    - `\r`：回车符
    - `\t`：tab 键
    - `\u` 或 `\U`：Unicode 字符
    - `\\`：反斜杠自身
    
- 非解释字符串：该类字符串使用反引号括起来，当使用多行字符串时使用这种形式。

`string` 类型的零值为长度为零的字符串，即空字符串 `""`。

Go种的字符串是根据长度限定的，而非特殊字符`\0`，其长度可以使用内置函数`len()`来获取，长度的基本含义是字符串在内存中所占字节的个数。

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

### 5. 类型别名

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

## 类型转换

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

比较特殊的是浮点型可以转换为整型，转换时会将小数部分去掉，只保留整数部分

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

### 1. bool与string

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

### 2. int/float与字符串

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

### 3. string与[]rune

string类型本身就是一个rune类型的切片，用for range遍历字符串可以返回每个字符，返回的字符类型是rune。但[]rune类型也经常需要转换为string，和单个rune类型相似，都可以直接进行显示类型转换，如下：

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

