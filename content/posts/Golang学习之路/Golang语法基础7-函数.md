---
title: Golang语法基础7-函数
date: 2019-11-25
lastmod: 2020-06-16
tags: [Go语法]
categories: [Golang学习之路]
slug: Golang syntax 7-function 
---

## 1. 函数声明与定义

Go中有三种类型的函数：

1. 普通的带有名字的函数
2. 匿名函数
3. 方法（Methods，在结构体部分介绍）

普通函数声明的基本格式为

```go
func functionName(param1 type1, param2 type2,...) (ret1 type1, ret2 type2,...)
```

定义就需要添加大括号和函数体

```go
func functionName(param1 type1, param2 type2,...) (ret1 type1, ret2 type2,...) {
    ...
}
```

两个括号中分别为参数列表和返回值列表，参数个数和返回值个数允许为0。左大括号必须与声明语句在同一行，流程控制部分已经见过这个规定，这是编译器强制规定。

当函数执行到代码块最后一行，也就是最后一个`}`之前，或者执行到`return`语句的时候就会退出。

main函数是每个程序必须包含的，一般来说是启动后第一个执行的函数，但如果有init()函数会先执行该函数。

main函数既没有参数，也没有返回值，如果添加了两者中任一者，都会引发构建错误。main函数一旦返回就表示程序已成功执行并立即退出。同样，init()函数也没有返回值

## 2. 函数调用

同一个包内，可以直接使用函数名调用该函数，不同包内，需要添加包名，如下所示

```go
pack1.functionName(arg1, arg2, ..., argn)
```

`pack1`是包名，`functionName`是该包中的一个函数，括号里是传入的实参。一个简单的调用其它函数的例子如下

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
//Output:
In main before calling greeting
In greeting: Hi!!!!!
In main after calling greeting
```

Go中不允许函数重载

## 3. 函数参数与返回值

除了mian()和init()函数外，其它函数都可以拥有参数和返回值。而且任意一个有返回值的函数都必须以`return`或`panic`语句结尾，`return`可以返回多个值，多值返回是Go的一大特性。

### 3.1 参数传递类型

Go中的参数传递类型有两种：按值传递和按引用传递。

Go默认使用按值传递来传递参数，也就是传递参数的副本，因此参数在函数中被更改后不会影响原值。如果希望函数运行的同时改变原变量的值，应该添加取地址符&，传递变量的指针，也就是按引用传递，按引用传递时，传入的是指针的副本，但指向的值依然是原变量。

函数调用时，切片、映射、接口、通道这些引用类型默认使用按引用传递

几乎在任何情况下，按引用传递的消耗都比按值传递小

按引用传递可以直接修改外部变量的值，因此被修改的变量不再需要使用`return`返回

### 3.2 命名参数

函数定义时，形参一般都有名字，不过也可以定义没有形参名的函数，只有形参类型，比如` func f(int, int, float64) `，返回值同样如此。只有类型的返回值称为非命名返回值，有名字的返回值称为命名返回值。一个例子如下

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
//Output:
num = 10, 2x num = 20, 3x num = 30    
num = 10, 2x num = 20, 3x num = 30 
```

命名返回值会被初始化为相应类型的零值，返回时只需要一条简单的不带参数的return语句（带参数也不会出错）。

当需要返回多个非命名返回值时，需要使用括号包围，如`(int, int)`，但对命名返回值，即使只有一个返回值，也要用括号包围。

### 3.3 空白符

空白符`_`用来匹配不需要的返回值，然后丢弃掉，之前已经介绍过。

### 3.4 变长参数

如果函数最后一个参数是`...type`的形式，那么函数就可以处理一个变长的参数，这个长度可以是0，这样的函数称为变参函数

```go
func myFunc(a, b, arg ...int) {}
```

变长参数的本质是一个切片，如下例

```go
func Greeting(prefix string, who ...string)
Greeting("hello:", "Joe", "Anna", "Eileen")
```

变量`who`的值为 `[]string{"Joe", "Anna", "Eileen"} `

如果参数本身就存在一个切片类型中，比如切片`slice1`，则可以通过`slice1...`的形式传递参数，之前的切片部分使用append函数时已经这样使用过

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
//Output:
The minimum is: 0
The minimum in the slice is: 1
```

一个接受变长参数的函数可以将这个参数作为其它函数的参数进行传递

```go
func F1(s ...string) {
	F2(s...)
	F3(s)
}

func F2(s ...string) { }
func F3(s []string) { }
```

### 3.5 函数作为参数

函数可以作为其它函数的参数进行传递，然后在其它函数内调用执行，只要函数返回值个数、返回值类型和返回值顺序同调用函数的形参列表定义相同，称之为回调。下面是一个将函数作为参数的简单例子 

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
//Output:
The sum of 1 and 2 is: 3
```

## 4. 内置函数

Go语言拥有一些不需要导入就可以使用的内置函数，之前已经接触过一些，比如len, cap, append，以下是内置函数列表

| 名称               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| close              | 用于管道通信                                                 |
| len、cap           | len 用于返回某个类型的长度或数量（字符串、数组、切片、map 和管道）；cap 是容量的意思，用于返回某个类型的最大容量（只能用于切片和 map） |
| new、make          | new 和 make 均是用于分配内存：new 用于值类型和用户定义的类型，如自定义结构，make 用于内置引用类型（切片、map 和管道）。它们的用法就像是函数，但是将类型作为参数：new(type)、make(type)。new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针。它也可以被用于基本类型：`v := new(int)`。make(T) 返回类型 T 的初始化之后的值，因此它比 new 进行更多的工作 |
| copy、append       | 用于复制和连接切片                                           |
| panic、recover     | 两者均用于错误处理机制                                       |
| print、println     | 底层打印函数，在部署环境中建议使用 fmt 包                    |
| complex、real imag | 用于创建和操作复数                                           |

