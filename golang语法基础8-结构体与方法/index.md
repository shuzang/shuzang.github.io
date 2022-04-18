# Golang语法基础8-结构体与方法


结构体是一种非常重要的结构，用到的地方非常多，在 Go 中，结构体还是实现面向对象编程的基础。

<!--more-->

## 1. 结构体

### 1.1 定义与初始化

结构体定义方式如下：

```go
type identifier struct {
    field1 type1
    field2 type2
    ...
}
```

大括号中的每一行称为一个字段，每个字段都有一个类型和一个名字，在一个结构体中，字段名必须是唯一的。结构体名和字段名的命名遵循可见性规则，即使用首字母的大小写来表示可导出和不可导出。但是需要注意，一个可导出的结构体类型中可以存在不可导出的字段。

结构体的字段可以是任意类型，甚至可以是结构体本身、函数或者接口。一个简单的结构体定义示例如下

```go
type T struct {
    a,b int
}
```

结构体是自定义数据类型，因此我们可以向基本数据类型一样声明/定义一个结构体类型的变量，声明时会分配内存并默认使用每个字段类型的零值来初始化。我们也可以手动初始化一个结构体，使用点号符给字段赋值，示例如下。另外，访问结构体内字段的值时同样使用点号符，这种使用点号符赋值和获取字段值的方式叫做**选择器(selector)**，

```go
var s T
s.a = 5
s.b = 8
```

由于结构体也是值类型，使用new函数创建。注意，使用 new 得到的 t 是指向结构体的指针。

```go
var t *T = new(T)
t := new(T) // 简单方便地写法，最常用
```

至此我们注意到，使用结构体时我们可能遇到两种类型：结构体类型和结构体指针类型，这两种类型都可以通过选择器的方式来使用，如下，v.i 和 p.i 都可以得到正确的值，在理解的时候可以想象底层对结构体指针 p 自动做了解引用，如 (*p).i。

```go
type myStruct struct { i int }
var v myStruct    // v是结构体类型变量
var p *myStruct   // p是指向一个结构体类型变量的指针
v.i
p.i
(*p).i
```

除使用选择器初始化结构体字段外，一种更简短更常用的结构体初始化方法如下

```go
ms := struct1{10, 15.5, "Chris"}  //结构体类型
ms := &struct1{10, 15.5, "Chris"}  //结构体指针类型
ms := &struct1{f1:15.5，i1:10}  //括号内声明字段名，这样可以不按定义的字段顺序，甚至省略部分字段
```

其中第二行称为混合字面量语法，但底层仍然会调用`new()`，因此与使用 `new()` 初始化是等同的。以`type Point struct {x,y int}`为例，这几种初始化方式的内存布局如下

<img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20191126_3EkqRU.jpg" alt="结构体内存布局" style="zoom: 80%;" />

从上图可以看出，结构体和它包含的数据在内存中是以连续块的形式存在的，即使结构体中嵌套其它的结构体，同样如此。

```go
type Rect1 struct {Min, Max Point }
type Rect2 struct {Min, Max *Point }
```

