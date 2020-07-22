# ftp 文件服务器

vsftpd 是“very secure FTP daemon”的缩写，是一个完全免费的、开放源代码的 ftp 服务器软件。特点是：非常高的安全性需求、带宽限制、良好的可伸缩性等。

## 工作原理

vsftpd 使用 ftp 协议，该协议属于应用层协议。它是典型的 c/s 架构，ftp 服务端用来存储文件，ftp 客户端可以通过 ftp 协议连接服务端实现上传和下载资源。

ftp 使用 tcp 的 21 端口进行**命令传输**，然后用 tcp 的 20 端口进行**数据传输**（主动模式）。默认是**被动模式**。

## 安装部署

```shell
[root@localhost ~]# yum install vsftpd ftp lftp -y

#vsftpd: 为服务端软件
#ftp、lftp： 为客户端工具，推荐使用lftp
```

## 启动服务

`systemctl start vsftpd`

## 匿名用户访问

用 ftp 客户端匿名登录需要输入**用户名**及**密码**验证，匿名用户名为：ftp 或者 anonymous，密码为空。

用 lftp 客户端匿名登录则不需要输入以上信息。

```shell
# ftp客户端连接：
[root@localhost ~]# ftp 192.168.115.129
ftp: connect: Connection refused
ftp>

# lftp客户端连接:
[root@localhost ~]# lftp 192.168.115.129
lftp 192.168.115.129:~>
```

这里我们重点讲 lftp 使用方法，ftp 客户端工具使用方法大致相同。

## 下载命令

### get 下载单个文件

可以先切换到本地指定目录（data）进行文件的下载，保存

```shell
lftp 192.168.115.132:~> lcd /data/
lcd ok, local cwd=/data
lftp 192.168.115.132:~> get xiaobei

# 下载到指定目录(默认使用源文件名，可以自定义名)：
lftp 192.168.115.132:/> get xiaobei -o /tmp
```

ps:当客户端已经连接上服务端，cd 是用于切换服务器中的目录命令，如果想切换客户端本地的目录则使用 lcd 命令

### mget 批量下载

```shell
lftp 192.168.115.132:/> mget *
```

**默认配置**只能进行文件的读取和下载，不能进行写入和上传文件：

```shell
lftp 192.168.115.132:/> put /etc/fstab
put: Access failed: 550 Permission denied. (fstab)
lftp 192.168.115.132:/> mkdir abc
mkdir: Access failed: 550 Permission denied. (abc)
```

可以看到上传命令和创建命令都失败了，没有相应的权限！

**开启匿名用户创建文件，重命名，删除，上传权限**

```shell
# 地址 /etc/vsftpd/vsftpd.conf
#开启上传权限
anon_upload_enable=YES

#开启创建文件权限
anon_mkdir_write_enable=YES

#开启重命名，删除权限
anon_other_write_enable=YES
anon_umask=022
```

重启服务，再次进入，发现还是没法创建目录，但是报错信息不一样:mkdir: Access failed: 550 Create directory operation failed.

这是因为目录没有写权限，给 pub 目录授权，**小总结**：要想匿名用户有写的权限，一是需要服务端配置文件开启写的权限，二是所在的目录本身有其他用户写的权限！

## 上传命令

### put 上传单个文件

    要想使用上传命令，需要开启上传权限和可写权限
    语法：put [OPTS] <lfile> [-o <rfile>]
    直接上传不改名，可以省去-o refile,如果不知道本地目录有哪些文件，可以使用!dir 查看，如下：
    lftp 192.168.115.132:/pub> !dir
    anaconda-ks.cfg favicon.png
    lftp 192.168.115.132:/pub> put /etc/fstab
    501 bytes transferred

### mput 批量上传

上传多个文件，可以使用 put 和 mput 命令上传，多个文件之间用空格分隔，如果想使用通配符，只有 mput 命令支持

```shell
lftp 192.168.115.132:/pub> put /etc/fstab /etc/favicon.png

lftp 192.168.115.132:/pub> mput /etc/f*
```

### 下载目录

```shell
# 下载远程服务器pub目录下的xiaobei 到本地的/root下
lftp 192.168.115.132:/> lcd
lcd ok, local cwd=/root
lftp 192.168.115.132:/> mirror -c xiaobei

# -c 支持断点续传
```

### 上传目录

```shell
lftp 192.168.115.132:/> mirror -cR /tmp

# 默认上传目录后，是不能查看目录内容的，需要添加mask权限
vim /etc/vsftpd/vsftpd.conf
anon_umask=022

```

### 同步远程目录

```shell
# 同步当前目录文件：
lftp 192.168.0.130:/pub> lcd
lcd 成功, 本地目录=/root
lftp 192.168.0.130:/pub> mirror -ce
-e 该参数会将远程目录pub下的文件同步到本地的root目录下，并且删除本
地多余的文件（远程目录没有的文件）
# 同步指定目录，如果是想同步整个目录，需要在本地端路径加上目录名：
lftp 192.168.0.130:/pub> mirror -ce xiaobei /root/xiaobei
#同步指定目录文件，如果是同步指定目录下的文件，不同步目录本身，就不需
要指目录了：
lftp 192.168.0.130:/pub> mirror -ce xiaobei /root
# xiaobei为远程服务器pub目录下的一个目录
# root为本地的目录
```

### 删除目录

```shell
# 删除目录：
lftp 192.168.0.103:/pub> rm -rf abc
# 删除空目录：
lftp 192.168.0.103:/pub> rmdir abc
rmdir ok, `abc' removed
```

### 删除文件

```shell
lftp 192.168.115.132:/pub> rm -rf xiao
```

## 本地用户访问

如果使用本地用户访问，需要服务端有对应的用户存在才行

```shell
# 服务端设置用户名及密码：
[root@localhost ~]# id xiaobei
uid=1000(xiaobei) gid=1000(xiaobei) 组=1000(xiaobei)

[root@localhost ~]# echo "xiaobei" |passwd --stdin xiaobei
#更改用户 xiaobei 的密码 。
#passwd：所有的身份验证令牌已经成功更新
```

修改配置文件，可以设置不让匿名用户登录 ，只能本地用户登录

```shell
vim /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
local_enable=YES
```

重启服务。然后再次访问

```shell
systemctl restart vsftpd
[root@www ~]# lftp 192.168.115.132
192.168.115.132:~> ls

#上面这个登录，表示匿名用户已经无法登录了。
[root@www ~]# lftp xiaobei:xiaobei@192.168.115.132
lftp xiaobei@192.168.115.132:~> ls
lftp xiaobei@192.168.115.132:~>
```

### 限制系统用户越狱

在安装 vsftpd 后不做配置的话，系统用户是可以向上切换到其他目录的。在配置文件中，添加如下选项

```shell
vim /etc/vsftpd/vsftpd.conf
chroot_local_user=YES
chroot_list_enable=NO
allow_writeable_chroot=YES
# chroot_local_user： 是否将所有用户限制在主目录,YES为启用，NO禁
用.(该项默认值是NO)
# chroot_list_enable： 是否启动限制用户（特例）的名单 YES为启用，
NO禁用(包括注释掉也为禁用)
```

- 如果想全部限制，所有用户都不能切换家目录，就可以像上面的配置

```shell
chroot_local_user=YES
chroot_list_enable=NO
allow_writeable_chroot=YES
```

- 如果想让部分用户有切换的家目录的权限，则需要开启限制

```shell
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
chroot_list中写上要放行用户
```
