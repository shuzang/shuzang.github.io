# Golang查漏补缺-bufio读取的字符串无法直接进行类型转换


如下列代码，在使用bufio包中的ReadString读取字符串之后，这个字符串无法进行类型转换，每次使用strconv.Atoi()函数返回值均为0。

```go
inputReader := bufio.NewReader(os.Stdin)
fmt.Println("Please input a number")
t, err := inputReader.ReadString('\n')
if err != nil {
    fmt.Println("can't read number!")
}
fmt.Println("the number is", t)
if num, ok := strconv.Atoi(t); ok != nil {
    fmt.Println("convert to int error", num)
}

//input
25
//output
Please input a number
the number is 25

convert to int error 0
```

因为这种写法其实经常遇到，之前编程的时候遇到这种情况没怎么注意，以为是算法问题，就换思路写了，直到这次只能用这种思路，才发现这里出现了问题。

[Stackoverflow](https://stackoverflow.com/questions/31464142/what-is-wrong-with-this-go-switch-statement)上相关的问题回答说这是因为ReadString读取字符串成功后会把`'\n'`一起加在字符串后面。查找包说明发现函数的原型和解释如下

```go
func (b *Reader) ReadString(delim byte) (string, error)
/* ReadString reads until the first occurrence of delim in the input, returning a string containing the data up to and including the delimiter. If ReadString encounters an error before finding a delimiter, it returns the data read before the error and the error itself (often io.EOF). ReadString returns err != nil if and only if the returned data does not end in delim. For simple uses, a Scanner may be more convenient.
*/
```

这种情况给出的建议是利用strings.Trim...系列去除末尾添加的字符，比如，对上面的错误程序

```go
inputReader := bufio.NewReader(os.Stdin)
fmt.Println("Please input a number")
t, err := inputReader.ReadString('\n')
if err != nil {
    fmt.Println("can't read number!")
}
fmt.Println("the number is", t)
t = strings.TrimRight(t, "\n")
if num, ok := strconv.Atoi(t); ok == nil {
    fmt.Println("convert succeed", num)
}
//input 
25
//output
Please input a number
the number is 25

convert succeed 25
```



---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2019/golang%E6%9F%A5%E6%BC%8F%E8%A1%A5%E7%BC%BA-bufio%E8%AF%BB%E5%8F%96%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%97%A0%E6%B3%95%E7%9B%B4%E6%8E%A5%E8%BF%9B%E8%A1%8C%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2/  

