---
title: Golang查漏补缺-对自定义类型排序
date: 2019-08-16
tags: [Go语法]
categories: [Golang学习之路]
---

PAT乙级25分的题好多需要根据一个结构体类型的某个字段进行排序，第一次遇到时确实不知所措，然后查了不少解决方案，这里做个总结。

这一问题一般归结为对自定义类型排序，当然，基本指的是结构体，搜到的解决方案也基本是利用sort包。

sort包基本的排序都是针对切片的，直接调用的话能找到整型、浮点型和字符串三种类型切片的排序，最简单的整型排序如下：

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	s := []int{5, 2, 6, 3, 1, 4} // unsorted
	sort.Ints(s)
	fmt.Println(s)
}
//Output:[1 2 3 4 5 6]
```

## 调用sort.Sort()

不少文章都是通过调用sort.Sort()实现的对结构体的排序，如[youyu岁月](https://segmentfault.com/a/1190000008062661)和[Donne](https://segmentfault.com/a/1190000016198631)的文章就是使用的这种办法。

```go
func Sort(data Interface)
```

Sort函数会调用一次data.Len确定长度，调用O(n*log(n))次data.Less和data.Swap进行排序。但函数不能保证排序的稳定性。而调用Sort首先需要实现一个接口。

```go
type Interface interface {
        // Len is the number of elements in the collection.
        Len() int
        // Less reports whether the element with
        // index i should sort before the element with index j.
        Less(i, j int) bool
        // Swap swaps the elements with indexes i and j.
        Swap(i, j int)
}
```

只要实现了Len, Less, Swap三个方法，就能调用Sort实现对结构体的排序，而其中主要就是Less比较逻辑的实现。

先看一下sort包本身对[]int类型的排序实现

```go
// 首先定义了一个[]int类型的别名IntSlice 
type IntSlice []int
// 获取此 slice 的长度
func (p IntSlice) Len() int           { return len(p) }
// 比较两个元素大小 升序
func (p IntSlice) Less(i, j int) bool { return p[i] < p[j] }
// 交换数据
func (p IntSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }
// sort.Ints()内部调用Sort() 方法实现排序
// 注意 要先将[]int 转换为 IntSlice类型 因为此类型才实现了Interface的三个方法 
func Ints(a []int) { Sort(IntSlice(a)) }
```

然后我们以一个人的结构体为例，结构体中包括其姓名和年龄，按照其年龄对结构体进行排序。

```go
type Person struct {
	Name string
	Age  int
}
type ByAge []Person
```

仿照[]int实现三个方法

```go
func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }
```

然后写个完整的程序测试看一看

```go
package main

import (
	"fmt"
	"sort"
)

type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%s: %d", p.Name, p.Age)
}

// ByAge implements sort.Interface for []Person based on
// the Age field.
type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

func main() {
	people := []Person{
		{"Bob", 31},
		{"John", 42},
		{"Michael", 17},
		{"Jenny", 26},
	}

	fmt.Println(people)
	// There are two ways to sort a slice. First, one can define
	// a set of methods for the slice type, as with ByAge, and
	// call sort.Sort. In this first example we use that technique.
	sort.Sort(ByAge(people))
	fmt.Println(people)
}

//输出结果如下：
[Bob: 31 John: 42 Michael: 17 Jenny: 26]
[Michael: 17 Jenny: 26 Bob: 31 John: 42]
```

如果要对某个结构体中多个字段进行排序，可以利用嵌套结构体实现，如下面的代码，该代码来自[youyu岁月](https://segmentfault.com/a/1190000008062661)

```go
package main

import (
    "fmt"
    "sort"
)

type Person struct {
    Name string
    Age  int
}

type Persons []Person

// Len()方法和Swap()方法不用变化
// 获取此 slice 的长度
func (p Persons) Len() int { return len(p) }

// 交换数据
func (p Persons) Swap(i, j int) { p[i], p[j] = p[j], p[i] }

// 嵌套结构体  将继承 Person 的所有属性和方法
// 所以相当于SortByName 也实现了 Len() 和 Swap() 方法
type SortByName struct{ Persons }

// 根据元素的姓名长度降序排序 （此处按照自己的业务逻辑写）
func (p SortByName) Less(i, j int) bool {
    return len(p.Persons[i].Name) > len(p.Persons[j].Name)
}

type SortByAge struct{ Persons }

// 根据元素的年龄降序排序 （此处按照自己的业务逻辑写）
func (p SortByAge) Less(i, j int) bool {
    return p.Persons[i].Age > p.Persons[j].Age
}

func main() {
    persons := Persons{
        {
            Name: "test123",
            Age:  20,
        },
        {
            Name: "test1",
            Age:  22,
        },
        {
            Name: "test12",
            Age:  21,
        },
    }

    fmt.Println("排序前")
    for _, person := range persons {
        fmt.Println(person.Name, ":", person.Age)
    }
    sort.Sort(SortByName{persons})
    fmt.Println("排序后")
    for _, person := range persons {
        fmt.Println(person.Name, ":", person.Age)
    }
}
```

## 调用sort.Slice()

更简单的是直接调用sort.Slice()，当然，由于golang官方的包说明稳定没法访问，一直查看的是国内的[标准库说明文档](https://studygolang.com/pkgdoc)，完全没有意识到它已经落后了，可能许久未更新，完全没有提到sort包中的Slice函数，但利用这个函数进行自定义类型排序是真有用，还是翻出去看[官方说明](https://golang.org/pkg/sort/)比较好。

```go
func Slice(slice interface{}, less func(i, j int) bool)
```

sort.Slice()以给定的比较函数排序切片，但并不保证排序的稳定性，想要稳定性，可以调用sort.SliceStable()。因为切片类型的广泛使用，调用sort.Slice()能满足大部分的需求，同时减少了自己实现接口需要实现的Len和Swap两个方法，写法上也更加精炼。一个例子如下：

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	people := []struct {
		Name string
		Age  int
	}{
		{"Gopher", 7},
		{"Alice", 55},
		{"Vera", 24},
		{"Bob", 75},
	}
	sort.Slice(people, func(i, j int) bool { return people[i].Name < people[j].Name })
	fmt.Println("By name:", people)

	sort.Slice(people, func(i, j int) bool { return people[i].Age > people[j].Age })
	fmt.Println("By age:", people)
}
//Output
By name: [{Alice 55} {Bob 75} {Gopher 7} {Vera 24}]
By age: [{Bob 75} {Alice 55} {Vera 24} {Gopher 7}]
```

值得注意的一点是，这里的排序结果依赖于less函数的内容，并不是默认升序，如上述程序。



