# Golang语法基础11-错误处理


Go中有两套错误处理的方式

- 普通错误： 在函数和方法中返回错误对象作为它们的唯一或最后一个返回值 ，如果返回nil，表示没有错误
- 异常：使用panic和recover

主调函数总是应该检查收到的错误，不要忽略，否则可能导致严重的后果。

<!--more-->

## 1. 普通错误

Go有一个预定义的error接口类型

```go
type error interface {
    Error() string
}
```

errors包中有一个errorString结构体实现了该接口，其实 errors 包实现很简单，如下

```go
package errors

// New returns an error that formats as the given text.
// Each call to New returns a distinct error value even if the text is identical.
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```

### 1.1 定义错误

当我们需要一个新的错误类型，可以使用`errors`包的`errors.New`函数接收合适的信息来创建，如下例

```go
err := errors.New("math - square root of negative number")
```

从上面的实现中可以看到调用 errors.New 其实就是将传入的字符串给了结构体 errorString，由于该结构体实现了 error 接口，因此 New 函数返回的时候将结构体赋给了一个 error 接口变量，所以以后我们在主调函数输出该返回值时，会输出结构体的值。

拿一个计算平方根的函数举例，可以这样使用

```go
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New("math - square root of negative number")
    }
    //implementation of Sqrt
}
```

然后像下面这样调用`Sqrt`函数

```go
if f, err := Sqrt(-1); err != nil {
    fmt.Printf("Error: %s\n", err)
}
```

使用`fmt.Printf`将错误信息打印出来，定义的错误信息通过会有像`Error:`这样的前缀，所以错误信息的内容不要以大写字母开头，如上例中使用`math`而不是`Math`

大部分情况自定义错误类型都是很有用的方法，可以将底层错误信息之外的其它有用信息打印出来。如果有不同的错误条件，就可以用类型断言判断错误场景，并做一些补救和恢复

```go
switch err := err.(type) {
	case ParseError:
		PrintParseError(err)
	case PathError:
		PrintPathError(err)
	...
	default:
		fmt.Printf("Not a special error, just %s\n", err)
}
```

包也可以用额外的方法定义特定的错误，如net.Error:

```go
package net
type Error interface {
	Timeout() bool   // Is the error a timeout?
	Temporary() bool // Is the error temporary?
}
```

如前所述，所有的例子都遵循同一种命名规范： 错误类型以 “Error” 结尾，错误变量以 “err” 或 “Err” 开头。 

### 1.2 输出更多的错误信息

`fmt.Errorf()`是一个常用的`fmt`包的函数，用于输出更多的错误信息，而不是单单一个字符串。如前面平方根的例子

```go
if f < 0 {
    return 0, fmt.Errorf("math: squre root of negative number %g", f)
}
```

实际上，`fmt.Errorf`和`errors.New`一样都返回`error`类型的变量

## 2. 异常

或者称为运行时异常，指的是那些无法预测的错误。比如数组下标越界或类型断言失败，就会触发异常，并且伴随着程序终止返回一个`runtime.Error`接口类型的值，这个值和普通错误的区别在于有一个`RuntimeError()`方法

### 2.1 panic

`panic`就是Go中用于生成异常的方法。当错误不可修复、程序无法继续运行时，使用`panic`函数来产生一个中止程序的运行时错误，`panic`接收任意类型的参数，通常是字符串，这个参数会在程序终止时被打印出来。中止程序和打印参数的过程由Go runtime负责。一个例子如下

```go
package main

import "fmt"

func main() {
	fmt.Println("Starting the program")
	panic("A severe error occurred: stopping the program!")
	fmt.Println("Ending the program")
}
```

输出如下

```go
Starting the program
panic: A severe error occurred: stopping the program!

goroutine 1 [running]:
main.main()
        F:/Gotest/main.go:7 +0x9c
exit status 2
```

一个检测到错误然后使用panic抛出异常的完整例子如下

```go
if err != nil {
	panic("ERROR occurred:" + err.Error())
}
```

但panic抛出异常并中止程序是最后的办法，如果有修复的可能就不应该使用

### 2.2 panicking

在多层嵌套的函数调用中使用panic，可以马上中止当前函数的执行，所有的defer语句都会保证执行并把控制权交还给接收到panic的函数调用者。这样向上冒泡直到最顶层，并执行（每层的）defer，在栈顶处程序崩溃，并在命令行中用传给panic的值报告错误情况：这个终止的过程就是panicking。

### 2.3 recover

`revocer`函数用于让程序从`panicking`重新获得控制权，停止终止过程进而恢复正常执行，但`recover`只能在`defer`修饰的函数中使用。如果正常执行，调用`recover`会返回nil，没有其它效果。

简而言之，panicking有两个结果，一个是程序终止，一个是遇到defer修饰的recover()函数然后恢复。一个例子如下

```go
func protect(g func()) {
	defer func() {
		log.Println("done")
		// Println executes normally even if there is a panic
		if err := recover(); err != nil {
			log.Printf("run time panic: %v", err)
		}
	}()
	log.Println("start")
	g() //   possible runtime-error
}
```

将panic，defer和recover结合的完整例子如下

