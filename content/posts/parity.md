---
date: "2018-08-22 16:51:13"
title: 基于以太坊的Parity联盟链部署
---

公司项目中使用公网上的以太坊私链，交易速度比较慢，于是这几天都在鼓捣基于以太坊的联盟链，parity是可以构建出一个基于PoA共识的私链，而且兼容以太坊的合约。这篇文章主要是记录自己的踩坑经历，主要实现了节点的搭建，合约的部署以及本地以太坊浏览器的启动。

## 部署联盟链

parity的文档：https://wiki.parity.io/Demo-PoA-tutorial

### 安装

首先是下载parity，在mac下是直接brew安装即可。

```
brew tap paritytech/paritytech

brew install parity
```

### 创世区块

创世区块的配置文件：

```on
// demo-spec.json

{
  "name": "DemoPoA",
  "engine": {
    "authorityRound": {
      "params": {
        "stepDuration": "5",
        "validators": {
          "list": [
            "0x00bd138abd70e2f00903268f3db08f2d25677c9e",
            "0x00aa39d30f0d20ff03a22ccfc30b7efbfca597c2"
          ]
        }
      }
    }
  },
  "params": {
    "gasLimitBoundDivisor": "0x400",
    "maximumExtraDataSize": "0x20",
    "minGasLimit": "0x1388",
    "networkID": "0x2323",
    "eip155Transition": 0,
    "validateChainIdTransition": 0,
    "eip140Transition": 0,
    "eip211Transition": 0,
    "eip214Transition": 0,
    "eip658Transition": 0
  },
  "genesis": {
    "seal": {
      "authorityRound": {
        "step": "0x0",
        "signature": "0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
      }
    },
    "difficulty": "0x20000",
    "gasLimit": "0x5B8D80"
  },
  "accounts": {
    "0x0000000000000000000000000000000000000001": {
      "balance": "1",
      "builtin": {
        "name": "ecrecover",
        "pricing": { "linear": { "base": 3000, "word": 0 } }
      }
    },
    "0x0000000000000000000000000000000000000002": {
      "balance": "1",
      "builtin": {
        "name": "sha256",
        "pricing": { "linear": { "base": 60, "word": 12 } }
      }
    },
    "0x0000000000000000000000000000000000000003": {
      "balance": "1",
      "builtin": {
        "name": "ripemd160",
        "pricing": { "linear": { "base": 600, "word": 120 } }
      }
    },
    "0x0000000000000000000000000000000000000004": {
      "balance": "1",
      "builtin": {
        "name": "identity",
        "pricing": { "linear": { "base": 15, "word": 3 } }
      }
    },
    "0x004ec07d2329997267ec62b4166639513386f32e": {
      "balance": "10000000000000000000000"
    }
  }
}
```

### node0

node0节点：

```toml
## node0.toml

[parity]
chain = "demo-spec.json"
base_path = "parity0"

[network]
port = 30300

[rpc]
port = 8546
apis = ["web3", "eth", "net", "personal", "parity", "parity_set", "traces", "rpc", "parity_accounts"]
interface = "0.0.0.0"
cors = ["*"]
hosts = ["all"]


[websockets]
port = 8456


[account]
password = ["node.pwds"]


[mining]
engine_signer = "0x00bd138abd70e2f00903268f3db08f2d25677c9e"
reseal_on_txs = "none"

```

### node1

node1节点：

```toml
## node1.toml

[parity]
chain = "demo-spec.json"
base_path = "parity1"


[network]
port = 30301


[rpc]
port = 8541
apis = ["web3", "eth", "net", "personal", "parity", "parity_set", "traces", "rpc", "parity_accounts"]



[websockets]
port = 8451


[ipc]
disable = true


[account]
password = ["node.pwds"]


[mining]
engine_signer = "0x00aa39d30f0d20ff03a22ccfc30b7efbfca597c2"
reseal_on_txs = "none"


[ui]
disable = true
```

### 启动并创建账户

启动

```
parity --config node0.toml --fat-db=on
parity --config node1.toml --fat-db=on
```

创建账户：

```
curl --data '{"jsonrpc":"2.0","method":"parity_newAccountFromPhrase","params":["node0", "node0"],"id":0}' -H "Content-Type: application/json" -X POST localhost:8546
```

