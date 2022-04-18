# 数据结构-二叉树


树是反映事物之间层次关系的一种结构，比如家谱树、硬盘目录结构树等。

使用树的原因是这种层次结构在管理上有更高的效率，以查找为例，顺序查找的时间复杂度是O(n)，而二分查找的时间复杂度是O(log<sub>2</sub>n)，可以看到查找效率得到了很大的提高，这是因为二分查找本质上是对一颗树的查找。

## 1. 树

### 1.1 定义

树（Tree）是由 n（n$\geq$0）个结点构成的有限集合，当 n=0 时，称为空树，而对于任一颗非空树（n>0），它具有如下性质：

- 树中有一个称为「根（Root)」的特殊结点，用 r 表示
- 其余结点可分为 m(m>0) 个互不相交的有限集 T<sub>1</sub>, T<sub>2</sub>, ... , T<sub>m</sub>，其中每个集合本身又是一棵树，称为原来树的「子树（SubTree）」

![树的定义](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vLDeO.png)

在判断是否是一颗树的时候，有以下注意点：

- 子树是不相交的；
- 除根节点外，每个节点有且仅有一个父节点；
- 一颗 N 个结点的树有 N-1 条边

![一些非树的例子](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vLRSI.png)

### 1.2 术语

与树相关的一些术语如下表所示

| 术语                       | 英文       | 描述                                                         |
| -------------------------- | ---------- | ------------------------------------------------------------ |
| 结点的度                   | Degree     | 结点的子树个数                                               |
| 树的度                     |            | 树的所有结点中最大的度数                                     |
| 叶结点                     | Leaf       | 度为0的结点                                                  |
| 父结点                     | Parent     | 有子树的结点是其子树的根结点的父节点                         |
| 子结点                     | Child      | 若A结点是B结点的父结点，则称B结点是A结点的子结点；<br>子结点也称孩子结点 |
| 兄弟结点                   | Sibling    | 具有同一父结点的各结点彼此是兄弟结点                         |
| 路径                       | Path       | 从结点n<sub>1</sub>到n<sub>k</sub>的路径为一 个结点序列n<sub>1</sub> , n<sub>2</sub> , … , n<sub>k</sub> , n<sub>i</sub>是 n<sub>i+1</sub>的父结点 |
| 路径长度                   |            | 路径所包含边的个数为路径的长度。                             |
| 祖先结点                   | Ancestor   | 沿树根到某一结点路径上的所有结点都是这个结点的祖先结点       |
| 子孙结点                   | Descendant | 某一结点的子树中的所有结点是这个结点的子孙                   |
| 结点的层次<br>(结点的深度) | Level      | 规定根结点在1层，其它任一结点的层数是其父结点的层数加1       |
| 树的深度                   | Depth      | 树中所有结点中的最大层次是这棵树的深度                       |
| 结点的高度                 | Height     | 结点的深度从上往下数，而结点的高度从下往上数                 |

以定义中的树T解释如下图

![术语解释](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vLhOf.png)

### 1.3 表示方法

树的表示使用儿子兄弟表示法，如下图所示，可以在合理表示的同时最大限度节省存储空间

![儿子兄弟表示法](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vLHYj.png)

上图经过旋转，就可以形成一颗二叉树，因此，对数的处理就变成对二叉树的处理过程

![旋转成为二叉树](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vLOlq.png)

## 2. 二叉树

### 2.1 定义

二叉树T：一个有穷的结点集合，可以为空。不为空时，是由根结点和称为其左子树T<sub>L</sub>和右子树T<sub>R</sub>的两个不相交的二叉树组成。

二叉树有如下五种基本形态

![二叉树的五种基本形态](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vLxmT.png)

同时，这里还有几种特殊的二叉树

1. 斜二叉树（Skewed Binary Tree)

   ![斜二叉树](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vOFpR.png)

