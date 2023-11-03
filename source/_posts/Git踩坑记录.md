---
title: Git踩坑记录
date: 2021.10.29
author: Aik
img: /source/images/xxx.jpg
top: true
hide: false
cover: true
coverImg: /images/1.jpg
password: 
toc: true
mathjax: true
summary: 记录了Git使用过程中遇到的一些问题
categories: Git
tags:
  - Git
---

# Git踩坑记录

### .gitgnore文件

.gitignore只能忽略掉那些原来没有被追踪（track）的文件，所以如果有一些文件提交到了git仓库当中，接受了git追踪,那么直接修改.gitignore是无效的。

比如一些配置文件，本地还要，直接删除仓库中的文件，也就删除了跟踪，提交上去后再配置gitignore就生效了

先执行git rm --cached public/ -r然后提交上去，后面这个文件的改动就会被忽略了,可能需要 -r

```
git rm --cached public/ -r
```
可以查看在你上次提交之后是否有对文件进行再次修改，此时能看到有delete记录
```
git status	
```

### git pull

直接用

```
git pull origin source
```

或者用下面的：

```
git fetch origin master:temp//从远程仓库获取新版本并创建一个temp分支
git diff temp //比较分支master和刚下载下来的temp分支的差异
git merge temp //比较过后，你觉得没有问题就可以将temp分支合并到master分支
git branch -d temp //你不想保留temp分支，就可以使用这个命令删除temp分支
```

