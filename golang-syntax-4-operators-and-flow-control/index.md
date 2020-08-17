# Golang语法基础4-运算符与流程控制


本文介绍 Golang 中的运算符和流程控制

<!--more-->

## 1. 运算符

Go语言的运算符有6种，如下表所示

| 运算符类型 | 运算符                                      |
| ---------- | ------------------------------------------- |
| 算术运算符 | +，-，*，/，%，++，--                       |
| 关系运算符 | ==，!=，>，<，>=，<=                        |
| 逻辑运算符 | &&，\|\|，!                                 |
| 位运算符   | &，\|，\^，<<，>>，&^                       |
| 赋值运算符 | =，+=，-=，*=，/=，%=，<<=，>>=，&=，^=，!= |
| 其它运算符 | &，*                                        |

基本的运算规则都与C语言相同，一些注意事项列举如下

算术运算符中自增自减运算符只能作为语句使用，不能用于表达式

```go
a++ // 允许
a-- // 允许
a = a++ //不允许，编译错误
a[i] = b[i++] //不允许，编译错误
```

整数除以0会导致编译错误，如果编译时未检出会导致程序崩溃。浮点数除以0.0会返回无穷大，用`+Inf`表示

逻辑运算符具有短路效果

位运算符是对整数在内存中的二进制位进行操作的，假定A = 60，B = 13，则

```go
A = 0011 1100
B = 0000 1101

A&B = 0000 1100 // 结果为12
A|B = 0011 1101 // 结果为61
A^B = 0011 0001 // 结果为49
A<<2 = 1111 0000 // 结果为240
A>>2 = 0000 1111 // 结果为15
```

其它运算符中的`&`是取地址符，`*`是指针变量

运算符的优先级是不同的，下表从上往下代表优先级从高到低

| 优先级 | 运算符                       |
| ------ | ---------------------------- |
| 7      | ^ ，!                        |
| 6      | * ，/ ，% ，<< ，>>， &， &^ |
| 5      | +， -， \| ，^               |
| 4      | ==， !=， < ，<= ，>=， >    |
| 3      | <-                           |
| 2      | &&                           |
| 1      | \|\|                         |

二元运算符的运算方向均是从左到右，必要时可以使用括号提升优先级或更清楚地表达。

## 2. 控制结构

除去顺序结构外，Go语言提供的基本流程控制结构包括

- 条件结构
  -  if-else 结构
  - switch 结构
  - select 结构，用于channel的选择（协程与通道部分）
- 循环结构 
  - for
  - for-range

同时，Go还提供了关键字`break`、`continue`和`goto`用来辅助进行流程控制，以及`return`语句提前结束执行。

注1：在这些结构中，Go都省略了条件语句两侧的小括号，使视觉上更加简洁。

注2：除`case`关键字后的语句，即使代码块只有一行，大括号也不可省略

注3：左大括号必须和关键字在同一行，对多分支结构中的`else`关键字，右大括号也要和它在一行。这两条规则是编译器的强制规定。

### 2.1 if-else 结构

if-else 结构的基本形态与C语言相同。可以省略`else`关键字变成单分支结构，也可以添加`else if`变成多分支，但为了代码简洁，过多的分支最好换用switch结构实现。

```go
if condition1 {
    // do something
} else {
    // do something else
}
```

当双分支结构在代码块的末尾时，通常会将else中原本的代码块迁移出来放在最后，如

```go
if condition {
    return true
}
return false
```

Go中 if 还可以在条件语句前添加一个初始化语句，以分号分隔

```go
if initialization; condition {
	// do something
}
```

例如

```go
if val := 10; val > max {
	// do something
}
```

但需要注意的时，使用简短方式 `:=` 声明和初始化的变量作用域只限于 if 结构的代码块内，属于局部变量。

由于Go语言并行赋值的特性，if 语句经常用于测试多返回值函数的错误。返回某个值以及true表示成功，返回零值（或nil）以及false表示失败

```go
if value, ok := readData(); ok {
…
}
```

当不使用true或false时，也可以使用一个error类型的变量来代替作为第二个返回值，成功执行，error的值为nil，失败返回的值会包含相应的错误信息。

```go
var err error
if err := file.Chmod(0664); err != nil {
	fmt.Println(err)
	return err
}
```

