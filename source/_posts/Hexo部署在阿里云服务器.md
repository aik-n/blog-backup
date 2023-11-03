---

title: hexo部署在阿里云服务器
date: 2021.12.6
author: Aik
img: /source/images/xxx.jpg
top: false
hide: false
cover: false
coverImg: /images/1.jpg
password: 
toc: true
mathjax: true
summary: 将本地的hexo生成的静态网页部署到阿里云服务器上
categories: Hexo
tags:
  - Hexo
  - 云服务器
---

### 1.准备工作

1. 购买阿里云的ECS服务器

2. 注册一个域名
    其中包括域名购买前的实名认证。

  拥有一个域名则可以直接通过域名访问网站，而不是ip地址。

3. 域名需要进行备案

### 2.云服务器上的相关设置

**重置实例的密码**

![image-20211206193031052](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20211206193031052.png)

**远程连接Linux实例**

通过阿里云自带的VNC进行远程连接。点击远程连接，然后选择VNC远程连接下方的立即登录。如果是第一次登陆需要重置VNC密码，之后登录。

![image-20211206193922280](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20211206193922280.png)

在打开的界面中输入账号，默认为root，密码为刚刚重置的实例密码，回车后进入云服务器后台.

**配置安全组**

因为我们需要通过80端口访问nginx服务，而阿里云默认禁止80端口访问权限，所以要手动添加安全组，让阿里云给相应的端口和IP放行，这样我们才能通过公网+端口的方式访问ECS服务器。

![image-20211206194637063](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20211206194637063.png)

### 3.把Hexo博客部署到阿里云

在进行下面的配置过程中要注意是在本地配置还是服务器端，是root用户还是git用户。

#### 3.1 安装nginx

我们使用nginx作为Web服务器，所以需要先安装nginx服务。具体步骤如下：

使用**root用户**远程登录服务器

安装nginx依赖环境，安装期间又y/n提示一律选择y。

```
#yum install gcc-c++
#yum install -y pcre pcre-devel
#yum install -y zlib zlib-devel
#yum install -y openssl openssl-devel
```

下载nginx安装包（如果后面有出错可以换成[官网最新的稳定版本](http://nginx.org/en/download.html) )

```
#wget -c https://nginx.org/download/nginx-1.20.2.tar.gz
```

将安装包解压到/usr/local目录下

```
#tar -xvf nginx-1.20.2.tar.gz -C /usr/local
```

进入/usr/local目录，确认nginx解压到该目录下

```
#cd /usr/local
#ls
```

进入nginx-1.20.2目录，会发现该目录下有一个configure文件，执行该配置文件

```
#cd nginx-1.20.2/
#ls
#./configure
```

编译并安装nginx（如果这一步出错，那么就去下载最新的nginx包）

```
#make
#make install
```

查找nginx安装目录

```
#whereis nginx
```

进入安装目录

```
#cd /usr/local/nginx
#ls
```

由于nginx默认通过80端口访问，而Linux默认情况下不会开发该端口号，因此需要开放Linux的80端口供外部访问。

```
#/sbin/iptables -I INPUT -p tcp –-dport 80 -j ACCEPT
```

进入/usr/local/nginx/sbin目录，启动nginx

```
#cd sbin
#./nginx
```

没有任何消息，代表启动成功。此时，便可以通过“公网IP+端口”的方式访问 http://xx.xx.xxx.xxx:80/ 进入nginx欢迎页面了。 **注：可以使用./nginx -s stop命令停止服务**。

#### 3.2 配置nginx服务器路由

专门为hexo创建一个部署目录/home/www/hexo,用于存放hexo生成的静态页面。

```
#mkdir -p /home/www/hexo
```

进入/usr/local/nginx/conf目录，打开该文件夹下的nginx.conf配置文件。

```
#cd /usr/local/nginx/conf
#ls
#vim nginx.conf
```

进入后按insert键由命令模式切换到编辑模式。

- 将其中的部署根目录（root）修改为/home/www/hexo

- 将域名（server_name）修改为www.域名网址，**如果暂时没有域名就填阿里云实例的公网IP，以后有了再改回来**

- 查看监听端口（listen）的系统默认值是否为80（不用修改）。

完成以上修改后，先按Esc由编辑模式切换到命令模式，再输入:wq命令保存并退出编辑器。

![image-20211206201125452](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20211206201125452.png)

#### 3.3 安装node.js

退回根目录，安装node.js

```
#cd ~
#curl -sL https://rpm.nodesource.com/setup_10.x | bash -
#yum install -y nodejs
```

查看安装结果，打印版本号即为安装成功

```
#node -v
#npm -v
```

#### 3.4 安装Git

使用yum命令安装Git，安装期间有提示一律选yes

```
#yum install git
```

安装成功后，查看版本号

```
#git --version
```

#### 3.5 创建git用户

为了实现博客的自动部署，我们后面要使用公钥免密登录服务器。为了安全起见，最好不要使用root用户免密登录。因此，我们要创建一个新的git用户，用于远程公钥免密登录服务器。

创建git用户

```
#adduser git
```

修改git用户的权限

```
chmod 740 /etc/sudoers
```

打开文件

```
#vim /etc/sudoers
```

进入后按insert键由命令模式切换到编辑模式。找到 root ALL=(ALL) ALL，在下面添加一行 **git ALL=(ALL) ALL**。修改完成后，先按Esc由编辑模式切换到命令模式，再输入:wq命令保存并退出编辑器。

![image-20211206201552116](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20211206201552116.png)

保存退出后改回权限

```
#chmod 400 /etc/sudoers
```

设置git用户的密码

```
#sudo passwd git
```

设置密码：***\**\*\*\*\****，这样我们就可以使用git用户远程登录阿里云服务器了。

#### 3.6 给git用户配置SSH免密公钥登录

使用**git用户**免密公钥登录阿里云服务器的原理是：在本地计算机生成一个公钥文件和一个秘钥文件（类似于一个钥匙配一把锁)，然后使用FTP工具将公钥文件上传到阿里云服务器，并公钥安装到authorized_keys列表中去（即：将公钥文件的内容拷贝到authorized_keys文件中去）。这样本地计算机便可以通过ssh方式免密连接我们的阿里云服务器了。

