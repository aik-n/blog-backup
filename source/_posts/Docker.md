---
title: Docker
date: 2021.11.3
author: Aik
img: ../images/deepLearning.jpg
top: true
hide: false
cover: true
coverImg: /images/1.jpg
password: 
toc: true
mathjax: true
summary: Docker的常见命令以及如何在Docker中安装各种服务
categories: 开发
tags:
  - Docker
---

## Docker常用命令

查看所有镜像

```bash
docker images -a
```

查看所有容器

```bash
docker ps -a
```

启动容器

```bash
docker start redis_6379 #容器名，或者使用容器id	
```

重新启动容器

```bash
docker restart redis_6379
```

删除镜像

```bash
docker rmi #{镜像id，可通过images -a查看}	
```

删除容器

```bash
docker rm redis_6379 #容器名，或者使用容器id
```

Docker设置自启

```bash
systemctl enable docker
```

容器设置自启

```bash
docker update --restart=always #容器名，或者使用容器id
```

启动镜像

```bash
docker run --name #{name} -p 6379:6379 \
-v /xxxx/xxx.conf:/xxxx/xxx.conf \	
-itd redis redis-server /redis.conf	#指定配置文件加载
```

- **冒号前面表示宿主机，后面表示镜像**

- -v：将宿主机上的文件挂载到镜像中

- -d：后台运行

- -p：将镜像中的6379端口映射到宿主机上的6379端口

- --privileged=true：使container内的root拥有真正的root权限

- --name：给定容器名称

- -e：设置环境变量

- -it：创建交互式容器，以交互的方式启动。容器运行的命令如果是bash命令而不是那些一直挂起的命令（比如运行ping，sleep），会自动退出。




Docker内的容器使用ping

```
#进入容器内部
docker exec -it nginx_80 bash
#备份原文件
mv /etc/apt/sources.list /etc/apt/sources.list.bak
#添加源地址
echo "deb http://archive.debian.org/debian/ jessie main" >>/etc/apt/sources.list
echo "deb-src http://archive.debian.org/debian/ jessie main" >>/etc/apt/sources.list

apt-get update
apt-get install iputils-ping
```



## Docker配置服务

### Redis

#### 安装

拉取redis镜像

```bash
docker pull redis
```

查看当前存在的镜像

```bash
docker images -a
```

