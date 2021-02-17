---
date: "2019-08-27 14:03:12"
title: crystal开发环境
---

突然搞明白了 crystal 的 vscode 插件的正确使用姿势，记录一下。

# 安装 crystal

```
brew install crystal
```

# 安装 vscode 插件

https://marketplace.visualstudio.com/items?itemName=faustinoaq.crystal-lang

# 安装 scry

scry 是 crystal 的 language server 的 client 工具，在本地安装 scry 就可以做到代码跳转了。

```
$ git clone https://github.com/crystal-lang-tools/scry.git

$ cd scry

$ shards build -v
Dependencies are satisfied
Building: scry
crystal build -o /Users/lucas/Documents/demos/crystal/scry/bin/scry src/scry.cr
```

/Users/lucas/Documents/demos/crystal/scry/bin/scry 就是编译出来的二进制的路径

# 配置插件

```json
  "crystal-lang.compiler": "crystal",
  "crystal-lang.server": "/Users/lucas/Documents/demos/crystal/scry/bin/scry",
  "crystal-lang.maxNumberOfProblems": 20,
  "crystal-lang.mainFile": "${workspaceRoot}/src/main.cr",
  "crystal-lang.processesLimit": 5,
  "crystal-lang.hover": true,
  "crystal-lang.problems": "build",
  "crystal-lang.implementations": true,
  "crystal-lang.completion": true,
  "crystal-lang.logLevel": "info",
```

把上面的配置加到 vscode 的 settings 文件中，就可以愉快的开发啦。
