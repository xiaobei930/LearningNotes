# 常见服务器

    nginx、tengine、httpd、tomcat、IIS、lighttpd
    httpd|IIS 政府类，银行用的居多
    nginx|tomcat|tengine 社区，电商用的多
    nginx 新浪、
    httpd 中国人民政府网站、兴业银行...
    IIS 招商银行、工商银行、农业银行...
    tengine(2012) 简书、csdn、淘宝...

# nginx 发展

- 第一个公开版本 0.1.0 发布于 2004 年 10 月 4 日。
- Nginx 的 1.4.0 稳定版已经于 2013 年 4 月 24 日。
- Nginx 目前最新的版本是 1.18.0，于 2020 年 4 月 21 号发布。

# nginx 的安装方式

- YUM 安装部署
- 源码安装部署

## yum 安装

### 配置 yum 仓库

- 登录http://nginx.org官网，点击右侧的download：

![nginx服务安装及配置文件-nginx 1](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/nginx服务安装及配置文件-nginx1.jpg)

- 点击 当前页最下方的 stable and mainline：

![nginx服务安装及配置文件-nginx 2](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/nginx服务安装及配置文件-nginx2.jpg)

- 然后选择 RHEL/CentOS
- 复制配置文件，创建 nginx.repo 文件

```shell
vim /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$base
arch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$release
ver/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
```

- 现在就可以直接安装了，默认情况是安装稳定版的

### 安装稳定版本

```shell
[root@localhost ~]# yum install nginx -y
[root@localhost ~]# nginx -v
nginx version: nginx/1.16.0
```

### 安装主线版本

```shell
# 先安装yum的扩展包，开启nginx的主线版本仓库：
yum install yum-utils -y
yum-config-manager --enable nginx-mainline
yum install nginx -y
```

这个时候 nginx 1.16 的版本会直接被升级。

```shell
[root@localhost ~]# nginx -v
nginx version: nginx/1.17.1
```

## 源码安装

登录http://nginx.org官网，点击右侧的download：

![nginx服务安装及配置文件-nginx3](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/nginx服务安装及配置文件-nginx3.png)

- 主线版本：也叫开发版本，目前最新但是还没有经过大量测试的版本。
- 稳定版本：稳定版通常是经过大量测试的，相对比较稳定的版本，企业中我们也会使用稳定版。
- 历史版本：通常是往期的稳定版本

### 下载包

```shell
# 选择想要下载的版本，直接单击右键复制地址下载：
wget http://nginx.org/download/nginx-1.16.0.tar.gz
```

### 安装依赖

```shell
yum install gcc gcc-c++ pcre pcre-devel zlib zlib-devel
openssl openssl-devel -y
```

### 解压包

```shell
tar xf nginx-1.16.0.tar.gz
cd nginx-1.16.0
```

### 预编译

    预编译主要是用来检查系统环境是否满足安装软件包的条件，
    并生成Makefile文件，该文件为编译、安装、升级nginx指明了相应参数。
    ./configure --help 可以查看预编译参数
    --prefix 指定nginx编译安装的目录；
    --user=*** 指定nginx的属主
    --group=*** 指定nginx的属主与属组
    --with-*** 指定编译某模块
    --without-** 指定不编译某模块
    --add-module 编译第三方模块

开始预编译

```shell
./configure --prefix=/usr/local/nginx --user=nginx --group=nginx
```

    make clean : 重新预编译时，通常执行这条命令删除上次的编译文件
    make build : 编译，默认参数，可省略build参数
    make install : 安装
    make modules : 编译模块
    make upgrade : 在线升级

### 编译&安装

`make & make install`

### 查看版本

```shell
/usr/local/nginx/sbin/nginx -v
nginx version: nginx/1.16.0
```

### 启动 nginx

`/usr/local/nginx/sbin/nginx`

### 检查进程及端口

```shell
# 查看进程：
[root@localhost ~]# ps -ef|grep nginx

# 查看端口
[root@localhost ~]# netstat -ntlp|grep 80

```

可以看到端口及进程表示 nginx WEB 服务已经搭建成功！

# nginx 常用命令

    使用/usr/local/nginx/sbin/nginx -h 命令查看可用参数：
    [root@localhost ~]# /usr/local/nginx/sbin/nginx -h
    nginx version: nginx/1.16.0
    Usage: nginx [-?hvVtTq][-s signal] [-c filename][-pprefix] [-g directives]
    Options:
    -?,-h : this help
    -v : show version and exit
    -V : show version and configure options then exit
    -t : test configuration and exit
    -T : test configuration, dump it and exit
    -q : suppress non-error messages during configuration testing
    -s signal : send signal to a master process: stop,quit, reopen, reload
    -p prefix : set prefix path (default:/usr/local/nginx/)
    -c filename : set configuration file (default:conf/nginx.conf)
    -g directives : set global directives out of configuration file

## 命令解读

    -v          可查看nginx的版本。
    -V          可查看nginx的详细信息，包括编译的参数。
    -t          可用来测试nginx的配置文件的语法错误。
    -T          可用来测试nginx的配置文件语法错误，同时还可以通过重定向备份nginx的配置文件。
    -q          如果配置文件没有错误信息时，不会有任何提示，如果有错误，则提示错误信息，与-t配合使用。
    -s          发送信号给master处理：
    stop        立刻停止nginx服务，不管请求是否处理完
    quit        优雅的退出服务，处理完当前的请求退出
    reopen      重新打开日志文件，原日志文件要提前备份改名。
    reload      重载配置文件
    -p          设置nginx家目录路径，默认是编译时的安装路径
    -c          设置nginx的配置文件，默认是家目录下的配置文件
    -g          设置nginx的全局变量，这个变量会覆盖配置文件中的变量。

