# 数据结构-树


树是反映事物之间层次关系的一种结构，比如家谱树、硬盘目录结构树等。

使用树的原因是这种层次结构在管理上有更高的效率，以查找为例，顺序查找的时间复杂度是O(n)，而二分查找的时间复杂度是O(log<sub>2</sub>n)，可以看到查找效率得到了很大的提高，这是因为二分查找本质上是对一颗树的查找。

## 1. 树

### 1.1 定义

树（Tree）是由 n（n$\geq$0）个结点构成的有限集合，当 n=0 时，称为空树，而对于任一颗非空树（n>0），它具有如下性质：

- 树中有一个称为「根（Root)」的特殊结点，用 r 表示
- 其余结点可分为 m(m>0) 个互不相交的有限集 T<sub>1</sub>, T<sub>2</sub>, ... , T<sub>m</sub>，其中每个集合本身又是一棵树，称为原来树的「子树（SubTree）」

![树的定义](https://user-images.githubusercontent.com/26682846/74151227-aa7d9500-4c46-11ea-9267-4f8e91ce8a23.png)

在判断是否是一颗树的时候，有以下注意点：

- 子树是不相交的；
- 除根节点外，每个节点有且仅有一个父节点；
- 一颗 N 个结点的树有 N-1 条边

![一些非树的例子](https://user-images.githubusercontent.com/26682846/74151236-ae111c00-4c46-11ea-906d-ee237150635a.png)

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

![术语解释](https://user-images.githubusercontent.com/26682846/74151271-c1bc8280-4c46-11ea-9bde-710b7c66d730.png)

### 1.3 表示方法

树的表示使用儿子兄弟表示法，如下图所示，可以在合理表示的同时最大限度节省存储空间

![表示方法](https://user-images.githubusercontent.com/26682846/74151277-c6813680-4c46-11ea-9740-eaeac2c3d223.png)

上图经过旋转，就可以形成一颗二叉树，因此，对数的处理就变成对二叉树的处理过程

![旋转成为二叉树](https://user-images.githubusercontent.com/26682846/74151300-d4cf5280-4c46-11ea-9001-8ca9110bf508.png)

## 2. 二叉树

### 2.1 定义

二叉树T：一个有穷的结点集合，可以为空。不为空时，是由根结点和称为其左子树T<sub>L</sub>和右子树T<sub>R</sub>的两个不相交的二叉树组成。

二叉树有如下五种基本形态

![二叉树的五种基本形态](https://user-images.githubusercontent.com/26682846/74151327-e6b0f580-4c46-11ea-9a1a-b454b9a8c467.png)

同时，这里还有几种特殊的二叉树

1. 斜二叉树（Skewed Binary Tree)

   ![斜二叉树](https://user-images.githubusercontent.com/26682846/74151331-ea447c80-4c46-11ea-85b8-3e54545898d3.png)

2. 完美二叉树（Perfect Binary Tree），也称作满二叉树（Full Binary Tree）

   ![满二叉树](https://user-images.githubusercontent.com/26682846/74151352-f7fa0200-4c46-11ea-8e57-64a4c021fbfa.png)

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

![数组结构存储](https://user-images.githubusercontent.com/26682846/74151367-0516f100-4c47-11ea-9376-b865f63ab5fa.png)

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

![链表结构存储](https://user-images.githubusercontent.com/26682846/74151372-06e0b480-4c47-11ea-922e-7fb1595c4595.png)

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
        fmt.Println(BT.data)
		PreOrderTraverse(BT.left)
		PreOrderTraverse(BT.right)
	}
}
```

上述程序使用了递归的方法，也可以使用非递归的方法，基本的思路是使用堆栈。

![先序遍历](https://user-images.githubusercontent.com/26682846/74235098-4374e480-4d09-11ea-8bd3-61cca6688afc.png)

如上图所示，在树的遍历过程中，每个结点被遇到三次，在第一次遇到时，我们将结点入栈，在最后一次离开时，我们将结点出栈。先序遍历就是在第一次遇到结点(入栈)时访问结点，因此程序如下：

```go
func PreOrderTraversal(BinTree BT) {
    var T BinTree = BT
    var S Stack = CreatStack(MaxSize)
    for T != nil || !IsEmpty(S) {
        for T != nil {
            Push(S,T)
            fmt.Println(T.data)
            T = T.left
        }
        if !IsEmpty(S) {
            T = Pop(S)
            T = T.right
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
        fmt.Println(BT.data)
		PreOrderTraverse(BT.right)
	}
}
```

![中序遍历](https://user-images.githubusercontent.com/26682846/74235103-453ea800-4d09-11ea-8452-791eeaabcaaa.png)

中序遍历也可以使用非递归的方法实现。实际上，前序、中序和后序走的路线是相同的，唯一的区别是访问结点的时机不同，在中序遍历中，是在第二次遇到结点时访问结点，如上图所示，因此中序非递归遍历的程序如下

```go
func PreOrderTraversal(BinTree BT) {
    var T BinTree = BT
    var S Stack = CreatStack(MaxSize)
    for T != nil || !IsEmpty(S) {
        for T != nil {
            Push(S,T)
            fmt.Println(T.data)
            T = T.left
        }
        if !IsEmpty(S) {
            T = Pop(S)
            T = T.right
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
func InOrderTraverse(BinTree BT) {
	if BT != nil {  
		PreOrderTraverse(BT.left)
		PreOrderTraverse(BT.right)
        fmt.Println(BT.data)
	}
}
```

![后序遍历](https://user-images.githubusercontent.com/26682846/74235106-47086b80-4d09-11ea-9886-739a51b3e3dc.png)

后序遍历是在第三次遇到结点时访问结点，它的非递归实现要复杂一点，需要增加一个栈标记到达结点的次序

```go
func PostOrderTraversal(BinTree BT) {
    var T BinTree = BT
    var S Stack = CreatStack(MaxSize)
    tag := make(map[BinTree]bool)
    for T != nil || !IsEmpty(S) {
        for T != nil {
            Push(S,T)
            map[T] = true
            T = T.left
        }
        if !IsEmpty(S) {
            T = S.top()
            if map[T] {
                map[T] == false
                T = T.right
            }else{
                T = Pop(S,T)
                fmt.Println(T.data)
                T = nil
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
    Q := CreatQueue(MaxSize)
    AddQ(Q, BT)
    for !IsEmptyQ(Q) {
        T = DeleteQ(Q)
        fmt.Println(T.data)
        if T.left != nil {
            AddQ(Q, T.left)
        }
        if T.right != nil {
            AddQ(Q, T.right)
        }
    }
}
```

## 3. 二叉搜索树



## 参考资料

[浙江大学-数据结构-中国大学MOOC平台](https://www.icourse163.org/course/ZJU-93001?tid=1450069451)

[浙江大学-数据结构-B站](https://www.bilibili.com/video/av43521866)

[浙江大学-数据结构练习-PTA平台](https://pintia.cn/problem-sets/434/problems/type/6)
