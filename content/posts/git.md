---
date: "2018-05-14 10:04:02"
title: git常用操作
---

整理一下常用的git操作，不用再到处找了。

# git放弃本地修改，强制更新

```
git fetch --all
git reset --hard origin/master
```



# git修改远程仓库地址

```
git remote set-url origin url
```


# cherry-pick

当你通过一番挣扎终于搞定一个bug,顺手提交到 git 服务器,心里一阵暗爽. 这时发现你当前所在的分支是 master !!!

这个分支不是开发者用来提交代码的,可惜现在剁手也晚了.

1. 先切换到master

```
git checkout master

git log
```

2. 复制提交的commit id

3. 切换到dev, cherry-pick

```
git checkout dev

git cherry-pic ${commit_id}
```

# 常用开发流程

git checkout -b feature1

git commit之后，进行rebase

git pull --rebase

gca!

git rvm
