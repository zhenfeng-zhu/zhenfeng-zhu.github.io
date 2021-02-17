---
date: "2018-05-05 15:32:59"
title: node踩坑
---

# module

首先第一个就是es6的module。

看到别人写的

```
import { a } from "./module";
```

所以自己也想要这么写，但是每次运行的时候都会报错。

```
// demo2.js
export const a = "hello";

//demo1.js
import { a } from "./demo2";

function hello() {
  console.log(a);
}
```

```
zhuzhenfengdeMacBook-Pro :: node/node-example » node demo1.js
/Users/zhuzhenfeng/Documents/github/node/node-example/demo1.js:1
(function (exports, require, module, __filename, __dirname) { import { a } from "./demo2";
                                                                     ^

SyntaxError: Unexpected token {
    at new Script (vm.js:74:7)
    at createScript (vm.js:246:10)
    at Object.runInThisContext (vm.js:298:10)
    at Module._compile (internal/modules/cjs/loader.js:646:28)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:689:10)
    at Module.load (internal/modules/cjs/loader.js:589:32)
    at tryModuleLoad (internal/modules/cjs/loader.js:528:12)
    at Function.Module._load (internal/modules/cjs/loader.js:520:3)
    at Function.Module.runMain (internal/modules/cjs/loader.js:719:10)
    at startup (internal/bootstrap/node.js:228:19)
zhuzhenfengdeMacBook-Pro :: node/node-example »
```

后来仔细查了资料之后，才发现，node现在还不能这么使用。

如下是解决的办法。

给每个js文件都以mjs命名。

```
// module.js
export const a = "hello";

// useModule.js
import { a } from "./module.mjs";

function hello() {
  console.log(a);
}

hello();
```

```
zhuzhenfengdeMacBook-Pro :: node/node-example » node --experimental-modules useModule.mjs
(node:15282) ExperimentalWarning: The ESM module loader is experimental.
hello
zhuzhenfengdeMacBook-Pro :: node/node-example »
```

这样才能够正常使用。

