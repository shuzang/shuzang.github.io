# Golang语法基础5-字符串操作与输入输出


本篇介绍字符串的相关操作，涉及`string`和`strconv`两个标准库，以及介绍输入输出的基本方法，涉及`fmt`和`bufio`两个标准库。

<!--more-->

## 1. 字符串操作

对字符串的操作无论在什么语言里都是很重要的，因此在基本数据类型中介绍过字符串之后，这里仍然专门拿出一篇来介绍关于字符串的处理。

如我们之前所述，内置的字符串操作是字符串的拼接，通过拼接符`+`来完成。

```go
s := "hel" + "lo,"
s += "world!"
fmt.Println(s) 
//Output: 
//hello, world!
```

但使用拼接符`+`并不是最高效的做法，同时，由于字符串是一种内容不可变的值类型，无法直接通过索引操作其内的任意字符。Go语言内置了`strings`包来提供对字符串的种种操作，方便我们使用，当然，很多时候也会使用`strconv`包，这个包的使用我们在类型转换部分提到过。

### 1.1 判断包含关系

`HasPrefix` 判断字符串 `s` 是否以 `prefix` 开头：

```go
strings.HasPrefix(s, prefix string) bool
```

`HasSuffix` 判断字符串 `s` 是否以 `suffix` 结尾：

```go
strings.HasSuffix(s, suffix string) bool
```

示例如下

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	var str string = "This is an example of a string"
	fmt.Printf("T/F? Does the string \"%s\" have prefix %s? ", str, "Th")
	fmt.Printf("%t\n", strings.HasPrefix(str, "Th"))
}
//Output:
//T/F? Does the string "This is an example of a string" have prefix Th? true
```

更一般化的，`Contains` 判断字符串 `s` 是否包含 `substr`：

```go
strings.Contains(s, substr string) bool
```

### 1.2 获取子字符串的位置

`Index` 返回字符串 `str` 在字符串 `s` 中的索引（`str` 的第一个字符的索引），-1 表示字符串 `s` 不包含字符串 `str`：

```go
strings.Index(s, str string) int
```

`LastIndex` 返回字符串 `str` 在字符串 `s` 中最后出现位置的索引（`str` 的第一个字符的索引），-1 表示字符串 `s` 不包含字符串 `str`：

```go
strings.LastIndex(s, str string) int
```

如果需要查询非 ASCII 编码的字符在父字符串中的位置，使用以下函数来对字符进行定位：

```go
strings.IndexRune(s string, r rune) int
```

示例如下

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	var str string = "Hi, I'm Marc, Hi."

	fmt.Printf("The position of \"Marc\" is: ")
	fmt.Printf("%d\n", strings.Index(str, "Marc"))

	fmt.Printf("The position of the first instance of \"Hi\" is: ")
	fmt.Printf("%d\n", strings.Index(str, "Hi"))
	fmt.Printf("The position of the last instance of \"Hi\" is: ")
	fmt.Printf("%d\n", strings.LastIndex(str, "Hi"))

	fmt.Printf("The position of \"Burger\" is: ")
	fmt.Printf("%d\n", strings.Index(str, "Burger"))
}
/*Output:
The position of "Marc" is: 8
The position of the first instance of "Hi" is: 0
The position of the last instance of "Hi" is: 14
The position of "Burger" is: -1
*/
```

### 1.3 统计出现次数

`Count` 用于计算字符串 `str` 在字符串 `s` 中出现的非重叠次数：

```go
strings.Count(s, str string) int
```

示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	var str string = "Hello, how is it going, Hugo?"
	var manyG = "gggggggggg"

	fmt.Printf("Number of H's in %s is: ", str)
	fmt.Printf("%d\n", strings.Count(str, "H"))

	fmt.Printf("Number of double g's in %s is: ", manyG)
	fmt.Printf("%d\n", strings.Count(manyG, "gg"))
}
/*Output:
Number of H's in Hello, how is it going, Hugo? is: 2
Number of double g’s in gggggggggg is: 5
*/
```

### 1.4 替换与重复

`Replace` 用于将字符串 `str` 中的前 `n` 个字符串 `old` 替换为字符串 `new`，并返回一个新的字符串，如果 `n = -1` 则替换所有字符串 `old` 为字符串 `new`：

```go
strings.Replace(str, old, new, n) string
```

`Repeat` 用于重复 `count` 次字符串 `s` 并返回一个新的字符串：

```go
strings.Repeat(s, count int) string
```

示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	var origS string = "Hi there! "
	var newS string

	newS = strings.Repeat(origS, 3)
	fmt.Printf("The new repeated string is: %s\n", newS)
}
//Output:
//The new repeated string is: Hi there! Hi there! Hi there!
```