2. 完美二叉树（Perfect Binary Tree），也称作满二叉树（Full Binary Tree）

   ![满二叉树](https://picped-1301226557.cos.ap-beijing.myqcloud.com/1vOZnK.png)

3. 完全二叉树（Complete Binary Tree）：有n个结点的二叉树，对树中结点按从上到下、从左到右顺序进行编号，编号为i（1≤ i ≤ n）结点与满二叉树中编号为i结点在二叉树中位置相同

二叉树有几个重要性质

1. 一个二叉树第 i 层的最大结点数为：$2^{i-1}, i\ge1$

2. 深度为 k 的二叉树有最大结点总数为：$2^k-1, k\ge1$

3. 对任何非空二叉树T，若n<sub>0</sub>表示叶结点的个数，n<sub>2</sub>是度为2的非叶结点个数，那么两者满足关系$n_0=n_2+1$，推导过程如下：

   > 设$n_0$为叶结点的个数，$n_1$是度为1的结点个数，$n_2$为度为2的结点个数
   >
   > 按照边的数量建立等式
   > $$
   > n_0 + n_1 + n_2 -1 = 0 \times n_0 + 1 \times n_1 + 2 \times n_2
   > $$
   > 移项可得结果 $n_0=n_2+1$

现在对二叉树的抽象数据类型进行定义

```
类型名称：二叉树
数据对象集：一个有穷的结点集合。若不为空，则由根结点和其左、右二叉子树组成
操作集：BT∈BinTree, Item∈ElementType，重要操作有：
   1. Boolen IsEmpty(BinTree BT)：判断BT是否为空；
   2. void Traversal(BinTree BT)：遍历，按某顺序访问每个结点；
   3. BinTree CreatBinTree()：创建一个二叉树。
```

常用的遍历方法有：

- void PreOrderTraversal( BinTree BT )：先序遍历—根、左子树、右子树
- void InOrderTraversal( BinTree BT )：中序遍历—左子树、根、右子树
- void PostOrderTraversal( BinTree BT )：后序遍历—左子树、右子树、根
- void LevelOrderTraversal( BinTree BT )：层次遍历—从上到下、从左到右

### 2.2 存储结构

#### 数组

完全二叉树可以按照从上到下、从左到右的顺序进行存储，如下图所示，一个n个结点的完全二叉树的结点序号有如下规则：

1. 非根结点（序号i>1）的父结点的序号$i/2$；
2. 结点（序号为i）的左孩子结点的序号是$2i$，需要满足$2i \le n$，否则没有左孩子；
3. 结点（序号为i）的右孩子结点的序号是$2i+1$，需要满足$2i+1 \le n$，否则没有右孩子。

![数组结构存储](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vO3ct.png)

一般的二叉树也可以采用这种结构，只要按照完全二叉树的形式将空结点在数组中对应的值置空即可，但会造成空间的浪费...

#### 链表

链表是最常用的表示一般二叉树的方法。一个简单表示如下

```go
type TreeNode struct{
    Data ElementType
    Left *TreeNode
    Right *TreeNode
}
```

以一个简单的二叉树为例，基本的结点结构和完整的二叉树链表如下所示

![链表结构存储](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vOyuV.png)

### 2.3 遍历

#### 先序遍历

先序遍历的过程为：

1. 访问根结点；
2. 先序遍历其左子树；
3. 先序遍历其右子树；

对应的程序实现如下

```Go
func PreOrderTraverse(root *TreeNode) {
    if root != nil {
        visit(root.Data)
        PreOrderTraverse(root.Left)
        PreOrderTraverse(root.Right)
    }
}
```

上述程序使用了递归的方法，也可以使用非递归的方法，基本的思路是使用堆栈。

![先序遍历](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vXQVU.png)

如上图所示，在树的遍历过程中，每个结点被遇到三次，在第一次遇到时，我们将结点入栈，在最后一次离开时，我们将结点出栈。先序遍历就是在第一次遇到结点(入栈)时访问结点，因此程序如下：

```go
func PreOrderTraversal(root *TreeNode)
	stack := CreatStack()
	for root != nil || stack.Len() != 0 {
		for root != nil {
			stack.PushBack(root)
			visit(root.Data)
			 root = root.Left
		}
		if stack.Len() != 0 {
			root = stack.Remove(stack.Back()).(*TreeNode)
			root = root.Right
		}
	}
}
```

#### 中序遍历

中序遍历的过程为：

1. 中序遍历其左子树
2. 访问根结点
3. 中序遍历其右子树

对应的程序实现如下

```go
func InOrderTraverse(root *Treenode) {
    if root != nil {  
        InOrderTraverse(root.Left)
        visit(root.Data)
        InOrderTraverse(root.Right)
    }
}
```

![中序遍历](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vX0aD.png)

中序遍历也可以使用非递归的方法实现。实际上，前序、中序和后序走的路线是相同的，唯一的区别是访问结点的时机不同，在中序遍历中，是在第二次遇到结点时访问结点，如上图所示，因此中序非递归遍历的程序如下

```go
func InOrderTraversal(root *TreeNode)
	stack := CreatStack()
	for root != nil || stack.Len() != 0 {
		for root != nil {
			stack.PushBack(root)
			root = root.Left
		}
		if stack.Len() != 0 {
			root = stack.Remove(stack.Back()).(*TreeNode)
			visit(root.Data)
			root = root.Right
		}
	}
}
```

中序遍历的特殊之处在于，对于二叉搜索树，通过中序遍历可以得到一个递增的有序序列。

#### 后序遍历

后序遍历的过程为：

1. 后序遍历其左子树
2. 后序遍历其右子树
3. 访问根结点

对应的程序实现如下

```go
func PostOrderTraverse(root *TreeNode) {
	if root != nil {  
		PostOrderTraverse(root.Left)
		PostOrderTraverse(root.Right)
		visit(root.Data)
	}
}
```

![后序遍历](https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200224_1vXrPH.png)

后序遍历是在第三次遇到结点时访问结点，它的非递归实现要复杂一点，需要增加一个栈标记到达结点的次序

```go
func PostOrderTraversal(BinTree BT) {
    stack := Creatstack()
    tag := make(map[*TreeNode]bool)
    for root != nil || stack.Len() != 0 {
        for root != nil {
            stack.PushBack(root)
            root = root.Left
        }
        if stack.Len() != 0 {
            root = stack.Back().Value.(*TreeNode)
            if !tag[root] {
                tag[root] = true
                root = root.Right
            } else {
                root = stack.Remove(stack.Back()).(*TreeNode)
                visit(root.Data)
                root = nil
            }
        }
    }
}
```

#### 层次遍历

二叉树遍历的核心问题是二维结构的线性化，层次遍历的思想是利用队列，首先将根结点入队，然后开始执行循环：

1. 从队列中取出一个元素
2. 访问该元素所指结点
3. 若该元素所指结点的左右孩子结点非空，则将其左右孩子的指针顺序入队

程序实现如下

```go
func LevelOrderTraversal(root *TreeNode) {
    if root == nil {
        return
    }
    queue := CreatQueue()
    queue.PushBack(root)
    for queue.Len() != 0 {
        root = queue.Remove(queue.Front()).(*TreeNode)
        visit(root.Data)
        if root.Left != nil {
            queue.PushBack(root.Left)
        }
        if root.Right != nil {
            queue.PushBack(root.Right)
        }
    }
}
```

树的层序遍历其实就是对树执行广度优先搜索。

注：**还有一种遍历方法叫做 莫里斯（Morris）遍历，可以将空间复杂度降到 O(1)**

## 3. 运用递归求解问题

树本身就是通过递归定义的，因此很多与树相关的问题都可以通过递归来解决。每个递归层次中，我们只关注当前节点的问题，子节点通过递归调用函数来解决。

递归的思路共有两种：自顶向下和自底向上。**自顶向下**指的是先对当前节点值进行处理，然后将处理结果通过递归调用函数传递给子节点。**自底向上**则指在每个递归层次首先调用递归函数处理子节点，然后根据返回值和当前节点的值得到答案。

下面以一些最常见的题目来说明如何用递归法求解二叉树问题，使用的二叉树节点的定义如下

```go
 type TreeNode struct {
     Val int
     Left *TreeNode
 }
```

### 3.1 二叉树的深度

输入一棵二叉树的根节点，求该树的深度。树的深度是指从根节点到最远叶节点的最长路径的节点数。

自顶向下的方法中，基本思想是，如果我们知道当前节点深度，那么子节点的深度就是当前节点深度加1，在递归调用时，将当前节点深度作为参数，这样所有节点都可以知道自己的深度，我们只需要在遇到叶节点时更新树的深度即可。初始条件定义为根结点深度=1.

```go
var answer int
func maxDepth(root *TreeNode, depth int) {
    if root == nil {
        return
    }
    if root.Left == nil && root.Right == nil {
        if depth > answer {
            answer = depth
        }
    }
    maxDepth(root.Left, depth + 1)
    maxDepth(root.Right, depth + 1)
}
```

自底向上的方法中，当前节点的最大深度就等于以左节点为根的子树和以右节点为根的子树的深度最大值+1

```go
func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    HL := maxDepth(root.Left)
    HR := maxDepth(root.Right)
    if HL > HR {
        return HL + 1
    }else{
        return HR + 1
    }
}
```

与深度相对的，有时候也会求解二叉树的最小深度，最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

```go
func minDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    if root.Left == nil && root.Right == nil {
        return 1
    }
    HL := minDepth(root.Left)
    HR := minDepth(root.Right)
    if root.Left == nil {
        return HR + 1
    }else if root.Right == nil {
        return HL + 1
    }else{
        if HL < HR {
            return HL + 1
        }else{
            return HR + 1
        }        
    }
}
```

### 3.2 对称二叉树

给定一个二叉树，检查它是否是镜像对称的。

```go
func isSymmetric(root *TreeNode) bool {
    return isMirror(root, root)
}

func isMirror(l,r *TreeNode) bool {
    if l == nil && r ==nil {
        return true
    }
    if l == nil || r == nil {
        return false
    }
    if l.Val != r.Val {
        return false
    } 
    return isMirror(l.Left, r.Right) && isMirror(l.Right, r.Left)
}
```

### 3.3 路径总和

给定一个二叉树和一个目标和，判断该树中是否存在根节点到叶子节点的路径，这条路径上所有节点值相加等于目标和。

示例: 给定如下二叉树，以及目标和 sum = 22，

```go
              5
             / \
            4   8
           /   / \
          11  13  4
         /  \      \
        7    2      1
```

返回 `true`, 因为存在目标和为 22 的根节点到叶子节点的路径 `5->4->11->2`。

递归的思路非常简单

```go
func hasPathSum(root *TreeNode, sum int) bool {
    if root == nil {
        return false
    }
    sum -= root.Val
    if root.Left == nil && root.Right == nil {
        return sum == 0
    }
    return hasPathSum(root.Left,sum) || hasPathSum(root.Right,sum)
}
```

迭代的解题思路是利用遍历，不断更新目标和并与当前节点比较。所有的遍历方式都可用，下面是 BFS 的示例。

```go
func hasPathSum(root *TreeNode, sum int) bool {
    if root == nil {
        return false
    }
    queue := list.New()
    queue.PushBack(root)
    for queue.Len() != 0 {
        root = queue.Remove(queue.Front()).(*TreeNode)
        if root.Left == nil && root.Right == nil {
            if root.Val == sum {
                return true
            }
        }
        if root.Left != nil {
            root.Left.Val += root.Val
            queue.PushBack(root.Left)
        }
        if root.Right != nil {
            root.Right.Val += root.Val
            queue.PushBack(root.Right)
        }
    }
    return false
}
```

如果需要记录路径，可以使用如下方案

```go
func pathSum(root *TreeNode, sum int) [][]int {
    var ret [][]int
    var path []int
    
    return dfs(root,path,ret,sum)
}

func dfs(root *TreeNode,path []int,ret [][]int,sum int) [][]int{
        if root == nil {
            return ret
        }
        sum -= root.Val
        path = append(path,root.Val)
        
        if root.Left == nil && root.Right == nil {
            if sum == 0 {
                slice := make([]int,len(path))
	            copy(slice,path)
                ret = append(ret,slice)
            }
            return ret
        }       
        if root.Left != nil {
            ret = dfs(root.Left,path,ret,sum)
        }
        if root.Right != nil {
            ret = dfs(root.Right,path,ret,sum)
        }
        return ret
}
```

## 4. 其它常见题型

### 4.1 翻转二叉树

对一棵二叉树进行镜像翻转，比如输入为

```go
     4
   /   \
  2     7
 / \   / \
1   3 6   9
```

那么翻转后的输出为

```go
     4
   /   \
  7     2
 / \   / \
9   6 3   1
```

递归的理解是翻转后的树是将左子树和右子树分别翻转后再进行翻转，写出来的程序有点像后序遍历

```go
func invertTree(root *TreeNode) *TreeNode {
    if root != nil {
        root.Left = invertTree(root.Left)
        root.Right = invertTree(root.Right)
        root.Left, root.Right = root.Right, root.Left
    }
    return root
}    
```

第三条翻转语句也可以放在左右子树翻转之前，即先对当前结点的左右子树翻转，再分别翻转左右子树。

```go
func invertTree(root *TreeNode) *TreeNode {
    if root != nil {
        root.Left, root.Right = root.Right, root.Left
        root.Left = invertTree(root.Left)
        root.Right = invertTree(root.Right)
    }
    return root
} 
```

### 4.2 二叉树的锯齿形层次遍历

给定一个二叉树，返回其节点值的锯齿形层次遍历。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。例如：给定二叉树 `[3,9,20,null,null,15,7]`,

 ```go
    3
   / \
  9  20
    /  \
   15   7
 ```

返回锯齿形层次遍历如下

```go
[
  [3],
  [20,9],
  [15,7]
]
```

与层次遍历的想法基本相同，只是添加了一个层次判断，在奇数层按原来的方法构造切片，在偶数层反向构造切片。由于Go的特性，这种方法很容易实现。

```go
func zigzagLevelOrder(root *TreeNode) [][]int {
    if root == nil {
        return nil
    }
    result := [][]int{}
    queue := list.New()
    queue.PushBack(root)
    for tmp := 1; queue.Len() != 0; tmp++ {
        level := []int{}
        currentLevel := queue.Len()
        for i := 0; i < currentLevel; i++ {
            root := queue.Remove(queue.Front()).(*TreeNode) 
            if tmp % 2 == 1 {
                level = append(level, root.Val)                 
            }else{
                level = append([]int{root.Val},level...)
            }            
            if root.Left != nil {
                queue.PushBack(root.Left)
            }
            if root.Right != nil {
                queue.PushBack(root.Right)
            }
        }
        result = append(result, level)
    }
    return result
}
```

### 4.3 由树的两种遍历序列还原二叉树

假设树中没有重复元素，根据两种遍历序列构造出原来的二叉树。需要注意的是，两种遍历序列中必须有中序遍历，也就是说，只给出先序和后序是无法确定棵二叉树的。

首先介绍如何根据先序和中序遍历确定一棵二叉树，步骤如下

1. 根据先序遍历序列的第一个结点确定根结点
2. 根据根结点在中序遍历序列中分割出左右两个子序列
3. 对左子树和右子树分别递归使用相同的方法继续分解

```go
func buildTree(preorder []int, inorder []int) *TreeNode {
        if len(inorder) == 0{
            return nil
        }
        idx := -1
        for i,v:=range inorder{
            if v == preorder[0]{
                idx = i
            }
        }
        root := &TreeNode{Val:preorder[0]}
        root.Left = buildTree(preorder[1:idx+1],inorder[:idx])
        root.Right = buildTree(preorder[idx+1:],inorder[idx+1:])
        return root
}
```

后序遍历的思想类似

```go
func buildTree(inorder []int, postorder []int) *TreeNode {
    if len(inorder) == 0 {
        return nil
    }
    idx := -1
    for i,v := range inorder {
        if v == postorder[len(postorder)-1] {
            idx = i
        }
    }
    root := &TreeNode{Val:postorder[len(postorder)-1]}
    root.Left = buildTree(inorder[:idx], postorder[:idx])
    root.Right = buildTree(inorder[idx+1:], postorder[idx:len(postorder)-1])
    return root
}
```

## 参考资料														

[1] [中国大学MOOC平台-浙江大学数据结构](https://www.icourse163.org/course/ZJU-93001?tid=1450069451)

[2] [bilibili-浙江大学数据结构](https://www.bilibili.com/video/av43521866)


