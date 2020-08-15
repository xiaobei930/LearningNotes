# SHELL 编程入门&案例实战

## Linux 技术学习的重点和方法

1 学习 Linux 技术，学习重点不是 Linux 系统简介、系统安装、命令操作、目录功能、配置 IP、用户权限等，学习重点是基于 Linux 平台掌握各个软件服务、应用程序：
NTP、DHCP、NFS、Vsftpd、Samba、Apache、Nginx、MYSQL、Mariadb、PHP、Tomcat、Rsync、Redis、Mycat、LVS、Keepalived、MQ、ZK、Haproxy、ES、ELK、Jenkins、SVN、GIT、Docker、K8S、Openstack；

2 作为运维人员，如何才能学好 Linux 软件服务、应用程序，有哪些学习技巧或者步骤方法呢？学习软件服务具体哪方面的内容呢？

 掌握软件服务的功能、应用的场景和场合、实现企业哪些需求；
 熟悉软件服务的工作原理、工作的流程，是如何对外服务的；
 独立安装、部署软件服务，并且日常管理、启动、停止、升级；
 掌握软件服务的目录功能、配置文件每个参数的含义；
 能够对软件服务、应用程序配置文件参数进行优化、调整；
 独立维护软件服务、应用程序日常的故障、第一时间解决故障。

## SHELL 编程概念剖析

### SHELL 是什么

 用户默认登陆 Linux 系统（Linux 内核+SHELL 程序）之后，是不能直接操作 Linux 内核（Linux 操作系统），跟 Linux 内核是不能直接联系的，需要借助 SHELL 程序来实现连接，SHELL 程序也被称为 SHELL 外壳，是附属在 Linux 内核的外层的。

 SHELL 是用户使用者和 Linux 内核之间的沟通桥梁，SHELL 可以接收用户输入的指令，SHELL 将接收到的指令传递给 Linux 内核，Linux 内核处理完成之后，会将处理的数据返回给 SHELL，SHELL 将数据处理最终返回给用户终端。

 SHELL 是外壳程序，SHELL 也被称为命令解释器，是附属在 Linux 内核外层，负责接收用户输入的 Linux 指令的，同时将指令传递给 Linux 内核的，SHELL 外壳程序也有很多的种类，默认 Linux 操作系统自带 SHELL 程序：BASH。
![SHELL编程入门&案例实战-shell-01](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/SHELL编程入门&案例实战-shell-01.jpg)

### SHELL 编程的概念?

 SHELL 编程主要是将用户操作的单个、多个 Linux 指令，按照一定的逻辑关系、顺序，堆积在文本文件（脚本文件）中，脚本文件中所有的 Linux 指令最终会以特定 SHELL 解释器（BASH）去执行，从而完成企业中某个具体的业务需求。
3 SHELL 编程意义？

 SHELL 编程主要是为了将手工操作的重复、繁琐的 Linux 指令、任务，变成流程化、简单化、自动化，以提高运维人员工作效率，减轻运维人员的工作量，加快企业自动化运维的脚步进程。

### SHELL 编程开发 Nginx WEB 一键部署脚本（思路剖析）

 从 Nginx 官网下载 Nginx 软件包稳定版本：nginx-1.16.0.tar.gz；
 通过 tar 工具解压软件包，tar -xzvf nginx-1.16.0.tar.gz；
 Cd 切换至解压源代码目录：cd nginx-1.16.0/；
 解决编译 Nginx 软件服务依赖环境、库文件、GCC 编译器；
 预编译，./configure --prefix=/usr/local/nginx/ --user=www --group=www --with-http_stub_status_module
 编译，make
 安装，make install
 启动，/usr/local/nginx/sbin/nginx
 查看 Nginx 进程，ps -ef|grep nginx
 查看 Nginx 监听端口，netstat -tnlp|grep 80
 关闭 selinux 和开启、关闭防火墙规则：setenforce 0；

### 4、SHELL 编程开发 Nginx WEB 一键部署脚本（v1 版本）

```shell
#!/bin/bash
#2020年8月15日21:42:51
#auto install nginx web
#by author xiaobei
########################
yum install -y wget gzip net-tools make gcc
yum install -y pcre pcre-devel zlib-devel
wget -c http://nginx.org/download/nginx-1.16.0.tar.gz
ls -l nginx-1.16.0.tar.gz
tar -xzvf nginx-1.16.0.tar.gz
cd nginx-1.16.0/
useradd -s /sbin/nologin www -M
./configure --prefix=/usr/local/nginx --user=www --group=www --with-http_stub_status_module
make
make install
ls -l /usr/local/nginx/
/usr/local/nginx/sbin/nginx
ps -ef|grep nginx
netstat -tnlp|grep 80
setenforce 0
systemctl stop firewalld.service
```

### SHELL 编程开发 Nginx WEB 一键部署脚本（v2 版本）

```shell
#!/bin/bash
#2020年8月15日21:42:51
#auto install nginx web
#by author xiaobei
########################
NGX_VER="1.15.0"
NGX_SRC="nginx-$NGX_VER"
NGX_YUM="yum install -y"
NGX_DIR="/data/app/nginx"
NGX_SOFT="nginx-${NGX_VER}.tar.gz"
NGX_URL="http://nginx.org/download"
NGX_ARGS="--user=www --group=www --with-http_stub_status_module --with-http_mail_module"
$NGX_YUM wget gzip net-tools make gcc
$NGX_YUM pcre pcre-devel zlib-devel
wget -c $NGX_URL/$NGX_SOFT
ls -l $NGX_SOFT
tar -xzvf $NGX_SOFT
cd $NGX_SRC/
useradd -s /sbin/nologin www -M
./configure --prefix=$NGX_DIR $NGX_ARGS
make
make install
ls -l $NGX_DIR
$NGX_DIR/sbin/nginx
ps -ef|grep nginx
netstat -tnlp|grep 80
setenforce 0
systemctl stop firewalld.service
```
