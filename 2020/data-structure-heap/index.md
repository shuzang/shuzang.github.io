# 数据结构-堆


普通的队列是一种先进先出的数据结构，在此基础上，还有一种叫做 **优先队列** 的结构。顾名思义，优先队列就是具有优先级的队列，其中，元素被赋予优先级，具有最高优先级的元素将最先被访问。

<!--more-->

优先级队列的一个典型使用场景是计算机的进程调度，在进程调度算法中有一种称为优先级法，就是使用优先队列这种结构。优先队列可以使用（有序）数组或（有序）链表实现，但最常用的实现方法是 **堆**。

注：这里的堆不是堆栈

## 1. 什么是堆

堆是一棵树，其中每个节点的值都大于等于/小于等于其子孙节点的值。每个节点的值都大于等于其子孙节点值的堆叫做大顶堆，否则叫做小顶堆。

习惯上，不加限定提到「堆」时往往指二叉堆。二叉堆是一棵用数组表示的完全二叉树，它满足堆的性质：

1. 每个结点中存有一个元素（或者说一个权值）
2. 任一结点的权值是其子树所有结点的最大值（或最小值）

下面是大顶堆和小顶堆的一个例子

![大顶堆与小顶堆示例](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200311_8EHOyj.png)

二叉堆和二叉搜索树虽然乍一看有相似之处，但它们的区别很大：

1. 结点顺序。在二叉搜索树中，左子树中的节点必须比当前节点小，右子树中的节点必须比当前节点大，但最大堆中不论左右子树，节点都必须比当前节点小或者大；
2. 平衡：二叉搜索树必须在平衡状态下，大部分操作复杂度才能达到O(log n)，而二叉堆一定是平衡的，构建树的方式决定了它的复杂度。
3. 搜索：二叉树搜索很快，堆则不一定，因为堆的主要任务不是搜索，而是及时获取最大或最小结点。
4. 二叉搜索树必须是严格的大小关系，堆可以有等于。

## 2. 堆的基本操作

堆的最基本操作是插入和删除，除此之外，给定一组值快速构建堆也是常见的问题。在Go中，我们将堆定义为一个切片，为了便于操作，下标从1开始

```go
var heap = make([]int, 1)
```

**注：下面的例子均以最大堆为例**

### 2.1 堆的插入

插入操作是指向二插堆中插入一个元素，其基本流程如下

1. 将插入的元素放在数组的最后一个元素
2. 如果新插入的结点的值大于它父亲节点的值，就与之交换，重复此过程直到不满足或者到根。这一过程叫做**向上调整**

一个例子如下

![堆的插入](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200311_8EbZ01.png)

程序实现如下

```go
func Insert(heap []int], key int) {
    heap = append(heap, key)
    child = len(heap)-1
    for {
        parent := (child-1)/2
        if parent < 0 || heap[parent] >= heap[child] {
            break
        }
        heap[parent],heap[child] = heap[child],heap[parent]
        child = parent
    }
}
```

### 2.2 堆的删除

删除操作指删除堆中最大的元素，即删除根结点。其基本流程如下

1. 读取根结点（最大值）元素
2. 交换根结点和最后一个结点，然后删除最后一个结点
3. 在新的根结点的儿子中，找一个最大的，与该结点交换，重复此过程直到底层。这一过程叫做**向下调整**

一个例子如下

![堆的删除](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200311_8EbGnA.png)

程序实现如下

```go
func Delete(heap []int) int {
    var parent, child int
    Max,t := heap[0],len(heap)-1
    heap[0], heap[t] = heap[t], heap[0]
    heap = heap[:t]
    for parent = 0; parent*2 +1 < len(heap); parent = child {
        child = parent * 2 + 1
        if child + 1 < len(heap) && heap[child+1] > heap[child] {
            child++
        }
        if heap[child] <= heap[parent] {
            break
        }
        heap[child], heap[parent] = heap[parent], heap[child]        
    } 
    return Max
}
```

### 2.3 堆的建立

通过将一组元素逐个插入到一个初始为空的堆，可以建立一个最大堆，但这种方法的复杂度为$O(n log_2n)$。实际上，这里存在一种可以在线性时间内$O(log_2n)$建立堆的方法，流程如下

1. 将 N 个元素按输入顺序构造切片，使其满足完全二叉树的特性
2. 从倒数第一个非叶子结点起，逐个往前按照**向下调整**的思路调整各结点位置，使其满足堆的特性

一个例子如下

![堆的建立](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200311_8Ebah8.png)

程序实现如下

```go
//假设传入的数组已经按顺序填好元素
func build(heap []int) {
    for i := (len(heap)-2)/2; i >= 0; i-- {
        for parent = i; parent * 2 + 1< len(heap); parent = child {
           child = parent * 2 + 1
           if child + 1 < len(heap) && heap[child+1] > heap[child] {
               child++
           }
           if heap[child] <= heap[parent] {
               break
           }
           heap[child], heap[parent] = heap[parent], heap[child]        
       }        
    } 
}
```

## 3. 实现优化

实际上我们注意到，关于堆的操作，最重要的有两个，一个是向上调整，用于插入，一个是向下调整，用于删除和建立堆。我们将这两个操作提取出来

