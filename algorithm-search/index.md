# 算法-搜索


搜索是最常用的算法之一，但线性的搜索进行介绍没有太大的意义，本文介绍搜索中一种广为使用的方法：二分查找。

<!--more-->

## 1. 原始问题

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

## 2. 进阶

进阶的二分查找问题一般与当前元素的左右邻居有关。

我们以「第一个错误的版本」题目为例，解释这种情况，题目描述如下

> 你是产品经理，目前正在带领一个团队开发新的产品。不幸的是，你的产品的最新版本没有通过质量检测。由于每个版本都是基于之前的版本开发的，所以错误的版本之后的所有版本都是错的。
>
> 假设你有 `n` 个版本 `[1, 2, ..., n]`，你想找出导致之后所有版本出错的第一个错误的版本。
>
> 你可以通过调用 `bool isBadVersion(version)` 接口来判断版本号 `version` 是否在单元测试中出错。实现一个函数来查找第一个错误的版本。你应该尽量减少对调用 API 的次数。

**示例:**

```
给定 n = 5，并且 version = 4 是第一个错误的版本。

调用 isBadVersion(3) -> false
调用 isBadVersion(5) -> true
调用 isBadVersion(4) -> true

所以，4 是第一个错误的版本。 
```

线性扫描的思路很简单，但时间复杂度是 $O(n)$，二分查找是一种自然而然想到的办法，但实际的编程过程可能有一些问题。

我们将二分过程分为两个场景。场景一中，isBadVersion(mid) 返回 false，说明 mid 左侧（包括自身）所有版本都是正确的，因此令 left = mid + 1

```
场景一：isBadVersion(mid) => false

 1 2 3 4 5 6 7 8 9
 G G G G G G B B B       G = 正确版本，B = 错误版本
 |       |       |
left    mid    right
```

场景二中，`isBadVersion(mid)` 返回 true，说明 mid 右侧版本都不是第一个错误的版本，但不确定 mid 自身是不是第一个错误的版本，因此令 right = mid

```
场景二：isBadVersion(mid) => true

 1 2 3 4 5 6 7 8 9
 G G G B B B B B B       G = 正确版本，B = 错误版本
 |       |       |
left    mid    right
```

初始的条件设定是 left = 1, right = n，然后我们来讨论 for 循环结束条件的设定。

首先我们知道，由于 left 和 right 不断接近，最后一定能到达 left + 1 = right 的情况，由于 right 指向的一定是 false，所以可能有两种情况

```
情况一：
 G      B            G      B
 |      |      —>           |
left  right             left,right
情况二：
 B      B            B      B
 |      |      —>    |
left  right      left,right
```

此时 mid = (left + right)/2，由数学知识我们知道 mid == left，如果是情况一，left = mid + 1，也就是会等于 right，如果是情况二，right = mid，也就是会等于left。其实此时我们已经可以确定第一个错误版本，两种情况下，错误版本都是 left 或 right 指向的值，所以在 left == right 的时候就可以跳出循环了。最终的程序如下

```go
func firstBadVersion(n int) int {
    left,right := 1,n
    
    for left < right {
        mid := (left + right) / 2
        if isBadVersion(mid) {
            right = mid
        }else{
            left = mid + 1
        }
    }
    return left
}
```

那么，如果循环条件是 left <= right 会发生什么呢，此时恒定为以下情况，循环体内 right = mid，并没有变，所以就陷入了无限循环。

```
       B
       |
 left,mid,right
```

所以遇到 mid 和 左右邻居相关的情况时，采用上面这种思路来进行分析。

还有一些其它的情况，我们通常需要在查找完成后进行一些后处理。

## 3. 总结

正所谓二分查找**思路很简单，细节是魔鬼**，大多数区别之处在于 mid 究竟是 +1 还是 -1，for 循环究竟是 < 还是 <=，要仔细地区分，还是要在练习中不断体会。

因为二分查找是不断折半的，其实就相当于对一棵二叉树的查找，其时间复杂度为 $O(logn)$

同时，二分查找除了维持前后索引和中间索引三个值，不需要其它空间，因此空间复杂度为 $O(1)$