## 5. 匿名函数与闭包

匿名函数是类似 `func(x, y int) int { return x + y }` 这样没有名字的函数。

匿名函数可以被直接调用，下面是一个计算从 1 到 1 百万整数的总和的匿名函数。表示参数列表的第一对括号必须紧挨着关键字 `func`，因为匿名函数没有名称。花括号 `{}` 涵盖着函数体，最后的一对括号表示对该匿名函数的调用。 

```go
func() {
	sum := 0
	for i := 1; i <= 1e6; i++ {
		sum += i
	}
}()
```

另外，匿名函数可以像其它函数一样接受参数，下例展示了如何传递参数到匿名函数中

```go
func (u string) {
	fmt.Println(u)
	…
}(v)
```

还应该知道的，匿名函数可以被赋值给某个变量，如`fplus := func(x, y int) int { return x + y }`，这样函数的地址就保存到了变量中，之后可以通过变量名对函数进行调用：`fplus(3, 4)`

所谓闭包就是函数及其引用环境的组合，这么说比较难理解，举个例子

```go
func f(i int) func() int {
    return func() int {
        i++
        return i
    }
}
```

在这里例子里，返回值是一个函数，这个函数本身没有定义变量，而是引用了它所在环境的变量 i，这就形成了一个闭包。从这里可以看出，闭包与匿名函数息息相关，因为匿名函数被用作函数返回值非常合适。下面是一个完整的例子

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
//Output:
1 - 21 - 321
```

从这里例子中我们注意到 x 的值是不断累加的，这也就意味着闭包函数其实会保存并积累其中的变量的值，不管外部函数退出与否，它都能够继续操作外部函数中的局部变量。 这里可以理解为函数被赋值给 f 之后，其实将整个闭包包括环境都赋值给了 f，变量 f 的生存周期内，其值是不变的，所以结果才会累积。

这种返回值为另一个函数的函数的形式也被称之为工厂函数，在需要创建一系列相似的函数的时候非常有用。下面的函数演示了如何动态返回追加后缀的函数： 

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

现在可以生成如下函数

```go
addBmp := MakeAddSuffix(".bmp")
addJpeg := MakeAddSuffix(".jpeg")
```

然后调用它们

```go
addBmp("file") // returns: file.bmp
addJpeg("file") // returns: file.jpeg
```

## 6. defer和追踪

关键字 defer 是 Go 中一个非常有用的特性，作用是将某个语句或函数推迟到函数返回之前执行。准确的说，defer 的执行时机有三种：

1. 包含 defer 语句的函数返回前
2. 包含 defer 语句的函数执行到末尾
3. 所在的 goroutine 发生 panic 时

一个例子如下

```go
func main() {
    defer fmt.Println("Fourth")
    fmt.Println("First")
    fmt.Println("Third")
}
//Output:
First
Third
Fourth
```

defer 语句中调用的函数参数的值在 defer 语句被定义时就确定了，如下例

```go
i := 1
defer fmt.Println("Deferred print:", i)
i++
fmt.Println("Normal print:", i)
// Output:
Normal print: 2
Deferred print: 1
```

但与匿名函数结合起来后，变量的值在函数运行时才会确定

```go
func f1() (r int) {
    r = 1
    defer func() {
        r++
        fmt.Println(r)
    }()
    r = 2
    return
}

func main() {
    f1()
}
// Output:
3
```

上例中出现了 return 语句，defer 与 return 的执行顺序比较复杂，这里要先理解两件事

1. defer 函数执行时机是外层函数设置返回值之后，即将返回之前
2. return xxx 操作并不是原子的

下面的例子中， return 0 实际上可以拆分为 r = 0; return 两条语句，因此输出是1不是0

```go
func f1() (r int) {
    defer func() {
        r++
    }()
    return 0
}
func main() {
    fmt.Println(f1())
}
```

来一个更复杂的例子

```go
func double(x int) int {
    return x + x
}

func triple(x int) (r int) {
    defer func() {
        r += x
    }()
    return double(x)
}

func main() {
    fmt.Println(triple(3))
}
// Output:
9
```

上面的例子实际上等价于

```go
func triple(x int) (r int) {
    r = double(x)
    func() {
        r += x
    }()
    return
}
```

多个 defer 同时使用时，以逆序执行，即后进先出

```go
func f() {
	for i := 0; i < 5; i++ {
		defer fmt.Printf("%d ", i)
	}
}
//Output:
4 3 2 1 0
```

defer 关键字一般用于释放某些已分配的资源或在函数执行完进行一些收尾工作，比如

1. 关闭文件流

    ```go
   //open a file
   defer file.Close()
   ```

2. 解锁一个加锁的资源

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
   //open a database connection
   defer disconnectFromDB()
   ```

## 7. 编写规范

Go是编译型的语言，因此函数的编写顺序无关紧要，但鉴于可读性的需求，最好遵循一定的编程规范，我这里采用的是Uber开源在github的编码规范，有两条主要规则

1. 函数应按粗略的调用顺序排序

2. 同一文件中的函数应按接收者排序，意即可被外部访问的函数（参考可见性规则）应放在前面，普通工具函数放在后面。另外，在类型定义（结构体、接口等）后，可被外部访问的函数前，可能会出现类似于`newXYZ()`这样的新建某个类型的函数。一个简单的例子如下

   ```go
   type something struct{ ... }
   
   func newSomething() *something {
       return &something{}
   }
   
   func (s *something) Cost() {
     return calcCost(s.weights)
   }
   
   func calcCost(n []int) int {...}
   ```

最后，main函数放在所有函数的最后。