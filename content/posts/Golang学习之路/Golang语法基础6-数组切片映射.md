---
title: Golang语法基础6-数组、切片与映射
date: 2019-11-18
lastmod: 2020-06-16
tags: [Go语法]
categories: [Golang学习之路]
slug: Golang syntax 6-array slice and map
---

只有基本的数据类型无法适应所有的情况，当需要存储和处理大量数据时，通常会使用数组、映射、链表等数据结构，称之为容器(container)。在Go语言中，由于数组不够灵活，增加了切片(slice)类型，切片也是一种容器。

C语言没有提供容器封装，开发者需要自己根据性能需求进行封装，或者使用第三方提供的容器。C++ 语言的容器通过标准库提供，如 vector 对应数组，list 对应双链表，map 对应映射等。

本篇只介绍数组和切片，映射类型放在下篇介绍，链表类型在数据结构中介绍。

## 1. 数组

数组是有限个相同类型的数据的集合，Go语言中数组声明的格式为

```go
var identifier [len]type
```

数组元素可以是任意基本类型，数组本身，结构体甚至接口（空），当元素类型是数组本身时，即为二维或多维数组。

应注意，数组长度也是数组类型的一部分，`[5]int`和`[10]int`是两个不同类型。

数组元素通过索引来读取或修改，不同于字符串，数组是可变的。索引的范围从`0`到`len(arr)-1`，内置函数`len()`可用来获取数组长度，数组长度最大为2Gb。

声明数组时所有的元素都会被自动初始化为元素类型的零值，初始化的过程是按照数组元素的顺序进行的。

当数组元素较少时，可以在声明时直接进行初始化，一些表达方式如下

```go
var arr1 = [3]int{1, 2, 3}
var arr2 = [10]int{1, 2, 3} //剩下的元素会自动补全，其值为0
var arr3 = [3]string{2:"test"}  //只有索引2被赋予了实际的值，其它元素都初始化为空字符串
```

数组长度的位置也可以使用省略号，此时数组长度会根据提供的值的个数自动推断

```go
arr := [...]int{1, 2, 3}
fmt.Println(len(arr)) //3
```

数组元素较多时使用for循环初始化

```go
var arr [100]int

//使用for循环初始化
for i:=0; i < len(arr); i++ {
    arr[i] = i * 2
}
```

Go语言中数组是一种值类型，而不像C语言是指向首元素的指针，因此可以使用内置函数`new()`来创建数组（`new()`用来创建值类型，返回所创建类型的指针）

```go
var arr1 = new([5]int)
```

这种方式和`var arr2 [5]int`的区别是，arr1的类型是`*[5]int`，而arr2的类型是`[5]int`，一个简单的式子可以帮助理解

```go
arr2 := *arr1
```

当像上式这样进行赋值时，我们把arr1的值做了一次拷贝，因此修改arr2不会对arr1产生影响。同理，函数中数组作为参数传入时，传入值类型的数组不会改变原值，但是直接传入数组作为参数，当数组很大时会消耗很多内存，可以传入数组的指针或使用切片来解决。以下是传入指针的例子。

```go
package main
import "fmt"
func f(a [3]int) { fmt.Println(a) }
func fp(a *[3]int) { fmt.Println(a) }

func main() {
    var ar = [3]int{1, 2, 3} 
	f(ar) 	// passes a copy of ar
	fp(&ar) // passes a pointer to ar
}
```

数组可以组装成多维数组，一个二维数组可以理解为一个数组类型的数组，以下演示一个二维数组的声明

```go
// 声明一个二维整型数组，两个维度的长度分别是 4 和 2
var array [4][2]int
// 声明并初始化数组中索引为 1 和 3 的元素
array = [4][2]int{1: {20, 21}, 3: {40, 41}}
// 声明并初始化数组中指定的元素
array = [4][2]int{1: {0: 20}, 3: {1: 41}}
```

## 2. 切片

切片（slice）就是动态数组，可以理解为对数组一个连续片段的引用，是一个引用类型。

### 2.1 声明与使用

切片声明的格式如下，基本就是去掉了数组声明中的长度

```go
var identifier []type
```

未初始化的切片默认为nil，长度为0。切片的初始化格式为

```go
var slice []type = arr[start:end]
```

