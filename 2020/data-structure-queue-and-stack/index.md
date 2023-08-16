# 数据结构-队列与栈


队列与栈是最常使用的两种数据结构，其中，队列的核心特征是先入先出，栈的核心特征是后入先出，只要符合这两个特征，就属于队列（栈），不因实现形式的不同（数组或链表）而有差别，可以根据具体情况选择使用起来更方便的实现形式。

在本文中，我们对队列与栈的核心功能，循环队列这种特殊结构，以及队列和栈的主要应用，尤其是广度优先搜索和深度优先搜索进行介绍。

<!--more-->

## 1. 队列

队列是一个先入先出的数据结构，插入时新元素只能添加到队列末尾，取出时只能获取第一个元素。也因此我们需要维护两个指针。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200222_screen-shot-2018-05-03-at-151021-1593830694330.png)



### 1.1 队列实现

队列的实现可以使用动态数组或链表，最基本的功能包括：插入，删除，取第一个元素以及判空。由于 Golang 切片的易操作性，队列的实现即为简单，front 和 rear 指针被隐藏了起来。

```go
queue := make([]int,0) // 建立队列
queue = append(queue, value)  // 插入元素
queue = queue[1:] // 删除元素
value := queue[0] // 取第一个元素
if len(queue) == 0 {} // 判空
```

但当我们使用链表实现时，头尾指针是必需的，维持头和尾两个指针可以将插入和删除操作的复杂度保持在$O(1)$。

### 1.2 利用标准库实现

Go 没有提供内置的队列库，是因为可以很容易使用链表实现，只要保证新元素添加到链表尾，取元素从链表头取即可。

```go
queue := list.New() // 建立队列
queue.PushBack(value) // 入队：添加新元素到末尾
queue.Front().Value.(valueType) // 获取队首元素
queue.Remove(queue.Front()).(valueType) // 出队：删除队首元素
queue.Len() // 队列长度
```

### 1.3 循环队列

队列对空间的浪费比较严重，这从上面的程序中可以看出，因为数组在在不停地延长，而头指针之前指向的空间都没有被释放。循环队列是解决该问题的一种办法，主要思路是重用之前被浪费的存储。

![插入元素 23](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200222_%E5%BE%AA%E7%8E%AF%E9%98%9F%E5%88%97.png)

循环队列使用一段固定的空间，当尾指针指向队列尾时，如果有新元素添加进来而队列非空，可以将尾指针重新指向最开始的存储空间。循环队列的核心是队列空和满的判定，我们少用一个元素来区分队列空和队列满，约定 front 指向队列头元素，rear 指向队列尾元素的下一个位置，队列内容为 $[front,rear)$。

```go
// 循环队列结构定义
type MyCircularQueue struct {
    queue []int
    front,rear int // front指向第一个元素，rear指向最后一个元素的下一个位置
}


/** Initialize your data structure here. Set the size of the queue to be k. */
func Constructor(k int) MyCircularQueue {
    return MyCircularQueue{make([]int,k+1),0,0}
}


/** Insert an element into the circular queue. Return true if the operation is successful. */
func (this *MyCircularQueue) EnQueue(value int) bool {
    if this.IsFull() {
        return false
    }
    this.queue[this.rear] = value
    this.rear = (this.rear + 1) % len(this.queue) //尾指针防止溢出
    return true
}


/** Delete an element from the circular queue. Return true if the operation is successful. */
func (this *MyCircularQueue) DeQueue() bool {
    if this.IsEmpty() {
        return false
    }
    this.front = (this.front + 1) % len(this.queue) //头指针防止溢出
    return true
}


/** Get the front item from the queue. */
func (this *MyCircularQueue) Front() int {
    if this.IsEmpty() {
        return -1
    }
    return this.queue[this.front]
}


/** Get the last item from the queue. */
func (this *MyCircularQueue) Rear() int {
    if this.IsEmpty() {
        return -1
    }
    return this.queue[(this.rear - 1 + len(this.queue)) % len(this.queue)] //尾指针返回正确的位置
}


/** Checks whether the circular queue is empty or not. */
func (this *MyCircularQueue) IsEmpty() bool {
    if this.front == this.rear {
        return true
    }
    return false
}


/** Checks whether the circular queue is full or not. */
func (this *MyCircularQueue) IsFull() bool {
    if (this.rear + 1) % len(this.queue) == this.front {
        return true
    }
    return false
}
```

