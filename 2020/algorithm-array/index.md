# 算法-数组类问题


数组的访问时间为 $O(1)$，这是它最大的优势，但限于数组固定的大小，平常使用最多的是动态数组。在 Golang 中，其实就是[切片]^(slice)，动态数组的初始化、访问、修改、迭代、添加、删除等都是 Golang 语法的内容，这里不再介绍。事实上，一维数组的大部分问题都很好解决，我们在本文中仅介绍二维数组的一些典型问题，更多维的数组思路是相似，而且由于其复杂性，实际上很少出现。

## 1. 方向转换

下面是一个叫做「对角线遍历」的例子，在这个例子中，我们可以理解如何在二维数组中转换前进方向，这是一个很有用的技巧。

螺旋矩阵是指一个呈螺旋状的矩阵，它的数字由第一行开始到右边不断变大，向下变大，向左变大，向上变大，如此循环。给定m和n，返回一个大小为m*n的螺旋矩阵

**示例：**

```in
input:
3 3
Output:
1 2 3
8 9 4
7 6 5
```

**思路：**

1. 每一步走完，判断是否超出矩阵范围，若超出，计算向哪个方向转，无论是否转向，计算下一步所在位置的x,y坐标
2. 螺旋矩阵的转向顺序是固定的，为`右->下->左->上`

**代码：**

```go
func spiralArray(m, n int) {
	var result [][]int
	for i := 0; i < m; i++ {
		result = append(result, make([]int, n))
	}
    //x,y为当前坐标，d为方向
	x, y, d := 0, 0, 0
    //以“右->下->左->上”顺序循环，dx,dy是每一种转向的坐标变化方式
	dx := []int{0, 1, 0, -1}
	dy := []int{1, 0, -1, 0}
	for t := 0; t < m*n; t++ {
		result[x][y] = t
		nx, ny := x+dx[d], y+dy[d] //计算下一个坐标
		if nx < 0 || nx >= m || ny < 0 || ny >= n || result[nx][ny] != 0 {
			d = (d + 1) % 4 //下一个坐标有问题，采用下一种转向方式
			nx = x + dx[d]
			ny = y + dy[d]
		}
		x = nx
		y = ny
	}
	w := bufio.NewWriter(os.Stdout)
	for i := 0; i < m; i++ {
		for j := 0; j < n; j++ {
			w.WriteString(fmt.Sprintf("%d", result[i][j]))
			if j != n-1 {
				w.WriteString(fmt.Sprintf(" "))
			}
		}
		w.WriteString(fmt.Sprintf("\n"))
	}
	w.Flush()
}
```

## 2. 对撞指针

对撞指针是数组类问题另一个常用的技巧，其含义是：将指针 i 和 j 分别指向数组第一个元素和最后一个元素，然后指针 i 不断向前，指针 j 不断递减，直到 i == j（终止条件是可以根据题目变化的）。

下面我们以两数之和为例给出一个对撞指针使用的示范。题目描述如下：

```
给定一个已按照升序排列的有序数组，找到两个数使得它们相加之和等于目标数。函数应该返回这两个下标值 index1 和 index2，其中 index1 必须小于 index2。

说明：
- 返回的下标值（index1 和 index2）不是从零开始的。
- 你可以假设每个输入只对应唯一的答案，而且你不可以重复使用相同的元素

示例
输入: numbers = [2, 7, 11, 15], target = 9
输出: [1,2]
解释: 2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。
```

普通的办法需要两层循环，时间复杂度是$O(n^2)$，利用对撞指针，我们可以将复杂度降低到 $O(n)$

```go
func twoSum(numbers []int, target int) []int {
    i,j := 0,len(numbers)-1
    for i < j {
        if numbers[i] + numbers[j] > target {
            j--
        }else if numbers[i] + numbers[j] < target {
            i++
        }else{
            return []int{i+1,j+1}
        }
    }
    return nil
}
```

## 3. 滑动窗口

对撞指针是两个指针向中间靠拢，滑动窗口是两个指针向同一侧移动，如果还记得计算机网络的内容，这里就非常容易理解。滑动窗口同样可以将时间复杂度控制在 $O(n)$ 级别。

我们以「长度最小的子数组」为例给出一个滑动窗口解题的例子，题目描述如下

```go
给定一个含有 n 个正整数的数组和一个正整数 s ，找出该数组中满足其和 ≥ s 的长度最小的子数组，并返回其长度。如果不存在符合条件的子数组，返回 0。

示例
输入：s = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
```

暴力法的复杂度显然是 $O(n^2)$，我们尝试用滑动窗口求解

```go
func minSubArrayLen(s int, nums []int) int {
    n := len(nums)
   
    i,j := 0,0
    sum := 0
    min := n+1
    
    for i < n {
        if j < n && sum < s {
            sum += nums[j]
            j++
        }else{
            sum -= nums[i]
            i++
        }
        if sum >= s && j-i < min {
            min = j-i
        }
    }
    if min == n+1 {
        return 0
    }
    return min
}
```

滑动窗口最重要的是确定何时移动两个指针，当然，边界和初始值也非常重要。

当然，总体来看的话，其实后面两种方法都属于双指针的技巧。

---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2020/algorithm-array/  

