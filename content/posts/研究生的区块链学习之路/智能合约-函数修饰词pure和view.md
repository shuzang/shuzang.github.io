---
title: 函数修饰词pure和view
author: 深入浅出区块链
date: 2020-05-04
tags: [智能合约]
categories: [研究生的区块链学习之路]
slug: pure and view keyword
---

转自[深入理解Solidity-函数](https://learnblockchain.cn/docs/solidity/contracts.html#view)

这两个函数修饰词的作用是告诉编译器函数是否会读取/修改状态，view 表示保证不修改状态，pure 表示保证不读取也不修改状态。Solidity v0.4.17 之前没有这两个修饰词，而是使用 constant 关键字，和 view 的含义相同，不过在 v0.5.0 之后被移除，现在只能使用这两个 view 和 pure。

## 1. view 视图函数

Getter 方法会被自动标记为 `view`，除此之外，一个 view 修饰的例子如下

```solidity
pragma solidity  >=0.5.0 <0.7.0;

contract C {
    function f(uint a, uint b) public view returns (uint) {
        return a * (b + 42) + now;
    }
}
```

view 保证函数不修改状态，以下操作会被认为是修改状态

1. 修改状态变量。
2. 产生事件。
3. 创建其它合约。
4. 使用 `selfdestruct`。
5. 通过调用发送以太币。
6. 调用任何没有标记为 `view` 或者 `pure` 的函数。
7. 使用低级调用。
8. 使用包含特定操作码的内联汇编。

## 2. pure 纯函数

pure 保证不读取也不修改状态，不修改的定义上面已经提到，下面的操作被认为是读取状态

1. 读取状态变量。
2. 访问 `address(this).balance` 或者 `.balance`。
3. 访问 `block`，`tx`， `msg` 中任意成员 （除 `msg.sig` 和 `msg.data` 之外）。
4. 调用任何未标记为 `pure` 的函数。
5. 使用包含某些操作码的内联汇编。

一个 pure 修饰的例子如下

```solidity
pragma solidity >=0.5.0 <0.7.0;

contract C {
    function f(uint a, uint b) public pure returns (uint) {
        return a * (b + 42);
    }
}
```

