# Golang语法基础9-接口与反射


## 1. 接口

上一篇谈到类型 T（或 *T）上的所有方法的集合叫做类型 T（或 *T）的方法集，接口可以用来定义方法集，但是这种定义是抽象的，不包含方法的代码实现，接口中也不能包含变量。

接口定义的方式如下

```go
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
```

不像大多数面向对象编程语言，Go中可以为定义好的接口声明一个变量，叫做接口值：`var ai Namer`，其本质是一个指针，未初始化时其值为nil。

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

### 1.1 接口类型值

上例中可以看到，不仅接口的值为nil，其类型也是nil，这里做一下详细说明。

变量的类型在声明时指定，且不能改变，就称为静态类型。定义的接口本身属于静态类型，但声明后接口的值却是是动态的，由于接口的本质是一个指针，这个指针会指向实现了接口类型的值。如下例，变量i本身是静态类型Namer，但却先后指向实现了接口类型的St1和St2两个空结构体

```go
type Namer interface {
	Name()
}

type St1 struct{}
func (St1) Name() {}
type St2 struct{}
func (St2) Name() {}

func main() {
	var i Namer = St1{}
	fmt.Printf("type is %T\n", i)
	fmt.Printf("value is %v\n", i)
	i = St2{}
	fmt.Printf("type is %T\n", i)
	fmt.Printf("value is %v\n", i)
}
//Output:
type is main.St1
value is {}
type is main.St2
value is {}
```

接口的类型值想要为nil，当且仅当动态值和动态类型都为nil。

### 1.2 接口实现

方法集的实现基于某种类型(比如结构体)，这是上一节讲到的，因此接口实现的过程就是方法集中方法实现的过程。实现了接口的变量可以赋值给接口值，如下例

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

接口被隐式地实现，因此类型不需要显式声明它实现了某个接口。

一个不包含任何方法的接口称为空接口，形如：interface{}。同时因为空接口不包含任何方法，所以任何类型都默认实现了空接口。

举个例子，fmt 包中的 Println() 函数，可以接收多种类型的值，比如：int、string、array等。这是因为它的形参就是接口类型，可以接收任意类型的值。

```go
func Println(a ...interface{}) (n int, err error) {}
```

多个类型可以实现同一个接口，如下例

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

一个类型也可以实现多个接口，这意味着类型除实现某个接口的方法外，还可以实现其它的方法

```go
type Shape interface {
	Area() float32
}

type Object interface {
	Perimeter() float32
}

type Circle struct {
	radius float32
}

func (c Circle) Area() float32 {
	return math.Pi * (c.radius * c.radius)
}

func (c Circle) Perimeter() float32 {
	return 2 * math.Pi * c.radius
}

func main() {
	c := Circle{3}
	var s Shape = c
	var p Object = c
	fmt.Println("area: ", s.Area())
	fmt.Println("perimeter: ", p.Perimeter())
}
//Output:
area:  28.274334
perimeter:  18.849556
```

### 1.3 接口嵌套

Go中的接口可以包含一个或多个其它接口，这相当于直接将这些内嵌接口的方法列举在外层接口中。下例中`File`接口包含了`ReadWrite`和`Lock`接口的所有方法

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

### 1.4 类型断言

一个接口类型的变量可以包含任何类型的值，因此需要有一种方式来检测其动态类型，这种方式就是类型断言

```go
v := varI.(T)
```

varI是一个接口变量，T是待检测类型，这一语句的作用是检测varI的动态类型是否和T一致，实质是将varI转换为T类型的值

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

不过，也有可能类型断言是无效的，为了更安全的使用类型断言，可以使用如下方式，在上例中也有说明

```go
if v, ok := varI.(T); ok {  // checked type assertion
    Process(v)
    return
}
// varI is not of type T
```

不确定接口变量当前类型时，就需要进行多次断言，这时使用switch最为简便，要求是所有`case`语句中列举的类型(nil除外)都必须实现对应的接口。

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

类型断言还有一种用法，就是测试它是否实现了某个接口，如下

```go
type Stringer interface {
    String() string
}

if sv, ok := v.(Stringer); ok {
    fmt.Printf("v implements String(): %s\n", sv.String()) // note: sv, not v
}
```

### 1.5 使用指针接收者和值接收者实现接口

虽然上一节中提到作用于变量的方法不区分变量是指针还是值，但是遇到接口时，会变得稍微复杂一点。

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

Go语言的反射系统无法获取到一个可执行文件空间或者一个包中的所有类型信息，仅仅是在运行时通过reflect包来访问指定的类型信息。

 https://mp.weixin.qq.com/s/qJVfEWngSDg3It3UytPuxA 

### 2.1 方法和类型的反射

变量的最基本信息是类型和值，反射包的`Type`用来表示一个Go类型，反射包的`Value`为Go值提供了反射接口。

两个简单的函数，`reflect.TypeOf`和`reflect.ValueOf`，返回被检查对象的类型和值。例如，x被定义为`var x float64 = 3.4`，那么`reflect.TypeOf(x)`返回`type: float64`，`reflect.ValueOf(x)`返回`value: 3.4`

其本质是检查接口的动态类型和动态值，可从以下函数签名看出

```go
func TypeOf(i interface{}) Type
func ValueOf(i interface{}) Value
```

因此，使用Valueof转换获得的值，依然拥有自己的类型和值，反射包的Value有不少方法都可以作用于它，比如`kind`方法返回一个常量来表示类型，`Type`方法也返回值的类型，`Int`和`Float`等方法可以获取存储在内部的值，`Interface`方法可以还原接口值。示例如下

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


