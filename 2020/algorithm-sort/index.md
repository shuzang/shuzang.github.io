# 算法-排序


排序算法分为**内部排序**（待排序记录存放在内存中进行的排序过程）和**外部排序**（由于待排序记录数量大，以致内存一次不能容纳全部记录，在排序过程中需要对外存进行访问）。我们一般提到的基本都属于内部排序，一共可以分为5大类、8小类，如下

- 插入排序：直接插入排序、希尔排序
- 选择排序：简单选择排序、堆排序
- 交换排序：冒泡排序、快速排序
- 归并排序
- 基数排序

这篇文章对这 8 类排序算法进行详细说明，不过在此之前，先介绍排序稳定性的概念，因为时间复杂度、空间复杂度和排序稳定性是排序算法的三个重要度量。

排序稳定性其实就是相同的两个数在排序前后的先后位置是否发生了变化，具体的数学定义如下

> 假设 $r_i = r_j(1 \le i \le n, 1 \le j \le n, i \ne j)$，且在排序前的序列中 $r_i$ 领先于 $r_j$ （即 $i \le j$）。如果排序后 $r_i$ 仍领先于 $r_j$ ，则称所用的排序方法是稳定的；反之，若可能使得排序后的序列中 $r_j$ 领先于 $r_i$，则称所用的排序方法是不稳定的。

注意，下面我们所有的讨论都是基于递增排序的

## 1. 直接插入排序

核心思想是：将序列的第一个记录看成是一个有序的子序列，然后从第二个记录开始逐个进行插入，直至整个序列有序为止。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200402_2077144-b2538c0df361e00d.jpg)



可以使用两层循环实现，第一层循环遍历所有待比较数组元素，第二层循环将本轮选择的元素与已经排好序的元素进行比较。程序如下

```go 
func InsertSort(nums []int) []int {
  var i, j, tmp int // tmp用来临时存储本轮待比较元素
  for i = 1; i < len(nums); i++ {
	tmp = nums[i] // 临时存储本轮待比较元素
	for j = i; j > 0 && tmp < nums[j-1]; j-- {
	  nums[j] = nums[j-1] // 数组元素后移
	}
	nums[j] = tmp // 在正确的位置插入元素
  }
  return nums
}
```

时间复杂度为$O(N^2)$，最好情况，也就是待排序数组有序情况下，时间复杂度为$O(N)$，即单纯的遍历一遍。

空间复杂度为$O(1)$。

直接插入排序是稳定的。

## 2. 希尔排序

核心思想是：先将整个待排序记录序列分割成若干个子序列分别进行直接插入排序，待整个序列中的记录 “基本有序” 时，再对全体记录进行直接插入排序。实现步骤如下：

1. 按照增量分成子序列
2. 对子序列进行排序（直接插入）
3. 缩小增量，重复以上步骤，直到增量为1

增量选取待排序序列长度的一半，然后每次取当前增量的一半。

![](http://upload-images.jianshu.io/upload_images/2077144-5d740374bdf12582.jpg)



程序实现如下

```go
func ShellSort(nums []int) []int {
  var i, j int
  // dk为增量，初始为待排序数组长度的一半，然后每轮循环减半
  for dk := len(nums) / 2; dk > 0; dk /= 2 {
	// 内层循环是间隔为dk的插入排序
	for i = dk; i < len(nums); i++ {
	  tmp := nums[i]
	  for j = i; j >= dk && tmp < nums[j-dk]; j -= dk {
		nums[j] = nums[j-dk]
	  }
	  nums[j] = tmp
	}
  }
  return nums
}
```

希尔排序是不稳定的，其空间复杂度为$O(1)$，最好情况时间复杂度为$O(N)$，最坏情况为$O(N^2)$

但这是我们按折半方式选取增量时得到的结果，这种情况下，因为增量不互质，小增量很可能不起作用，某些循环的插入排序没有意义。我们可以通过选择较好的增量序列来优化算法，有两个有名的增量序列是

1. Hibbard增量序列：$dk = 2^k -1$，该增量序列最坏情况的时间复杂度为$O(N^{3/2})$，理论上平均时间复杂度为 $T_avg = O(N^{5/4})$
2. Sedgewick增量序列：$\{1,5,19,41,109,……\}$，计算公式为 $9 \times 4^i - 8 \times 2^i + 1$ 或 $4^i - 3 \times 2^i +1$。理论上其时间复杂度为 $T_avg = O(N^{7/6}), T_worst = O(N^{4/3})$

一般没有特别说明的情况下，增量序列可以选择... 3, 2, 1 或... 5, 3, 2, 1 这样从1开始的已知质数序列。程序实现时可以定义一个增量序列数组，然后遍历其内容。不过最常使用的依然是按折半方式选取的增量。

## 3. 简单选择排序

核心思想是：在要排序的一组数中，选出最小（或者最大）的一个数与第一个位置的数进行交换，然后在剩下的数中再找最小（或者最大）的与第二个位置的数交换，依次类推，直至第 n-1 个元素（倒数第二个数）与第 n 个元素 (最后一个数) 比较为止。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200402_2077144-c5a9c8acc5d75bb6.jpg)