官网下载[配置文件][https://redis.io/docs/management/config]，本次选用6.2版本的

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231011113853383.png" alt="image-20231011113853383" style="zoom:50%;" />

修改redis.conf配置文件

```bash
bind 127.0.0.1 #注释掉这部分，这是限制redis只能本地访问

protected-mode no #默认yes，开启保护模式，限制为本地访问

daemonize no#默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，改为yes会使配置文件方式启动redis失败

logfile "/var/log/redis.log"

requirepass 123456 #密码
```

使用redis镜像创建容器，以redis.conf配置文件启动，-v 代表路径映射

```bash
docker run --name redis_6379 --privileged=true -p 6379:6379 \
-v /usr/local/redis/6379/conf/redis.conf:/usr/local/etc/redis/redis.conf \
-v /usr/local/redis/6379/data/:/data \
-v /usr/local/redis/6379/log/redis.log:/var/log/redis.log \
-d redis \
redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```

--appendonly yes表示开启redis持久化

**一定要给挂载的文件赋予权限**，不然启动的时候会提示 Fatal error, can't open config file '/usr/local/etc/redis/redis.conf': Permission denied.

```bash
chmod 777 /usr/local/redis/6379/conf/redis.conf	#该用户、所在组、其他用户能rwx(读写运行)
chmod 777 /usr/local/redis/6379/log/redis.log
```

测试redis，以"-it"参数创建交互式容器

```
docker exec -it redis_6379 bash
```

![image-20231004143333225](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231004143333225.png)

开放防火墙端口

```bash
firewall-cmd --zone=public --add-port=6379/tcp --permanent
```

重新加载防火墙

```bash
firewall-cmd --reload
```

查看开放的端口号

```bash
firewall-cmd --list-all
```



#### linux中卸载redis

windows10自带的WSL可以在本机上创建一个linux子系统，但还是建议通过docker来安装服务。

卸载通过apt方式安装的redis步骤：

1.停止redis服务

杀端口号

```bash
kill -9 #{端口号}
```

连接客户端进行关闭

```bash
redis-cli
auth #{密码认证，没有设置密码不需要这一步}
shutdown
```

2.移除Redis安装包

```bash
sudo apt-get remove redis-server
```

3.删除Redis的配置文件和数据文件

```bash
sudo rm /etc/redis/redis.conf
```

4.清除Redis的所有日志和配置文件

```bash
sudo apt-get purge redis-server
```

5.删除Redis的依赖软件包

```bash
sudo apt-get autoremove
```

### MySQL

拉取镜像

```bash
docker pull mysql
```

创建并启动容器

```bash
docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
```

防火墙设置

```bash
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

### Nginx

创建挂载目录

```
touch /usr/local/nginx/nginx.conf
mkdir /usr/local/nginx/logs
mkdir /usr/local/nginx/html
mkdir /usr/local/nginx/conf
```

下载nginx镜像

```bash
docker pull nginx
```

创建并启动容器

```
docker run  --name nginx_80 -p 80:80 -d nginx
```

进入容器内部

```
docker exec -it nginx_80 bash
```

nginx配置文件在该目录下

```
cd /etc/nginx
```



![image-20231007164744757](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231007164744757.png)

默认首页html文件在该目录下

```
cd /usr/share/nginx/html
```

![image-20231007164847807](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231007164847807.png)

日志文件在该目录下

```
cd /var/log/nginx
```

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231007164922423.png" alt="image-20231007164922423" style="zoom: 80%;" />

复制配置文件到指定挂载的目录下

```bash
docker cp nginx_80:/etc/nginx/nginx.conf /usr/local/nginx
docker cp nginx_80:/etc/nginx/conf.d /usr/local/nginx/conf
docker cp nginx_80:/usr/share/nginx/html/ /usr/local/nginx/html
docker cp nginx_80:/var/log/nginx/ /usr/local/nginx/logs
```

将需要代理的项目文件放在html目录下

挂载路径的方式创建并启动

```bash
docker run  --name nginx_80 -p 80:80 \
-v /usr/local/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v /usr/local/nginx/logs:/var/log/nginx \
-v /usr/local/nginx/html:/usr/share/nginx/html \
-v /usr/local/nginx/conf:/etc/nginx/conf.d \
-e TZ=Asia/Shanghai --net=host \
--privileged=true -d nginx
```

-e TZ=Asia/Shanghai 代表设置时区

**nginx.conf**配置文件

```

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/json;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

    upstream backend {
    	#192.168.1.102为本机的局域网IP地址
        server 192.168.1.102:8081 max_fails=5 fail_timeout=10s weight=1;	
        #server 127.0.0.1:8082 max_fails=5 fail_timeout=10s weight=1;
    }  
}
```

如果请求后端接口的时候报502网关错误：failed (111: Connection refused) while connecting to upstream

如果是docker下安装的nginx，可以试着把代理端口设置为本机的IP地址（局域网或WSL）

![image-20231008170803084](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231008170803084.png)

还不行就将wsl设置白名单

将nginx.conf配置文件中的代理IP设置为172.25.208.1重新创建容器，之后管理员命令执行下面的命令

```bash
New-NetFirewallRule -DisplayName "WSL" -Direction Inbound  -InterfaceAlias "vEthernet (WSL)"  -Action Allow
```

**default.conf**

```
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        # proxy_pass http://localhost:8081;
        # add_header Access-Control-Allow-Origin '*';
        # add_header Access-Control-Allow-Methods 'POST,PUT,GET,DELETE';
        # add_header Access-Control-Allow-Headers 'version, access-token, user-token, Accept, apiAuth, User-Agent, Keep-Alive, Origin, No-Cache, X-Requested-With, If-Modified-Since, Pragma, Last-Modified, Cache-Control, Expires, Content-Type, X-E4M-With';
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}
    location /api {  
            default_type  application/json;
            #internal;  
            keepalive_timeout   30s;  
            keepalive_requests  1000;  
            #支持keep-alive  
            proxy_http_version 1.1;  
            rewrite /api(/.*) $1 break;  
            proxy_pass_request_headers on;
            #more_clear_input_headers Accept-Encoding;  
            proxy_next_upstream error timeout;  
            # proxy_pass http://127.0.0.1:8081;
            proxy_pass http://backend;
        }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```



### RabbitMQ

下载镜像

```bash
docker pull rabbitmq:3-management	# -management是自带Web管理界面的，否则要手动安装插件
```

安装镜像文件

```bash
docker run \
-e RABBITMQ_DEFAULT_USER=xwy \
-e RABBITMQ_DEFAULT_PASS=123456 \
--name rabbit \
--hostname rabbit1 \
-p 15672:15672 \
-p 5672:5672 \
-d \
rabbitmq:3-management
```

--hostname 代表主机名，集群部署一定要配置

15672是管理界面的端口号，通过localhost:15672可以访问

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231019192751595.png" alt="image-20231019192751595" style="zoom: 67%;" />

