

目录下所有文件的快照的一个提交记录，但仅提交与上一个版本不同的部分文件。

```bash
git commit					# 提交
```

分支，指向某个提交记录。使用分支就相当于是指基于这个提交和它的所有父提交进行新的工作。

```bash
git branch <newImage>		# 创建分支
git checkout <newImage> 	# 切换分支
git commit <newImage>		# 在新的分支上提交记录
```

如果是创建一个新的分支同时切换到新创建的分支，直接一行命令

```bash
git checkout -b <your-branch-name>
```

合并分支

```bash
git merge <branch-name>		# 合并分支（并行）
git rebase <branch-name>	# 另一种合并分支的方式（串行接在要合并的分支后面）
```

head通常情况下指向分支名，随着commit命令向前移动。

分离的head其实就是让其指向了某个具体的提交记录而不是分支名。

```bash
cat .git/HEAD 				# 查看HEAD的指向
git symbolic-ref HEAD		# 

git checkout <hash值>
```

查看提交记录的hash值，hash值通常很长``fed2da64c0efc5293610bdd892f82a58e8cbc5d8``，所以提出了相对引用\^向上移动一个提交记录，\~<nums>向上移动nums个提交记录

```bash
git log 					# 查看提交的记录 
git checkout master^		# 切换到master的父节点
git checkout HEAD^			# 切换到head所在节点的父节点
git checkout HEAD^2			# 切换到head所在的第二个父节点，merge操作产生的多个节点
git checkout HEAD~4			# 一次性后退4步
git checkout HEAD~^2~2		# 链式操作，移动到指定父节点
git branch bugWork HEAD~^2~	# 链式操作，在指定父节点处建立分支

```

强制修改分支位置

```bash
git branch -f master HEAD~3 # 将master分支强制指向HEAD的第3级父提交
```

撤销变更

```bash
git reset	HEAD~1			# 通过把分支记录回退几个提交记录来实现撤销改动
git revert HEAD				# 相当于再提交一个改动的版本，使得改动后的版本和reset回退的版本相同，用于推送到远程仓库
```







整理提交记录

```bash
git cherry-pick <节点hash值...>  # 将一些提交复制到当前所在的位置HEAD下
```

交互式 rebase 指的是使用带参数 `--interactive` 的 rebase 命令, 简写为 `-i`，Git 会打开一个 UI 界面并列出将要被复制到目标分支的备选提交记录。

```bash
git rebase -i HEAD~4 			# 从4级父节点处打开 
git commit --amend				# 修改
```



tag，建立一个指向一个提交记录的锚点

```bash
git tag v1 c1
```

`git describe` 的语法是：

```
git describe <ref>
```

`<ref>` 可以是任何能被 Git 识别成提交记录的引用，如果你没有指定的话，Git 会以你目前所检出的位置（`HEAD`）。

它输出的结果是这样的：

```bash
<tag>_<numCommits>_g<hash>
```

`tag` 表示的是离 `ref` 最近的标签， `numCommits` 是表示这个 `ref` 与 `tag` 相差有多少个提交记录， `hash` 表示的是你所给定的 `ref` 所表示的提交记录哈希值的前几位。

当 `ref` 提交记录上有某个标签时，则只输出标签名称



**远程仓库**

远程仓库并不复杂, 在如今的云计算盛行的世界很容易把远程仓库想象成一个富有魔力的东西, 但实际上它们只是

你的仓库在另个一台计算机上的拷贝。你可以通过因特网与这台计算机通信 —— 也就是增加或是获取提交记录

话虽如此, 远程仓库却有一系列强大的特性

- 首先也是最重要的的点, 远程仓库是一个强大的备份。本地仓库也有恢复文件到指定版本的能力, 但所有的信息

  都是保存在本地的。有了远程仓库以后，即使丢失了本地所有数据, 你仍可以通过远程仓库拿回你丢失的数

  据。

- 还有就是, 远程让代码社交化了! 既然你的项目被托管到别的地方了, 你的朋友可以更容易地为你的项目做贡献



远程分支反映了远程仓库(在你上次和它通信时)的**状态**。这会有助于你理解**本地的工作**与**公共工作**的差别 —— 这

是你与别人分享工作成果前至关重要的一步。

远程分支有一个特别的属性，在你检出时自动进入分离 HEAD 状态。Git 这么做是出于不能直接在这些分支上进行

操作的原因, 你必须在别的地方完成你的工作, （更新了远程分支之后）再用远程分享你的工作成果。

- `<remote name>/<branch name>`远程分支/本地分支，大多数是origin/
- 当添加新的提交时 `o/master` 也不会更新。这是因为 `o/master` 只有在远程仓库中相应的分支更新了以后才会更新。

```bash
git fetch	# 从远程仓库下载本地仓库中缺失的提交记录,更新远程分支指针至最后
```

`git fetch` 并不会改变你本地仓库的状态。它不会更新你的 `master` 分支，也不会修改你磁盘上的文件。

``git pull``等同于``git fetch + git merge``

``git pull --rebase``等同于``git fetch + git rebase``



远程追踪

```bash
local branch "master" set to track remote branch "o/master"
```

```
git checkout -b totallyNotMaster o/master
```

就可以创建一个名为 `totallyNotMaster` 的分支，它跟踪远程分支 `o/master`。

另一种设置远程追踪分支的方法就是使用：`git branch -u` 命令，执行：

```
git branch -u o/master foo
```

这样 `foo` 就会跟踪 `o/master` 了。如果当前就在 foo 分支上, 还可以省略 foo：

```
git branch -u o/master
```



``git push <remote> <place>``

切到本地仓库中的“master”分支，获取所有的提交，再到远程仓库“origin”中找到“master”分支，将远程仓库中没有的提交记录都添加上去，搞定之后告诉我。