表示slice是数组arr从start索引到end-1索引之间的元素构成的子集，切片的大小可以和数组相等，但应注意到终止索引的项并不包含在切片内。一些切片的方式如下

```go
var arr = [5]int{1,2,3,4,5}

s := arr[:] 
s := arr[0:5] //这两个切片都等于整个数组

s := arr[:3]
s := arr[0:3] //这两式输出都是[1,2,3]

s := arr[2:] 
s := arr[2:5] //这两式输出都是[3,4,5]
```

切片是可索引的，但切片的索引与原数组的索引不一定相同，如上例最后一行，s[0] = arr[2]。

切片的长度在运行时可修改，最小为0最大为相关数组的长度，具体的长度值可通过`len()`函数获得。

`cap()`函数可以计算切片的容量，也就是切片最长可以达到多少。举个例子，如果 s 是一个切片，`cap(s)` 就是从 `s[0]` 到数组末尾的数组长度。切片的长度永远不会超过它的容量。容量之所以从s[0]开始计数，是因为切片只能向后移动，任何试图获取切片第一个元素之前的数组元素的做法都会导致编译错误。

```go
arr := [5]int{1, 2, 3, 4, 5}
s := arr[2:4] //len(s)为2，cap(s)为3
```

两个直接创建切片的例子如下

```go
s := [3]int{1,2,3}[:]
x := []int{2,3,4,5,6}
```

但本质上这两者都是先创建的数组，然后取了与数组等长的切片。

注：切片本身已是引用，它没有指针，因此不要对它使用取地址符。

在上面的数组部分我们谈到当数组很大时，直接将数组作为参数传给函数会占用大量内存，因此我们介绍了如何传入数组的指针，这里我们再介绍如何传入切片：我们应当在函数中声明参数为切片类型，调用函数时，把数组分片，创建一个切片引用传递给该函数，示例如下

```go
func sum(a []int) int {
	s := 0
	for i := 0; i < len(a); i++ {
		s += a[i]
	}
	return s
}

func main() {
	var arr = [5]int{0, 1, 2, 3, 4}
	sum(arr[:])
}
```

数组作为值类型使用`new()`来创建，而切片作为引用类型，需要使用`make()`。

```go
var slice []type = make([]type, len)
slice := make([]type, len) //简写形式
```

其中第二个参数len是数组的长度，也是slice的初始长度，例如定义`s1 := make([]int, 10)`，那么`cap(s1) == len(s1) == 10`

也可以在声明时利用第三个参数指定切片容量

```go
slice := make([]type, len, cap)
```

因此，下面两种方法可生成相同切片

```go
make([]int, 50, 100)
new([100]int)[0:50]
```

字符串可以看作是一个不可变的字节数组，因此也可以切分为切片使用。

### 2.2 常用操作

由于切片的灵活性，会经常使用切片进行一些操作，这里简单介绍几种。

#### 重组

使用`make`创建切片的时候可以指定容量，因此必要时可以改变切片长度直到达到容量上限，改变切片长度的过程称为切片重组（reslice），如将切片扩展1位

```go
s = s[0:len(s)+1] //len(s)+1 <= cap(s)
```

#### 复制

增加切片的容量必须创建一个新的更大的切片并把原分片的内容都拷贝过来。切片拷贝使用`copy()`函数，函数原型如下

```go
copy(destSlice, srcSlice []T) int
```

作用是将 srcSlice 复制到 destSlice，两者类型必须一致，返回值为实际复制的元素个数。源地址和目标地址可能会有重叠。复制的元素个数是 srcSlice 和 dstSlice 的长度最小值。示例如下

```go
sl_from := []int{1, 2, 3}
sl_to1 := make([]int, 5)
sl_to2 := make([]int, 2)
n1 := copy(sl_to1, sl_from) // n1 = 3, s1_to1 = [1,2,3,0,0]
n2 := copy(sl_to2, sl_from) // n2 = 2, s1_to2 = [1,2]
```

#### 追加

追加也是一种切片扩容的方式，主要使用`append()`函数，函数原型是

```go
func append(s []T, x ...T) []T
```

作用是将0个或多个具有相同类型T的元素追加到切片s后面病并回新的切片，追加的元素类型需要和原切片的元素同类型。如果 s 的容量不足以存储新增元素，append 会分配新的切片来保证已有切片元素和新增元素的存储。因此，返回的切片可能已经指向一个不同的相关数组了。append 方法总是返回成功，除非系统内存耗尽了。

