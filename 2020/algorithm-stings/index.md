# 算法-字符串类问题


做 Leetcode 每日一题的时候遇到了子串判断类的问题，想起一直没仔细的去看过 KMP 等字符串常用的算法，所以这里学习一下。

<!---more-->

## 1. 重复的子字符串

这就是今天遇到的题目，题目描述为

```
给定一个非空的字符串，判断它是否可以由它的一个子串重复多次构成。给定的字符串只含有小写英文字母，并且长度不超过10000。

示例 1:
输入: "abab"
输出: True
解释: 可由子字符串 "ab" 重复两次构成。

示例 2:
输入: "aba"
输出: False

示例 3:
输入: "abcabcabcabc"
输出: True
解释: 可由子字符串 "abc" 重复四次构成。 (或者子字符串 "abcabc" 重复两次构成。)
```

首先确认边界条件

1. 字符串长度满足：0 < len(s) <= 10000，所以不需要考虑空字符串
2. 字符串的子串不包括自己，也就是说，「一个字符串由自己重复 1 次构成」这种说法不成立，可由示例2得出

然后确认基本思路

1. 遍历所有可能的子串长度，从 1 到 len(s)/2，然后判断这个长度的子串是否可能为结果。取 len(s)/2 是因为超过字符串长度一半的子串不可能通过重复构成原字符串；
2. 字符串的长度一定是子串长度的倍数，否则也不可能是结果；
3. 考虑如何取给定长度的子串，由于字符串由子串重复构成，那么只需要从第一个字符开始取即可，长度为 n 的子串就从第一个字符开始取 n 个字符构成子串；

最后根据该思路编写代码

```go
func repeatedSubstringPattern(s string) bool {
    // 字符串长度可能是1到len(s)/2
    for i := 1; i <= len(s)/2; i++ {
        // 只有 len(s) 是子字符串长度的倍数才有可能
        if len(s) % i != 0 {
            continue
        }
        // 子串为 s[:i]，然后对字符串的其它部分进行判断
        var flag bool
        for j := i; j < len(s); j += i {
            if s[j:j+i] != s[:i] {
                flag = true
                break
            }
        }
        // 中途没有跳出才说明整个字符串由子串重复构成
        if !flag {
            return true
        }
    }
    // 所有长度的子串都无法重复构成原字符串
    return false
}
```

代码编写完成后要考虑最后一件事，就是条件和循环的边界条件，这是最容易产生错误的地方

1. ` i <= len(s)/2`：当 `i == len(s)/2` 的时候，子串为 `s[:len(s)/2]`，以字符串 `abab` 为例，子串为 `ab`，因为 Go 的切片不会取最后一个字符，所以必须添加 `=` 号，不然会漏掉一种情况；

2. `j < len(s)`：我们考虑最后一次循环的 j，此时 `s[j:j+i]` 中 `j+i` 可能会越界超出 len(s)，导致 panic。遇到这种情况我们最常用的做法是修改条件为 `j + i < len(s)`，然后我们来考虑边界条件。

   当 `j + i == len(s)` 的时候， `s[j:j+i]` 其实原本想表达的含义是最后一个字符串，是应该取的，但是 j + i 产生越界，所以要额外处理。

修改后的程序如下

```go
func repeatedSubstringPattern(s string) bool {
    // 字符串长度可能是1到len(s)/2
    for i := 1; i <= len(s)/2; i++ {
        // 只有 len(s) 是子字符串长度的倍数才有可能
        if len(s) % i != 0 {
            continue
        }
        // 子串为 s[:i]，然后对字符串的其它部分进行判断
        var flag bool
        for j := i; j + i <= len(s); j += i {
            var tmp string
            if j + i == len(s) {
                tmp = s[j:]
            }else{
                tmp = s[j:j+i]
            }
            if tmp != s[:i] {
                flag = true
                break
            }
        }
        // 中途没有跳出才说明整个字符串由子串重复构成
        if !flag {
            return true
        }
    }
    // 所有长度的子串都无法重复构成原字符串
    return false
}
```

## 2. 模式匹配BF算法

上面的问题可以使用 KMP 算法解决，但解释 KMP 算法之前必须先了解 BF 算法。

模式匹配问题为：假设有两个字符串 S 和 T，设 S 为主串，判断 T 是否为 S 的子串，如果是，返回子串在主串中第一个出现的位置，如果不是，返回 -1。

```
示例 1:
输入: haystack = "hello", needle = "ll"
输出: 2

示例 2:
输入: haystack = "aaaaa", needle = "bba"
输出: -1
```

最笨的办法，也就是暴力法，是穷举 S 所有的子串，判断是否和 T 相同，该算法就称为 BF（Brute Force）算法。为了介绍通用的算法，这里放弃 Go 切片的优势，采用逐个字符匹配的方式。代码如下

