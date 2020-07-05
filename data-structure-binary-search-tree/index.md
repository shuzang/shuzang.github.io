# 数据结构-二叉搜索树




## 1. 二叉搜索树

### 1.1 定义

二叉搜索树（BST，Binary Search Tree），也称二叉排序树或二叉查找树。其定义如下

二叉搜索树：是一颗二叉树，可以为空，如果不为空，满足下列性质

1. 非空左子树的所有结点值小于其根结点的结点值
2. 非空右子树的所有结点值小于其根结点的结点值
3. 左右子树都是二叉搜索树

### 1.2 基本操作

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
func searchBST(root *TreeNode, val int) *TreeNode {
    if root == nil {
        return nil
    }
    if val < root.Val {
        return searchBST(root.Left, val)
    }else if val > root.Val {
        return searchBST(root.Right, val)
    }else{
        return root
    }
}
```

迭代的实现思路如下

```go
func IterFind(ElementType X, BinTree BST){
    for BST != nil {
        if X > BST->Data 
            BST = BST->Right
        else if X < BST->Data
            BST = BST->Left
        else
            return BST;
    }
    return nil
}
```

查找最大和最小元素只需要记住两点

- 最大元素一定在树的最右分支的端结点上
- 最小元素一定在树的最左分支的端结点上

![查找最大最小元素](/images/数据结构4-二叉搜索树/3SfTds.png)

查找最小元素的递归方法参考实现如下

``` go
func FindMin(root *TreeNode) *TreeNode {
    if root == nil 
        return nil
    else if root->Left == nil
        return root
    else
        return FindMin(root->Left)
}
```

查找最大元素的迭代方法参考实现如下

```go
func findMax(root *TreeNode) *TreeNode {
	if root != nil {
		for root.Right != nil {
			root = root.Right
		}
	}
	return root
}
```

#### 插入

二叉搜索树插入的关键是找到要插入的位置，可以采用和查找类似的方法，一个参考实现如下

```go
func insertIntoBST(root *TreeNode, val int) *TreeNode {
	if root == nil {
		root = &TreeNode{val, nil, nil}
	} else {
		if val > root.Val {
			root.Right = insertIntoBST(root.Right, val)
		} else if val < root.Val {
			root.Left = insertIntoBST(root.Left, val)
		}
	}
	return root
}
```

#### 删除

二叉树的删除比较复杂，要考虑三种情况

1. 要删除的是叶节点，则直接删除，并修改其父结点指针为NULL

   ![二叉搜索树删除叶结点](/images/数据结构4-二叉搜索树/3S7N0U.png)

2. 要删除的结点只有一个孩子，则将其父结点的指针指向要删除结点的孩子结点

   ![二叉搜索树要删除的结点只有一个孩子](/images/数据结构4-二叉搜索树/3SHE4J.png)

3. 要删除的结点有左右两颗子树，则用另一个结点替代被删除的结点，可以是右子树的最小结点，也可以是左子树的最大结点

   ![二叉搜索树要删除的结点有两颗子树](/images/数据结构4-二叉搜索树/3SqOAO.png)

参考实现如下

```go
func deleteNode(root *TreeNode, key int) *TreeNode {
	if root == nil {
		return nil
	}
	if key < root.Val {
		root.Left = deleteNode(root.Left, key)
	} else if key > root.Val {
		root.Right = deleteNode(root.Right, key)
	} else {
		if root.Left != nil && root.Right != nil {
			root.Val = findMin(root.Right).Val
			root.Right = deleteNode(root.Right, root.Val)
		} else {
			if root.Left == nil {
				root = root.Right
			} else if root.Right == nil {
				root = root.Left
			}
		}
	}
	return root
}
```

## 2. 平衡二叉树

### 2.1 定义

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

### 2.2 平衡性调整

平衡二叉树同样是一颗二叉搜索树，插入和删除的算法和二叉搜索树相同，但是插入和删除结点后，会造成树高和平衡因子的变化，从而不满足平衡二叉树的条件，因此需要对其进行调整。

以插入为例，将所有情况和对应的处理措施分为四种

#### RR

意为插入的结点在失衡点右子树的右子树中（左右孩子处理办法都一样），其解决办法是进行一次**左旋转**，如下图所示，14为新插入的结点。需要注意的是失衡点不一定是根结点。

![AVL树左旋转](/images/数据结构4-二叉搜索树/3kH9Fx.png)

简单的代码实现如下

```go
func leftRotate(root *TreeNode) *TreeNode {
	tmp := root.Right
	root.Right = tmp.Left
	tmp.Left = root

	root.Height = max(getHeight(root.Left), getHeight(root.Right)) + 1
	tmp.Height = max(getHeight(tmp.Left), getHeight(tmp.Right)) + 1
	return tmp
}
```

#### RL

意为插入的结点在失衡点右子树的左子树中（左右孩子都一样），其解决办法是**先进行一次右旋转，再进行一次左旋转**，如下图所示，插入结点分别为10.5和11.5

![AVL树先右后左旋转](/images/数据结构4-二叉搜索树/3kOXZQ.png)

简单的代码实现如下

```go
func rightThenLeftRotate(root *TreeNode) *TreeNode {
	tmp := rightRotate(root.Right)
	root.Right = tmp
	return leftRotate(root)
}
```

#### LL

意为插入的结点在失衡点左子树的左子树中（左右孩子都一样），其解决办法是进行一次**右旋转**，如下图所示，6为插入的新结点

![AVL树右旋转](/images/数据结构4-二叉搜索树/3kvTR1.png)

简单的代码实现如下

```go
func rightRotate(root *TreeNode) *TreeNode {
	tmp := root.Left
	root.Left = tmp.Right
	tmp.Right = root

	root.Height = max(getHeight(root.Left), getHeight(root.Right)) + 1
	tmp.Height = max(getHeight(tmp.Left), getHeight(tmp.Right)) + 1
	return tmp
}
```

#### LR

意为插入的结点在失衡点左子树的右子树中（左右孩子都一样），其解决办法是**先进行一次左旋转，再进行一次右旋转**，如下图所示，插入结点为8.5和9.5

![AVL树先左后右旋转](/images/数据结构4-二叉搜索树/3Ahc9g.png)

简单的代码实现

```go
func leftThenRightRotate(root *TreeNode) *TreeNode {
	tmp := leftRotate(root.Left)
	root.Left = tmp
	return rightRotate(root)
}
```

对于AVL树的所有操作都建立在这四个基本操作之上，在插入和删除操作完成后，调用调整函数，然后在调整函数中分情况调用这四个函数。调整函数的实现如下

```go
func ajust(root *TreeNode) *TreeNode {
	if root == nil {
		return nil
	}
	compare := getHeight(root.Right) - getHeight(root.Left)
	if compare == 2 {
		if getHeight(root.Right.Right) > getHeight(root.Right.Left) {
			root = leftRotate(root)
		} else {
			root = rightThenLeftRotate(root)
		}
	} else if compare == -2 {
		if getHeight(root.Left.Left) > getHeight(root.Left.Right) {
			root = rightRotate(root)
		} else {
			root = leftThenRightRotate(root)
		}
	}
	return root
}
```

其中用到寻找最大值和获取高度两个工具函数，实现如下

```go
func getHeight(root *TreeNode) int {
	if root == nil {
		return 0
	}
	return root.Height
}

