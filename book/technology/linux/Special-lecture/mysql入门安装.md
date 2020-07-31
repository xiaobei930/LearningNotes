# Mysql 数据库简介

MySQL 是一种关系数据库管理系统，关系数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵
活性。MySQL 所使用的 SQL 语言是用于访问数据库的最常用标准化语言。

关系数据库管理系统(Relational Database Management System，RDBMS)，是将数据组织为相关的行和列的系统，而管理关系数据库的计算机软件就是关系数据库管理系统 。

数据库一般分为以下两种：

- 关系型数据库；
- 非关系型数据库。

> 常用的关系型数据库软件有: MYSQL、Mariadb、Oracle、SQL Server、PostgreSQL、DB2 等常用的非关系型数据库软件有: Redis、memcached。

MySQL 现阶段有免费版本和付费版本，公司里面使用 MySQL 多数都是使用免费版本。

Mariadb 数据库是 MySQL 数据库原来的团队，后续独立出来进行开发的完全开源版本。

## mysql 引擎

MySQL 引擎有很多，企业里面主流的 myisam、innodb 两种

**MyISAM**：主要强调的是性能，其执行数度比 InnoDB 类型更快，但不提供事务支持，不支持外键，如果执行大量的 SELECT(查询)操作，MyISAM 是更好的选择，支持表锁。**myisam 引擎查询性能高**。

**InnoDB**：提供事务支持事务、外键、行级锁等高级数据库功能，可执行大量的 INSERT 或 UPDATE，**InnoDB 引擎写性能高**。

# Mysql 数据库安装

MySQL 安装方式有两种，一种是 yum/rpm 安装，另外一种是 tar 源码安装。

## yum 安装

Yum 安装方法很简单，执行命令如下即可:

    Centos6: yum install –y mysql-server mysql-devel mysql
    Centos7:yum install –y mariadb mariadb-devel mariadbserver

```shell
# 安装完成之后可以查询一下软件是否安装成功：
[root@localhost ~]# rpm -qa|grep mariadb
mariadb-libs-5.5.60-1.el7_5.x86_64
mariadb-server-5.5.60-1.el7_5.x86_64
mariadb-5.5.60-1.el7_5.x86_64
mariadb-devel-5.5.60-1.el7_5.x86_64
Mariadb 和 mysql 数据库软件命令和配置都是类似的。
```

## 源码安装 5.5 版本

```shell
cd /usr/src
wget http://mirrors.163.com/mysql/Downloads/MySQL5.5/mysql-5.5.60.tar.gz
tar xf mysql-5.5.60.tar.gz
cd mysql-5.5.60/

yum install gcc ncurses-devel libaio bison gcc-c++ git cmake ncurses-devel ncurses -y
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql55/ \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DMYSQL_DATADIR=/data/mysql \
-DSYSCONFDIR=/usr/local/mysql55/ \
-DMYSQL_USER=mysql \
-DMYSQL_TCP_PORT=3306 \
-DWITH_XTRADB_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_EXTRA_CHARSETS=1 \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DEXTRA_CHARSETS=all \
-DWITH_BIG_TABLES=1 \
-DWITH_DEBUG=0
make && make install
cp support-files/my-large.cnf /usr/local/mysql55/my.cnf
cp support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
mkdir -p /data/mysql
useradd -s /sbin/nologin mysql
chown -R mysql. /data/mysql
/usr/local/mysql55/scripts/mysql_install_db --user=mysql
--datadir=/data/mysql --basedir=/usr/local/mysql55
/etc/init.d/mysqld start
加入service服务：
chkconfig --add mysqld
chkconfig --level 35 mysqld on
service mysqld restart
```

### 常见报错

```shell
客户端连接报错
ERROR 2002 (HY000): Cant connect to local MySQL server
through socket '/var/lib/mysql/mysql.sock' (2)
# 分析原因：
找不到套接字，怎么找？
[root@node3 ~]# ps -ef |grep mysqld
root 5165 1 0 21:22 pts/0 00:00:00 /bin/sh
/usr/local/mysql55/bin/mysqld_safe --datadir=/data/mysql --pid-file=/data/mysql/node3.pid
mysql 5536 5165 0 21:22 pts/0 00:00:01
/usr/local/mysql55/bin/mysqld --basedir=/usr/local/mysql55 --datadir=/data/mysql --plugindir=/usr/local/mysql55/lib/plugin --user=mysql --logerror=/var/log/mariadb/mariadb.log --pidfile=/data/mysql/node3.pid --socket=/tmp/mysql.sock --port=3306

#解决方案一：
[root@node3 ~]# mysql -S /tmp/mysql.sock

#解决方案二：
[root@node3 ~]# ln -s /tmp/mysql.sock
/var/lib/mysql/mysql.sock
```