```go
package main

import (
	"fmt"
)

func badCall() {
	panic("bad end")
}

func test() {
	defer func() {
		if e := recover(); e != nil {
			fmt.Printf("Panicing %s\r\n", e)
		}
	}()
	badCall()
	fmt.Printf("After bad call\r\n") // <-- wordt niet bereikt
}

func main() {
	fmt.Printf("Calling test\r\n")
	test()
	fmt.Printf("Test completed\r\n")
}
```

输出

```go
Calling test
Panicing bad end
Test completed
```

## 3. 错误处理的一些原则

这是在编写自己的包时需要遵循的一些原则，有助于别人调用和程序正常运行

1. 在包内部，总是在panic后使用recover，不要让panic返回的层次超出当前包的范围
2. 向包的调用者返回错误值，而不是panic
3. 在包内部的不可导出函数中有深层次调用时，将panic转换成error来通知调用者出错信息

根据此原则，一个`parse`包如下，作用是把输入的字符串解析为整数切片 

```go
package parse

import (
	"fmt"
	"strings"
	"strconv"
)

// A ParseError indicates an error in converting a word into an integer.
type ParseError struct {
    Index int      // The index into the space-separated list of words.
    Word  string   // The word that generated the parse error.
    Err error // The raw error that precipitated this error, if any.
}

// String returns a human-readable error message.
func (e *ParseError) String() string {
    return fmt.Sprintf("pkg parse: error parsing %q as int", e.Word)
}

// Parse parses the space-separated words in in put as integers.
func Parse(input string) (numbers []int, err error) {
    defer func() {
        if r := recover(); r != nil {
            var ok bool
            err, ok = r.(error)
            if !ok {
                err = fmt.Errorf("pkg: %v", r)
            }
        }
    }()

    fields := strings.Fields(input)
    numbers = fields2numbers(fields)
    return
}

func fields2numbers(fields []string) (numbers []int) {
    if len(fields) == 0 {
        panic("no words to parse")
    }
    for idx, field := range fields {
        num, err := strconv.Atoi(field)
        if err != nil {
            panic(&ParseError{idx, field, err})
        }
        numbers = append(numbers, num)
    }
    return
}
```

该包定义了自己的`ParseError`，当没有东西需要转换或转换成整数失败时，产生panic，但可导出的Parse函数会从panic中recover并整合信息返回一个错误给调用者。

一个调用parse包的实例如下

```go
package main

import (
	"fmt"
	"./parse/parse"
)

func main() {
    var examples = []string{
            "1 2 3 4 5",
            "100 50 25 12.5 6.25",
            "2 + 2 = 4",
            "1st class",
            "",
    }

    for _, ex := range examples {
        fmt.Printf("Parsing %q:\n  ", ex)
        nums, err := parse.Parse(ex)
        if err != nil {
            fmt.Println(err) // here String() method from ParseError is used
            continue
        }
        fmt.Println(nums)
    }
}
```

输出

```go
Parsing "1 2 3 4 5":
  [1 2 3 4 5]
Parsing "100 50 25 12.5 6.25":
  pkg: pkg parse: error parsing "12.5" as int
Parsing "2 + 2 = 4":
  pkg: pkg parse: error parsing "+" as int
Parsing "1st class":
  pkg: pkg parse: error parsing "1st" as int
Parsing "":
  pkg: no words to parse
```

## 4. 闭包处理错误

根据上面的原则，只要有函数返回，就应该检查是否有错误发生，但这会导致重复乏味的代码。结合 defer/panic/recover 机制和闭包可以得到一种更加优雅的写法。不过这种写法的限制是所有函数都需要是同一种签名。

假设所有函数签名都是下面这种形式

```go
func f(a type1, b type2)
```

参数的数量和类型是不相关的，我们给这个类型一个名字：

```
fType1 = func f(a type1, b type2)
```

使用两个辅助函数帮忙完成整个过程

1. check：用来检查是否有错误和panic发生的函数

    ```go
   func check(err error) { if err != nil { panic(err) } }
   ```

2.  errorhandler：接收一个 fType1 类型的函数 fn 并返回一个调用 fn 的函数, 其中包含有 defer/recover 机制

    ```go
   func errorHandler(fn fType1) fType1 {
   		return func(a type1, b type2) {
   		defer func() {
   			if err, ok := recover().(error); ok {
   				log.Printf("run time panic: %v", err)
   			}
   		}()
   		fn(a, b)
   	}
   }
   ```
   
   当错误发生时会 recover 并打印在日志中，check() 函数会在所有的被调函数中调用，像这样： 
   
   ```go
   func f1(a type1, b type2) {
   	f, _, err := // call function/method
   	check(err)
   	t, err := // call function/method
   	check(err)
   	_, err2 := // call function/method
   	check(err2)
   }
   ```

通过这种机制，所有的错误都会被 recover，并且调用函数后的错误检查代码也被简化为调用 check(err) 即可。在这种模式下，不同的错误处理必须对应不同的函数类型；它们（错误处理）可能被隐藏在错误处理包内部。可选的更加通用的方式是用一个空接口类型的切片作为参数和返回值。 

最最重要的一点，Goroutine 中抛出的异常，只能在本协程中使用 recover 捕获，主协程是无法接收到的，同时，子协程发生的 panic 如果没有被捕获，会导致整个程序中断。
