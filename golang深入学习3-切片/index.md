# Golang深入学习3-切片


本篇理解切片的底层实现和扩容方式。

<!--more-->

## 1. 实现

切片的定义位于 `src/runtime/slice.go`，如下

```go
type slice struct {
    array unsafe.Pointer   // 用来存储实际数据的数组指针，指向一块连续的内存
    len   int              // 切片中元素的数量
    cap   int              // array数组的长度
}
```

所以可见切片和字符串很相似，实质都是一个指针，只不过除了长度 len 还有一个容量字段 cap。一个简单的图解如下

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20200724_3Ek0rd.png)

图中的 x 和 y 都是从数组 [5]int{2,3,5,7,11} 上获取的切片，也就是指向该数组的不同位置。

上篇介绍字符串的时候提到字符串虽然底层是指针，但不允许等于 nil，它的空值是空字符串 `""`。但切片是可以等于 nil 的，只要其底层指针等于 nil，一般情况是切片声明而未初始化的时候出现该情况，这个时候因为没有指向任何内存区域，切片的长度和容量信息都是无效的，不过还是可以获取到。

注：因为切片等于 nil 一般意味着没有初始化，也就没有使用的价值，所以很少将切片直接和 nil 作比较，使用更多的还是判断切片的长度是否为0（len(s) == 0）

```go
func main() {
	var s []int
	fmt.Println(s, len(s), cap(s))
	fmt.Println(s == nil)
}
// Output
[] 0 0
true
```

切片一旦初始化，底层指针就指向了一个确定的内存区域，但指向的内存区域大小可以为0，也就是切片中没有任何元素，此时切片的长度也是 0，但和未初始化时得到的长度绝不是一个含义。

```go
func main() {
	var s []int
	s = []int{}
	fmt.Println(s, len(s), cap(s))
	fmt.Println(s == nil)
}
// Output
[] 0 0
false
```

## 2. 扩容

切片的长度是当前所包含的元素个数，容量是可容纳的最大元素个数。这里的含义是，初始化时指定的容量就代表在内存已经预分配了与容量相等的空间，其后访问、添加、删除切片的元素都和数组相似，只操作指针，不会造成内存重新分配。

```go
func main() {
	s := make([]int, 0, 3)
	fmt.Printf("%p ", s)
	fmt.Println(s, len(s), cap(s))
	s = append(s, 1)
	fmt.Printf("%p ", s)
	fmt.Println(s, len(s), cap(s))
}
// Output
0xc000124180 [] 0 3
0xc000124180 [1] 1 3
```

但是，如果追加的元素数量超过了容量，那么会导致内存的重新分配。

```go
func main() {
	s := make([]int, 0, 3)
	fmt.Printf("%p ", s)
	fmt.Println(s, len(s), cap(s))
	s = append(s, 1, 2, 3, 4)
	fmt.Printf("%p ", s)
	fmt.Println(s, len(s), cap(s))
}
// Output
0xc000124180 [] 0 3
0xc000146030 [1 2 3 4] 4 6
```

内存的重新分配就是切片的扩容，其逻辑是，为切片分配一块更大的内存，然后将旧切片的元素复制到新切片中。

一个很有意思的情况如下，将切片 s 赋值给一个新的切片 l，然后对原切片 s 进行扩容和修改，不会影响到切片 l

```go
func main() {
	s := make([]int, 3, 3)
	fmt.Printf("%p %v\n", s, s)
	l := s[:]
	s = append(s, 1)
	s[0] = 1
	fmt.Printf("%p %v\n", s, s)
	fmt.Printf("%p %v", l, l)
}
// Output
0xc000124180 [0 0 0]
0xc000146030 [1 0 0 1]
0xc000124180 [0 0 0]
```

最后一个值得注意的问题是切片每次扩容会扩大多少，这个逻辑位于 `src/runtime/slice.go` 文件中的 growslice 函数中，其中 old.len 是旧长度，old.cap 是旧容量，newcap 是新容量，cap 是需要的容量，

```go
// ...省略
newcap := old.cap
doublecap := newcap + newcap
if cap > doublecap {
    newcap = cap
} else {
    if old.len < 1024 {
        newcap = doublecap
    } else {
        // Check 0 < newcap to detect overflow
        // and prevent an infinite loop.
        for 0 < newcap && newcap < cap {
            newcap += newcap / 4
        }
        // Set newcap to the requested cap when
        // the newcap calculation overflowed.
        if newcap <= 0 {
            newcap = cap
        }
    }
}
// ...省略
```

简单描述就是：

1. 如果需要的容量超过原切片容量的两倍，直接使用需要的容量作为新容量；
2. 如果原切片的长度小于 1024，新切片的容量翻倍；
3. 如果原切片的长度大于1024，则每次增加25%，直到新容量超过所需要的容量；

第二条的翻倍倒是可以确认，但第一条和第三条经验证却不太符合