程序实现如下

```go
func SelectionSort(nums []int) []int {
  var min int
  for i := 0; i < len(nums)-1; i++ {
	min = i
	for j := i + 1; j < len(nums); j++ {
	  if nums[j] < nums[min] {
		min = j
	  }
	}
	if min != i {
	  nums[i], nums[min] = nums[min], nums[i]
	}
  }
  return nums
}
```

选择排序是不稳定的，空间复杂度是$O(1)$，时间复杂度为$O(N^2)$

## 4. 堆排序

我们知道一个堆的根元素永远是最小值（小顶堆）或最大值（大顶堆），所以堆排序的核心思想是：首先将传入的所有元素构造成堆，然后每次取出根元素加入新的数组，最后对堆进行调整。直到新的元素都被取出后，新的数组就是原来数组的有序序列。

程序代码：

```go
// 堆排序
func HeapSort(nums []int) []int {
	result := []int{}
	// 初始化堆
	for i := (len(nums) - 1) / 2; i >= 1; i-- {
		nums = heapAdjust(nums, i)
	}
	//每次取出根元素并重新调整堆
	for len(nums)-1 != 0 {
		t := len(nums) - 1
		nums[1], nums[t] = nums[t], nums[1]
		result = append([]int{nums[t]}, result...)
		nums = nums[:t]
		nums = heapAdjust(nums, 1)
	}
	return result
}

// 堆调整函数，调整数组使其符合堆的要求
func heapAdjust(nums []int, start int) []int {
	var parent, child int
	for parent = start; parent*2 < len(nums); parent = child {
		child = parent * 2
		if child+1 < len(nums) && nums[child+1] > nums[child] {
			child++
		}
		if nums[child] <= nums[parent] {
			continue
		}
		nums[child], nums[parent] = nums[parent], nums[child]
	}
	return nums
}
```

堆排序是不稳定的，空间复杂度是 $O(1)$，时间复杂度是 $O（nlog_2n）$（注：一次堆调整的时间复杂度是 $O（log_2n）$）

## 5. 冒泡排序

核心思想是：在要排序的一组数中，自上而下的对相邻的两个数依次进行比较和调整，较大的数往下沉，较小的数往上冒。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200402_2077144-d98f8e37f35366d6.jpg)

冒泡是最简单的排序，代码实现如下

```go
func BubbleSort(nums []int) []int {
  for i := len(nums) - 1; i > 0; i-- {
	for j := 0; j < i; j++ {
	  if nums[j] > nums[j+1] {
		nums[j], nums[j+1] = nums[j+1], nums[j]
	  }
	}
  }
  return nums
}
```

冒泡是稳定的排序算法，空间复杂度是$O(1)$，最好情况下时间复杂度是$O(N)$，最坏情况下时间复杂度是$O(N^2)$。

上面的算法即使数组中途已经有序，也会一遍又一遍的判断，通过加入一个标志字段，可以节省一部分时间。

