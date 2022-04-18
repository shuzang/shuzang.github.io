# Golang深入学习2-string


Go 中字符串虽然作为基本数据类型，但本质依然是字符数组，本篇文章理解 Go 底层 string 类型是如何实现的，以及探讨它与 []byte 之间的关系。

<!--more-->

## 1. string

标准库 `builtin` 声明了 Go 所有的预定义标识符，其中对 string 的描述如下

> string is the set of all strings of 8-bit bytes, conventionally but not necessarily representing UTF-8-encoded text. A string may be empty, but not nil. Values of string type are immutable.

意思是字符串是字节的一个序列，约定但不必须是 UTF-8 编码的文本。字符串可以为空但不能是nil，其值不可变。Go 中字符串的源码定义在 `src/runtime/string.go` 中，如下

```go
type stringStruct struct {
	str unsafe.Pointer
	len int
}
```

所以 Go 中字符串是一个结构体，其中包含两个字段，第一个字段 str 是个指针，第二个字段 len 是字符串长度。str 指针虽然是 unsafe.Pointer 类型，但它最后其实指向了一个 byte 类型的数组，如下

```go
//go:nosplit
func gostringnocopy(str *byte) string {
	ss := stringStruct{str: unsafe.Pointer(str), len: findnull(str)}
	s := *(*string)(unsafe.Pointer(&ss))
	return s
}
```

所以我们理解了字符串的赋值其实是指针的复制，同时我们还注意到字符串长度其实调用了 findnull 函数

```go
func findnullw(s *uint16) int {
	if s == nil {
		return 0
	}
	p := (*[maxAlloc/2/2 - 1]uint16)(unsafe.Pointer(s))
	l := 0
	for p[l] != 0 {
		l++
	}
	return l
}
```

在 findnull 的实现中，`maxAlloc` 是**允许用户分配的最大虚拟内存空间**。在 64 位，理论上可分配最大 `1 << heapAddrBits` 字节。在 32 位，最大可分配小于 `1 << 32` 字节。所以，求长度的逻辑是：如果指针悬空，那么字符串长度为0，否则将指针转换为一个字符数组的指针，然后判断这个字符数组的每个值是否存在，第一个为0的值对应的索引就是字符串的长度。

字符串的值不可改变这个特性是通过禁止访问 str 指针指向的内存的值实现的，但 str 指针本身的值是可以改变的，也就是说它指向的内存区域可以改变，所以字符串可以重复赋值

```go
s := "hello" // str 指针指向"hello"的内存
s = "world" // str 指针指向"world"的内存
```

字符串同时也支持切片操作，我们可以理解为 str 的重新赋值和 len 的重新计算，比如下面的语句中，hello 和 world 其实都指向 s 所指向的内存区域，只是指针的位置不一样。

```go
s := "hello, world"
hello := s[:5]
world := s[7:]
```

最后，虽然字符串底层指向一个 byte 数组，单独访问其元素得到的类型也是 byte，但使用 for range 语法遍历时，单个值的类型却是 rune。

```go
func main() {
	s := "hello"
	fmt.Printf("%T ", s[1])
	for _, v := range s {
		fmt.Printf("%T", v)
		break
	}
}
// Output
uint8 int32
```

这里主要是因为 Go 专门做了一个解码操作，如下，注意这里的代码不是真的底层实现，只是用来说明逻辑的

```go
func forOnString(s string, forBody func(i int, r rune)) {
    for i := 0; len(s) > 0; {
        r,size := utf8.DecodeRuneInString(s)
        forBody(i,r)
        s = s[size:]
        i += size
    }
}
```

## 2. 转换

由以上可知字符串单个字符可能是 byte 或 rune，这也是我们使用字符串时经常做的强制类型转换。它们隐含者内存的重新分配，代价可能是不一样的，所以这里研究一下。

### 2.1 string->[]byte

string 转换 []byte，源码实现如下

```go
func stringtoslicebyte(buf *tmpBuf, s string) []byte {
	var b []byte
	if buf != nil && len(s) <= len(buf) {
		*buf = tmpBuf{}
		b = buf[:len(s)]
	} else {
		b = rawbyteslice(len(s))
	}
	copy(b, s)
	return b
}

// rawbyteslice allocates a new byte slice. The byte slice is not zeroed.
func rawbyteslice(size int) (b []byte) {
	cap := roundupsize(uintptr(size))
	p := mallocgc(cap, nil, false)
	if cap != uintptr(size) {
		memclrNoHeapPointers(add(p, uintptr(size)), cap-uintptr(size))
	}

	*(*slice)(unsafe.Pointer(&b)) = slice{p, size, int(cap)}
	return
}
```