### 1.5 大小写转换

`ToLower` 将字符串中的 Unicode 字符全部转换为相应的小写字符：

```go
strings.ToLower(s) string
```

`ToUpper` 将字符串中的 Unicode 字符全部转换为相应的大写字符：

```go
strings.ToUpper(s) string
```

示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	var orig string = "Hey, how are you George?"
	var lower string
	var upper string

	fmt.Printf("The original string is: %s\n", orig)
	lower = strings.ToLower(orig)
	fmt.Printf("The lowercase string is: %s\n", lower)
	upper = strings.ToUpper(orig)
	fmt.Printf("The uppercase string is: %s\n", upper)
}
/*Output:
The original string is: Hey, how are you George?
The lowercase string is: hey, how are you george?
The uppercase string is: HEY, HOW ARE YOU GEORGE?
*/
```

### 1.6 修剪与分割

可以使用 `strings.TrimSpace(s)` 来剔除字符串开头和结尾的空白符号；

可以使用 `strings.Trim(s, "cut")` 来将开头和结尾的指定字符串（`cut`）去除掉。

如果只想剔除开头或者结尾的字符串，则可以使用 `TrimLeft` 或者 `TrimRight` 来实现。

`strings.Fields(s)` 将会利用 1 个或多个空白符号来作为动态长度的分隔符将字符串分割成若干小块，并返回一个 slice，如果字符串只包含空白符号，则返回一个长度为 0 的 slice。

`strings.Split(s, sep)` 用于自定义分割符号来对指定字符串进行分割，同样返回 slice。

因为这 2 个函数都会返回 slice，所以习惯使用 for-range 循环来对其进行处理

### 1.7 拼接

`Join` 用于将元素类型为 string 的 slice 使用分割符号来拼接组成一个字符串：

```go
strings.Join(sl []string, sep string) string
```

示例

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "The quick brown fox jumps over the lazy dog"
	sl := strings.Fields(str)
	fmt.Printf("Splitted in slice: %v\n", sl)
	for _, val := range sl {
		fmt.Printf("%s - ", val)
	}
	fmt.Println()
	str2 := "GO1|The ABC of Go|25"
	sl2 := strings.Split(str2, "|")
	fmt.Printf("Splitted in slice: %v\n", sl2)
	for _, val := range sl2 {
		fmt.Printf("%s - ", val)
	}
	fmt.Println()
	str3 := strings.Join(sl2,";")
	fmt.Printf("sl2 joined by ;: %s\n", str3)
}
/*Output:
Splitted in slice: [The quick brown fox jumps over the lazy dog]
The - quick - brown - fox - jumps - over - the - lazy - dog -
Splitted in slice: [GO1 The ABC of Go 25]
GO1 - The ABC of Go - 25 -
sl2 joined by ;: GO1;The ABC of Go;25
*/
```

## 2. 输入输出

编写程序进行数据的读写必不可少，一般会用到fmt, os和bufio三个包，下面对一些读写方式进行总结。

### 2.1 读取用户输入

用户输入来自三种：标准输入，字符串，io.Reader类型

#### 来自标准输入

读取用户输入一般指的是读取用户的键盘（控制台）输入，定义为标准输入os.Stdin，最常用的方法是使用fmt包提供的Scan开头的函数。

```go
func Scan(a ...interface{}) (n int, err error)
func Scanf(format string, a ...interface{}) (n int, err error)
func Scanln(a ...interface{}) (n int, err error)
```

Scan从标准输入扫描文本，将成功读取的空白分隔的值传递给函数的参数。换行视为空白。返回成功扫描的条目个数和遇到的任何错误。如果读取的条目比提供的参数少，会返回一个错误报告原因。

Scanf从标准输入扫描文本，根据format 参数指定的格式将成功读取的空白分隔的值传递给函数的参数。返回成功扫描的条目个数和遇到的任何错误。

Scanln类似Scan，但会在换行时才停止扫描。最后一个条目后必须有换行或者到达结束位置。

一个示例程序如下：

```go
package main
import "fmt"

var firstName, lastName string

func main() {
   fmt.Println("Please enter your full name: ")
   fmt.Scanln(&firstName, &lastName)
   // fmt.Scanf("%s %s", &firstName, &lastName)
}
```

#### 来自字符串

读取来自字符串的输入一般使用fmt包中SScan开头的函数

