---
title: "Github Action自动部署blog"
date: 2021-02-18T13:20:11+08:00
draft: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
disableHLJS: true
disableShare: true
disableHLJS: false
---

之前我采用的方式是两个github repo的方式： 

一个叫hugo-blog，用于存放blog的源文件

一个叫zhenfeng-zhu.github.io，用于存放生成之后的文件

然后通过写一个shell脚本，将生成之后的文件推向zhenfeng-zhu.github.io仓库中，同时将blog的源文件也做了一个backup。后来使用了一个github action的方式，
就不用在两个仓库中进行折腾，一切都由github action来做了。

# 方案

## 设置workflow

首先创建一个.github/workflows/gh-pages.yml
```yaml
name: github pages

on:
  push:
    branches:
      - main  # Set a branch to deploy

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

我们看下这个workflow的步骤：

1. name就是github pages

2. on的意思代表，是什么行为会触发这个action的构建。
   这里我们设置的是当push到main分支的时候进行构建。

3. jobs就是具体的工作，这里指定了几个步骤：
	 - 首先是在Ubuntu 18.04下进行构建
	 - 这些步骤都是用的actions/checkout@v2模板进行，拉取submodules。
	 - setup hugo：采用的是peaceiris/actions-hugo@v2模板
	 - build：简单的hugo命令
	 - deploy：采用peaceiris/actions-gh-pages@v3方式，把自己的github token也配置上。具体可以点进去看下这个步骤做了什么操作。


然后把博客的源文件，放在main分支里，当我们push之后，就会发现出现了一个gh-pages分支。

## 设置github pages
打开该repo的settings，选到GitHub Pages。

我们选择分支是gh-pages即可。