## 源码安装 5.7 版本

```shell
# 安装依赖：
yum install gcc ncurses-devel libaio bison gcc-c++  git cmake  ncurses-devel openssl openssl-devel -y
​
wget http://nchc.dl.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz
tar xf boost_1_59_0.tar.gz
mv boost_1_59_0 /usr/local/boost
# Boost库是为C++语言标准库提供扩展的一些C++程序库的总称，由Boost社区组织开发、维护
​
wget http://mirrors.163.com/mysql/Downloads/MySQL-5.7/mysql-5.7.28.tar.gz
tar xf  mysql-5.7.28.tar.gz
cd mysql-5.7.28
​
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql57 \
-DMYSQL_UNIX_ADDR=/data/mysql57/mysql.sock \
-DMYSQL_DATADIR=/data/mysql57 \
-DSYSCONFDIR=/usr/local/mysql57 \
-DMYSQL_USER=mysql \
-DMYSQL_TCP_PORT=3307 \
-DWITH_XTRADB_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_EXTRA_CHARSETS=1 \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DEXTRA_CHARSETS=all \
-DWITH_BIG_TABLES=1 \
-DWITH_DEBUG=0 \
-DENABLE_DTRACE=0 \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/usr/local/boost
​
make && make install
​
mkdir -p /data/mysql57
useradd -s /sbin/nologin mysql
chown -R mysql. /data/mysql57
​
cp support-files/mysql.server /etc/init.d/mysqld57
chmod +x  /etc/init.d/mysqld57
​
vim /usr/local/mysql57/my.cnf
[mysqld]
basedir=/usr/local/mysql57/
datadir=/data/mysql57/
port=3307
pid-file=/data/mysql57/mysql.pid
socket=/data/mysql57/mysql.sock
​
[mysqld_safe]
log-error=/data/mysql57/mysql.log
​
#初始化
/usr/local/mysql57/bin/mysqld --initialize --user=mysql --datadir=/data/mysql57 \
--basedir=/usr/local/mysql57/
​
/etc/init.d/mysqld57 start
​
## 登录后修改密码：
> alter user user() identified by "123";
​
### 或者跳过权限修改：
跳过权限：
vim /usr/local/mysql57/my.cnf
在[mysqld]字段下添加：
skip-grant-tables
​
然后重启服务：
/etc/init.d/mysqld restart
​
免密进入数据库：
如果只启动了一个数据库服务，可以直接用下面的命令进入，要是有多个服务启动，会默认进入3306的数据库，到时可以指定IP和端口进入。
/usr/local/mysql57/bin/mysql
更新密码为空：
update mysql.user set authentication_string=password('') where user="root";
​
# MySQL5.7默认监听ipv6地址，用ipv4地址也是可以正常访问，如果想改为ipv4，可以在[mysqld]字段下配置一下参数，然后重启服务：
bind-address=0.0.0.0
```

# mysql 安装目录介绍

```shell
# 源码安装mysql 版本:
​
mysql 主配置目录:/usr/local/mysql55
mysql 数据目录：/data/mysql
mysql 命令目录：/usr/local/mysql55/bin/* 比如：mysql、mysqld等。
mysql 配置文件：/usr/local/mysql55/my.cnf
mysql 启动文件：/usr/local/mysql55/support-files/mysql.server 或者
是/etc/init.d/mysqld
mysql 日志文件：/data/mysql
​
​
​
# yum 安装mariadb程序：
​
mariadb 主配置目录:/var/lib/mysql
mariadb 数据目录：/var/lib/mysql
mariadb 命令目录：/usr/bin
mariadb 默认配置文件：/etc/my.cnf
mariadb 启动文件：/usr/bin
mariadb 日志文件：/var/log/mariadb
```