```go
sl3 := []int{1, 2, 3}
sl3 = append(sl3, 4, 5, 6) // sl2 = [1,2,3,4,5,6]
```

#### 删除

删除切片元素没有专用语法，需要使用切片本身的特性。分为三种情况：从开始位置删除，从中间位置删除，从末尾删除

**从开始位置删除**

直接移动数据指针

```go
a = []int{1, 2, 3}
a = a[1:] // 删除开头1个元素
a = a[N:] // 删除开头N个元素
```

不移动数据指针，而是将后面的数据向开头移动

```go
a = []int{1, 2, 3}
a = append(a[:0], a[1:]...) // 删除开头1个元素
a = append(a[:0], a[N:]...) // 删除开头N个元素
```

使用`copy()`函数

```go
a = []int{1, 2, 3}
a = a[:copy(a, a[1:])] // 删除开头1个元素
a = a[:copy(a, a[N:])] // 删除开头N个元素
```

**从中间位置删除**

对剩余的元素做一次整体移动，可以使用`copy()`或`append()`

```go
a = []int{1, 2, 3, ...}
a = append(a[:i], a[i+1:]...) // 删除中间1个元素
a = append(a[:i], a[i+N:]...) // 删除中间N个元素
a = a[:i+copy(a[i:], a[i+1:])] // 删除中间1个元素
a = a[:i+copy(a[i:], a[i+N:])] // 删除中间N个元素
```

**从末尾删除**

```go
a = []int{1, 2, 3}
a = a[:len(a)-1] // 删除尾部1个元素
a = a[:len(a)-N] // 删除尾部N个元素
```

删除开头和末尾都是删除中间的特殊情况

#### 插入

插入的一般方式是使用两次`append()`函数

```go
a = append(a[:i], append([]T{x}, a[i:]...)...) //在索引i的位置插入元素x
a = append(a[:i], append(make([]T, j), a[i:]...)...) //在索引i的位置插入长度为j的新切片
a = append(a[:i], append(b, a[i:]...)...) //在索引i的位置插入切片b的所有元素
```

映射(map)其实就是数据结构里的哈希表，但不少语言都已经把它作为了内置的数据类型。映射是元素对的无序集合，由键(key)和值(value)两部分构成，可以通过键快速查找值（比线性查找快，但实际上比通过数组或切片索引直接读取要慢）。

## 3. 映射

### 3.1  声明与初始化

Golang 中的 map 是引用类型，声明方法如下

```go
//语法格式
var mapname map[keytype]valuetype
//示例
var map1 map[string]int
```

凡是可以用 `==` 或 `!=` 操作符比较的类型都可以作为**键的类型**，比如string、int、float、只包含基本类型的结构体、指针和接口，而数组、切片以及含有数组切片的结构体无法作为键类型。**值的类型**是任意的，当值类型是一些复杂结构时，往往有比较特殊的用途，比如

1. 函数。值类型为函数时可以视作分支结构，key用来选择要执行的函数。

2. 空接口。我们可以用空接口作为值类型存储任意类型的值，只是在使用前需要做一次类型断言。

3. 切片。通过将值类型定义为切片类型，应对一个Key对应多个值的情况，示例如下

   ```go
   mp1 := make(map[int][]int)
   ```

map 可以动态增长，声明时不关心长度，使用时其长度使用内置函数`len()`获取。

未初始化的 map 值为 nil，如果此时试图给map添加元素会导致运行时错误，因此添加元素必须首先初始化。map初始化的方法有两种

1. 直接使用大括号，在数组与切片的初始化中已经见过这种方法，示例如下

   ```go
   var mapLit map[string]int
   mapLit = map[string]int{"one":1, "two":2}
   ```

2. 使用make，map是引用类型，因此使用make初始化。以make方式初始化其实相当于`mapLit := map[string]int{}`

   ```go
   mapLit := make(map[string]int)
   ```

虽然map可以动态增长，没有长度限制，但是也可以在一开始标明其初始容量

```go
mapLit := make(map[string]int, 100)
```

当map增长到容量上限后，继续增加新的键值对，map的大小会自动加1，因此容量对map并没有多大影响。

### 3.2 访问与删除map中的元素