使用链表实现循环队列时在细节上可能有所不同。

## 2. 队列与BFS

广度优先搜索（BFS）的实现与队列是密不可分的，最常用在树和图中，使用非常普遍。下面使用伪代码提供两个模板，一个是遍历，一个是找最短路径，这两个模板足够应付绝大多数题目。

### 2.1 遍历

遍历的伪代码模板如下，核心思想就是将初始节点入队，然后在循环中对队列头元素的所有邻居节点进行处理，如果没有被访问过就入队。

```go
func BFS(root Node) {
    create queue  // 创建队列，存储所有待处理结点
    create visited // 标记已访问过的结点
    add root to queue // 根结点入队
    // BFS
    for queue is not empty {
        var cur Node = the first node in queue
        remove the first node from queue // 实际情况可将以上两步合并为一步
        add cur to visited // 标记当前结点为已访问
        for curNeighbor  := range the neighbors of cur {
            if curNeighbor not in visited {
                 add next to queue
            }     
        }
    }
}
```

以「岛屿数量」题目为例，给出一个改模板的使用示例。[leetcode题目地址](https://leetcode-cn.com/problems/number-of-islands/)

```go
// 题目描述
给定一个由 '1'（陆地）和 '0'（水）组成的的二维网格，计算岛屿的数量。一个岛被水包围，并且它是通过水平方向或
垂直方向上相邻的陆地连接而成的。你可以假设网格的四个边均被水包围。

示例输入:
11000
11000
00100
00011

输出: 3
// 程序
func numIslands(grid [][]byte) int {
    var res int
    for i := 0; i < len(grid); i++ {
        for j :=0 ; j < len(grid[i]); j++ {
            // 将连成一篇的陆地算作1个
            if grid[i][j] == '1' {
                BFS(grid,i,j)
                res++
            }
            
        }
    }
    return res
}

type point struct {
    x,y int
}

func BFS(grid [][]byte, x,y int) {
    queue := []point{}
    queue = append(queue,point{x,y})
    grid[x][y] = '0'
    
    //以“右->下->左->上”顺序循环，dx,dy是每一种转向的坐标变化方式
	dx := []int{0, 1, 0, -1}
	dy := []int{1, 0, -1, 0}

    for len(queue) != 0 {
        cur := queue[0]
        queue = queue[1:]
        
        for i := 0; i < 4; i++ {
            nx,ny := cur.x + dx[i],cur.y + dy[i]
            // 坐标超出界限或当前邻居结点为水域，进入下一个循环
            if nx < 0 || nx >= len(grid) || ny < 0 || ny >= len(grid[cur.x]) || grid[nx][ny] == '0' { 
                continue
            }
            queue = append(queue,point{nx,ny})
            grid[nx][ny] = '0' // 入队时标记已访问，防止重复入队，陷入循环
        }
    }
}
```

### 2.2 最短路径

寻找最短路径的伪代码模板如下

```go
// 如果找到返回最短路径
func BFS(root, target Node) int {
    create queue          // 创建队列，存储所有待处理结点
    create visited        // 标记已访问过的结点
    var shortestPath int  // 根结点到当前节点的最短路径
    // 初始化
    add root to queue
    add root to visited
    // BFS
    for queue is not empty {
        shortestPath++
        int size = queue.length();
        for  i = 0; i < size; i++ {
            var cur Node = the first node in queue;
            return shortestPath if cur is target;
            for curNeighbor := range the neighbors of cur {
                if curNeighbor is not in visited {
                    add next to queue
                    add next to visited
                }
            }
            remove the first node from queue
        }
    }
    return -1;          // 找不到最短路径
}
```

注：如果确定没有循环（比如树中）或者确实希望将结点多次加入队列，则不需要使用 visited 标记已访问结点。

这里有一些总结的技巧

1. 已访问列表可充分利用原题的数组或链表值实现
2. 已访问列表可使用 map 实现
3. 使用 map 实现时，值类型为 bool 或 int 可用于不同的场景
4. 对当前节点的所有邻居节点进行访问，如果是数组，可使用方向数组完成，其它场景则根据具体情况决定

## 3. 栈

如下图，栈是一个后入先出的数据结构，插入（Push）和删除（Pop）都只能在栈顶操作。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200222_screen-shot-2018-06-02-at-203523.png)

### 3.1 栈实现

使用动态数组的简单实现如下