## 命令演示

如果觉得每次都需要输入绝对路径执行命令麻烦，可以通过以下几种方法实现直接使用 nginx 命令。

    1、做软连接：
    ln -s /usr/local/nginx/sbin/* /usr/local/sbin
    然后重新读取下配置文件
    . /etc/profile
    
    ps:软连接做在PATH路径是第一位，因为yum安装的在/usr/sbin/目录下，which安装PATH的顺序找到第一个，就不找了。
    
    2、配置环境变量：
    echo "export PATH=/usr/local/nginx/sbin:$PATH" > /etc/profile.d/nginx.sh
    然后重新读取下配置文件
    source /etc/profile
    
    ps:最好写在$PATH前面，否则，如果安装了yum版的nginx,直接执行nginx会启动yum版的nginx，因为which nginx，会先找到/usr/sbin/nginx文件
    
    3、设置别名：
    alias nginx='/usr/local/nginx/sbin/nginx'
    ps:which优先找别名

### 启动

`nginx`

### 立即停止 nginx

`nginx -s stop`

### 优雅停止 nginx

`nginx -s quit`

### 重新打开日志文件

该命令可用于日志切割，定期执行。

    [root@localhost logs]# ls
    access.log error.log nginx.pid
    [root@localhost logs]# mv access.log{,.bak}
    [root@localhost logs]# ls
    access.log.bak error.log nginx.pid
    [root@localhost logs]# /usr/local/nginx/sbin/nginx -s reopen
    [root@localhost logs]# ls
    access.log access.log.bak error.log nginx.pid

### 重载配置文件

修改工作进程数，重启服务，对比前后进程数

```shell
[root@localhost logs]# ps -ef|grep nginx
root 2685 1 0 23:56 ? 00:00:00 nginx:master process /usr/local/nginx/sbin/nginx
nginx 2686 2685 0 23:56 ? 00:00:00 nginx:worker process
root 2691 2532 0 23:57 pts/1 00:00:00 grep --color=auto nginx

[root@localhost logs]# vim
/usr/local/nginx/conf/nginx.conf
[root@localhost logs]# /usr/local/nginx/sbin/nginx -s reload
[root@localhost logs]# ps -ef|grep nginx
root 2685 1 0 23:56 ? 00:00:00 nginx:master process /usr/local/nginx/sbin/nginx
nginx 2694 2685 0 23:58 ? 00:00:00 nginx:worker process
nginx 2695 2685 0 23:58 ? 00:00:00 nginx:worker process
nginx 2696 2685 0 23:58 ? 00:00:00 nginx:worker process
root 2698 2532 0 23:58 pts/1 00:00:00 grep --color=auto nginx
```

### 启动指定的配置文件

在/data/目录下拷贝一份 nginx 的配置文件，然后修改用户名为 www。
注意：配置文件中引用的其他配置文件路径也要做一个修改

    [root@localhost logs]# cp /usr/local/nginx/conf/nginx.conf /data/
    [root@localhost logs]# vim /data/nginx.conf
    [root@localhost logs]# /usr/local/nginx/sbin/nginx -c /data/nginx.conf
    [root@localhost logs]# ps -ef|grep nginx
    root 2736 1 0 00:05 ? 00:00:00 nginx:master process /usr/local/nginx/sbin/nginx -c /data/nginx.conf
    www 2737 2736 0 00:05 ? 00:00:00 nginx:worker process
    www 2738 2736 0 00:05 ? 00:00:00 nginx:worker process
    www 2739 2736 0 00:05 ? 00:00:00 nginx:worker process
    root 2741 2532 0 00:05 pts/1 00:00:00 grep --color=auto nginx

### 设置全局变量

    通过设置全局变量，让 nginx 在前端运行。
    [root@localhost logs]# /usr/local/nginx/sbin/nginx -g "daemon off;"
    现在当前 nginx 在前端运行，
    输入 ctrl +c，则 nginx 就退出了。
    可以使用 ctrl +z 放置后台运行。

# nginx 配置文件路径

不同安装方式，nginx 的文件存放路径也有所不同。

## 源码安装配置文件路径

在安装目录下的 conf 目录下，比如我的安装目录是/usr/local/nginx，那么他的配置文件就在/usr/local/nginx/conf 目录下

## yum 安装配置文件路径

在/etc/nginx/目录（主配置文件）与/etc/nginx/conf.d 目录下。

# nginx 配置文件的结构

通常源码安装的 nginx 的配置文件，会是下面这种结构，yum 安装的有细微差异（大致是一样的，只是 server 是通过 include 引用的独立配置文件）。

```shell
...
events {
    ...
}
http {
    ...
    server {
    ....
        location / {
            root html;
            ...
        }
    }
}
```

nginx 的配置指令可以分为两大类：指令块与单个指令。

**指令块**就是像 events，http，server 等；

**单独指令**就是像 root html;这样的。

nginx 规定指令块可以嵌套，如 http 块中可以嵌套 server 指令，server 块中可以嵌套 location 指令，指令可以同时出现在不同的指令块，如 root 指令可以同时出现在 http、server 和 location 指令块，**需要注意的是在 location 中定义的指令会覆盖 server，http 的指令**。