```go
func BubbleSort(nums []int) []int {
  for i := len(nums) - 1; i > 0; i-- {
    var flag bool  
	for j := 0; j < i; j++ {
	  if nums[j] > nums[j+1] {
		nums[j], nums[j+1] = nums[j+1], nums[j]
		flag = true
	  }
	}
    if !flag { //如果当前数组已有序，跳出循环
      break
    }
  }
  return nums
}
```

## 6. 快速排序

快排递归方法的核心思想是： 先在待排序的记录中选择一个数（通常是第一个数或者最后一个数）作为基准，设立两个指向标记，分别指向第一个元素和最后一个元素，然后进行第一趟划分，若最后一个数大于或等于基准值，则使指向最后一个数的指向标记向前移动；若最后一个数小于基准值，则将最后一个数与第一个数进行交换，此时将指向第一个数的指向标记向前移动。则第一趟得到的记录刚好以基准值为分界线，以上述同样的方法再对分界线两边的部分进行排序, 基准值就等于说找到了最终位置。

非递归方法的核心思想是：利用栈来存储两个指向标记的值，其它与递归的快速排序不相上下。

![一趟排序的过程](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200402_2077144-e6fddeb29cbeae9a.jpg)

![整个快速排序的过程](https://picped-1301226557.cos.ap-beijing.myqcloud.com/2077144-2a1063ef6b6e61ae.jpg)

递归程序代码如下

```go
/* 快速排序的递归解法, low 是第一个指向标记，high 是第二个指向标记。
初始参数传入时，low和high应该为第一个元素和最后一个元素
*/

func QuickSort(nums []int, low, high int) []int {
	if low < high {
		pivot := nums[low]
		i, j := low, high
		for i < j {
			for ; i < j && nums[j] >= pivot; j-- {
			}
			nums[i], nums[j] = nums[j], nums[i]
			for ; i < j && nums[i] <= pivot; i++ {
			}
			nums[j], nums[i] = nums[i], nums[j]
		}
		nums[i] = pivot
		nums = QuickSort(nums, low, i-1)
		nums = QuickSort(nums, i+1, high)
	}
	return nums
}
```

快速排序不稳定，平均情况下，时间复杂度为 $ O(nlogn) $，传入数组本身有序是最坏情况，时间复杂度为$O(n^2)$，空间复杂度（考虑递归调用的最大深度）在平均情况下为$O(logn)$，在最坏情况下为$O(n)$

## 7. 归并排序

核心思想是：将两个（或者两个以上）的有序表合并成一个新的有序表，即把待排序序列分为若干个子序列，每个子序列都是有序的。然后再把有序子序列合并为整体有序序列，两个子序列归并时，准备两个指针从两个子序列从头扫描到尾，将扫描到的元素按序插入结果数组。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200402_2077144-142c6fe3cc7df2ad.jpg)

两个子序列归并的代码如下

```go
// 归并两个子序列
func merge(left, right []int) []int {
	result := make([]int, 0)
	m, n := 0, 0
	for m < len(left) && n < len(right) {
		if left[m] > right[n] {
			result = append(result, right[n])
			n++
		} else {
			result = append(result, left[m])
			m++
		}
	}
	result = append(append(result, left[m:]...), right[n:]...)
	return result
}
```

归并排序的递归程序代码如下

```go
func MergeSort(nums []int) []int {
	if len(nums) < 2 {
		return nums
	}
	i := len(nums) / 2
	left := MergeSort(nums[:i])
	right := MergeSort(nums[i:])
	result := merge(left, right)
	return result
}
```

归并排序的非递归程序如下

```go
func MergeSort(nums []int) []int {
	for i := 1; i <= len(nums); i *= 2 {
		var j int
		for ; j <= len(nums)-2*i; j += 2 * i {
			result := merge(nums[j:i+j], nums[i+j:i*2+j])
			copy(nums[j:i*2+j], result)
		}
		if i+j < len(nums) {
			copy(nums[j:], merge(nums[j:i+j], nums[i+j:]))
		}
	}
	return nums
}
```

归并排序是稳定的，时间复杂度为$O（nlogn）$，空间复杂度为$O(n)$

