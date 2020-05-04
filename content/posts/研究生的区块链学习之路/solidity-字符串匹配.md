---
title: 字符串匹配
author: fravoll
date: 2020-05-04
tags: [Solidity]
categories: [研究生的区块链学习之路]
slug: String Equality Comparison
---

翻译自：[Fravoll-String Equality Comparison](https://fravoll.github.io/solidity-patterns/string_equality_comparison.html)

比较两个给定字符串是否相等，是 Solidity 编程中最常见的一种情况，但语言本身并没有提供内置函数用于字符串比较，本文介绍两种可用方法并分析 Gas 消耗情况。

## 1. 两种种比较方法

下面介绍社区中提出过的比较方法

### 1.1 StringUtils 库

第一种方法是使用 Ethereum 基金会提供的 StringUtils 库，它对每个字符进行成对比较，如果有一个字符对不匹配，则返回false。这种办法可以返回正确的结果，对于短字符串和字符不同发生在字符串前面的情况仅消耗少量 Gas。但是对于相等的字符串和长字符串，这种方法的 Gas 消耗较高，因为必须做很多比较才能得到正确结果。因此，字符串比较的两个可衡量的因素是字符串平均长度和正确率。

### 1.2 哈希函数

作者提出使用哈希函数进行比较，同时检查所提供的字符串的长度，从一开始就剔除长度不匹配的字符串。其步骤如下

1. 检查两个字符串是否有相同长度，通过转换为 `bytes` 类型完成，因为 `bytes` 类型有内置长度函数。如果相同进入第2步，如果不相同返回结果；
2. 使用 `keccak256()` 函数对两个字符串求哈希，然后比较计算得到的哈希值，从而确定是否相等。

一个示例代码如下

```solidity
# 这段代码未经安全审计，使用有风险
function hashCompareWithLengthCheck(string a, string b) internal returns (bool) {
    if(bytes(a).length != bytes(b).length) {
        return false;
    } else {
        return keccak256(abi.encodePacket(a)) == keccak256(abi.encodePacket(b));
    }
}
```

`abi.encodePacket(...) returns (bytes)` 用于对给定参数执行[紧打包编码](https://solidity-cn.readthedocs.io/zh/develop/abi-spec.html#abi-packed-mode)，官方文档中不推荐使用 `keccak256(...)` 直接计算哈希，而是使用 `keccak256(abi.encodePacked(...))`

## 2. Gas 消耗分析

在 Remix 编写代码测试了三种不同情况的字符串比较的 Gas 消耗

1. 比较哈希
2. 比较每个字符，同时比较字符串长度
3. 比较哈希，同时比较字符串长度

结果如下表所示，输入列为输入的待比较字符串，输出列的单位为 Wei

| Input A                        | Input B                    | Hash | Character + Length | Hash + Length |
| :----------------------------- | :------------------------- | ---: | -----------------: | ------------: |
| abcdefghijklmnopqrstuvwxyz     | abcdefghijklmnopqrstuvwxyz | 1225 |               7062 |          1261 |
| abcdefghijklmnopqrstuvwxy**X** | abcdefghijklmnopqrstuvwxyz | 1225 |               7012 |          1261 |
| **X**bcdefghijklmnopqrstuvwxyz | abcdefghijklmnopqrstuvwxyz | 1225 |                912 |          1261 |
| a**X**cdefghijklmnopqrstuvwxyz | abcdefghijklmnopqrstuvwxyz | 1225 |               1156 |          1261 |
| ab**X**defghijklmnopqrstuvwxyz | abcdefghijklmnopqrstuvwxyz | 1225 |               1400 |          1261 |
| abcdefghijkl                   | abcdefghijklmnopqrstuvwxyz | 1225 |                690 |           707 |
| a                              | a                          | 1225 |                962 |          1261 |
| ab                             | ab                         | 1225 |               1156 |          1261 |
| abc                            | abc                        | 1225 |               1450 |          1261 |

可以看出，哈希+字符串长度 的比较方式 Gas 消耗更加稳定，这种方式比较高效。