```go
func main() {
	// 1. 需要的容量超过原切片容量的两倍，新容量应为需要的容量5
	s1 := make([]int, 2)
	fmt.Printf("%p %v\n", s1, cap(s1))
	s1 = append(s1, 3, 4, 5)
	fmt.Printf("%p %v\n", s1, cap(s1))

	// 2. 原切片长度小于1024，新容量应当翻倍为4
	s2 := make([]int, 2)
	fmt.Printf("%p %v\n", s2, cap(s2))
	s2 = append(s2, 3)
	fmt.Printf("%p %v\n", s2, cap(s2))

	// 3. 原切片长度大于1024，新容量递增25%1次，应当为1500
	s3 := make([]int, 1200)
	fmt.Printf("%p %v\n", s3, cap(s3))
	s3 = append(s3, 3)
	fmt.Printf("%p %v", s3, cap(s3))
}
// Output
0xc0000120b0 2
0xc00000a390 6
0xc0000120f0 2
0xc0000104c0 4
0xc000100000 1200
0xc00010c000 1536
```

这是因为扩容的那一段核心源码后面还有一段新容量的处理过程，如下

```go
var overflow bool
	var lenmem, newlenmem, capmem uintptr
	// Specialize for common values of et.size.
	// For 1 we don't need any division/multiplication.
	// For sys.PtrSize, compiler will optimize division/multiplication into a shift by a constant.
	// For powers of 2, use a variable shift.
	switch {
	case et.size == 1:
		lenmem = uintptr(old.len)
		newlenmem = uintptr(cap)
		capmem = roundupsize(uintptr(newcap))
		overflow = uintptr(newcap) > maxAlloc
		newcap = int(capmem)
	case et.size == sys.PtrSize:
		lenmem = uintptr(old.len) * sys.PtrSize
		newlenmem = uintptr(cap) * sys.PtrSize
		capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
		overflow = uintptr(newcap) > maxAlloc/sys.PtrSize
		newcap = int(capmem / sys.PtrSize)
	case isPowerOfTwo(et.size):
		var shift uintptr
		if sys.PtrSize == 8 {
			// Mask shift for better code generation.
			shift = uintptr(sys.Ctz64(uint64(et.size))) & 63
		} else {
			shift = uintptr(sys.Ctz32(uint32(et.size))) & 31
		}
		lenmem = uintptr(old.len) << shift
		newlenmem = uintptr(cap) << shift
		capmem = roundupsize(uintptr(newcap) << shift)
		overflow = uintptr(newcap) > (maxAlloc >> shift)
		newcap = int(capmem >> shift)
	default:
		lenmem = uintptr(old.len) * et.size
		newlenmem = uintptr(cap) * et.size
		capmem, overflow = math.MulUintptr(et.size, uintptr(newcap))
		capmem = roundupsize(capmem)
		newcap = int(capmem / et.size)
	}
```

其中 et 是切片中的元素类型，sys.PtrSize 是一个指针的大小，64位系统中为8，主要调用的处理函数 roundupsize 来自 `src/runtime/msize.go` 文件，如下

```go
// Returns size of the memory block that mallocgc will allocate if you ask for the size.
func roundupsize(size uintptr) uintptr {
    // _MaxSmallSize   = 32768
	if size < _MaxSmallSize {
        // smallSizeMax    = 1024
		if size <= smallSizeMax-8 {
			return uintptr(class_to_size[size_to_class8[divRoundUp(size, smallSizeDiv)]])
		} else {
			return uintptr(class_to_size[size_to_class128[divRoundUp(size-smallSizeMax, largeSizeDiv)]])
		}
	}
    // _PageSize = 1 << 13
	if size+_PageSize < size {
		return size
	}
	return alignUp(size, _PageSize)
}

```

这里面涉及到了一些常量和两个函数，内容如下