## 8. 基数排序

以上的排序都是基于大小比较的，这种思路下的最坏时间复杂度的下界是$O(nlogn)$，想要更快，可以在比较大小的同时做一些其它的事情，这就是基数排序。

基数排序是基于桶排序的，假设我们有 N 个数字，每个整数的值在 0到99之间（于是有 M=100 个不同的值），线性时间内对它们进行排序的方法是：建立一个长度为100的数组，每个数组的值是一个指向链表的指针，遍历所有数字，将与数组下标相同的值添加到该数组元素对应的链表中，最后按下标从小到大输出所有非空的数组元素对应的链表。每个数组元素可以看作一个桶，桶里装着所有相同的值，这样排序的时间复杂度是$O(M+N)$

![桶排序](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200402_%E6%A1%B6%E6%8E%92%E5%BA%8F.png)

依然假设我们有 N=10 个整数，每个整数的值在0到999之间（于是有 M=1000个不同的值），如果使用同样的方法构造数组，则数组太大，这里使用的办法就是基数排序了。对于序列：64，8，216，512，27，729，0，1，343，125，首先建立长度为10的数组（这里的10就是基数），然后按照个位数字将数据放入与数组下标相同的桶，这是第一趟排序，也就是下图 Pass 1（第2行）行，此时所有元素个位有序；第二趟排序按十位数入桶，对应第3行到第5行，此时得到的序列是：0，1，8，512，216，125，27，729，343，64；最后一趟排序按百位数入桶，对应第6行到最后一行，最终得到序列0，1，8，27，64，125，216，343，512，729，此时得到的序列就是有序的。关键字就是每一位的数字，顺序是从个位到百位，这叫做低位优先。

![基数排序](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200402_%E5%9F%BA%E6%95%B0%E6%8E%92%E5%BA%8F.png)

基数排序是稳定的，时间复杂度为$O(d(n+rd)$，d 代表关键字长度，上例为3（每个数字最多3位），n 代表要排序的数的总数，上例为10，rd代表关键字基数，上例为10（0~9 共10种情况）。空间复杂度为$O(n)$

基数排序不仅用于数字的排序，也用于多关键字的排序，以扑克牌为例，有花色和面值两种关键字，排序如下

![多关键字排序](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200402_%E5%A4%9A%E5%85%B3%E9%94%AE%E5%AD%97%E6%8E%92%E5%BA%8F.png)

注意这里提到用了「主位优先」，在数字排序的例子里含义其实就是高位优先，与我们之前使用的低位优先正好相反。这里如果使用低位优先（也叫做次位优先），就是按面值建立桶。

## 9. 排序算法比较

所有排序算法时间, 空间复杂度汇总

| 排序方法 | 最好时间    | 平均时间     | 最坏时间    | 辅助空间    | 稳定性 |
| -------- | ----------- | ------------ | ----------- | ----------- | ------ |
| 直接插入 | $O(n)$      | $O(n^2)$     | $O(n^2)$    | $O(1)$      | 稳定   |
| 希尔排序 | $O(n)$      | $O(n^{1.3})$ | $O(n^2)$    | $O(1)$      | 不稳定 |
| 简单选择 | $O(n)$      | $O(n^2)$     | $O(n^2)$    | $O(1)$      | 不稳定 |
| 快速排序 | $O(nlogn)$  | $O(nlogn)$   | $O(n^2)$    | $O(logn)$   | 不稳定 |
| 冒泡排序 | $O(n)$      | $O(n^2)$     | $O(n^2)$    | $O(1)$      | 稳定   |
| 堆排序   | $O(nlogn)$  | $O(nlogn)$   | $O(nlogn)$  | $O(1)$      | 不稳定 |
| 归并排序 | $O(nlogn)$  | $O(nlogn)$   | $O(nlogn)$  | $O(n)$      | 稳定   |
| 基数排序 | $O(d(r+n))$ | $O(d(r+n))$  | $O(d(r+n))$ | $O(rd + n)$ | 稳定   |

对于链表来说，可执行的排序包括直接插入、简单选择、快排、冒泡、和归并。