```go
func Sscan(str string, a ...interface{}) (n int, err error)
func Sscanf(str string, format string, a ...interface{}) (n int, err error)
func Sscanln(str string, a ...interface{}) (n int, err error)
```

SScan开头的函数基本和Scan相似，唯一的不同是多了第一个参数str，代表从字符串str扫描文本。一个示例程序如下：

```go
package main

import "fmt"

func main() {
	var name string
	var age int
	n, err := fmt.Sscanf("Kim is 22 years old", "%s is %d years old", &name, &age)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%d: %s, %d\n", n, name, age)

}
//Output
2: Kim, 22
```

#### 来自io.Reader类型

主要是使用Fscan开头的函数

```go
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
```

同样，与Scan的不同是第一个参数r是io.Reader类型，示例程序如下：

```go
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	var (
		i int
		b bool
		s string
	)
	r := strings.NewReader("5 true gophers") //返回一个io.Reader类型
	n, err := fmt.Fscanf(r, "%d %t %s", &i, &b, &s)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fscanf: %v\n", err)
	}
	fmt.Println(i, b, s)
	fmt.Println(n)
}

5 true gophers
3
```

### 2.2 输出指定内容

输出和读取输入基本是是相反的，各函数原型如下：

```go
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```

示例程序如下：

```go
package main

import (
	"fmt"
	"io"
	"os"
)


func main() {
	const name, age = "Kim", 22
	fmt.Printf("%s is %d years old.\n", name, age)
    
	s := fmt.Sprintf("%s is %d years old.\n", name, age)
	io.WriteString(os.Stdout, s) // Ignoring error for simplicity.
	
    n, err := fmt.Fprintf(os.Stdout, "%s is %d years old.\n", name, age)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fprintf: %v\n", err)
	}
	fmt.Printf("%d bytes written.\n", n)
}
//Output
Kim is 22 years old.
Kim is 22 years old.
Kim is 22 years old.
21 bytes written.
```

### 2.3 使用bufio包

使用fmt进行读写，读写次数比较多时，时间耗费极大。go提供了一个bufio包，使用该包可以大幅提高文件读写效率。

#### bufio包原理

介绍来自[茹姐-GO语言基础进阶教程：bufio包](https://zhuanlan.zhihu.com/p/73690883)。io操作本身的效率并不低，低的是频繁的访问本地磁盘的文件。所以bufio就提供了缓冲区(分配一块内存)，读和写都先在缓冲区中，最后再读写文件，来降低访问本地磁盘的次数，从而提高效率。

![利用缓冲区读写文件](https://pic3.zhimg.com/80/v2-eafcc5129ec4afd2ed89cecc5824c57e_hd.jpg)

#### 读入

```go
package main
import (
    "fmt"
    "bufio"
    "os"
)

func main() {
    inputReader := bufio.NewReader(os.Stdin)
    fmt.Println("Please enter some input: ")
    input, err := inputReader.ReadString('\n')
    if err == nil {
        fmt.Printf("The input was: %s\n", input)
    }
}

```

`bufio.NewReader()` 构造函数的签名为：`func NewReader(rd io.Reader) *Reader`

`inputReader` 是一个指向 `bufio.Reader` 的指针。`inputReader := bufio.NewReader(os.Stdin)` 这行代码，将会创建一个读取器，并将其与标准输入绑定。返回的读取器对象提供一个方法 `ReadString(delim byte)`，该方法从输入中读取内容，直到碰到 `delim` 指定的字符，然后将读取到的内容连同 `delim` 字符一起放到缓冲区。在上面的例子中，我们会读取键盘输入，直到回车键（\n）被按下。

#### 写出

```go
package main

import (
    "os"
    "fmt"
    "bufio"
)

func main() { 
     w1 := bufio.NewWriter(os.Stdout)

     for i:=1;i<=1000;i++{
        w1.WriteString(fmt.Sprintf("%d:hello",i))
     }
     w1.Flush()
}

```

### 2.4 文件读写

主要是使用os.Open函数打开文件，以及defer关键字和close方法在程序结束时关闭文件。

```go
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
)

func main() {
    inputFile, inputError := os.Open("input.dat")
    if inputError != nil {
        fmt.Printf("An error occurred on opening the inputfile\n" +
            "Does the file exist?\n" +
            "Have you got acces to it?\n")
        return // exit the function on error
    }
    defer inputFile.Close()

    inputReader := bufio.NewReader(inputFile)
    for {
        inputString, readerError := inputReader.ReadString('\n')
        fmt.Printf("The input was: %s", inputString)
        if readerError == io.EOF {
            return
        }      
    }
}

```

未完待续...


