---
tags: [Git]
title: Git 配置文件
created: '2022-04-29T16:30:13.010Z'
modified: '2022-05-02T07:35:59.297Z'
---

# Git 配置文件

```
[color]
    status = auto
    diff = auto
    branch = auto
    ui = true
[alias]
    lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --
    lg1 = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    st = status -sb
    cm = commit
    df = difftool
    co = checkout
    br = branch
    lp = log --pretty=oneline
[core]
    autocrlf = input
    excludesfile = /Users/jiageng/.gitignore_global //全局忽略文件
    editor = vim
    quotepath = false
[diff]
    tool = vimdiff
[difftool]
    prompt = false
[pull]
    ff = only
```