如果 key1 是 map1 的 key，那么 `map1[key1]` 就是对应 key1 的值，map中就通过这种类似数组索引的方式访问元素

```go
val1 := map1[key1]
```

上式将 key1 对应的值赋给了 val1，但反过来，也可以通过这种形式设置对应key1的值，如下

```go
map1[key1] = val1
```

访问 map 中不存在的 key 会获得它所对应的值类型的空值，因此我们还需要有一种办法来判断键值对是否存在，这样才能区分到底是键值对本身不存在，还是值是空值。实际上通过键来访问值会返回两个结果，如下

```go
val1, ok := map1[key1]
```

当键值对存在时，ok 的值为 true，而当键值对不存在时，ok 的值为 false。如果只想判断某个键值对是否存在，可以将返回的真正的值设置为匿名变量

```go
_, ok := map1[key1]
```

map中元素的删除使用内置函数`delete()`，格式如下

```go
delete(mapname, keyname)
```

如果键值对不存在，删除操作也不会产生错误

```go
mapLit := map[string]int{"one": 1, "two": 2}
delete(mapLit, "one")
```

但 Golang 并没有提供清空 map 中所有元素的方法，清空 map 的唯一办法就是重新 make 一个新的 map

```go
mapLit := map[string]int{"one": 1, "two": 2}
mapLit = make(map[string]int)
```

### 3.3 遍历map

for-range可用于遍历map

```go
for key, value := range map1 {
	...
}
```

其中第一个返回值 key 是 map 中的 key 值，第二个返回值 value 则是 key 对应的 value 值。如果只关心值，可以省略键

```go
for _, value := range map1 {
	...
}
```

而如果只关心键，则可以省略值

```go
for key := range map1 {
	fmt.Printf("key is: %d\n", key)
}
```

还需要知道的一点是，for-range 结构虽然能遍历整个 map，但我们并不知道 map 中键值对排列的顺序，并不是按 key 的顺序排列的，也不是按 value 的顺序排列。

如果想要为 map 排序，那么就需要先通过遍历将 map 的所有数据复制到切片中，再对切片排序，最后打印出来

```go
// the telephone alphabet:
package main

import (
	"fmt"
	"sort"
)

var barVal = map[string]int{"alpha": 34, "bravo": 56, "charlie": 23}

func main() {
	fmt.Println("unsorted:")
	for k, v := range barVal {
		fmt.Printf("Key: %v, Value: %v ; ", k, v)
	}
	keys := make([]string, len(barVal))
	i := 0
	for k := range barVal {
		keys[i] = k
		i++
	}
	sort.Strings(keys)
	fmt.Println()
	fmt.Println("sorted:")
	for _, k := range keys {
		fmt.Printf("Key: %v, Value: %v ; ", k, barVal[k])
	}
}
//Output:
unsorted:
Key: alpha, Value: 34 ; Key: bravo, Value: 56 ; Key: charlie, Value: 23 ; 
sorted:
Key: alpha, Value: 34 ; Key: bravo, Value: 56 ; Key: charlie, Value: 23 ; 

```

上例按key进行了排序并输出，如果想要更好的显示，可以使用结构体切片

```go
type name struct {
	key string
	value int
}
```

### 3.4 map类型的切片

map类型的切片是一个很有意思的结构，构造它需要使用两次`make()`函数，第一次分配切片，第二次分配切片中的每个map元素

```go
package main
import "fmt"

func main() {
	// Version A:
	items := make([]map[int]int, 5)
	for i:= range items {
		items[i] = make(map[int]int, 1)
		items[i][1] = 2
	}
	fmt.Printf("Version A: Value of items: %v\n", items)

	// Version B: NOT GOOD!
	items2 := make([]map[int]int, 5)
	for _, item := range items2 {
		item = make(map[int]int, 1) // item is only a copy of the slice element.
		item[1] = 2 // This 'item' will be lost on the next iteration.
	}
	fmt.Printf("Version B: Value of items: %v\n", items2)
}
//Output:
Version A: Value of items: [map[1:2] map[1:2] map[1:2] map[1:2] map[1:2]]
Version B: Value of items: [map[] map[] map[] map[] map[]]

```

应该意识到，for-range 结构中，value 只是值的拷贝，对它做操作不会影响原值，因此上例中第二种写法是错误的，真正的 map 元素并没有得到初始化。

