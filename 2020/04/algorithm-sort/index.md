# 算法-排序


排序算法又分为内部排序（待排序记录存放在计算机存储器中进行的排序过程）和外部排序（由于待排序记录数量大，以致内存一次不能容纳全部记录，在排序过程中需要对外存进行访问）。这里主要描述内部排序。内部排序可以分为（大的方面 5 类，小的方面 8 类）：插入排序（直接插入排序、希尔排序）、选择排序（简单选择排序、堆排序）、交换排序（冒泡排序、快速排序）、归并排序、基数排序；以下对这 8 类排序算法进行一一详细说明。

![](http://upload-images.jianshu.io/upload_images/2077144-6c368d2239e22de5.png)

排序稳定性的定义如下

> 假设 $k_i = k_j(1 \le i \le n, 1 \le j \le n, i \ne j)$，且在排序前的序列中 $r_i$ 领先于 $r_j$ （即 $i \le j$）。如果排序后 $r_i$ 仍领先于 $r_j$ ，则称所用的排序方法是稳定的；反之，若可能使得排序后的序列中 $r_j$ 领先于 $r_i$，则称所用的排序方法是不稳定的。

注意，下面我们所有的讨论都是基于递增排序的

## 1. 插入排序

核心思想是：将序列的第一个记录看成是一个有序的子序列，然后从第二个记录开始逐个进行插入，直至整个序列有序为止。

![](http://upload-images.jianshu.io/upload_images/2077144-b2538c0df361e00d.jpg)



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

时间复杂度为$O(N^2)$。最好情况，也就是待排序数组有序情况下，时间复杂度为$O(N)$，即单纯的遍历一遍。空间复杂度为$O(1)$。直接插入排序是稳定的。

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

但按折半方式选取增量时，因为增量不互质，小增量很可能不起作用，某些循环的插入排序没有意义。我们可以通过选择较好的增量序列优化算法，有两个有名的增量序列是

1. Hibbard增量序列：$dk = 2^k -1$，该增量序列最坏情况的时间复杂度为$O(N^{3/2})$，理论上平均时间复杂度为 $T_avg = O(N^{5/4})$
2. Sedgewick增量序列：$\{1,5,19,41,109,……\}$，计算公式为 $9 \times 4^i - 8 \times 2^i + 1$ 或 $4^i - 3 \times 2^i +1$。理论上其时间复杂度为 $T_avg = O(N^{7/6}), T_worst = O(N^{4/3})$

平时在没有说明的情况下，增量序列可以选择... 3, 2, 1 或... 5, 3, 2, 1 这样从1开始的已知质数序列。程序实现时可以定义一个增量序列数组，然后遍历其内容。

## 3. 冒泡排序

核心思想是：在要排序的一组数中，自上而下的对相邻的两个数依次进行比较和调整，较大的数往下沉，较小的数往上冒。

![](http://upload-images.jianshu.io/upload_images/2077144-d98f8e37f35366d6.jpg)

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
	var flag bool
	for i := len(nums) - 1; i > 0; i-- {
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

冒泡比较适合单链表排序

## 4. 选择排序

核心思想是：在要排序的一组数中，选出最小（或者最大）的一个数与第一个位置的数进行交换，然后在剩下的数中再找最小（或者最大）的与第二个位置的数交换，依次类推，直至第 n-1 个元素（倒数第二个数）与第 n 个元素 (最后一个数) 比较为止。

![](http://upload-images.jianshu.io/upload_images/2077144-c5a9c8acc5d75bb6.jpg)

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

## 5. 堆排序

我们知道一个堆的根元素永远是最小值（小顶堆）或最大值（大顶堆），所以堆排序的核心思想是：首先将传入的所有元素构造成堆，然后每次取出根元素加入新的数组，最后对堆进行调整。直到新的元素都被取出后，新的数组就是原来数组的有序序列。

程序代码：

![](http://upload-images.jianshu.io/upload_images/2077144-1f9f26963b67ec88.PNG)

堆调整程序图

![](http://upload-images.jianshu.io/upload_images/2077144-ecd81407e1740188.PNG)

堆排序是不稳定的，空间复杂度是 $O(1)$，时间复杂度是 $O（nlog_2n）$（注：一次堆调整的时间复杂度是 $O（log_2n）$）

## 6. 快速排序

快排递归方法的核心思想是： 先在待排序的记录中选择一个数（通常是第一个数或者最后一个数）作为基准，设立两个指向标记，分别指向第一个元素和最后一个元素，然后进行第一趟划分，若最后一个数大于或等于基准值，则使指向最后一个数的指向标记向前移动；若最后一个数小于基准值，则将最后一个数与第一个数进行交换，此时将指向第一个数的指向标记向前移动。则第一趟得到的记录刚好以基准值为分界线，以上述同样的方法再对分界线两边的部分进行排序, 基准值就等于说找到了最终位置。

非递归方法的核心思想是：利用栈来存储两个指向标记的值，其它与递归的快速排序不相上下。

![一趟排序的过程](http://upload-images.jianshu.io/upload_images/2077144-e6fddeb29cbeae9a.jpg)

![整个快速排序的过程](http://upload-images.jianshu.io/upload_images/2077144-2a1063ef6b6e61ae.jpg)

递归程序代码如下

```go
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
		QuickSort(nums, low, i-1)
		QuickSort(nums, i+1, high)
	}
	return nums
}
```

非递归程序代码如下

![](http://upload-images.jianshu.io/upload_images/2077144-7614627c5dcc2d7f.PNG)

![](http://upload-images.jianshu.io/upload_images/2077144-64e139257bbb4e16.PNG)

快速排序不稳定，平均情况下，时间复杂度为$O(nlogn)$，传入数组本身有序是最坏情况，时间复杂度为$O(n^2)$，空间复杂度（考虑递归调用的最大深度）在平均情况下为$O(logn)$，在最坏情况下为$O(n)$

## 7. 归并排序

思想：将两个（或者两个以上）的有序表合并成一个新的有序表，即把待排序序列分为若干个子序列，每个子序列都是有序的。然后再把有序子序列合并为整体有序序列。

![](http://upload-images.jianshu.io/upload_images/2077144-142c6fe3cc7df2ad.jpg)

程序代码：

![](http://upload-images.jianshu.io/upload_images/2077144-6c60ca8a6a985ffc.PNG)

![](http://upload-images.jianshu.io/upload_images/2077144-038df22e7facd4b3.PNG)

![](http://upload-images.jianshu.io/upload_images/2077144-7f33f1597dd79792.PNG)



归并排序是稳定的，时间复杂度为$O（nlogn）$，空间复杂度为$O(n)$

## 8. 基数排序

思想：【低位优先，链式队列】，将数字按位数划分出 n 个关键字，每次针对一个关键字进行排序，然后针对排序后的序列进行下个关键字的排序，循环至所有关键字都是用过，则排序完成。

程序代码：建立下标为 0--9 的队列，然后依次按关键字（低位优先）将数据入队，然后再出队。直至所有数据都排完。

时间复杂度：O（d*n）d 代表数字的个数，n 代表要排序的数的总数

空间复杂度：O（n）

稳定性：稳定

* * *

排序算法时间, 空间复杂度汇总:

![](http://upload-images.jianshu.io/upload_images/2077144-9785539c2b812eb4.jpg)

排序算法比较图


