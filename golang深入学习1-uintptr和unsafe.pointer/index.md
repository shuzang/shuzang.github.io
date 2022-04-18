# Golang深入学习1-uintptr 和 Unsafe.Pointer


本篇介绍 Go 中的指针、uintptr 和 unsafe.Pointer 三个概念。

<!--more-->

## 1. 指针

Go 中指针的概念与 C 相同，只是指针本身不能进行运算。

任何一个变量在运行时都有一个地址，这个地址代表它在内存中的位置，在变量名前使用取地址符 `&` 可以获取该变量的内存地址，比如

```go
ptr := &v 
```

其中 v 是一个变量，ptr 用来保存它的内存地址，ptr 就是指针。当然，准确的来说，ptr 是一个指针变量，作为变量，仍然是用来保存数据，只不过这里它特别用来保存一个内存地址而已。

一个指针变量需要使用 `*` 来声明，如下

```go
var ptr *int
```

`*` 代表 ptr 保存一个地址，而 `int` 代表这个地址指向的内存所保存的数据是 int 类型。这里要注意，*int 和 *string 是不同的指针类型，不能相互赋值，如下例，指针的类型与它所指向的内存地址保存的数据类型有关。

```go
func main() {
	var a *int
	b := "hello"
	a = &b
}
// Output
.\main.go:6:4: cannot use &b (type *string) as type *int in assignment
```

指针在声明但未赋值时其值为 nil，不指向任何内存

```go
func main() {
	var a *int
	fmt.Println(a)
}
// Output
<nil>
```

将一个变量的地址赋值给指针后，进行使用要利用 `*` 符号，否则其值就是所保存的内存地址

```go
func main() {
	var a *int
	b := 10
	a = &b
	fmt.Println(a, *a)
}
// Output
0xc000012098 10
```

## 2. unsafe.Pointer

我们已经知道不同的指针类型间不能相互赋值，另外，它们也不能进行类型转换

```go
func main() {
	var a *int
	b := 10
	a = &b
	c := (*float64)(a)
}
// Output
.\main.go:7:17: cannot convert a (type *int) to type *float64
```

unsafe.Pointer 是特别定义的一种指针类型，它可以包含任意类型变量的地址，也就是说无论 int、float64 还是其它类型的变量，内存地址都可以交给它保存。

```go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	b := 10
	var a *int = &b
	c := unsafe.Pointer(a)
	fmt.Println(c)
}
// Output
0xc000012098
```

从上面的程序注意到两点

1. unsafe.Pointer 来自 unsafe 包，不是内置类型；
2. unsafe.Pointer 一般作为指针类型转换的桥梁使用

unsafe.Pointer 本质是一个 *int，在 unsafe 包中定义如下

```go
type ArbitraryType int
type Pointer *ArbitraryType
```

unsafe.Pointer 的使用说明如下

1. 任意类型的指针可以转换为一个Pointer类型值
2. 一个Pointer类型值可以转换为任意类型的指针
3. 一个uintptr类型值可以转换为一个Pointer类型值
4. 一个Pointer类型值可以转换为一个uintptr类型值

下面是一个使用 unsafe.Pointer 作为桥梁进行类型转换的例子

```go
func main() {
	b := 10
	var a *int = &b
	c := (*float64)(unsafe.Pointer(a))
}
```

## 3. uintptr

uintptr 确确实实是 Go 的基本类型，属于整型的一种，被设计为足够容纳一个指针。在 Go 中指针是不允许进行运算的，但 uintptr 可以，所以它的意义在于将其它类型的指针转换成它进行运算，然后再转换回原本的类型。

```go
type User struct {
	Name string
	Age int
}

func main() {
	u := new(User)
	name :=(*string)(unsafe.Pointer(u))
	*name = "test"

	age := (*int)(unsafe.Pointer(uintptr(unsafe.Pointer(u)) + unsafe.Offsetof(u.Age)))
	*age = 18
	fmt.Println(u)
}
```

思路: 如果想对 Name 和 Age 进行赋值，那首先应该先拿到相应的地址，然后对地址内容修改，所以我们可以分别定义一个 string 和  int 分别指向 Name 和 Age的地址
对于 u 这个结构体，u 的首地址就是 Name 的地址，但是 u 是结构体指针，所以只需要将u转换成 *string 即可
对于 Age，我们已经拿到了Name的地址，在此基础上进行偏移即可，也就是 unsafe.Offsetof(u.Age) 的偏移量， 但是 `Pointer`不能进行指针运算，随意需要将 `Pointer` 转换成 `uintptr`

## 4. 说明

这里有两点要说明。

第一，理解 unsafe.Pointer 和 uintptr 是因为 Go 源码中很多实现都使用了这两个概念，比如接口类型的定义

```go
type iface struct {
	tab  *itab
	data unsafe.Pointer
}

type itab struct {
	inter *interfacetype 
	_type *_type 
	hash  uint32 
	_     [4]byte
	fun   [1]uintptr 
}
```

第二，使用 unsafe.Pointer 和 uintptr 要非常注意，因为它们绕过了类型系统直接操作内存地址。