```go
func strStr(haystack string, needle string) int {
    // 子串为空，返回0
    if len(needle) == 0 {
        return 0
    }

    // = 号是考虑子串等于主串的情况
    for i := 0; i + len(needle) <= len(haystack); i++ {
        var flag bool
        for j := 0; j < len(needle); j++ {
            if haystack[i+j] != needle[j] {
                flag = true
                break
            }
        }
        if !flag {
            return i
        }
    }

    return -1
}
```

## 3. 模式匹配KMP算法

实际上，没有必要从主串 S 的每一个字符开始穷举每种情况。Knuth、Morris、Pratt 对该算法进行了改进，提出了 KMP 算法。

 设 `S = abaabaabeca，T = abaabe`，KMP 的流程如下

从S第1个字符开始：i=1, j=1，比较两个字符是否相等，如果相等，则 i++, j++；等到第一次匹配不相等的时候，BF算法会继续从 S 第 2个字符开始和 T 第一个字符进行比较，但 BMP 的做法是：S 的索引不需要移动，T 的索引回退 3 个，如下图

<img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200824_epub_27600261_316.jfif" style="zoom:50%;" />

这样做的原因是 T 串中 j 索引前面的两个字符和 S 串中 i 索引前面的两个字符相同，都是 `ab`，所以现在的关键变成了我们怎么知道 i 前面的字符和 j 前面的字符相同，有几个字符相同（这决定了 j 回退几个位置）

直观的想法是进行第二次的比较，比较 T 开头的字符和 i 前面的字符，但其实不需要，因为在 j 回退之前，i 前面的字符必然和 j 前面的字符相同，如下

<img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200824_epub_27600261_319.jfif" style="zoom:50%;" />

所以问题转化成了 T 内部字符的比较，假设 T 中当前 j 指向的前面的所有字符为 $T'$，上图中 $T' = abaab$，那么只需要比较 $T'$ 的前缀和后缀即可。判断其前缀后缀是否相等，并寻找相等前缀后缀的最大长度。

1. 长度为1：前缀 a，后缀 b，不相等
2. 长度为2：前缀 ab，后缀 ab，相等
3. 长度为3：前缀 aba，后缀 aab，不相等
4. 长度为4：前缀 abaa，后缀 baab，不相等

相等前缀后缀的最大长度为 2，则 j 可以回到 2+1=3 个位置继续比较。如果将这个回退的位置表示为 next[j]，令 $T' = t_1t_2...t_{j-1}$，则可得公式

<img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200824_epub_27600261_324.jfif" style="zoom: 50%;" />

由于字符串 T 的长度有限，next[] 其实是一个固定的数组

| j       | 1    | 2    | 3    | 4    | 5    | 6    |
| ------- | ---- | ---- | ---- | ---- | ---- | ---- |
| T       | a    | b    | a    | a    | b    | e    |
| next[j] | 0    | 1    | 1    | 2    | 2    | 3    |

假设 next[j] = k，$T' = t_1t_2...t_{j-1}$，那么 T 的相等前缀、后缀最大长度为 k-1

<img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200824_epub_27600261_326.jfif" style="zoom:50%;" />

我们在求 next[j+1] 的时候可以考虑动态规划递推的办法

1. $t_k = t_j$：那么 next[j+1] = k+1，即相等前缀和后缀的长度比 next[j] 多1，如下图

   <img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200824_epub_27600261_327.jfif" style="zoom:50%;" />

2. $t_k \neq t_j$：那么回退找 next[k] = k' 的位置，比较 $t_{k'}$ 和 $t_j$ 是否相等

   <img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200824_epub_27600261_328.jfif" style="zoom:50%;" />

   如果相等，则 next[j+1] = k'' + 1，如果不相等，继续向前找，直到找到 next[i] = 0 停止

   <img src="https://picped-1301226557.cos.ap-beijing.myqcloud.com/BC_20200824_epub_27600261_329.jfif" style="zoom:50%;" />

求解 next 的代码实现如下

```go
func getNext(T string, next []int) {
    j,k := 1,0
    next[1] = 0
    for j < len(T) {
        if k == 0 || T[j] == T[k] {
            j,k = j+1,k+1
            next[j] = k
        }else{
            k = next[k]
        }
    }
}
```

KMP 算法的代码如下

```go
func KMP(S,T string, pos int, next []int) int {
    i,j := pos, 1
    for i <= len(S) && j <= len(T) {
        if j == 0 || S[i] == T[j] {
            i++
            j++
        }else{
            j = next[j]
        }
    }
    if j > len(T) {
        return i-len(T)
    }else{
        return 0
    }
}
```



---

> 作者:   
> URL: https://shuzang.github.io/2020/algorithm-stings/  