```
curl --data '{"jsonrpc":"2.0","method":"parity_newAccountFromPhrase","params":["user", "user"],"id":0}' -H "Content-Type: application/json" -X POST localhost:8546
```

```
curl --data '{"jsonrpc":"2.0","method":"parity_newAccountFromPhrase","params":["node1", "node1"],"id":0}' -H "Content-Type: application/json" -X POST localhost:8541
```

这样就创建了3个账户，其中node0和node1是见证者  user是初始发钱的。

因为parity ui是要连接8546端口，所以这里就让node0的rpc的端口是8546。

### 节点互通和转账

让node0和node1节点相通，其实就是让两个节点成为一个网络：

```
// 获取node0的encode
curl --data '{"jsonrpc":"2.0","method":"parity_enode","params":[],"id":0}' -H "Content-Type: application/json" -X POST localhost:8546

// 调用node1的rpc，将node0加入， RESULT就是上一步获取的
curl --data '{"jsonrpc":"2.0","method":"parity_addReservedPeer","params":["enode://RESULT"],"id":0}' -H "Content-Type: application/json" -X POST localhost:8541
```

我们先给两个账户转账：

```
curl --data '{"jsonrpc":"2.0","method":"personal_sendTransaction","params":[{"from":"0x004ec07d2329997267Ec62b4166639513386F32e","to":"0x00Bd138aBD70e2F00903268F3Db08f2D25677C9e","value":"0xde0b6b3a7640000"}, "user"],"id":0}' -H "Content-Type: application/json" -X POST localhost:8540
```

从user中转了1个以太坊到了node0账户中，同样再转给node1。

## 部署合约

再使用truffle开发完合约之后，把账户部署到我们刚刚起来的联盟链。部署合约需要消耗一定的gas，truffle使用的是HD wallet的Provider，所以我们要先给一个钱包转一些以太币。

因为这里用的是metamask，在最初创建钱包的时候有设置12个助记词，所以先让钱包连接到node0节点：

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fuioo9rlayj30ab0h2gne.jpg)

创建一个账户，向那个账户转几个以太币。

然后在truffle中，配置如下：

```
// truffle.js

module.exports = {
  // See <http://truffleframework.com/docs/advanced/configuration>
  // to customize your Truffle configuration!
  networks: {
    development: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*"
    },
    parity: {
      provider: function () {
        return new HDWalletProvider('这里写助记词', "http://127.0.0.1:8546")
      },
      network_id: 3,
      gas: 4700000
    }
  }
};
```

然后在执行部署合约的时候，指定parity即可：

```
truffle migrate --network parity
```

## 部署以太坊浏览器

以太坊的浏览器找了好几个，最后选中了[etherchain-light](https://github.com/gobitfly/etherchain-light)。部署起来简单。

首先clone代码到本地，然后npm安装依赖。

```
git clone https://github.com/gobitfly/etherchain-light --recursive

cd etherchain-light && yarn
```

一定要用`—-recursive`，将所有git的子模块都下载下来。

修改config.js.example文件为config.js，然后把Provider改为HttpProvider，连接到node0的节点即可。

```
// config.js
var web3 = require('web3');
var net = require('net');

var config = function () {
  
  this.logFormat = "combined";
  // this.ipcPath = process.env["HOME"] + "/.local/share/io.parity.ethereum/jsonrpc.ipc";
  // this.provider = new web3.providers.IpcProvider(this.ipcPath, net);

  this.provider = new web3.providers.HttpProvider("http://127.0.0.1:8546")
  
  // ... 省略其余代码
}

module.exports = config;
```

执行`npm start`之后即可将以太坊浏览器运行起来。然后在浏览器中访问`http://localhost:3000`。

## 思考

PoA共识基于权威的共识机制，和基于raft协议的共识机制具体哪个更快？

Parity文档中没有找到和权限控制相关的模块，用它来做联盟链还有待确定。

Quorum是JP摩根开源的基于以太坊的联盟链，使用的raft算法，可以研究研究。

还不是很清楚，fabric已经是联盟链主流的情况下，选择以太坊做联盟链的好处有多大。

