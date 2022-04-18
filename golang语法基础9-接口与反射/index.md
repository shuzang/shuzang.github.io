# Golang语法基础9-接口与反射


本篇介绍 Golang 中的接口与反射。

<!--more-->

## 1. 接口

类型 T（或 *T）上的所有方法的集合叫做类型 T（或 *T）的方法集，我们可以使用接口来定义方法集，但是这种定义是抽象的，不包含方法的代码实现，接口中也不能包含变量。

接口定义的基本格式如下

```go
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
```

### 1.1 接口理解

接口可以是空的，也就是不包含任何方法，空接口的底层实现是一个名为 eface 的结构体

```go
type eface struct {
    _type *_type
    data  unsafe.Pointer
}

type _type struct {
    size       uintptr // type size
    ptrdata    uintptr // size of memory prefix holding all pointers
    hash       uint32  // hash of type; avoids computation in hash tables
    tflag      tflag   // extra type information flags
    align      uint8   // alignment of variable with this type
    fieldalign uint8   // alignment of struct field with this type
    kind       uint8   // enumeration for C
    alg        *typeAlg  // algorithm table
    gcdata    *byte    // garbage collection data
    str       nameOff  // string form
    ptrToThis typeOff  // type for pointer to this type, may be zero
}
```

非空接口的底层实现与空接口不同，是一个名为 iface 的结构体，非空接口中定义的方法的具体实现都放在 itab.fun 变量中

```go
type iface struct {
    tab  *itab
    data unsafe.Pointer
}

// layout of Itab known to compilers
// allocated in non-garbage-collected memory
// Needs to be in sync with
// ../cmd/compile/internal/gc/reflect.go:/^func.dumptypestructs.
type itab struct {
    inter  *interfacetype
    _type  *_type
    link   *itab
    bad    int32
    inhash int32      // has this itab been added to hash?
    fun    [1]uintptr // variable sized
}

type interfacetype struct {
    typ     _type
    pkgpath name
    mhdr    []imethod
}

type imethod struct {   //这里的 method 只是一种函数声明的抽象，比如  func Print() error
    name nameOff
    ityp typeOff
}
```

itab 可以简单理解为 接口类型+具体类型，interfacetype 是接口类型，包含包路径、方法等信息。_type 则是具体类型，空接口也包含这个字段。

关于接口值的理解是一件很重要的事，但是比较复杂。我们通过了解接口的内存布局来理解接口的本质，从而理解接口值。先看一个例子

```go
type Stringer interface {
    String() string
}

type Binary uint64

func (i Binary) String() string {
    return strconv.Uitob64(i.Get(), 2)
}

func (i Binary) Get() uint64 {
    return uint64(i)
}

func main() {
    b := Binary{}
    s := Stringer(b)
    fmt.Print(s.String())
}
```

对比非空接口的底层实现，发现接口在内存中实际上由两个成员组成，如下图，tab 指向虚表，data 指向实际引用的数据

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20191126_20150126190937373.png)

虚表描绘了实际的类型信息及接口的方法集，具体有哪些部分我们以及从 itab 结构体中看到了。注意在这里是 Stringer 接口的函数指针列表，而不是实际类型 Binary 的函数指针列表，只有在运行时遇到 s := Stringer(b) 这样的语句，才会生成接口对应的 Binary 类型的虚表。这样，当判定一种类型是否满足某个接口时，只需要判断类型的方法集是否完全包含接口的方法集即可。

### 1.2 接口值

我们可以将一个变量声明为接口类型，`var ai Namer`，未初始化时，接口值为 nil。

```go
type Shape interface {
	Area() float32
}

func main() {
	var s Shape
	fmt.Println("value of s is", s)
	fmt.Printf("type of s is %T\n", s)
}
//Output:
value of s is <nil>
type of s is <nil>
```

我们可以理解接口变量的值为 nil，那么为什么类型也是 nil 呢？上一小节我们理解接口的时候知道，接口其实就是定义在结构体中的两个指针，未初始化时这两个指针都是 nil，因此接口值是 nil，接口类型也是nil。

当我们初始化接口变量后，两个指针都有了具体指向的值，此时接口的值就是接口指向的类型的值，接口的类型就是实现了接口的变量类型。如下例，变量 i 本身是接口类型 Namer，先后指向实现了接口类型的 St1 和 St2 两个空结构体，其值和类型就发生了变化。

