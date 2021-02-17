---
date: "2020-11-20T10:07:39+08:00"
draft: false
title: Geth
---

## Geth简介

**go-ethereum**

go-ethereum客户端通常被称为geth，它是个命令行界面，执行在Go上实现的完整以太坊节点。通过安装和运行geth，可以参与到以太坊前台实时网络并进行以下操作：

- 挖掘真的以太币
- 在不同地址间转移资金
- 创建合约，发送交易
- 探索区块历史
- 及很多其他

> 网站: http://ethereum.github.io/go-ethereum/

> Github: https://github.com/ethereum/go-ethereum

> 维基百科: https://github.com/ethereum/go-ethereum/wiki/geth

> Gitter: https://gitter.im/ethereum/go-ethereum

## mac下安装geth

1. 首先安装homebrew，
2. 使用brew安装即可。在安装geth的时候，会将go也安装上。

```shell
brew tap ethereum/ethereum
brew install ethereum
```

3. 在命令行输入geth —help，如果出现

   ```shell
   zhuzhenengdeMBP:blog zhuzhenfeng$ geth --help
   NAME:
      geth - the go-ethereum command line interface

      Copyright 2013-2017 The go-ethereum Authors

   USAGE:
      geth [options] command [command options] [arguments...]
      
   VERSION:
      1.7.3-unstable-eea996e4
   ```

   证明安装成功。

## 使用Geth

1. 打开终端，输入以下命令，以开发的方式启动geth

   ```
   geth  --datadir “~/Documents/github/ethfans/ethdev” --dev
   ```

   --datadir 是指定geth的开发目录，引号的路径可以随便设置

2. 新开一个终端，执行以下命令，进入geth的控制台

   ```
   geth --dev console 2>>file_to_log_output
   ```

   该命令会将在console中执行的命令，生成一个文本保存在file_to_log_output文件中。

3. 再新开一个终端，查看打印出来的日志

   ```
   tail -f file_to_log_output
   ```

切换到geth控制台终端，geth有如下常用的命令

- `eth.accounts`

  查看有什么账户

- `personal.newAccount('密码')`

  创建一个账户

- `user1=eth.accounts[0]`

  可以把账户赋值给某一个变量

- `eth.getBalance(user1)`

  获取某一账户的余额

- `miner.start()`

  启动挖矿程序

- `miner.stop()`

  停止挖矿程序

- `eth.sendTransaction({from: user1,to: user2,value: web3.toWei(3,"ether")})`

  从user1向user2转以太币

- `personal.unlockAccount(user1, '密码')`

  解锁账户

以太坊启动挖矿程序的时候，头结点会产生以太币，在进行转账操作之后，必须进行挖矿才会使交易成功。