```go
const (
	_MaxSmallSize   = 32768
	smallSizeDiv    = 8
	smallSizeMax    = 1024
	largeSizeDiv    = 128
	_NumSizeClasses = 67
	_PageShift      = 13
)

var class_to_size = [_NumSizeClasses]uint16{0, 8, 16, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 352, 384, 416, 448, 480, 512, 576, 640, 704, 768, 896, 1024, 1152, 1280, 1408, 1536, 1792, 2048, 2304, 2688, 3072, 3200, 3456, 4096, 4864, 5376, 6144, 6528, 6784, 6912, 8192, 9472, 9728, 10240, 10880, 12288, 13568, 14336, 16384, 18432, 19072, 20480, 21760, 24576, 27264, 28672, 32768}
var size_to_class8 = [smallSizeMax/smallSizeDiv + 1]uint8{0, 1, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7, 7, 8, 8, 9, 9, 10, 10, 11, 11, 12, 12, 13, 13, 14, 14, 15, 15, 16, 16, 17, 17, 18, 18, 18, 18, 19, 19, 19, 19, 20, 20, 20, 20, 21, 21, 21, 21, 22, 22, 22, 22, 23, 23, 23, 23, 24, 24, 24, 24, 25, 25, 25, 25, 26, 26, 26, 26, 26, 26, 26, 26, 27, 27, 27, 27, 27, 27, 27, 27, 28, 28, 28, 28, 28, 28, 28, 28, 29, 29, 29, 29, 29, 29, 29, 29, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31}
var size_to_class128 = [(_MaxSmallSize-smallSizeMax)/largeSizeDiv + 1]uint8{31, 32, 33, 34, 35, 36, 36, 37, 37, 38, 38, 39, 39, 39, 40, 40, 40, 41, 42, 42, 43, 43, 43, 43, 43, 44, 44, 44, 44, 44, 44, 45, 45, 45, 45, 46, 46, 46, 46, 46, 46, 47, 47, 47, 48, 48, 49, 50, 50, 50, 50, 50, 50, 50, 50, 50, 50, 51, 51, 51, 51, 51, 51, 51, 51, 51, 51, 52, 52, 53, 53, 53, 53, 54, 54, 54, 54, 54, 55, 55, 55, 55, 55, 55, 55, 55, 55, 55, 55, 56, 56, 56, 56, 56, 56, 56, 56, 56, 56, 57, 57, 57, 57, 57, 57, 58, 58, 58, 58, 58, 58, 58, 58, 58, 58, 58, 58, 58, 58, 58, 58, 59, 59, 59, 59, 59, 59, 59, 59, 59, 59, 59, 59, 59, 59, 59, 59, 60, 60, 60, 60, 60, 61, 61, 61, 61, 61, 61, 61, 61, 61, 61, 61, 62, 62, 62, 62, 62, 62, 62, 62, 62, 62, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 63, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 64, 65, 65, 65, 65, 65, 65, 65, 65, 65, 65, 65, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66, 66}


// divRoundUp returns ceil(n / a).
func divRoundUp(n, a uintptr) uintptr {
	// a is generally a power of two. This will get inlined and
	// the compiler will optimize the division.
	return (n + a - 1) / a
}

// alignUp rounds n up to a multiple of a. a must be a power of 2.
func alignUp(n, a uintptr) uintptr {
	return (n + a - 1) &^ (a - 1)
}
```

由此可以得到处理逻辑：对前面的过程得到的新容量 cap，处理后调用 roundupsize 处理，如果容量小于 32768，那么依赖 class_to_size、size_to_class8 和 size_to_class128 三个数组中的两个去调整，得到的结果再稍微处理一下作为新容量。

以前面出错的两个例子计算，首先是需要的容量超过原切片容量两倍的，预处理得到的新容量为 5，然后因为 64 位系统中 int 类型为 8 个字节，调用 roundupsize 的方式是 `capmem = roundupsize(uintptr(newcap) * sys.PtrSize)`，也就是乘以8再传给 roundupsize 处理，这时 roundupsize 得到的参数是 40，40 < 32768，40 < 1024-8，`divRoundUp(size, smallSizeDiv)` 计算得到 5，获取 size_to_class8[5] = 4，再获取 class_to_size[4] = 48，最后返回，经过 `newcap = int(capmem / sys.PtrSize)` 处理得到新容量 48/8 = 6

```go
func main() {
	// 1. 需要的容量超过原切片容量的两倍，新容量应为需要的容量5
	s1 := make([]int, 2)
	fmt.Printf("%p %v\n", s1, cap(s1))
	s1 = append(s1, 3, 4, 5)
	fmt.Printf("%p %v\n", s1, cap(s1))

}
// Output
0xc0000120b0 2
0xc00000a390 6
```

第二个例子中，预处理得到的新容量是 1500，int 型8个字节，将 1500*8 = 12000 传入 roundupsize，12000 < 32768，但是 12000 > 1024-8，经 `divRoundUp(size-smallSizeMax, largeSizeDiv)` 计算得到 86，获取 size_to_class128[86] = 55，再获取 class_to_size[55] = 10880，返回后经  `newcap = int(capmem / sys.PtrSize)` 处理得到新容量 12288/8 = 1536

```go
func main() {
	// 3. 原切片长度大于1024，新容量递增25%1次，应当为1500
	s3 := make([]int, 1200)
	fmt.Printf("%p %v\n", s3, cap(s3))
	s3 = append(s3, 3)
	fmt.Printf("%p %v", s3, cap(s3))
}
// Output
0xc000100000 1200
0xc00010c000 1536
```

当然，这个处理过程非常复杂，平常使用一般不会自己去做这么复杂的计算，我们仅仅需要大致估计扩容后的大小就可以了，所以可以简单的使用前面的三条规则做估计，这里重复一下

1. 如果需要的容量超过原切片容量的两倍，直接使用需要的容量作为新容量；
2. 如果原切片的长度小于 1024，新切片的容量翻倍；
3. 如果原切片的长度大于1024，则每次增加25%，直到新容量超过所需要的容量；