func max(a int, b int) int {
	if a > b {
		return a
	} else {
		return b
	}
}
```



由于查找不涉及结点的增删，所以算法和实现同二叉搜索树相同。

## 3. 哈夫曼树

### 3.1 定义

设二叉树有 n 个叶子结点，每个叶子结点带有权值 w<sub>k</sub>，从根结点到每个叶子结点的长度位 l<sub>k</sub>，则每个叶子结点的**带权路径长度**（WPL）之和为：
$$
WPL = \sum_{k=1}^{n} w_k l_k
$$
WPL 最小的二叉树就叫做哈夫曼树（或者最优二叉树）。其特点有

- 没有度为1的结点

- n个叶子结点的哈夫曼树共有2n-1个结点，因为对二叉树而言有$n_0=n_2+1$

- 哈夫曼树的任意非叶节点的左右子树交换后仍然是哈夫曼树

- 对同一组权值存在不同构的两棵哈夫曼树，例子如下

  ![权值相同但不同构的哈夫曼树](/images/数据结构4-二叉搜索树/3GlEwT.png)

### 3.2 构造

构造哈夫曼树的思路极为简单，即每次把权值最小的两棵二叉树合并，以{1，2，3，4，5}这组数为例，构造过程如下

![哈夫曼树构造过程](/images/数据结构4-二叉搜索树/3G12VK.png)

其构造的关键在于每次寻找剩余结点中的最小值，最简单的实现是使用堆

```go
type TreeNode struct {
    Weight int
    Left *TreeNode
    Right *TreeNode
}