```go
type Namer interface {
	Name()
}

type St1 struct {
	a int
}

func (st1 St1) Name() {
	fmt.Println(st1.a)
}

type St2 struct{}

func (St2) Name() {}

func main() {
	var i Namer = St1{2}
	fmt.Printf("type is %T\n", i)
	fmt.Printf("value is %v\n", i)
	i = St2{}
	fmt.Printf("type is %T\n", i)
	fmt.Printf("value is %v\n", i)
}
//Output:
type is main.St1
value is {2}
type is main.St2
value is {}
```

### 1.3 接口的实现

接口的实现是一种隐式的实现，只要某个类型的方法集完全包含接口的方法集，就属于实现了该接口，实现了接口的变量可以赋值给接口值。

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
//Output:
The square has area: 25.000000
```

空接口可以声明为 interface{}，同时因为空接口不包含任何方法，所以任何类型都默认实现了空接口。举个例子，fmt 包中的 Println() 函数，可以接收多种类型的值，比如：int、string、array等。这是因为它的形参就是接口类型，可以接收任意类型的值。

```go
func Println(a ...interface{}) (n int, err error) {}
```

多个类型可以实现同一个接口，如下例，一个类型也可以实现多个接口。

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
//Output:
Looping through shapes for area ...
Shape details:  {5 3}
Area of this shape is:  15
Shape details:  &{5}
Area of this shape is:  25
```

### 1.4 接口嵌套

Go 中的接口可以包含一个或多个其它接口，这相当于直接将这些内嵌接口的方法列举在外层接口中。下例中`File`接口包含了`ReadWrite`和`Lock`接口的所有方法

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

### 1.5 类型断言

如果一个变量是接口变量，实际上其值的类型是不确定的，我们使用类型断言来检测值的具体类型

```go
v := varI.(T)
```

varI是一个接口变量，T是待检测类型，这一语句的作用是检测 varI 的动态类型是否和 T 一致，实质是将 varI 转换为 T 类型的值

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
//Output:
The type of areaIntf is: *main.Square
areaIntf does not contain a variable of type Circle
```

类型断言可能失败，为了更安全的使用类型断言，使用如下的方式

```go
if v, ok := varI.(T); ok {  // checked type assertion
    Process(v)
    return
}
// varI is not of type T
```

由于断言是一个比较的过程，因此需要多次尝试，使用 switch 语句最为简便，不过要求所有 `case` 语句中列举的类型(nil除外)都必须实现对应的接口。

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
//Output:
Type Square *main.Square with value &{5}
```

类型断言还有一种反向用法，就是测试它是否实现了某个接口，如下

```go
type Stringer interface {
    String() string
}

if sv, ok := v.(Stringer); ok {
    fmt.Printf("v implements String(): %s\n", sv.String()) // note: sv, not v
}
```

所以我们可以意识到，断言格式 `varI.(T)` 中的 `varI` 可以是任意变量，`T` 是任意类型，断言的实质就是将变量转换为 `T` 类型的值，然后进行比较。

### 1.6 使用指针接收者和值接收者实现接口

虽然作用于变量的方法不区分变量是指针还是值，但是遇到接口时，会变得稍微复杂一点。

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

在 `lst` 上调用 `CountInto` 时会导致一个编译器错误，因为 `CountInto` 需要一个 `Appender`，而它的方法 `Append` 只定义在指针上。 在 `lst` 上调用 `LongEnough` 是可以的，因为 `Len` 定义在值上。

在 `plst` 上调用 `CountInto` 是可以的，因为 `CountInto` 需要一个 `Appender`，并且它的方法 `Append` 定义在指针上。 在 `plst` 上调用 `LongEnough` 也是可以的，因为指针会被自动解引用。

Go语言规范中接口方法集的调用规则为：

- 类型 *T 的可调用方法集包含接受者为 *T 或 T 的所有方法集
- 类型 T 的可调用方法集包含接受者为 T 的所有方法
- 类型 T 的可调用方法集不包含接受者为 *T 的方法

## 2. 反射

反射是指在程序运行期间对程序本身进行访问和修改的能力。程序在编译时，变量被转换为内存地址，变量名不会被编译器写入到可执行部分，在运行程序时，程序无法获取自身的信息。

支持反射的语言可以在程序编译期将变量的反射信息，如字段名称、类型信息、结构体信息等整合到可执行文件中，并给程序提供接口访问反射信息，这样就可以在程序运行期获取类型的反射信息，并且有能力修改它们。

