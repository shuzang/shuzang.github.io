# 算法-搜索


搜索是最常用的算法之一，但线性的搜索进行介绍没有太大的意义，本文介绍搜索中一种广为使用的方法：二分查找。

<!--more-->

## 1. 典型示例

标准的二分查找问题描述为：给定一个 `n` 个元素有序的（升序）整型数组 `nums` 和一个目标值 `target` ，写一个函数搜索 `nums` 中的 `target`，如果目标值存在返回下标，否则返回 `-1`。二分查找程序如下

```go
func search(nums []int, target int) int {
    if nums == nil {
        return -1
    }
    l,r := 0, len(nums)-1
    
    for l <= r {
        mid := (l + r)/2
        if nums[mid] == target {
            return mid
        }else if nums[mid] < target {
            l = mid+1
        }else{
            r = mid-1
        }
    }
    return -1
}
```

可以看出，二分查找是一种在每次比较后将查找空间一分为二的算法，每次需要查找集合中的索引或元素时，都应考虑二分查找。二分查找的前提条件是集合有序，因此在无序情况下需要先对集合进行排序。一个二分查找过程总结如下

1. 预处理：如果集合无序，进行排序
2. 二分查找：使用循环或递归在每次比较后将查找空间划分为两半
3. 后处理：在剩余空间中确定可行的候选者

## 2. 模板

尽管二分查找的思路基本一致，但在细节处理上还是存在一些不同，用以适应不同的情况， Leetcode 探索专题给出了三个模板，这里介绍如下。

### 2.1 模板1

模板1是二分查找的标准形式，用来查找可以通过访问数组中单个索引来确定的元素。

```go
func binarySearch(nums []int, target int) int {
    if len(nums) == 0 {
        return -1
    }
    left,right := 0,len(nums)-1
    for left <= right {
        mid := (left + right)/2
        if nums[mid] == target {
            return mid
        }else if nums[mid] < target {
            left = mid + 1
        }else {
            right = mid - 1
        }
    }
    return -1
}
```

求 x 的平方根可以使用该思路编写程序

```go
func mySqrt(x int) int {
    l,r := 0,x
    res := -1
    
    for l <= r {
        mid := (l + r) / 2
        if mid * mid <= x {
            res = mid
            l = mid + 1
        }else{
            r = mid - 1
        }
    }
    return res
}
```

### 2.2 模板2

模板2是一种改进的写法，用于查找过程与当前元素的左邻居或右邻居有关的情况

```go
func binarySearch(nums []int, target int) int {
    if len(nums) == 0 {
        return -1
    }
    left,right := 0,len(nums)
    for left < right {
        mid := (left + right)/2
        if nums[mid] == target {
            return mid
        }else if nums[mid] < target {
            left = mid + 1
        }else {
            right = mid - 1
        }
    }
    if left != len(nums) && nums[left] == target {
        return left
    }
    return -1
}
```

峰值元素是指其值大于左右相邻值的元素，给定一个输入数组 `nums`，其中 `nums[i] ≠ nums[i+1]`，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回任何一个峰值所在位置即可。你可以假设 `nums[-1] = nums[n] = -∞`。该题可以使用模板2解决

```go
func findPeakElement(nums []int) int {
    left,right := 0,len(nums)
    for left < right {
        mid := (l+r)/2
        if nums[mid] < nums[mid+1] {
            left = mid + 1
        }else {
            right = mid - 1
        }
    }
    return -1
}
```

### 2.3 模板3

模板3是另一种高级写法，用于查找过程和当前元素的左右邻居都有关的情况。

```go
func binarySearch(nums []int, target int) int {
    if len(nums) == 0 {
        return -1
    }
    left,right := 0,len(nums)-1
    for left+1 < right {
        mid := (left + right)/2
        if nums[mid] == target {
            return mid
        }else if nums[mid] < target {
            left = mid
        }else {
            right = mid
        }
    }
    if nums[left] == target {
        return left
    }
    if nums[right] == target {
        return right
    }
    return -1
}
```

## 3. 总结

三个模板的核心区别之处在于 mid 究竟是 +1 还是 -1，for 循环究竟是 < 还是 <=，正所谓二分查找**思路很简单，细节是魔鬼**。要仔细地区分，还是要在练习中不断体会。

因为二分查找是不断折半的，其实就相当于对一棵二叉树的查找，其时间复杂度为 $O(logn)$

同时，二分查找除了维持前后索引和中间索引三个值，不需要其它空间，因此空间复杂度为 $O(1)$
