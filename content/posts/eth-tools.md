---
date: "2018-08-10 18:57:09"
title: 以太坊开发总结
---

最近因公司项目需要，做为一个打杂工程师，操起键盘和笔记本开始了以太坊的踩坑之旅。以太坊的开发比较新，变化也比较多，还好有[@cctanfujun](https://github.com/cctanfujun)的手把手带领下，半只脚踏入了以太坊的开发的大门。

在这篇文章中，我将会简单介绍一下以太坊的基本概念，以及我现在用到的一些工具，还有具体的一个开发流程。因为我还没有接触到如何上主链，所以这些都是基于测试链讲解。希望能给大家带来一些帮助。

什么是区块链

相信大家对区块链都有自己的理解，不仅仅是互联网公司，传统企业也在“币改转型”。

**简言之，区块链就是数据库。**它是特定数据的数据库，里面的数据不断增长，具有非凡特性：

1. 一旦数据存储于数据库，永远都无法被修改或删除。区块链上的每个记录会被永久保存下来。
2. 没有单独的个人或组织能维护该数据库。必须要上千个人才行，每个人都有数据库的副本。

什么是以太坊？

> **以太坊**（英语：Ethereum）是一个[开源](https://zh.wikipedia.org/wiki/%E5%BC%80%E6%BA%90)的有[智能合约](https://zh.wikipedia.org/wiki/%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6)功能的公共[区块链](https://zh.wikipedia.org/wiki/%E5%8C%BA%E5%9D%97%E9%93%BE)平台[[1\]](https://zh.wikipedia.org/wiki/%E4%BB%A5%E5%A4%AA%E5%9D%8A#cite_note-Gray-2014-04-07-1)[[2\]](https://zh.wikipedia.org/wiki/%E4%BB%A5%E5%A4%AA%E5%9D%8A#cite_note-Vigna-28-Oct-2015-2)。通过其专用[加密货币](https://zh.wikipedia.org/wiki/%E5%8A%A0%E5%AF%86%E8%B4%A7%E5%B8%81)[以太币](https://zh.wikipedia.org/wiki/%E4%BB%A5%E5%A4%AA%E5%B8%81)（Ether，又称“以太币”）提供[去中心化](https://zh.wikipedia.org/wiki/%E5%8E%BB%E4%B8%AD%E5%BF%83%E5%8C%96)的[虚拟机](https://zh.wikipedia.org/wiki/%E8%99%9A%E6%8B%9F%E6%9C%BA)（称为“以太虚拟机”Ethereum Virtual Machine）来处理[点对点](https://zh.wikipedia.org/wiki/%E7%82%B9%E5%AF%B9%E7%82%B9)合约。

为什么选择以太坊？

- 智能合约
- 代币
- 资料相对完善，相对容易开发

- 大佬对以太坊比较熟悉
- 大佬对以太坊比较熟悉
- 大佬对以太坊比较熟悉

重要的事情说三遍，有一个经验丰富的人带领，做东西肯定事半功倍。

自己动手写区块链

这里提供两个教程，一个是书，一个是视频。其中视频和书是对应的，不清楚是不是同一个作者。

[Blockchain Tutorial](https://github.com/liuchengxu/blockchain-tutorial)

[私有区块链，我们一起GO](https://www.imooc.com/learn/1021?mc_marking=cfb7fb7f097d2fca6dabbe4c5e71cf77&mc_channel=syb38)

以太坊开发

由于我是专注于后端的开发，现在的技术栈是

- node
- go

正式进入以太坊的开发。这是我这段时间接触到的一些资源：

- go-ethereum：也就是geth，官方的go版本的客户端

- solidity：智能合约编程语言
- truffle：智能合约的编程框架，基于nodejs
- Ganache：启动了多个节点本地私链
- Rinkeby：以太坊测试链
- Etherscan：以太坊区块链浏览器，可以查询交易
- MetaMask：chrome的钱包插件
- web3：官方封装的开发Dapp的库，可以调用合约
- truffle-hdwallet-provider：web3的确定性钱包provider

概念

账户和钱包

在以太坊中，一个账号就是一个地址（address），里面有余额。

钱包是保管私钥的地址， 私钥->公钥->地址   这是一个一一对应的关系，钱包里面可以有多个账户。

私钥不同的生成方法，对应着不同的钱包结构，因此分为了确定性钱包和非确定性钱包。

- 比特币最早的客户端（Satoshi client）就是非确定性钱包，钱包是一堆随机生成的私钥的集合。 客户端会预先生成 100 个随机私钥，并且每个私钥只使用一次。
- 确定性钱包则不需要每次转账都要备份，确定性钱包的私钥是对种子进行单向哈希运算生成的，种子是一串由随机数生成器生成的随机数。在确定性钱包中，只要有这个种子，就可以找回所有私钥

HD 钱包是目前常用的确定性钱包 ，说到 HD 钱包，大家可能第一反应会想到硬件钱包 （Hardware Wallet），其实这里的 HD 是 Hierarchical Deterministic（分层确定性）的缩写。

> 所谓分层，就是一个大公司可以为每个子部门分别生成不同的私钥，子部门还可以再管理子子部门的私钥，每个部门可以看到所有子部门里的币，也可以花这里面的币。也可以只给会计人员某个层级的公钥，让他可以看见这个部门及子部门的收支记录，但不能花里面的钱，使得财务管理更方便了。

生成规则是：

1. 生成一个助记词（参见 BIP39）
2. 该助记词使用 PBKDF2 转化为种子（参见 BIP39）
3. 种子用于使用 HMAC-SHA512 生成根私钥（参见 BIP32）
4. 从该根私钥，导出子私钥（参见 BIP32），其中节点布局由BIP44设置

DAPP

以太坊与其他加密货币的主要不同在于，以太坊不是单纯的货币，而是一个环境/平台。在这个平台上，任何人都可以利用区块链的技术，通过智能合约来构建自己的项目和DAPPS（去中心化应用）。DAPPS发布的方式通常是采用被称为“ICO”的众筹方式。简单来说，你需要用你的以太来购买相应DAPP的一些tokens。

代币

为什么不能在这些DAPPS中直接使用以太完成交易？为什么我们需要给DAPPS创造一种原生的货币？

因为即使在现实生活中，我们也在使用某种形式的Token来代替现金。比如：在游乐场里，你先用现金兑换代币，然后用代币来支付各种服务。在这个例子中，现金就是以太，代币就是token。

ERC20：以太坊token标准

简单来说，ERC20是开发者在自己的tokens中必须采用的一套具体的公式/方法，从而确保该token与ERC20兼容。在合约执行过程中，下面的四个行为是ERC20 tokens所需要完成的：

- 获得Token供给总量.
- 获得账户余额.
- 从一方向另一方转移Token.
- 认可Token作为货币性资产的使用.

大佬说：代币其实就是智能合约，而这个合约是发生了0个以太的转账。

Gas和挖矿

不少小哥哥或小姐姐会认为挖矿就是挖以太币，其实代币不用挖的，当你挖到了区块，代币是给你的奖励。因为任何一笔交易都需要记录，一个区块的大小也就几M，存储不了那么多交易信息，所以要持续挖区块来记录交易，同时只要是你发起了交易，就得付手续费，这些手续费也成为Gas，会按照一定的算法奖励给挖出区块的人。

接下来会讲一下，平时开发中如何创建钱包，如何转账，如何自己发代币，如何部署合约并调用。

环境准备

- 安装Ganache并启动
- 安装truffle框架

创建钱包

golang

依赖

```
github.com/ethereum/go-ethereum
```

首先要连接到测试链，测试链可以是本地的也可以是公网的。

```
func connectRPC() (*ethclient.Client, error) {
	// 连接测试链的节点
	//rpcClient, err := rpc.Dial("https://rinkeby.infura.io/v3/6c81fb1b66804f0698d49f2ec242afc9")
	rpcClient, err := rpc.Dial("http://127.0.0.1:7545")
	if err != nil {
		log.Fatalln(err)
		return nil, err
	}

	conn := ethclient.NewClient(rpcClient)
	return conn, nil
}
```

一般都选择以keystore的形式创建账户

```
func CreateWallet() (key, addr string) {
	ks := keystore.NewKeyStore("~/Documents/github/gowork/src/geth-demo/",
		keystore.StandardScryptN, keystore.StandardScryptP)
	account, _ := ks.NewAccount("password")
	key_json, err := ks.Export(account, "password", "password")
	if err != nil {
		log.Fatalln("导出账户错误: ", err)
		panic(err)
	}
	key = string(key_json)
	addr = account.Address.Hex()
	return
}
```

当然另一种创建账户的方式是用私钥：

```
func CreateWallet() (string, error) {
	key, err := crypto.GenerateKey()
	if err != nil {
		log.Fatalln(err)
		return "", nil
	}

	address := crypto.PubkeyToAddress(key.PublicKey).Hex()
	log.Println("address: ", address)

	privateKey := hex.EncodeToString(key.D.Bytes())
	log.Println("privateKey: ", privateKey)
	return address, nil
}
```

Node

node一般使用web3。创建web3对象的时候要使用一个provider，这个provider用来连接到测试链，可以是钱包的，也可以是一个HttpProvider。

创建web3

```
const web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:7545"));
```

或者使用`truffle-hdwallet-provider`来创建，使用这个的前提是，自己已经创建了一个钱包，并且这个钱包是HD的。

```
const Web3 = require('web3');
const HDWalletProvider = require('truffle-hdwallet-provider');

const provider = new HDWalletProvider(助记词, 测试链url);
const web3 = new Web3(provider);
```

创建账户

```

```