我们可以看到其实做了一次内存的重新分配，得到了新的字符数组 b，然后将 s 复制给 b。至于 copy 函数可以直接把 string 复制给 []byte，是因为 go 源码单独实现了一个`slicestringcopy`函数来实现，具体可以看`src/runtime/slice.go`。

### 2.2 []byte->string

[]byte 转换 string，源码如下

```go
func slicebytetostring(buf *tmpBuf, ptr *byte, n int) (str string) {
	if n == 0 {
		// Turns out to be a relatively common case.
		// Consider that you want to parse out data between parens in "foo()bar",
		// you find the indices and convert the subslice to string.
		return ""
	}
	if raceenabled {
		racereadrangepc(unsafe.Pointer(ptr),
			uintptr(n),
			getcallerpc(),
			funcPC(slicebytetostring))
	}
	if msanenabled {
		msanread(unsafe.Pointer(ptr), uintptr(n))
	}
	if n == 1 {
		p := unsafe.Pointer(&staticuint64s[*ptr])
		if sys.BigEndian {
			p = add(p, 7)
		}
		stringStructOf(&str).str = p
		stringStructOf(&str).len = 1
		return
	}

	var p unsafe.Pointer
	if buf != nil && n <= len(buf) {
		p = unsafe.Pointer(buf)
	} else {
		p = mallocgc(uintptr(n), nil, false)
	}
	stringStructOf(&str).str = p
	stringStructOf(&str).len = n
	memmove(p, unsafe.Pointer(ptr), uintptr(n))
	return
}

func stringStructOf(sp *string) *stringStruct {
	return (*stringStruct)(unsafe.Pointer(sp))
}
```

该转换的思路是新分配 s，然后将 b 复制给它，所以依然有内存的重新分配。

### 2.3 string->[]rune

源码如下，由于 byte 和 rune 类型的差异，比如进行内存的重新分配。

```go
func stringtoslicerune(buf *[tmpStringBufSize]rune, s string) []rune {
	// two passes.
	// unlike slicerunetostring, no race because strings are immutable.
	n := 0
	for range s {
		n++
	}

	var a []rune
	if buf != nil && n <= len(buf) {
		*buf = [tmpStringBufSize]rune{}
		a = buf[:n]
	} else {
		a = rawruneslice(n)
	}

	n = 0
	for _, r := range s {
		a[n] = r
		n++
	}
	return a
}

// rawruneslice allocates a new rune slice. The rune slice is not zeroed.
func rawruneslice(size int) (b []rune) {
	if uintptr(size) > maxAlloc/4 {
		throw("out of memory")
	}
	mem := roundupsize(uintptr(size) * 4)
	p := mallocgc(mem, nil, false)
	if mem != uintptr(size)*4 {
		memclrNoHeapPointers(add(p, uintptr(size)*4), mem-uintptr(size)*4)
	}

	*(*slice)(unsafe.Pointer(&b)) = slice{p, size, int(mem / 4)}
	return
}
```

### 2.4 []rune->string

源码如下，内存分配没得跑。

```go
func slicerunetostring(buf *tmpBuf, a []rune) string {
	if raceenabled && len(a) > 0 {
		racereadrangepc(unsafe.Pointer(&a[0]),
			uintptr(len(a))*unsafe.Sizeof(a[0]),
			getcallerpc(),
			funcPC(slicerunetostring))
	}
	if msanenabled && len(a) > 0 {
		msanread(unsafe.Pointer(&a[0]), uintptr(len(a))*unsafe.Sizeof(a[0]))
	}
	var dum [4]byte
	size1 := 0
	for _, r := range a {
		size1 += encoderune(dum[:], r)
	}
	s, b := rawstringtmp(buf, size1+3)
	size2 := 0
	for _, r := range a {
		// check for race
		if size2 >= size1 {
			break
		}
		size2 += encoderune(b[size2:], r)
	}
	return s[:size2]
}

func rawstringtmp(buf *tmpBuf, l int) (s string, b []byte) {
	if buf != nil && l <= len(buf) {
		b = buf[:l]
		s = slicebytetostringtmp(&b[0], len(b))
	} else {
		s, b = rawstring(l)
	}
	return
}

func rawstring(size int) (s string, b []byte) {
	p := mallocgc(uintptr(size), nil, false)

	stringStructOf(&s).str = p
	stringStructOf(&s).len = size

	*(*slice)(unsafe.Pointer(&b)) = slice{p, size, size}

	return
}
```

## 3. 总结

1. string 和 []byte，string 和 []rune 的转换都会进行内存的重新分配，有一定代价；
2. 直接访问 string 中的成员，类型为 byte，使用 for range 结构，类型为 rune；
3. 需要修改 string 中的成员时，需要转换 []byte；
