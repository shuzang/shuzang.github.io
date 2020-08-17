---
title: Golang查漏补缺-按指定长度构造二维切片
date: 2019-09-03
tags: [Golang]
categories: [Golang学习之路]
typora-root-url: ..\..\..\static
---

因为初始化时，数组长度必须为常量，所以当要求按给定的长度构造数组时，一般都使用切片来完成。一维的切片直接使用`arr := make([]int, len)`构造，`len`是指定的长度。二维的切片就不是很友好了，需要使用循环来完成。具体示例如下。

```go
package main

import (
	"fmt"
)

func main() {
	var m, n int
	fmt.Scanln(&m, &n)
	a := make([]int, m)
	var b [][]int
	for i := 0; i < m; i++ {
		b = append(b, make([]int, n))
	}
	fmt.Println(a, b)
}
//Output
[0 0 0] [[0 0] [0 0] [0 0]]
```