### 2.2 switch 结构

switch结构的基本形态依然同C语言相同

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

不同的是，Go中switch语句接受任意形式的表达式，如上例中var1可以是任何类型，而val1和val2可以是同类型的任意值，不局限于数值。

可以同时测试多个可能符合条件的值，使用逗号分隔，例如`case val1, val2, val3`

每个`case`分支都是唯一的，从上到下逐一测试，一旦成功匹配到某个分支，执行完对应的代码块后会**自动退出**整个 switch 结构，而不需要使用`break`结束。因此，程序不会自动的去执行剩下的case分支的代码，如果想继续执行，需要使用fallthrough关键字

```go
switch i {
	case 0: fallthrough
	case 1:
		f() // 当 i == 0 时函数也会被调用
}
```

case语句后不需要用大括号包围代码块，default分支可以出现在任何顺序，但最好放在最后

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
//Output:
//It's equal to 100
```

switch可以不提供条件语句，然后在每个case分支测试不同条件，可以替换分支比较多的if-else结构，简化代码

```go
switch {
	case i < 0:
		f1()
	case i == 0:
		f2()
	case i > 0:
		f3()
    default:
    	...
}
```

switch的条件语句还可以是初始化语句

```go
switch a, b := x[i], y[j] {
	case a < b: t = -1
	case a == b: t = 0
	case a > b: t = 1
}
```

### 2.3 for 结构

Go中只有for用于循环结构，没有C中的while或do while，基本形态如下

```go
for 初始化语句; 条件语句; 修饰语句 {
    ...
}
```

同样，for 关键字后的三个语句不需要小括号，左大括号需和关键字在同一行

可以只保留条件语句，这种情况下可以去掉所有分号，大致等同于其它语言的while循环

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

或者省略条件语句，但必须在循环体中存在条件判断以确保在某个时候退出循环，退出可以使用`break`或`return	`

```go
for i := 0; ; i++ {
    ...
}
```

或者三条语句全部省略，但同样需要在循环体中添加退出条件

```go
for {
    ...
}
```

### 2.4 for-range 结构

这是 Go 中特有的迭代结构，可以迭代数组、切片、map、字符串等任意一个集合，一般形式为

```go
for k, v := range set {
    ...
}
```

k 为索引，每次递增，v 为索引对应的值的拷贝。值得注意的是，由于 v 只是值的拷贝，任何对它的修改都不会影响集合中原来的值，除非索引是指针。

如果不需要索引，可以使用匿名变量`_`忽略它

```go
for _, v := range set{
    ...
}
```

但如果只需要索引而不需要值，可以直接省略不写

```go
for k := range set {
    ...
}
// Output: 0 1 2 ...
```

字符串通过for-range结构获取的元素是rune类型

## 3. 辅助关键字

`break`用来跳出循环，在for循环中跳出一层循环，在switch或select语句中，跳过整段代码块

```go
for i:=0; i<3; i++ {
    for j:=0; j<10; j++ {
        if j>5 {
            break   
        }
        print(j)
    }
    print("  ")
}
//Output:
//012345 012345 012345
```

`continue`用来忽略剩余的循环体直接进入下一次循环，只存在于for循环中

```go
for i := 0; i < 10; i++ {
    if i == 5 {
        continue
    }
    print(i)
    print(" ")
}
//Output:
//0 1 2 3 4 6 7 8 9
```

for、switch 或 select 语句都可以配合标签（label）形式的标识符使用，即某一行第一个以冒号（`:`）结尾的单词，下例中continue语句指向LABEL1，当执行到该语句时，就会跳转到LABEL1标签的位置起继续执行，不过此时注意循环体内的变量并不会被释放，当j==4循环跳出后，i会自动变成下一个循环的值，不会陷入无限循环。

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

标签的名称和一般的标识符相同，都是大小写敏感的，但为了可读性，一般全部使用大写字母。同变量相同，标签定义未使用也会导致编译错误。

`goto`关键字是配合标签使用的，但这种用法并不被推荐，因为很可能导致糟糕的代码结构，一如当年的PASCAL

逆序的标签虽然可能导致错误，但正序的标签(标签位于goto语句之后)则可以正常使用，但标签和goto之间不能有新定义变量的语句，否则会导致编译失败

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