```go
// 向上调整
func up(heap []int) {
    child := len(stones)-1
    for {
        parent := (child-1) / 2
        if parent < 0 || heap[parent] >= heap[child] {
            break
        }
        heap[parent],heap[child] = heap[child],heap[parent]
        child = parent
    }
}

// 向下调整
func down(heap []int, i int) {
    var parent,child int
    for parent := i; parent * 2 + 1 < len(heap); parent = child {
        child = parent * 2 + 1
        if child + 1 < len(heap) && heap[child+1] > heap[child] {
            child++
        }
        if heap[child] <= heap[parent] {
            break
        }
        heap[child],heap[parent] = heap[parent],heap[child]
    }
}
```

这样，插入、删除和建堆函数可以简化为

```go
// 插入
func Insert(heap []int], key int) {
    heap = append(heap, key)
    up(heap)
}

// 删除
func Delete(heap []int) int {
    Max := heap[0]
    heap[0]= heap[len(heap)-1]
    heap = heap[:len(heap)-1]
    down(heap, 0) 
    return Max
}

// 建堆
func build(heap []int) {
    for i := (len(heap)-2)/2; i >= 0; i-- {
        down(heap,i)       
    } 
}
```

Go 的标准库也提供了堆的相关实现，位于 container/heap 包中，该包提供了对任意类型（实现了heap.Interface接口）的堆操作。

heap 包默认使用最小堆，但在正式应用前需实现如下接口

```go
type Interface interface {
    sort.Interface
    Push(x interface{}) // 向末尾添加元素
    Pop() interface{}   // 从末尾删除元素
}
```

该接口的示例实现如下

```go
// An IntHeap is a min-heap of ints.
type IntHeap []int
func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *IntHeap) Push(x interface{}) {
    // Push and Pop use pointer receivers because they modify the slice's length,
    // not just its contents.
    *h = append(*h, x.(int))
}
func (h *IntHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}
```

实现了该接口后，就可以利用 Init 函数初始化一个堆，利用 Push 插入和利用 Pop 删除。相关的函数原型如下

```go
func Init(h Interface)
func Push(h Interface, x interface{})
func Pop(h Interface) interface{}
```

使用这些方法的一个示例如下

```go
// This example inserts several ints into an IntHeap, checks the minimum,
// and removes them in order of priority.
func Example_intHeap() {
    h := &IntHeap{2, 1, 5}
    heap.Init(h)
    heap.Push(h, 3)
    fmt.Printf("minimum: %d\n", (*h)[0])
    for h.Len() > 0 {
        fmt.Printf("%d ", heap.Pop(h))
    }
    // Output:
    // minimum: 1
    // 1 2 3 5
}
```

## 4. 优先队列

如我们开头所说，堆用来实现优先队列，下面给出一个利用标准库实现优先队列的例子

```go
// This example demonstrates a priority queue built using the heap interface.
package heap_test
import (
    "container/heap"
    "fmt"
)
// An Item is something we manage in a priority queue.
type Item struct {
    value    string // The value of the item; arbitrary.
    priority int    // The priority of the item in the queue.
    // The index is needed by update and is maintained by the heap.Interface methods.
    index int // The index of the item in the heap.
}
// A PriorityQueue implements heap.Interface and holds Items.
type PriorityQueue []*Item
func (pq PriorityQueue) Len() int { return len(pq) }
func (pq PriorityQueue) Less(i, j int) bool {
    // We want Pop to give us the highest, not lowest, priority so we use greater than here.
    return pq[i].priority > pq[j].priority
}
func (pq PriorityQueue) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
    pq[i].index = i
    pq[j].index = j
}
func (pq *PriorityQueue) Push(x interface{}) {
    n := len(*pq)
    item := x.(*Item)
    item.index = n
    *pq = append(*pq, item)
}
func (pq *PriorityQueue) Pop() interface{} {
    old := *pq
    n := len(old)
    item := old[n-1]
    item.index = -1 // for safety
    *pq = old[0 : n-1]
    return item
}
// update modifies the priority and value of an Item in the queue.
func (pq *PriorityQueue) update(item *Item, value string, priority int) {
    item.value = value
    item.priority = priority
    heap.Fix(pq, item.index)
}
// This example creates a PriorityQueue with some items, adds and manipulates an item,
// and then removes the items in priority order.
func Example_priorityQueue() {
    // Some items and their priorities.
    items := map[string]int{
        "banana": 3, "apple": 2, "pear": 4,
    }
    // Create a priority queue, put the items in it, and
    // establish the priority queue (heap) invariants.
    pq := make(PriorityQueue, len(items))
    i := 0
    for value, priority := range items {
        pq[i] = &Item{
            value:    value,
            priority: priority,
            index:    i,
        }
        i++
    }
    heap.Init(&pq)
    // Insert a new item and then modify its priority.
    item := &Item{
        value:    "orange",
        priority: 1,
    }
    heap.Push(&pq, item)
    pq.update(item, item.value, 5)
    // Take the items out; they arrive in decreasing priority order.
    for pq.Len() > 0 {
        item := heap.Pop(&pq).(*Item)
        fmt.Printf("%.2d:%s ", item.priority, item.value)
    }
    // Output:
    // 05:orange 04:pear 03:banana 02:apple
}
```

## 参考

[1] [Golang: 详解container/heap](https://ieevee.com/tech/2018/01/29/go-heap.html)

[2] [B站-浙江大学数据结构课程](https://www.bilibili.com/video/av43521866?p=51)

---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2020/data-structure-heap/  

