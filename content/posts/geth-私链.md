---
date: "2020-11-20T15:59:22+08:00"
draft: false
title: Geth 私链
---

在上一篇文章[《Geth入门》](https://github.com/zhenfeng-zhu/articles/issues/1)中，主要讲了开发环境下以太坊geth客户端的使用。今天简单说下私链的配置。

## genesis.json

```json
{
    "config": {
          "chainId": 10,
          "homesteadBlock": 0,
          "eip155Block": 0,
          "eip158Block": 0
    },
    "coinbase"   : "0x0000000000000000000000000000000000000000",
    "difficulty" : "0x40000",
    "extraData"  : "",
    "gasLimit"   : "0xffffffff",
    "nonce"      : "0x0000000000000042",
    "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
    "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
    "timestamp"  : "0x00",
    "alloc": { }
}
```

| 参数         | 描述                                       |
| ---------- | ---------------------------------------- |
| nonce      | nonce就是一个64位随机数，用于挖矿                     |
| mixhash    | 与nonce配合用于挖矿，由上一个区块的一部分生成的hash           |
| difficulty | 设置当前区块的难度，如果难度过大，cpu挖矿就很难，这里设置较小难度       |
| alloc      | 用来预置账号以及账号的以太币数量，因为私有链挖矿比较容易，所以我们不需要预置有币的账号，需要的时候自己创建即可以 |
| coinbase   | 矿工的账号，随便填                                |
| timestamp  | 设置创世块的时间戳                                |
| parentHash | 上一个区块的hash值，因为是创世块，所以这个值是0               |
| extraData  | 附加信息，随便填，可以填你的个性信息                       |
| gasLimit   | 该值设置对GAS的消耗总量限制，用来限制区块能包含的交易信息总和，因为我们是私有链，所以填最大。 |
| config     | Fatal: failed to write genesis block: genesis has no chain configuration ：这个错误信息，就是说，你的json文件中，缺少config部分。看到这个信息，我们不需要把geth退回到v1.5版本，而是需要加上config部分。 |

## 创建创世区块

打开终端，输入以下命令，在当前目录下创建创世区块。

```
geth --datadir "./" init genesis.json
```

可以发现在当前目录新增了两个文件夹：

- geth中保存的是区块链的相关数据
- keystore中保存的是该链条中的用户信息

## 启动私链

```
geth --datadir "./" --nodiscover console 2>>geth.log
```

- `--datadir`：代表以太坊私链的创世区块的地址
- `--nodiscover`：私链不要让公链上的节点发现

也可将此命令写入一个shell文件中，每次启动的时候执行脚本就可以了。

输入此命令后，就可以进入到geth的控制台中了，在这里可以进行挖矿，智能合约的编写。