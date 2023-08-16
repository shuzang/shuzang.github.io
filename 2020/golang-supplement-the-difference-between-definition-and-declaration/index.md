# Golang查漏补缺-定义与声明的区别


一直都比较纠结这个问题，所以搜了一下答案，记录在这里。

<!--more-->

变量的声明有两种情况：

1. 需要建立存储空间，这种称为定义性声明（defining declaration），简称定义
2. 不需要建立存储空间，这种称为引用性声明（referncing declaration）

所以广义的讲，声明包含定义，定义是声明的一个特例。

在 Go 中，基本变量类型在声明时都会分配存储空间并分配默认值，因此都属于定义性声明

```go
var (
	a int
    b float32
    c bool
)
```

但是，像切片、映射、通道等，声明时不会分配存储空间，要分配空间还必须使用 make 内置函数，因此它们是引用性声明

```go
var a []int
a = make([]int,3)
```

在 Go 的官方文档中，使用的也都是 declaration 这个词，统一用「声明」来描述



---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2020/golang-supplement-the-difference-between-definition-and-declaration/  