在**服务器端**将登陆用户切换到git用户，然后在~目录(根目录)下创建.ssh文件夹，用来存放公钥。

```
#su git
$cd ~
$mkdir .ssh
```

在**本地计算机**桌面右键打开GitBash，在本地生成公钥/私钥对(如果之前GitHub或者Gitee配置过SSH那么直接用之前的公钥就行，不用重新生成)。

```
$cd ~
$cd .ssh
$ssh-keygen
```

接下来，碰见系统询问就直接按回车键。此时便会在本地计算机的用户根目录（C:\Users\xxx（用户名））下自动生成.ssh（隐藏）文件夹，并在其中创建两个文件，分别为：id_rsa（私钥）和id_rsa.pub（公钥）。

在**本地计算机**上给私钥设置权限

```
$ chmod 700 ~/.ssh
$chmod 600 ~/.ssh/id_rsa
```

下载并安装FTP工具[FileZilla](https://drive.google.com/file/d/185do_JQhmvuvjbmx0wvscEdYY3KCZHD0/view?usp=sharing)

打开FileZilla，使用**git用户**通过22端口远程连接到阿里云服务器，将客服端生成的公钥上传到服务器的~/.ssh目录下。

![image-20211206204938014](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20211206204938014.png)

上传完成后切回**服务器端**，继续以**git用户**的身份进入服务器~/.ssh目录，新建一个authorized_keys文件，并将id_rsa.pub文件中公钥的内容拷贝到该文件中。 **（注：该步骤既可以用命令行操作，也可使用FTP工具操作。）**

```\
$cd ~/.ssh
$cp id_rsa.pub authorized_keys
$cat id_rsa.pub >> ~/.ssh/authorized_keys
```

在服务器上设置文件权限

```
$chmod 600 ~/.ssh/authorized_keys
$chmod 700 ~/.ssh
```

确保设置了正确的SELinux上下文

```
$restorecon -Rv ~/.ssh
```

现在，当使用ssh远程登录服务器时，将不会提示您输入密码（除非在创建密钥对时输入了密码）。

接下来在本地计算机上使用ssh方式连接我们的云服务器

```
$ssh git@xxx.xxx.xxx.xxx（阿里云公网IP）
```

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20211206205227726.png" alt="image-20211206205227726" style="zoom: 67%;" />

#### 3.7 配置Git仓库

在服务器上使用git用户创建一个Git仓库，并且在该仓库中新建一个post-receive钩子文件

```
$cd ~
$git init --bare hexo.git
$vi ~/hexo.git/hooks/post-receive
```

进入后按insert键由命令模式切换到编辑模式。输入：

```
git --work-tree=/home/www/hexo --git-dir=/home/git/hexo.git checkout -f
```

作用：让钩子文件删除/home/www/hexo目录下原有的文件，然后从blog.git仓库 clone 新的博客静态文件到/home/www/hexo目录下。

完成以上修改后，先按Esc由编辑模式切换到命令模式，再输入:wq命令保存并退出编辑器。

授予钩子文件可执行权限

```
$chmod +x ~/hexo.git/hooks/post-receive
$cd ~
$sudo chmod -R 777 /home/www/hexo
```

重启ECS服务器实例，至此就完成了所有关于服务器端的配置。

### 4. 其他配置

配置本地博客文件目录下的config.yml文件

![image-20211206205725224](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20211206205725224.png)

这样在hexo d的时候就会直接把文件生成在服务器端的/home/www/hexo目录下。

在后续买好域名，并备案后，就可以修改配置文件并且正常通过域名进行访问了。



参考文档：https://mp.weixin.qq.com/s/JTTUYJTvtdT6X2fvLUBFZg

