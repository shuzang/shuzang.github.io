# 智能合约知识点总结


项目进行过程中知识点的总结

## 1. 字符串比较

翻译自：[Fravoll-String Equality Comparison](https://fravoll.github.io/solidity-patterns/string_equality_comparison.html)

比较两个给定字符串是否相等，是 Solidity 编程中最常见的一种情况，但语言本身并没有提供内置函数用于字符串比较，本文介绍两种可用方法并分析 Gas 消耗情况。

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

### 1.3 Gas 消耗分析

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

## 2. 可见性与Getter函数

转自：[Solidity 0.6.4 中文文档](https://learnblockchain.cn/docs/solidity/contracts.html#getter)

Solidity 有两种函数调用：内部调用（Internal Function Calls）和外部调用（External Function Calls）。前者指直接或递归地调用合约内部函数，不会产生实际的 EVM 调用，因此也被称为「消息嗲用」，后者指从合约外部调用合约中的函数，会产生一个 EVM 调用。

### 2.1 可见性

因此，函数和状态变量有四种可见性。函数可以指定为 `external`，`public`，`internal` 或 `private`，对于状态变量， 默认是 `internal` 且不能设置为 `external`。

- `external`：外部函数作为合约接口的一部分，意味着我们可以从其他合约和交易中调用。 一个外部函数 `f` 不能从内部调用（即 `f` 不起作用，但 `this.f()`可以）。 当收到大量数据的时候，外部函数有时候会更有效率，因为数据不会从calldata复制到内存.
- `public`：public 函数是合约接口的一部分，可以在内部或通过消息调用。对于 public 状态变量， 会自动生成一个 getter 函数（见下面）。
- `internal`：这些函数和状态变量只能是内部访问（即从当前合约内部或从它派生的合约访问），不使用 `this` 调用。
- `private`：private 函数和状态变量仅在当前定义它们的合约中使用，并且不能被派生合约使用。

> 合约中的所有内容对外部观察者都是可见的。设置一些 `private` 类型只能阻止其他合约访问和修改这些信息， 但是对于区块链外的整个世界它仍然是可见的。

可见性标识符的定义位置，对于状态变量来说是在类型后面，对于函数是在参数列表和返回关键字中间，如下例

```solidity
pragma solidity  >=0.4.16 <0.7.0;

contract C {
    function f(uint a) private pure returns (uint b) { return a + 1; }
    function setData(uint a) internal { data = a; }
    uint public data;
}
```

在下面的例子中，`D` 可以调用 `c.getData（）` 来获取状态存储中 `data` 的值，但不能调用 `f` 。 合约 `E` 继承自 `C` ，因此可以调用 `compute`。

```solidity
pragma solidity >=0.4.0 <0.7.0;

contract C {
    uint private data;

    function f(uint a) private returns(uint b) { return a + 1; }
    function setData(uint a) public { data = a; }
    function getData() public returns(uint) { return data; }
    function compute(uint a, uint b) internal returns (uint) { return a+b; }
}

// 下面代码编译错误
contract D {
    function readData() public {
        C c = new C();
        uint local = c.f(7); // 错误：成员 `f` 不可见
        c.setData(3);
        local = c.getData();
        local = c.compute(3, 5); // 错误：成员 `compute` 不可见
    }
}

contract E is C {
    function g() public {
        C c = new C();
        uint val = compute(3, 5); // 访问内部成员（从继承合约访问父合约成员）
    }
}
```

### 2.2 Getter 函数

编译器自动为所有 **public** 状态变量创建 getter 函数。对于下面给出的合约，编译器会生成一个名为 `data` 的函数， 该函数没有参数，返回值是一个 `uint` 类型，即状态变量 `data` 的值。 状态变量的初始化可以在声明时完成。

```solidity
pragma solidity  >=0.4.0 <0.7.0;

contract C {
    uint public data = 42;
}

contract Caller {
    C c = new C();
    function f() public {
        uint local = c.data();
    }
}
```

getter 函数具有外部（external）可见性。如果在内部访问 getter（即没有 `this.` ），它被认为一个状态变量。 如果使用外部访问（即用 `this.` ），它被认作为一个函数。

```solidity
pragma solidity ^0.4.0 <0.7.0;

contract C {
    uint public data;
    function x() public {
        data = 3; // 内部访问
        uint val = this.data(); // 外部访问
    }
}
```

如果你有一个数组类型的 `public` 状态变量，那么你只能通过生成的 getter 函数访问数组的单个元素。 这个机制以避免返回整个数组时的高成本gas。 可以使用如 `data(0)` 用于指定参数要返回的单个元素。 如果要在一次调用中返回整个数组，则需要写一个函数，例如：

```solidity
pragma solidity >=0.4.0 <0.7.0;

contract arrayExample {
  // public state variable
  uint[] public myArray;

  // 指定生成的Getter 函数
  /*
  function myArray(uint i) public view returns (uint) {
      return myArray[i];
  }
  */

  // 返回整个数组
  function getArray() public view returns (uint[] memory) {
      return myArray;
  }
}
```

现在可以使用 `getArray()` 获得整个数组，而 `myArray(i)` 是返回单个元素。

下一个例子稍微复杂一些：

```solidity
pragma solidity ^0.4.0 <0.7.0;

contract Complex {
    struct Data {
        uint a;
        bytes3 b;
        mapping (uint => uint) map;
    }
    mapping (uint => mapping(bool => Data[])) public data;
}
```

这将会生成以下形式的函数

```solidity
function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
    a = data[arg1][arg2][arg3].a;
    b = data[arg1][arg2][arg3].b;
}
```

请注意，因为没有好的方法来提供映射的键，所以结构中的映射被省略。

## 3. 合约间调用

之前的实验合约间的调用没有成功，这次就仔细地研究一下合约间地调用机制。分为两种情况

1. 调用者和被调用者在一个sol文件中
2. 调用者和被调用者在不同的sol文件中

本文提到的合约调用方法的实质是抽象合约的使用。

### 3.1 同sol文件的智能合约调用

下面的智能合约中，Main和Add两个合约定义在一个Main.sol文件中，可以同时编译，然后逐个部署。

```js
pragma solidity ^0.5.0;


contract Main {
  Add add;
  
  constructor(address _m) public {
     add = Add(_m);
  }
  
  function Addnumber() public view returns (uint) {
    return add.add5(10);
  }
}

contract Add {
  function add5(uint s) public pure returns (uint){
      return 5+s;
  }
}
```

以使用Remix为例，点击编译按钮编译Main.sol文件，将会同时编译Main和Add两个合约。

![编译](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191108_68472178-e6294d00-025a-11ea-8b4b-41a53b471c18.png)

 然后首先部署Add合约，因为Main合约的部署需要Add的合约地址作为参数。切换到部署和运行选项卡，选择Add合约，点击`Deploy`，成功部署后，复制合约地址。

![deploy simple Add](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191108_68472256-0e18b080-025b-11ea-9a24-e324c82cd7b5.png)

然后重新选择Main合约，填入Add合约地址作为参数，点击部署按钮。

![deploy simple Main](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191108_68472285-1a047280-025b-11ea-8bb5-a1fb5c65574e.png)

测试合约间调用，由合约内容可知，Main合约中的Addnumber函数调用了Add合约的add5函数，传入参数为10，得到的结果应为15。展开左侧的`Deployed Contracts`，点击Addnumber进行调用，结果如下。

![call test](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191108_68472220-f5a89600-025a-11ea-9d2a-b39c8e39a810.png)

### 3.2 不同sol文件的智能合约调用

这一次我们测试不同sol文件的智能合约调用，来一个复杂一点的，两个合约分别是Add.sol和Main.sol。

Add.sol使用了一个结构体来定义数值，并通过映射定义查找表来寻找这个值。文件中定义了两个函数，numRegister用来向表中添加数值，addValue用来将从表中查到的指定值+5返回。之所以用这个结构是因为我们的项目里用到了，这里来测试一下可不可行。

```js
pragma solidity ^0.5.0;

contract Add {
    struct Num{
        uint value;
    }
    mapping(uint => Num) public lookupTable;
    
    function numRegister(uint key, uint _value) public {
        lookupTable[key].value = _value;
    }
    
    function addValue(uint key) public view returns (uint) {
        return lookupTable[key].value + 5;
    }

}
```

Main.sol没有多大变化

```js
pragma solidity ^0.5.0;

contract Main {
  Add add;
  
  constructor(address _m) public {
     add = Add(_m);
  }
  
  function Addnumber() public view returns (uint) {
    return add.addValue(5);
  }
}

contract Add {
      function addValue(uint key) public view returns (uint);
}
```

仍然是先编译部署Add合约，部署后调用numRegister函数写入数值5，并调用addValue函数测试返回。

![deploy comlex Add](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191108_68472316-2d174280-025b-11ea-96a6-51af8dd4fd4c.png)

接着编译部署Main合约，复制Add合约地址作为初始化参数，部署后调用Addnumber函数测试

![deploy comlex Main](https://picped-1301226557.cos.ap-beijing.myqcloud.com/YJS_20191108_68472331-399b9b00-025b-11ea-933f-8c03ff95c55a.png)

### 3.3 总结

合约内的调用方法是相同的，都要先实例化，然后传入被调合约地址，接着才能调用。而写在不同sol文件中时，需要额外声明被调合约的抽象合约，有些文章中说使用`call`，`callcode`或`delegatecall`，但并不建议，因为这三个函数都是非常底层的函数，破坏了类型的安全，只能作为最后的手段使用。

详细的解释参考了[StackExchange-Calling function from deployed contract](https://ethereum.stackexchange.com/questions/9733/calling-function-from-deployed-contract)

## 4. 函数修饰词pure和view

转自[深入理解Solidity-函数](https://learnblockchain.cn/docs/solidity/contracts.html#view)

这两个函数修饰词的作用是告诉编译器函数是否会读取/修改状态，view 表示保证不修改状态，pure 表示保证不读取也不修改状态。Solidity v0.4.17 之前没有这两个修饰词，而是使用 constant 关键字，和 view 的含义相同，不过在 v0.5.0 之后被移除，现在只能使用这两个 view 和 pure。

### 4.1 view 视图函数

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

### 4.2 pure 纯函数

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

## 5. 浮点数处理

首先声明，Solidity 中支持浮点数定义，但无法赋值和进行计算。文档中对其描述是「目前还不完全支持」，虽然这意味着以后可能会完全支持，但等不及了，下面记录几个可参考的资料。

1. 来自 [ethereum stackexchange](https://ethereum.stackexchange.com/questions/83785/what-fixed-or-float-point-math-libraries-are-available-in-solidity) 中的回答，介绍了一些可用的库；
2. [ABDK Math Quad](https://github.com/abdk-consulting/abdk-libraries-solidity/blob/master/ABDKMathQuad.md)，包含两个合约库，一个支持定点数，一个支持浮点数；
3. Mikhail Vladimirov 的 [Math in Solidity](https://medium.com/coinmonks/math-in-solidity-part-1-numbers-384c8377f26d) 系列文章，介绍如何在 Solidity 中处理各种数学运算，写的非常棒。

## 6. 地址类型

在智能合约中显式传入地址类型时，可能会出现如下错误

> Address checksum
>
> This looks like an address but has an invalid checksum. If this is not used as an address, please prepend '00'.

关于该问题的一个讨论见 <https://github.com/ethereum/EIPs/issues/55>

这是因为合约中现在使用地址类型必须做一个转换，不是简单的全部大写字母或小写字母，而是遵循一定的规则，这个规则见 [ethereum/EIPs#55]( https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md )

但是网上提供的解决方案一般是使用JS库中的转换函数，在智能合约中无法直接解决，好在，web3提供了一个[在线API接口](https://web3-tools.netlify.com/)，可以调用其`checkAddressChecksum`函数对地址进行转换，然后将转换后的结果直接用于合约代码。

## 7. Gas limit问题

在搭建的以太坊私链上进行智能合约部署时，出现了以下问题

```js
INFO [03-21|13:50:11.690] Served eth_sendTransaction               reqid=24 t=684.186µs    err="exceeds block gas limit"
Error: exceeds block gas limit undefined
```

出现该错误的原因如错误描述，是当前合约所需的gas超过了区块的最大gas。这可能与参数gasLimit有关。在创世区块的配置文件中，我们使用了默认的配置值，为`0x2fefd8`，转换为10进制即`3141592`。

注：[在线转换工具](http://tool.oschina.net/hexconvert/)

<!--more-->

### 原因查找

因为部署智能合约之前已经进行过挖矿，区块链中已有数个区块，我们查询目前的gasLimit值。

```js
>eth.getBlock(eth.blockNumber)
{
  difficulty: 131072,
  extraData: "0xd683010900846765746886676f312e3132856c696e7578",
  gasLimit: 3147727,
  gasUsed: 0,
  hash: "0x7e03472bcad02f6e85a3cdb21cfba856da58a4955dd2b6d21e3b8561446ae390",
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  miner: "0x79b43b2196723fff1485999aba45fda3e8b4df58",
  mixHash: "0x10a5130f4ea573f1f1599c11b8ade9ac3feb256c0414db1a277b7b63e8343d48",
  nonce: "0x6bba1166a347ba0f",
  number: 2,
  parentHash: "0xed8c7febfc1ab5e4e388bd886be1182635e77b0047f530c93af4eb31f898bd7c",
  receiptsRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  sha3Uncles: "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  size: 534,
  stateRoot: "0x741c086895803cc3f85c8e7fb738acfb42aa03a12a03edf246b1c14055123b78",
  timestamp: 1552396507,
  totalDifficulty: 263168,
  transactions: [],
  transactionsRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  uncles: []
}
```

发现此时区块的gasLimit值为`3147727`。

查找我们部署的合约web3代码

```js
var testContract = web3.eth.contract([{"constant":true,"inputs":[{"name":"a","type":"uint256"}],"name":"multiply","outputs":[{"name":"d","type":"uint256"}],"payable":false,"stateMutability":"pure","type":"function"}]);
var test = testContract.new(
   {
     from: web3.eth.accounts[0], 
     data: '0x6080604052348015600f57600080fd5b5060a58061001e6000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c8063c6888fa114602d575b600080fd5b605660048036036020811015604157600080fd5b8101908080359060200190929190505050606c565b6040518082815260200191505060405180910390f35b600060078202905091905056fea165627a7a72305820028fd57d2fec4df9170b559fe84245ed4f81bc40f3cad3c185c8035501bdb3220029', 
     gas: '4700000'
   }, function (e, contract){
    console.log(e, contract);
    if (typeof contract.address !== 'undefined') {
         console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
    }
 })
```

发现合约所需gas为`4700000`，比gasLimit值高，所以部署失败，出现了`Error: exceeds block gas limit undefined`的错误

### 解决办法

第一种解决办法是修改`genesis.json`中的gasLimit参数，设置一个更大的值。但这样做需要重新构建网络，极为繁琐。

另一种解决办法是通过geth命令的`--targetgaslimit`参数来调整gasLimit值

```js
> --targetgaslimit 4712388
```

这里没有调整成功，提示原因是端口在运行，可能和docker有关，不知道怎么解决。

该问题最后是通过调整web3中的gaslimit值解决的，因为这个简单的智能合约怎么看都不像能消耗4700000gas的样子，果然查询之后发现只消耗100000左右，于是将web3代码中的gaslimit调整到120000，重新部署，果然成功。

一个关于gaslimit的解释见：[以太坊Block Gaslimit动态调整机制分析](https://tinycalf.github.io/2018/07/02/20180702-%E4%BB%A5%E5%A4%AA%E5%9D%8A%E5%9D%97gas%E4%B8%8A%E9%99%90%E5%8A%A8%E6%80%81%E8%B0%83%E6%95%B4%E7%A0%94%E7%A9%B6/)


---

> 作者:   
> URL: https://shuzang.github.io/2020/summary-of-smart-contract-knowledge-points/  

