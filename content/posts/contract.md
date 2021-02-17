---
date: "2018-08-03 14:32:03"
title: contract
---

## 类型

Solidity是静态类型的语言。

### 值类型

- bool
- int/uint
- fixed/unfixed
- address
  - balance和transfer
  - send
  - call, callcode和delegatecall
- byte
- bytes 和 string
- 十六进制hex"0012"
- enum
- function

### 引用类型

- 数组

  uint[] 

- 结构体

  struct

- 映射

  mapping(key => value)

## 单元和全局变量

- 以太币的单位

  在数字后面加上 `wei`、 `finney`、 `szabo` 或 `ether`。默认是wei

- 时间单位

  数字后面带有 `seconds`、 `minutes`、 `hours`、 `days`、 `weeks` 和 `years`。默认是秒。

- 区块和交易

  - `block.blockhash(uint blockNumber) returns (bytes32)`：指定区块的区块哈希。
  - `block.coinbase` (`address`): 挖出当前区块的矿工地址
  - `block.difficulty` (`uint`): 当前区块难度
  - `block.gaslimit` (`uint`): 当前区块 gas 限额
  - `block.number` (`uint`): 当前区块号
  - `block.timestamp` (`uint`): 自 unix epoch 起始当前区块以秒计的时间戳
  - `gasleft() returns (uint256)`：剩余的 gas
  - `msg.data` (`bytes`): 完整的 calldata
  - `msg.gas` (`uint`): 剩余 gas - 自 0.4.21 版本开始已经不推荐使用，由 `gesleft()` 代替
  - `msg.sender` (`address`): 消息发送者（当前调用）
  - `msg.sig` (`bytes4`): calldata 的前 4 字节（也就是函数标识符）
  - `msg.value` (`uint`): 随消息发送的 wei 的数量
  - `now` (`uint`): 目前区块时间戳（`block.timestamp`）
  - `tx.gasprice` (`uint`): 交易的 gas 价格
  - `tx.origin` (`address`): 交易发起者（完全的调用链）

- 地址相关

  - `<address>.balance` (`uint256`):

    以 Wei 为单位的 [地址类型](http://solidity-cn.readthedocs.io/zh/develop/types.html#address) 的余额。

  - `<address>.transfer(uint256 amount)`:

    向 [地址类型](http://solidity-cn.readthedocs.io/zh/develop/types.html#address) 发送数量为 amount 的 Wei，失败时抛出异常，发送 2300 gas 的矿工费，不可调节。

  - `<address>.send(uint256 amount) returns (bool)`:

    向 [地址类型](http://solidity-cn.readthedocs.io/zh/develop/types.html#address) 发送数量为 amount 的 Wei，失败时返回 `false`，发送 2300 gas 的矿工费用，不可调节。

  - `<address>.call(...) returns (bool)`:

    发出低级函数 `CALL`，失败时返回 `false`，发送所有可用 gas，可调节。

  - `<address>.callcode(...) returns (bool)`：

    发出低级函数 `CALLCODE`，失败时返回 `false`，发送所有可用 gas，可调节。

  - `<address>.delegatecall(...) returns (bool)`:

    发出低级函数 `DELEGATECALL`，失败时返回 `false`，发送所有可用 gas，可调节。

- 合约相关

  `this` (current contract's type):

  当前合约，可以明确转换为 [地址类型](http://solidity-cn.readthedocs.io/zh/develop/types.html#address)。

  `selfdestruct(address recipient)`:

  销毁合约，并把余额发送到指定 [地址类型](http://solidity-cn.readthedocs.io/zh/develop/types.html#address)。

  `suicide(address recipient)`:

  与 selfdestruct 等价，但已不推荐使用。

## 控制结构

输入参数和我们常见的函数的参数相同

输出参数必须要在returns后面，和go的类似

```
function arithmetics(uint _a, uint _b) returns (uint o_sum, uint o_product) {
        o_sum = _a + _b;
        o_product = _a * _b;
}

function arithmetics(uint _a, uint _b) returns (uint , uint ) {
        o_sum = _a + _b;
        o_product = _a * _b;
    	return o_sum, o_product;
}
```

不能用switch和goto

内部函数调用，就和普通的方法调用一样。

从外部调用合约的函数，先创建一个合约实例（和类的对象一样），然后调用实例方法。

调函数要发送wei和gas，就像下图所示：

```
pragma solidity ^0.4.0;

contract InfoFeed {
    function info() public payable returns (uint ret) { return 42; }
}

contract Consumer {
    InfoFeed feed;
    function setFeed(address addr) public { feed = InfoFeed(addr); }
    function callFeed() public { feed.info.value(10).gas(800)(); }
}
```

`payable` 修饰符要用于修饰 `info`，否则，.value() 选项将不可用。

 可以通过new创建一个合约，和new出一个对象一样。

## 合约结构

- 状态变量

  状态变量是永久存储在合约中的值，其实可以理解为类的成员变量。

- 函数

  函数是合约的可执行单元，可以理解为类的成员函数。

- 函数修饰器

  以声明的形式改良函数语义。

- 事件

  以太坊的日志工具接口。

  ```
  event HighestBidIncreased(address bidder, uint amount); // 事件
  
  emit HighestBidIncreased(msg.sender, msg.value); // 触发事件
  ```

- 结构体

  理解为数据类。

- 枚举

合约函数可见性修饰

