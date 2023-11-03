---
title: Hexo搭建及维护
date: 2021.10.20
author: Aik
img: /source/images/xxx.jpg
top: true
hide: false
cover: true
coverImg: /images/1.jpg
password: 
toc: true
mathjax: true
summary: 记录了Hexo如何从0进行搭建以及后续的维护
categories: Hexo
tags:
  - Hexo
  - Git
  - node.js
---

# 基于Hexo的博客搭建及维护

本文将说明如何搭建一个自己的Hexo博客，以及后续如何对该博客进行维护。

---

## 基础搭建

### 准备工作

安装以下应用：
- [Git](http://git-scm.com/)
- [Node.js](https://nodejs.org/en/)

安装完成以后可以通过命令行的方式查看版本号

打开CMD

```
git --version   # 查看git版本号
node			# 查看node.js版本号
```

都能正确显示则表示可以进行下一步了。

### Hexo的安装

新建一个文件夹，后续所有操作都在此文件夹下进行。

安装淘宝的cnpm管理器，可能会提升后续的安装速度（可选），如安装失败则后续还是使用npm进行安装

```
npm -v
npm install -g cnpm --registry=http://registry.npm.taobao.org
cnpm -v
```

使用npm或者cnpm安装Hexo框架

```
npm install -g hexo-cli
hexo -v		# 查看hexo版本信息
```

初始化一个博客

```
hexo init	# 初始化博客
hexo s		# 启动本地blog服务
http://localhost:4000/ 		# 本地访问网页的地址，可以查看初始效果
```

### 通过把博客部署到Github以及Gitee上来进行访问
配置文件夹中的配置文件_config.yml：

  ```
  # Deployment
  ## Docs: https://hexo.io/docs/one-command-deployment
  deploy:
    type: 'git'
    repo:
      github: git@github.com:aik-n/aik-n.github.io.git
      gitee: git@gitee.com:aik-n/aik-n.github.io.git
    branch: master
  ```
  该配置文件因同时给定了两个地址，所以下面在部署时会同时部署到两边。
- **部署在Github上**

  在Github上创建一个新的仓库（以自己的为例）aik-n.github.io

  然后在文件夹中安装git部署插件

  ```
  npm install --save hexo-deployer-git
  ```

  之后通过hexo命令将博客部署到Github的仓库中去

  ```
  hexo clean		# 清理一下
  hexo g			# 生成
  hexo d			# 部署到远程Git仓库
  ```

  即可通过https://aik-n.github.io/来访问自己的博客

- 部署在Gitee上

  在Gitee上创建一个新的仓库，名字自己取（**建议用用户名，后面会说明原因**）
  
  然后在文件夹中安装git部署插件（上面安装过了这边就不用了）
  
  ```
  npm install --save hexo-deployer-git
  ```
  
  之后通过hexo命令将博客部署到Gitee的仓库中去
  
  ```
  hexo clean		# 清理一下
  hexo g			# 生成
  hexo d			# 部署到远程Git仓库
  ```
  
  即可通过https://aik-n.gitee.io/blog来访问自己的博客
  
  **注：**如果打开的页面无css样式，有以下解决方法：
  
  **方法一**：需要去配置文件_config.yml中修改参数
  
  ```
  # URL
  ## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
  url: https://gitee.com/aik-n/blog.git
  root: /blog
  ```
  
  但这样会出现一个问题，就是地址和路径只能填一个，导致如果同时在GitHub和Gitee上部署的话会有一边无法加载出Css样式，建议采用下面的方法。
  
  **方法二**：通过建立一个与自己个性仓库同名的仓库，如https://gitee.com/aik-n这个用户，创建站点并且不想以子目录的方式访问，那么就可以创建名为aik-n的仓库https://gitee.com/aik-n/aik-n，这样再部署完成后就可以直接通过https://aik-n.gitee.io进行访问。
  
  

---

## 后续维护

此时在Github上查看仓库中的文件可以发现和本地的文件并不相同，它只包含了网页的静态页面，像是主题以及文章源文件都是不在里面的。因此我们需要将源文件也上传到仓库里去，这样才能方便我们后续的维护工作。

### SSH配置

以本人电脑为例：C:/Users/xwy/.ssh目录下存放着公钥和密钥的文件信息。如果没配过的就把这文件夹下的全都删了，从头配起。

ssh文件夹右键，打开Git Bash窗口。

如果你是第一次使用，或者还没有配置过的话需要操作一下命令，自行替换相应字段。

```git
git config --global user.name "XXX"
git config --global user.email  "XXX@gmail.com"
```

- **GitHub**

  在终端中输入：

  ```
  ssh-keygen -t rsa -C "xxx@gmail.com"
  ```

  按三次回车，则会在当前目录下生成rsa开头的两个文件，打开rsa.pub文件，里面就是公钥，复制到Github里的SSH设置里去。

  验证一下是否连接：

  ```
  ssh -T git@github.com
  ```

  首次需要回复yes然后回车，如果成功则会有Hi XXX! You've successfully authenticated, but Github.com does not provide shell access.的内容返回。

- **Gitee**

  在终端中输入：

  ```
  ssh-keygen -t ed25519 -C "xxxxx@xxxxx.com"  # 这里邮箱填注册用到的邮箱
  ```

  按三次回车，则会在当前目录下生成ed25519开头的两个文件，打开id_ed25519.pub文件，里面就是公钥，复制到Gitee里的SSH设置里去。

  验证一下是否连接：

  ```
  ssh -T git@gitee.com
  ```

  首次需要回复yes再回车，如果成功则会有Hi XXX! You've successfully authenticated, but Gitee.com does not provide shell access.的内容返回

如果以上都配置好了但还是有错误提示的话，建议采用如下方法：

- 在ssh文件夹中建立一个**config**文件（没有后缀名），内容如下：

  ```
  # gitee
  Host gitee.com
  HostName gitee.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_ed25519
  
  # github
  Host github.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/github_id_rsa
  ```

  里面的文件地址自行修改，之后再进行上述的验证操作。

### 备份至仓库分支上
#### 直接备份
在Github/Gitee博客的仓库中创建一个分支**source**,并将其设置为默认分支

在博客的文件目录下打开Git Bash

查看一下本地分支：

```
git branch -a
```

其中绿色前面一个*的代表当前所处的分支，红色代表关联的远程分支，白色的代表存在的其他本地分支。

之后创建一个本地分支source，对应仓库里存放备份文件的分支：

```
git branch source
```

如果创建了多个分支，可以通过下面来进行切换：

```
git checkout 想要切换的分支名
```

在创建并切换到source分支后，开始将本地分支推到Git上：

```
git init		# 建立本地git仓库
git remote add origin https://xxx@xx.git # （将本地的仓库关联到GitHub（码云）上对应的仓库，后面的https改成仓库地址
git add .
git commit -m"提交信息"
git push origin source 		#代表把本地source分支的内容推到Github上仓库里的source下，创建新分支后的第一次需要加上-f强推
```

注意，其中git remote add origin 中的origin可以自己修改的，代表远程地址的别名，例如我想将本地仓库同时关联GitHub和Gitee，那么我可以执行下面两行：

```
git remote add origin https://gitee.com/aik-n/aik-n.github.io.git
git remote add origin1 https://github.com/aik-n/aik-n.github.io.git
```

一个origin，一个origin1。这样在push的时候，我们就可以选择往哪个上面推。

我们可以使用Git remote命令来查看相关信息：

```
git remote -v			# 可以查看远程仓库列表，有关联则会在下面都列出来
git remote show 仓库别名		# 可以查看该仓库的详细信息
git remote add 仓库别名 仓库地址		# 关联远程仓库
git remote rm 仓库别名		# 删除仓库
git remote rename 老别名 新别名		# 修改仓库名
```
#### 使用Hexo插件备份

利用Hexo框架自带的hexo-git-backup插件进行快捷备份。

安装插件：

```
npm install hexo-git-backup --save
```

配置博客目录的_config.yml文件

```
backup:
  type: git     
  message: hexo blog source   
  repository:  
    github: git@github.com:aik-n/aik-n.github.io.git,source #这里改成你自己的，分支“source”也改成自己的，如果Github不存在会自动新建
    gitee: git@gitee.com:aik-n/aik-n.github.io.git,source #这里改成你自己的，f“source”也改成自己的，如果Gitee不存在会自动新建
```

执行备份：

```
hexo backup
```


---
## 更换新的电脑后

在更换设备之后，我们只需要重新配置一下环境，然后把备份的项目从GitHub上拉下来就行了。

1. 下载[Git](http://git-scm.com/)和[Node.js](https://nodejs.org/en/)并安装

3. `git clone 仓库地址`
   
   或者直接把项目包下载下来解压
   
3. 在项目文件夹中安装Hexo框架以及后续部署使用的git部署插件，**切记不要用~~hexo init~~把项目初始化了**。

   ```
   npm install
   npm install -g hexo-cli			# Hexo框架安装（这一步可能不需要）
   npm install --save hexo-deployer-git		# git部署插件
   ```

4. 之后由于配置文件都是直接从云拉下来的，所以不用再进行更改了，直接hexo三联进行部署，git push进行后续维护，在进行备份时，记得配置好下面这两项：

   ```git
   git config --global user.name "XXX"
   git config --global user.email  "XXX@gmail.com"
   ```
   
   可能也还需要配置SSH

## 附录

Hexo的源文件说明：

 1. `_config.yml`站点的配置文件，需要拷贝；

2. `themes/`主题文件夹，需要拷贝；
3. `source`博客文章的.md文件，需要拷贝；
4. `scaffolds/`文章的模板，需要拷贝；
5. `package.json`安装包的名称，需要拷贝；
6. `.gitignore`限定在push时哪些文件可以忽略，需要拷贝；

7. `.git/`主题和站点都有，标志这是一个git项目，不需要拷贝；

8. `node_modules/`是安装包的目录，在执行npm install的时候会重新生成，不需要拷贝；

9. `public`是hexo g生成的静态网页，不需要拷贝；

10. `.deploy_git`同上，hexo g也会生成，不需要拷贝；

11. `db.json`文件，不需要拷贝。