![嵌套结构体内存布局](https://github.com/unknwon/the-way-to-go_ZH_CN/raw/master/eBook/images/10.1_fig10.2.jpg?raw=true)

一个使用结构体的完整例子如下

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
//Output:
The int is: 10
The float is: 15.500000
The string is: Chris
&{10 15.5 Chris}
```

### 1.2 结构体标签

实际上，一个完整的结构体定义，在字段名和类型外，还有一个标签(tag)部分。标签是一个字符串，用来对字段进行一定的说明，对程序功能没有太大的作用，因此前面才没有介绍。正如我们说的，它的主要作用就是对字段进行说明，标签只有包 `reflect` 能获取。

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
//Output:
An important answer
The name of the thing
How much there are
```

### 1.3 匿名字段与内嵌结构体

结构体的字段名其实和变量很相似，不需要时也可以用空白符 `_` 代替，但实际上，也可以直接省略，即整个字段只有类型，此时类型就是字段名，这种字段叫做**匿名字段**。如下例，通过类型 `t.float32` 来获取存储在匿名字段中的数据，也因为这种调用方式，一个结构体中对每一种数据类型只能有一个匿名字段。

```go
package main

import "fmt"

type T struct {
	a       int
	float32 // anonymous field
}

func main() {
	t := T{6, 7.5}
	fmt.Println(t.float32)
}

//Output:
{6 7.5}
```

由于结构体本身也是一种数据类型，因此也可以作为匿名字段使用，称为**内嵌结构体**。通过内嵌结构体可以实现 OO 编程种的继承。

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
//Output:
1 2 3 4
{1 2}
```

使用内嵌结构体的时候，很可能会出现命名冲突（继承来的字段名和当前结构体的某个字段名相同），这种情况下外层的名字会覆盖内层的名字，但两者的内存空间都会保留，下例中`d.b`的调用不会出错，指的是float32，而不是`B.b`，进行内层调用可以使用`d.B.b`

```go
type B struct {a, b int}
type D struct {B; b float32}
var d D
```

但下面这种情况，`c.a`的调用会导致编译器错误,只能由程序员手动修改

```go
type A struct {a int}
type B struct {a, b int}

type C struct {A; B}
var c C
```

### 1.4 结构体工厂

可以为结构体定义一个工厂来创建结构体实例，工厂的名字通常以new或New开头，这是一种很常用的方法。假设定义了如下File结构体类型

```go
type File struct {
    fd int //文件描述符
    name string //文件名
}
```

下面是为File结构体创建的工厂，返回一个指向结构体的指针

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }

    return &File{fd, name}
}
```

然后这样调用它

```go
f := NewFile(10, "./test.txt")
```

这种方式可以模拟OO编程中使用`new`的实例化，如果要完全等同，还需要利用可见性规则禁止使用Go内置的`new()`函数

```go
type matrix struct {
    ...
}

func NewMatrix(params) *matrix {
    m := new(matrix) // 初始化 m
    return m
}
```

然后在其它包里就只能使用工厂创建结构体实例

```go
package main
import "matrix"
...
wrong := new(matrix.matrix)     // 编译失败（matrix 是私有的）
right := matrix.NewMatrix(...)  // 实例化 matrix 的唯一方式
```

## 2. 方法

Go中的方法是作用在接收者上的一个函数，接收者是某种类型的变量。定义方法的一般格式如下

```go
func (recv receiver_type) methodName(parameter_list) (return_value_list) { ... }
```

在`func`关键字之后，方法名之前的括号中声明接收者和接收者类型。同样还可以看到，方法的本质仍然是函数，只不过是针对特定变量的函数，除了括号中的接收者声明，其它部分和普通函数没有不同。

接收者几乎可以是任何类型的变量，包括基本数据类型、数组的别名类型、结构体、函数等，但不可以是接口，因为方法是需要实现的，而接口只是抽象定义。

如果接收者变量`recv`已经在其它地方进行了初始化，Method1是它的方法名，那么方法的调用格式为`recv.Method1()`，同结构体相似，如果`recv`是指针，调用时自动解引用。

如果方法不需要使用`recv`的值，可以用空白符`_`替换它

```go
func (_ receiver_type) methodName(parameter_list) (return_value_list) { ... }
```

 类型 T（或 *T）上的所有方法的集合叫做类型 T（或 *T）的方法集（method set）。 

一个接收者变量加上它的方法等价于面向对象中的一个类，区别只在于Go中方法的代码与变量定义是分离的，只要在同一个包中即可。

因为方法是函数，所以方法同样不允许重载，但不同的接收者变量可以有相同名字的方法，即使它们在同一个包中，一个例子如下

```go
func (a *denseMatrix) Add(b Matrix) Matrix
func (a *sparseMatrix) Add(b Matrix) Matrix
```

一个结构体方法的例子如下

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
//Output:
The sum is: 22
Add them to the param: 42
The sum is: 7
```

一个非结构体类型(数组别名)方法的例子如下

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

变量和定义在它上面的方法必须在同一个包里定义，如下例是错误的，这也是为什么不能定义int这样的基本类型的方法，但可以定义基本类型的别名的方法

```go
package main

import "container/list"

func (p *list.List) Iter() {
	// ...
}

func main() {
	lst := new(list.List)
	for _= range lst.Iter() {
	}
}
```

### 2.1 函数和方法的区别

函数将变量作为参数：**Function1(recv)**

方法在变量上被调用：**recv.Method1()**

在接收者是指针时，方法可以改变接收者的值（或状态），这点函数也可以做到（当参数作为指针传递，即通过引用调用时，函数也可以改变参数的状态）。

**不要忘记 Method1 后边的括号 ()，否则会引发编译器错误：`method recv.Method1 is not an expression, must be called`**

接收者必须有一个显式的名字，这个名字必须在方法中被使用。

**receiver_type** 叫做 **（接收者）基本类型**，这个类型必须在和方法同样的包中被声明。

在 Go 中，（接收者）类型关联的方法不写在类型结构里面，就像类那样；耦合更加宽松；类型和方法之间的关联由接收者来建立。

**方法没有和数据定义（结构体）混在一起：它们是正交的类型；表示（数据）和行为（方法）是独立的。**

### 2.2 指针或值作为接收者

 如果想要方法改变接收者的数据，就在接收者的指针类型上定义该方法。否则，就在普通的值类型上定义方法。一个例子如下

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

指针方法和值方法都可以在指针或非指针上被调用，如上例，b1是值类型，而change()方法作用在指针类型上，b1.change()会被自动转换为(&b1).change()；b2是指针类型，但write()方法是值类型，b2.write()会被自动转换成(*b2).write()

 ### 2.3 利用方法读取结构体中的未导出字段

本文开始对结构体的介绍中，提到结构体对外部可见，而结构体中的字段对外部不可见是可能发生的，对于这种情况，读取或修改结构体中的字段值可以通过作用在结构体上的方法完成，一个例子如下

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

对其中定义的结构体字段进行调用

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

### 2.4 内嵌类型的方法与继承

 当一个匿名类型被内嵌在结构体中时，匿名类型的可见方法也同样被内嵌，这在效果上等同于外层类型 **继承** 了这些方法 ， 这个机制提供了一种简单的方式来模拟面向对象语言中的子类和继承相关的效果。一个示例如下

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

使用同名方法可以覆盖父类型中的方法，比如在上例中添加如下代码，会打印100

```go
func (n *NamedPoint) Abs() float64 {
	return n.Point.Abs() * 100.
}
```

因为一个结构体可以嵌入多个匿名类型，所以实际上可以实现简单的多重继承，如下例所示

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

### 2.5 在类型中嵌入功能

主要有两种方法来实现在类型中嵌入功能：

A：聚合（或组合）：包含一个所需功能类型的具名字段。

B：内嵌：内嵌（匿名地）所需功能类型，像前一节 10.6.5 所演示的那样。

 假设有一个 `Customer` 类型，我们想让它通过 `Log` 类型来包含日志功能，`Log` 类型只是简单地包含一个累积的消息（当然它可以是复杂的）。如果想让特定类型都具备日志功能，你可以实现一个这样的 `Log` 类型，然后将它作为特定类型的一个字段，并提供 `Log()`，它返回这个日志的引用。 

使用聚合方式实现如下

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

使用内嵌方式实现如下

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

内嵌的类型不需要指针，`Customer` 也不需要 `Add` 方法，它使用 `Log` 的 `Add` 方法，`Customer` 有自己的 `String` 方法，并且在它里面调用了 `Log` 的 `String` 方法。

如果内嵌类型嵌入了其他类型，也是可以的，那些类型的方法可以直接在外层类型中使用。

因此一个好的策略是创建一些小的、可复用的类型作为一个工具箱，用于组成域类型。

 
