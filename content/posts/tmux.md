---
date: "2019-08-28T12:03:46.801Z"
title: tmux
---

折腾一下 tmux

# 安装

```
brew install tmux
```

# 概念

- session：理解为一个会话，持久保存工作状态。
- window：可以理解为我们常说的 tab 页。
- pane：一个 window 被分成若干个 pane，理解为 iterm 的分屏。

# session

新建

```
tmux new -s your-session-name
```

断开

```
tmux detach
```

恢复

```
tmux attach-session -t your-session-name
或者
tmux a -t your-session-name
```

关闭

- kill-server
- kill-session
- kill-window
- kill-pane

```
tmux kill-session -t your-session-name

tmux kill-server
```

查看

```
tmux list-session
tmux ls
```

# tmux 的基础配置

`prefix` 是 tmux 的前缀键，默认是 ctrl+b 。只有按下前缀键，才会激活 tmux，然后再按其他的键进行 tmux 操作。这样可以避免与其他应用的快捷键进行冲突。

## 配置前缀

需要去tmux.conf中去配置

## 分屏

水平分屏：prefix+"，前缀键加引号
垂直分屏：prefix+%，前缀键加百分号