func Huffman(H MinHeap) {
    //假设MinHeap类型已实现标准库中堆的相关接口，H已存好所有权值
    var T TreeNode
    heap.Init(H)
    for i := 1; i < H.Len(); i++ {
        T = TreeNode{}
        T.Left = H.Pop(H)
        T.Right = H.Pop(H)
        T.Weight = T.Left.Weight + T.Right.Weight
        H.Push(H, T)
    }
    T = H.Pop(H)
    return T
}
```

### 3.3 哈夫曼编码

该问题的描述为：给定一段字符串，如何堆字符进行编码，可以使得该字符串的编码存储空间最少

> 例：假设有一段文本，包含58个字符，并由以下7个字符构成：a，e，i，s，t，空格(sp)，换行(nl)；这7个字符出现的次数不同，如何对这7个字符进行编码，可以使得总编码空间最少

如果用等长ASCII编码，则一共 58×8=464 位；如果用等长3位编码，则一共 58×3=174位；最后一种方法是不等长编码，即出现频率高的字符编码短，出现频率低的字符编码长。

使用不等长编码时，为了避免二义性，可以使用前缀码（prefix code），即任何字符的编码都不是另一字符编码的前缀。将二叉树用于编码，遵循下面的规则

- 左右分支：0，1
- 字符只在叶节点上

假设四个字符的频率分别为：a-4, u-1, x-2, z-1，两个可用的编码树如下

![编码树举例](/images/数据结构4-二叉搜索树/3GaWI1.png)

| 字符 | a    | e    | i    | s    | t    | sp   | nl   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 权值 | 10   | 15   | 12   | 3    | 4    | 13   | 1    |

按照构造哈夫曼树的方法，可以构造棵编码代价最小的二叉树，假设之前例子中的7个字符权值如上表，则可构造得到如下的哈夫曼编码树

![哈夫曼编码树](/images/数据结构4-二叉搜索树/3GB4TH.png)

## 4. 常见题型

### 4.1 验证二叉搜索树

给定一个二叉树，判断其是否是一个有效的二叉搜索树。

假设一个二叉搜索树具有如下特征：

- 节点的左子树只包含小于当前节点的数。
- 节点的右子树只包含大于当前节点的数。
- 所有左子树和右子树自身必须也是二叉搜索树。

```go
func isValidBST(root *TreeNode) bool {
    return isBST(root, math.MinInt64, math.MaxInt64)
}
func isBST(root *TreeNode, left, right int) bool {
    if root == nil {
        return true
    }
    if left >= root.Val || right <= root.Val {
    	return false
    }
    return isBST(root.Left, left, root.Val) && isBST(root.Right, root.Val, right)
}
```

## 参考资料

[1] [CSDN-Golang实现平衡二叉树](https://blog.csdn.net/qq_36183935/article/details/80315808)

 