Go 语言的反射系统无法获取到一个可执行文件空间或者一个包中的所有类型信息，仅仅是在运行时通过 reflect 包来访问指定的类型信息。

 https://mp.weixin.qq.com/s/qJVfEWngSDg3It3UytPuxA 

### 2.1 方法和类型的反射

Go 的反射包中比较重要和常用的有三个类型：`Kind`，`Type`，`Value`，其中 `Type`用来表示一个Go类型，`Value`为Go值提供了反射接口。

`reflect.TypeOf` 和 `reflect.ValueOf` 分别是定义在这两者之上的方法，函数原型如下

```go
func TypeOf(i interface{}) Type
func ValueOf(i interface{}) Value
```

其本质是检查接口的动态类型和动态值，一般用来返回被检查对象的类型和值。例如，x被定义为`var x float64 = 3.4`，那么`reflect.TypeOf(x)`返回`type: float64`，`reflect.ValueOf(x)`返回`value: 3.4`

因此，使用ValueOf转换获得的值，依然拥有自己的类型和值，反射包的Value有不少方法都可以作用于它，比如`kind`方法返回一个常量来表示类型，`Type`方法也返回值的类型，`Int`和`Float`等方法可以获取存储在内部的值，`Interface`方法可以还原接口值。示例如下

```go
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
//Output:
type: float64
value: 3.4
type: float64
kind: float64
value: 3.4
3.4
value is 3.40e+00
3.4
```

由于x是一个float64类型的值，因此使用v.Float()获取它的实际值，其它类型的值可以使用`Int(), Bool(), Complex, String()`

`Kind`方法总是返回底层类型，如

```go
type MyInt int
var m MyInt = 5
v := reflect.ValueOf(m)
```

`v.Kind()`将返回`reflect.Int`

### 2.2 通过反射修改值

无论如何，利用反射修改值总是应该极为谨慎的，而且，不是所有的反射值都可以修改，是否可修改可通过`CanSet()`方法检测。`v := reflect.ValueOf(x)`函数传递了一个x的拷贝，对拷贝的修改无法影响原值，因此，若想修改原值，应传递指针，`v = reflect.ValueOf(&x)`。此时v的类型是`*float64`，但依然不可修改，还需要最后一步，使用`Elem()`方法，至此才能使用`SetFloat()`修改原值，一个完整的例子如下

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
//Output:
settability of v: false
type of v: *float64
settability of v: false
The Elem of v is:  <float64 Value>
settability of v: true
3.1415
<float64 Value>
```

### 2.3 结构体的反射

对结构体进行反射，需要使用`NumField()`方法返回结构内的字段数量，然后通过for循环用索引获得每个字段的值`Field(i)`，最后可以使用索引调用签名在结构体上的方法：`Method(i).Call(nil)`

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
//Output:
main.NotknownType
struct
Field 0: Ada
Field 1: Go
Field 2: Oberon
[Ada - Go - Oberon]
```

但是尝试修改会得到错误，结构体中只有被导出字段才是可设置的。

## 3. 总结：面向对象

面向对象最重要的三个方面：封装、继承和多态，在Go中都可以寻找到替代的实现方式

- 封装(数据隐藏)：即Go中的可见性规则，不同于别的OO语言中的4种访问性，Go只有两种
- 继承： 用组合或内嵌实现，结构体一篇中谈到过这个。
- 多态：用接口实现。某个类型的实例可以赋给它所实现的任意接口类型的变量。类型和接口是松耦合的，并且多重继承可以通过实现多个接口实现。Go 接口不是 Java 和 C# 接口的变体，而且接口间是不相关的，并且是大规模编程和可适应的演进型设计的关键 

### 3.1 组合实现继承

组合的含义是包含所需功能的具名字段

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
//Output:
1 - Yes we can!
2 - After me the world will be a better place!
```

### 3.2 内嵌实现继承

内嵌的含义是匿名的实现所需功能

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
//Output:
Barak Obama
Log:{1 - Yes we can!
2 - After me the world will be a better place!}
```

内嵌看起来更简单一点，因此最合适的办法就是利用内嵌创建一些小的、可复用的类型作为工具箱，来被其它类型调用。

### 3.3 多重继承

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
//Output:
Our new CameraPhone exhibits multiple behaviors...
It exhibits behavior of a Camera: Click
It works like a Phone too: Ring Ring
```