```go
var stack []int

// 初始化
func Constructor() {
    stack = make([]int,0)
}

// 入栈
func Push(value int) bool {
	stack = append(stack, value)
	return true
}

// 出栈
func Pop() bool {
	if len(stack) == 0 {
		return false
	}
	stack = stack[:len(stack)-1]
	return true
}

// 获取栈顶元素
func Top() int {
	if len(stack) == 0 {
		return -1
	}
	return stack[len(stack)-1]
}

// 判空
func IsEmpty() bool {
	if len(stack) != 0 {
		return false
	}
	return true
}
```

### 3.2 内置库

与对了相同，栈也可以很容易地使用 Go 提供的链表库实现

```go
stack := list.New() // 建立队列
stack.PushBack(value) // 入队：添加新元素到末尾
stack.Back().Value.(valueType) // 获取队首元素
stack.Remove(queue.Back()).(valueType) // 出队：删除队首元素
stack.Len() // 队列长度
```

### 3.3 例子：有效括号

这是一个最经典也最简单的栈使用的例子，题目描述如下，[leetcode题目地址](https://leetcode-cn.com/problems/valid-parentheses/)

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。

注意空字符串可被认为是有效字符串。

```go
func isValid(s string) bool {
	a := map[rune]rune{
		'(': ')',
		'[': ']',
		'{': '}',
	}
	stack := []rune{}
	for _, v := range s {
		if len(stack) != 0 && a[stack[len(stack)-1]] == v {
			stack = stack[:len(stack)-1]
			continue
		}
		stack = append(stack, v)
	}
	if len(stack) != 0 {
		return false
	}
	return true
}
```

## 4. 栈和DFS

类似的，深度优先搜索（DFS）是栈的一种核心使用场景。DFS有两种实现方式，一种是递归，尽管递归的实现看起来没有使用栈，但实际上使用的是系统提供的隐式栈，也称为调用栈（Call Stack）；另一种就是显式的使用栈。

DFS 无法像 BFS 一样计算最短路径。

### 4.1 递归（隐式栈）

递归的模板如下

```go
func DFS(cur, target Node, visited map[Node]bool) bool {
    create visited // 标记已访问结点
    return true if cur is target
    for next : each neighbor of cur {
        if next is not in visited {
            add next to visted;
            return true if DFS(next, target, visited) == true;
        }
    }
    return false;
}
```

仍以队列中的「岛屿数量」一题为例，DFS 递归解法如下

```go
type point struct {
	x, y int //x,y分别为行号和列号
}

func numIslands(grid [][]byte) int {
	var res int
	for i := 0; i < len(grid); i++ {
		for j := 0; j < len(grid[i]); j++ {
            // 连在一起的陆地算一个
			if grid[i][j] == '1' {
				res++
				DFS(grid, i, j)
			}
		}
	}
	return res
}

func DFS(grid [][]byte, row, col int) {
    //以“右->下->左->上”顺序循环，dx,dy是每一种转向的坐标变化方式
	dx := []int{-1, 0, 1, 0}
	dy := []int{0, -1, 0, 1}

	grid[row][col] = '0'

	for i := 0; i < 4; i++ {
		nx, ny := row+dx[i], col+dy[i]
		if nx < 0 || nx >= len(grid) || ny < 0 || ny >= len(grid[row]) || grid[nx][ny] == '0' {
			continue
		}
        // 当前邻居结点满足条件，标记为已访问，并递归对该结点执行DFS
		grid[nx][ny] = '0'
		DFS(grid, nx, ny)
	}
}
```

### 4.2 显式栈

显式栈的模板如下

```go
func DFS(root,target Node) {
    create stack
    create visited
    add root to stack;
    for stack is not empty {
        var cur Node = the top element in stack
        remove cur from stack
        return true if cur is target
        for Node next : the neighbors of cur {
            if next is not in visited {
                add next to stack
                add next to visited
            }
        }      
    }
    return false
}
```

「岛屿数量」一题的 DFS 显式栈解法如下

```go
type point struct {
    x,y int //x,y分别为行号和列号
}
func numIslands(grid [][]byte) int {
	var res int
	for i := 0; i < len(grid); i++ {
		for j := 0; j < len(grid[i]); j++ {
			if grid[i][j] == '1' {
				res++
                DFS(grid,i,j)
			}
		}
	}
	return res
}

func DFS(grid [][]byte, row,col int) {   
    stack := []point{}
    stack = append(stack,point{row,col})
    grid[row][col] = '0'
    
    //以“右->下->左->上”顺序循环，dx,dy是每一种转向的坐标变化方式
    dx := []int{-1,0,1,0}
    dy := []int{0,-1,0,1}
    
    for len(stack) != 0 {
        cur := stack[len(stack)-1]
        stack = stack[:len(stack)-1]      
        
        for i := 0; i < 4; i++ {
            nx,ny := cur.x+dx[i],cur.y+dy[i]
            if nx < 0 || nx >= len(grid) || ny < 0 || ny >= len(grid[row]) || grid[nx][ny] == '0' {
                continue
            }
            // 满足条件的结点标记为已访问并入栈
            grid[nx][ny] = '0'
            stack = append(stack,point{nx,ny})
        }
    }  
}
```

## 5. 其它

最后一部分介绍两种非常常见的问法：用栈实现队列，用队列实现栈

### 5.1 用栈实现队列

使用两个栈实现，最普通的思路是，每次出队时，将栈中的元素弹入一个临时栈，然后取出栈底元素，最后将临时栈的元素再放回去。但这样时间复杂度比较高，另一种合适的做法是，设立一个入栈，一个出栈，入队时将元素放入入栈，出队时从出栈的栈顶取元素，如果出栈为空，将此时入栈的所有元素弹入出栈，然后取栈顶元素。这样做可以将时间复杂度缩减到 $O(1)$

```go
type MyQueue struct {
    in []int
    out []int
}


/** Initialize your data structure here. */
func Constructor() MyQueue {
    return MyQueue{}
}


/** Push element x to the back of queue. */
func (this *MyQueue) Push(x int)  {
    this.in = append(this.in,x)
}


/** Removes the element from in front of queue and returns that element. */
func (this *MyQueue) Pop() int {
    if len(this.out) == 0 {
        for i := len(this.in)-1; i >= 0; i-- {
            this.out = append(this.out,this.in[i])
        }
        this.in = nil
    }
    pop := this.out[len(this.out)-1]
    this.out = this.out[0:len(this.out)-1]
    return pop
}


/** Get the front element. */
func (this *MyQueue) Peek() int {
    if len(this.out)==0{
        for i:=len(this.in)-1; i>=0; i--{
            this.out = append(this.out, this.in[i])
        }
        this.in = nil
    }
    return this.out[len(this.out)-1]
}


/** Returns whether the queue is empty. */
func (this *MyQueue) Empty() bool {
    if len(this.in) == 0 && len(this.out) == 0{
        return true
    }else{
        return false
    }
}
```

### 5.2 用队列实现栈

使用两个队列实现，只能使用笨办法，每次出栈将所有元素放到临时队列，取队尾元素，然后再将临时队列的元素放回去。如果使用链表实现，可以将最后一步简化为交换两个队列的头指针。

```go
type MyStack struct {
    q []int
    t []int
}


/** Initialize your data structure here. */
func Constructor() MyStack {
    return MyStack{}
}


/** Push element x onto stack. */
func (this *MyStack) Push(x int)  {
    this.q = append(this.q,x)
}


/** Removes the element on top of the stack and returns that element. */
func (this *MyStack) Pop() int {
    if this.Empty() {
        return -1
    }
    for len(this.q) > 1 {
        this.t = append(this.t,this.q[0])
        this.q = this.q[1:]
    }
    pop := this.q[0]
    this.q = nil
    if this.t != nil {
        for i := 0; i < len(this.t); i++ {
            this.q = append(this.q,this.t[i])
        }
    }
    this.t = nil
    return pop
}


/** Get the top element. */
func (this *MyStack) Top() int {
    if this.Empty() {
        return -1
    }
    for len(this.q) > 1 {
        this.t = append(this.t,this.q[0])
        this.q = this.q[1:]
    }
    pop := this.q[0]
    this.q = nil
    this.t = append(this.t,pop)
    if this.t != nil {
        for i := 0; i < len(this.t); i++ {
            this.q = append(this.q,this.t[i])
        }
    }
    this.t = nil
    return pop
}


/** Returns whether the stack is empty. */
func (this *MyStack) Empty() bool {
    if len(this.q) == 0 {
        return true
    }else{
        return false
    }
}
```



---

> 作者: Shuzang  
> URL: https://shuzang.github.io/2020/data-structure-queue-and-stack/  

