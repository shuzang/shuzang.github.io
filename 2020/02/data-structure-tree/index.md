# 数据结构-树


树是反映事物之间层次关系的一种结构，比如家谱树、硬盘目录结构树等。

使用树的原因是这种层次结构在管理上有更高的效率，以查找为例，顺序查找的时间复杂度是O(n)，而二分查找的时间复杂度是O(log<sub>2</sub>n)，可以看到查找效率得到了很大的提高，这是因为二分查找本质上是对一颗树的查找。

## 1. 树

### 1.1 定义

树（Tree）是由 n（n$\geq$0）个结点构成的有限集合，当 n=0 时，称为空树，而对于任一颗非空树（n>0），它具有如下性质：

- 树中有一个称为「根（Root)」的特殊结点，用 r 表示
- 其余结点可分为 m(m>0) 个互不相交的有限集 T<sub>1</sub>, T<sub>2</sub>, ... , T<sub>m</sub>，其中每个集合本身又是一棵树，称为原来树的「子树（SubTree）」

![树的定义](https://s2.ax1x.com/2020/02/15/1vLDeO.png)

在判断是否是一颗树的时候，有以下注意点：

- 子树是不相交的；
- 除根节点外，每个节点有且仅有一个父节点；
- 一颗 N 个结点的树有 N-1 条边

![一些非树的例子](https://s2.ax1x.com/2020/02/15/1vLRSI.png)

### 1.2 术语

与树相关的一些术语如下表所示

| 术语       | 英文       | 描述                                                         |
| ---------- | ---------- | ------------------------------------------------------------ |
| 结点的度   | Degree     | 结点的子树个数                                               |
| 树的度     |            | 树的所有结点中最大的度数                                     |
| 叶结点     | Leaf       | 度为0的结点                                                  |
| 父结点     | Parent     | 有子树的结点是其子树的根结点的父节点                         |
| 子结点     | Child      | 若A结点是B结点的父结点，则称B结点是A结点的子结点；<br>子结点也称孩子结点 |
| 兄弟结点   | Sibling    | 具有同一父结点的各结点彼此是兄弟结点                         |
| 路径       | Path       | 从结点n<sub>1</sub>到n<sub>k</sub>的路径为一 个结点序列n<sub>1</sub> , n<sub>2</sub> , … , n<sub>k</sub> , n<sub>i</sub>是 n<sub>i+1</sub>的父结点 |
| 路径长度   |            | 路径所包含边的个数为路径的长度。                             |
| 祖先结点   | Ancestor   | 沿树根到某一结点路径上的所有结点都是这个结点的祖先结点       |
| 子孙结点   | Descendant | 某一结点的子树中的所有结点是这个结点的子孙                   |
| 结点的层次 | Level      | 规定根结点在1层，其它任一结点的层数是其父结点的层数加1       |
| 树的深度   | Depth      | 树中所有结点中的最大层次是这棵树的深度                       |

以定义中的树T解释如下图

![术语解释](https://s2.ax1x.com/2020/02/15/1vLhOf.png)

### 1.3 表示方法

树的表示使用儿子兄弟表示法，如下图所示，可以在合理表示的同时最大限度节省存储空间

![儿子兄弟表示法](https://s2.ax1x.com/2020/02/15/1vLHYj.png)

上图经过旋转，就可以形成一颗二叉树，因此，对数的处理就变成对二叉树的处理过程

![旋转成为二叉树](https://s2.ax1x.com/2020/02/15/1vLOlq.png)

## 2. 二叉树

### 2.1 定义

二叉树T：一个有穷的结点集合，可以为空。不为空时，是由根结点和称为其左子树T<sub>L</sub>和右子树T<sub>R</sub>的两个不相交的二叉树组成。

二叉树有如下五种基本形态

![二叉树的五种基本形态](https://s2.ax1x.com/2020/02/15/1vLxmT.png)

同时，这里还有几种特殊的二叉树

1. 斜二叉树（Skewed Binary Tree)

   ![斜二叉树](https://s2.ax1x.com/2020/02/15/1vOFpR.png)

2. 完美二叉树（Perfect Binary Tree），也称作满二叉树（Full Binary Tree）

   ![满二叉树](https://s2.ax1x.com/2020/02/15/1vOZnK.png)

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

![数组结构存储](https://s2.ax1x.com/2020/02/15/1vO3ct.png)

一般的二叉树也可以采用这种结构，只要按照完全二叉树的形式将空结点在数组中对应的值置空即可，但会造成空间的浪费...

#### 链表

链表是最常用的表示一般二叉树的方法。一个简单表示如下

```c
typedef struct TreeNode *BinTree;
typedef BinTree Position;
struct TreeNode{
    ElementType Data;
    BinTree Left;
    BinTree Right;
}
```

以一个简单的二叉树为例，基本的结点结构和完整的二叉树链表如下所示

![链表结构存储](https://s2.ax1x.com/2020/02/15/1vOyuV.png)

### 2.3 遍历

#### 先序遍历

先序遍历的过程为：

1. 访问根结点；
2. 先序遍历其左子树；
3. 先序遍历其右子树；

对应的程序实现如下

```Go
func PreOrderTraverse(BinTree BT) {
	if BT != nil {
        visit(BT.data)
		PreOrderTraverse(BT.left)
		PreOrderTraverse(BT.right)
	}
}
```

上述程序使用了递归的方法，也可以使用非递归的方法，基本的思路是使用堆栈。

![先序遍历](https://s2.ax1x.com/2020/02/15/1vXQVU.png)

如上图所示，在树的遍历过程中，每个结点被遇到三次，在第一次遇到时，我们将结点入栈，在最后一次离开时，我们将结点出栈。先序遍历就是在第一次遇到结点(入栈)时访问结点，因此程序如下：



```go
func PreOrderTraversal(BinTree BT)
	stack := CreatStack(MaxSize)
	for BT != nil || stack.Len() != 0 {
        for BT != nil {
            Push(stack,BT)
            visit(T.data)
            BT = BT.left
        }
        if stack.Len() != 0 {
            BT = Pop(stack)
            BT = BT.right
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
func InOrderTraverse(BinTree BT) {
	if BT != nil {  
		PreOrderTraverse(BT.left)
        visit(BT.data)
		PreOrderTraverse(BT.right)
	}
}
```

![中序遍历](https://s2.ax1x.com/2020/02/15/1vX0aD.png)

中序遍历也可以使用非递归的方法实现。实际上，前序、中序和后序走的路线是相同的，唯一的区别是访问结点的时机不同，在中序遍历中，是在第二次遇到结点时访问结点，如上图所示，因此中序非递归遍历的程序如下

```go
func InOrderTraversal(BinTree BT)
	stack := CreatStack(MaxSize)
	for BT != nil || stack.Len() != 0 {
        for BT != nil {
            Push(stack,BT)            
            BT = BT.left
        }
        if stack.Len() != 0 {
            BT = Pop(stack)
            visit(T.data)
            BT = BT.right
        }
    }
}
```

#### 后序遍历

后序遍历的过程为：

1. 后序遍历其左子树
2. 后序遍历其右子树
3. 访问根结点

对应的程序实现如下

```go
func PostOrderTraverse(BinTree BT) {
	if BT != nil {  
		PreOrderTraverse(BT.left)
		PreOrderTraverse(BT.right)
        visit(BT.data)
	}
}
```

![后序遍历](https://s2.ax1x.com/2020/02/15/1vXrPH.png)

后序遍历是在第三次遇到结点时访问结点，它的非递归实现要复杂一点，需要增加一个栈标记到达结点的次序

```go
func PostOrderTraversal(BinTree BT) {
    stack := Creatstacktack(Maxstackize)
    tag := make(map[BinTree]bool)
    forBT!= nil || stack.Len() != 0 {
        forBT!= nil {
           Push(stack,T)
           tag[T] = true
           BT= T.left
        }
        if stack.Len() != 0 {
           BT= stack.top()
           if tag[T] {
               tag[T] == false
               BT= T.right
           }else{
               BT= Pop(stack,T)
               visit(T.data)
               BT= nil
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
func LevelOrderTraversal(BinTree BT) {
    if BT == nil {
        return
    }
    queue := CreatQueue(MaxSize)
    Push(queue, BT)
    for queue.Len() != 0 {
        BT = Pop(queue)
        fmt.Println(BT.data)
        if BT.left != nil {
            Push(queue, BT.left)
        }
        if BT.right != nil {
            Push(queue, BT.right)
        }
    }
}
```

## 3. 二叉搜索树

### 3.1 定义

二叉搜索树（BST，Binary Search Tree），也称二叉排序树或二叉查找树。其定义如下

二叉搜索树：是一颗二叉树，可以为空，如果不为空，满足下列性质

1. 非空左子树的所有结点值小于其根结点的结点值
2. 非空右子树的所有结点值小于其根结点的结点值
3. 左右子树都是二叉搜索树

### 3.2 基本操作

二叉搜索树的基本操作包括：

- Position Find(ElementType X, BinTree BST)：从二叉搜索树BST中查找元素X，返回其所在结点的地址；
- Position FindMin(BinTree BST)：从二叉搜索树BST中查找并返回最小元素所在结点的地址；
- Position FindMax(BinTree BST)：从二叉搜索树BST中查找并返回最大元素所作结点的地址
- BinTree Insert(ElementType X, BinTree BST)
- BinTree Delete(ElementType X, BinTree BST)

#### 查找

查找的基本思路是从根结点开始，如果树为空，返回NULL，如果树非空，则将根结点的值和X进行比较，分情况处理：

1. 如果X小于根结点的值，在左子树中继续搜索
2. 如果X大于根结点的值，在右子树中继续搜索
3. 若两者值相等，搜索完成，返回指向该结点的指针

递归的实现思路如下

```go
func Find(ElementType X, BinTree BST) {
    if BST == nil 
        return nil;
    if X > BST->Data
        return Find(X, BST->Right);
    else if X < BST->Data
        return Find(X, BST->Left);
    else
        return BST;
}
```

迭代的实现思路如下

```go
func IterFind(ElementType X, BinTree BST){
    for BST != nil {
        if X > BST->Data 
            BST = BST->Right;
        else if X < BST->Data
            BST = BST->Left;
        else
            return BST;
    }
    return nil;
}
```

查找最大和最小元素只需要记住两点

- 最大元素一定在树的最右分支的端结点上
- 最小元素一定在树的最左分支的端结点上

![查找最大最小元素](https://s2.ax1x.com/2020/02/16/3SfTds.png)

查找最小元素的递归方法参考实现如下

``` go
func FindMin(BinTree BST){
    if BST == nil 
        return nil;
    else if BST->Left == nil
        return BST;
    else
        return FindMin(BST->Left);
}
```

查找最大元素的迭代方法参考实现如下

```go
func FindMax(BinTree BST){
    if BST != nil
        for BST->Right != nil
            BST = BST->Right;
    return BST;
}
```

#### 插入

二叉搜索树插入的关键是找到要插入的位置，可以采用和查找类似的方法，一个参考实现如下

```go
func Insert(ElementType X, BinTree BST){
    if BST == nil {
        BST = &BinTree{X, nil, nil}
    }else{
        if X < BST->Data
            BST->Left = Insert(X, BST->Left);
        else if X > BST->Data
            BST->Right=Insert(X,BST->Right);
    }
    return BST;
}
```

#### 删除

二叉树的删除比较复杂，要考虑三种情况

1. 要删除的是叶节点，则直接删除，并修改其父结点指针为NULL

   ![二叉搜索树删除叶结点](https://s2.ax1x.com/2020/02/16/3S7N0U.png)

2. 要删除的结点只有一个孩子，则将其父结点的指针指向要删除结点的孩子结点

   ![二叉搜索树要删除的结点只有一个孩子](https://s2.ax1x.com/2020/02/16/3SHE4J.png)

3. 要删除的结点有左右两颗子树，则用另一个结点替代被删除的结点，可以是右子树的最小结点，也可以是左子树的最大结点

   ![二叉搜索树要删除的结点有两颗子树](https://s2.ax1x.com/2020/02/16/3SqOAO.png)

参考实现如下

```go
func Delete(ElementType X, BinTree BST){
    if BST == nil 
        return nil
    if X < BST->Data
        BST->Left = Delete(X, BST->Left);
    else if X > BST->Data
        BST->Right = Delete(X, BST->Right);
    else
        if BST->Left != nil && BST->Right != nil {
            BST.Data = FindMin(BST->Right);
            BST->Right = Delete(BST->Data, BST->Right);
        }else{
            Tmp = BST;
            if BST->Left != nil
                BST = BST->Right;
            else if BST->Right == nil
                BST = BST->Left;
        }
    return BST;
}
```

## 4. 平衡二叉树

### 4.1 定义

平衡二叉树（Balanced Binary Tree），一般称为AVL树，它要么是一颗空树，要么满足任一结点左右子树高度差的绝对值不超过1。一般使用「平衡因子（Balance Factor, BF）」来描述这一高度差，设H<sub>L</sub>和H<sub>R</sub>分别为树T左右子树的高度，则BF满足：
$$
BF(T) = \left|H_L - H_R\right| \le 1
$$
此外，平衡二叉树的高度为$O(\log_2n)$,证明如下

> 设$f_n$为高度为n的AVL树所包含的最少结点数，则有
> $$
> f_n= \begin{cases} 1&(n=1)\ \\\ 2&(n=2)\ \\\ f_{n-1}+f_{n-2}+1& (n>2) \end{cases}
> $$
> 显然$\\{ f_n + 1 \\}$是一个斐波那契数列，由于斐波那契数列以指数的速度增长，因此AVL树的高度为$O(\log_2n)$

### 4.2 平衡性调整

平衡二叉树同样是一颗二叉搜索树，插入和删除的算法和二叉搜索树相同，但是插入和删除结点后，会造成树高和平衡因子的变化，从而不满足平衡二叉树的条件，因此需要对其进行调整。

以插入为例，将所有情况和对应的处理措施分为四种

#### RR

意为插入的结点在失衡点右子树的右子树中（左右孩子处理办法都一样），其解决办法是进行一次**左旋转**，如下图所示，14为新插入的结点。需要注意的是失衡点不一定是根结点。

![AVL树左旋转](https://s2.ax1x.com/2020/02/18/3kH9Fx.png)

简单的代码实现如下

```go
func rightRotate(BinTree AVL) {
    tmp := AVL.Right
    AVL.Right = tmp.Left
    tmp.Left = AVL
    
    AVL.Height = max(AVL.Left.Height, AVL.Right.Height) + 1
    tmp.Height = max(tmp.Left.Height, tmp.Right.Height) + 1
    return tmp
}
```

#### RL

意为插入的结点在失衡点右子树的左子树中（左右孩子都一样），其解决办法是**先进行一次右旋转，再进行一次左旋转**，如下图所示，插入结点分别为10.5和11.5

![AVL树先右后左旋转](https://s2.ax1x.com/2020/02/18/3kOXZQ.png)

简单的代码实现如下

```go
func rightThenLeftRotate(BinTree AVL) {
    tmp := rightRotate(AVL.Right)
    AVL.Right = tmp
    return leftRotate(AVL)
}
```

#### LL

意为插入的结点在失衡点左子树的左子树中（左右孩子都一样），其解决办法是进行一次**右旋转**，如下图所示，6为插入的新结点

![AVL树右旋转](https://s2.ax1x.com/2020/02/18/3kvTR1.png)

简单的代码实现如下

```go
func rightRotate(BinTree AVL) {
    tmp := AVL.Left
    AVL.Left = tmp.Right
    tmp.Right = AVL
    
    AVL.Height = max(AVL.Left.Height, AVL.Right.Height) + 1
    tmp.Height = max(tmp.Left.Height, tmp.Right.Height) + 1
    return tmp
}
```

#### LR

意为插入的结点在失衡点左子树的右子树中（左右孩子都一样），其解决办法是**先进行一次左旋转，再进行一次右旋转**，如下图所示，插入结点为8.5和9.5

![AVL树先左后右旋转](https://s2.ax1x.com/2020/02/19/3Ahc9g.png)

简单的代码实现

```go
func leftThenRightRotate(BinTree AVL) {
    tmp := LeftRotate(AVL.Left)
    AVL.Left = tmp
    return RightRotate(AVL)
}
```

对于AVL树的所有操作都建立在这四个基本操作之上，在插入和删除操作完成后，调用调整函数，然后在调整函数中分情况调用这四个函数。调整函数的实现如下

```go
func ajust(BinTree AVL) {
    if AVL.Right.Height - AVL.Left.Height == 2 {
        if AVL.Right.Right.Height > AVL.Right.Left.Height {
            AVL = leftRotate(AVL)
        }else{
            AVL = rightThenLeftRotate(AVL)
        }
    }else if AVL.Left.Height = AVL.Right.Height == 2 {
        if AVL.Left.Left.Height > AVL.Left.Right.Heigth {
            AVL = rightRotate(AVL)
        }else{
            AVL = leftThenRightRotate(AVL)
        }
    }
    return AVL
}
```



## 参考资料																																			

[1] [中国大学MOOC平台-浙江大学数据结构](https://www.icourse163.org/course/ZJU-93001?tid=1450069451)

[2] [bilibili-浙江大学数据结构](https://www.bilibili.com/video/av43521866)

[3] [CSDN-Golang实现平衡二叉树](https://blog.csdn.net/qq_36183935/article/details/80315808)
