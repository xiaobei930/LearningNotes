# nfs 文件共享服务器

NFS 是 network file sytem 的缩写，他最大的特点就是可以通过网络，让不同的机器，不同的系统实现文件共享。NFS 客户端可以将 NFS 服务器共享的目录挂载在本地的文件系统中，访问目录就如同访问自己本地目录一样。

## NFS 工作原理

![yum、nfs文件共享服务器-NFS原理](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/yum、nfs文件共享服务器-NFS原理.jpg)

1. 首先 NFS 服务器端开启 rpcbind；
2. 然后服务端开启 NFS 服务，这时 NFS 的各项功能都需要向 RPC 服务注册，这时 rpc 会通知 portmap 模块将可用的端口分配给 statd，rquotad 等;
3. 然后 NFS 客户端 RPC 服务就会通过网络向 NFS 服务端的 RPC 服务的 111 端口发出 NFS 文件存取功能的询问请求。
4. NFS 服务端的 RPC 服务找到对应的已注册的 NFSdaemon 端口后，通知 NFS 客户端的 RPC 服务。
5. 此时 NFS 客户端就可获取到 nfs 服务端各个进程的正确端口，然后通过 客户端 rpc 就直接与 NFS 服务器的 rpc 进行存取数据了（rpc 知道了 nfs 的 具体端口，就可以实现远程调用，即传输）。

## NFS 安装和部署

服务器和客户端都关闭防火墙，装好 nfs 服务组件
nfs 服务端：192.168.115.130
nfs 客户端：192.168.115.131

```shell
# 关闭防火墙：
systemctl stop firewalld && systemctl disable firewalld
# 临时关闭selinux:
setenforce 0
# 永久关闭selinux:
sed -i 's/=enforcring/=disabled/' /etc/selinux/config
# 安装nfs服务组件：
yum -y install nfs-utils

```

## 配置服务端

- 编辑/etc/exports 文件

```shell
/data/nfs 192.168.115.0/24(rw,sync)
# 格式：
# /data/nfs 要共享的目录，需要存在
# 192.168.115.0/24 谁能挂载使用，可以是网段，也可以指定具体ip
# (rw,sync) 挂载的一些参数，rw表示挂载为可读可写，sync表示同步
```

- 导出（广播）编辑的文件，并重启 rpc 和 nfs 服务

```shell
systemctl restart rpcbind
systemctl restart nfs
exportfs -r
```

## 配置客户端

- 可用 showmount 搜索网络中可用的共享文件
  `showmount -e 192.168.115.130`
- 创建目录，用于挂载
  `mkdir /mnt/nfs`
- 挂载

```shell
mount -t nfs 192.168.115.131:/data/nfs /mnt/nfs
#推荐使用：
mount -t nfs -o soft,timeo=1 192.168.115.131:/data/xiaobei/mnt/nfs
# soft： 软挂载，遇到报错会终止挂载，并返回信息，默认是硬挂载，一直尝试挂载。
# timeo: 超时时间，如果不设置，一直链接，可以设置小点
```

挂载完成之后，进入目录，可能会发现无法对目录中的文件进行修改。这主要是因为客户端访问服务器时，身份被压缩成 nobody，相对服务器文件系统来说，就是其他用户。所以要想编辑，需要在服务端对文件授权或者更改 exports 文件，设置 no_root_squash（不压缩客户端 root 身份）。

## 解读 exports 文件

```shell
[root@localhost ~]# exportfs -v
/data/nfs 192.168.115.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

其中：
rw:可读可写
ro:仅可读
sync:是指数据同步写入内存和磁盘
root_squash:如果客户端用 root 身份访问，则被压缩成 nobody,权限也将受到限制。
no_root_squash:也就是不压缩，客户端使用 root 身份登录，全有所有权限，很危险。
all_squash:不管访问者是什么身份，包括 root，全部压缩至匿名用户。
no_all_squash:保留访问用户的身份 uid 以及 gid,一般只能查看，不能修改，权限问题，但是可以强制保存。

## 报错处理

### 卸载时报错

```shell
umount.nfs4: /mnt/nfs: device is busy
umount -l /mnt/nfs 强行解除挂载
或者使用
fuser -m /mnt/nfs 将会显示使用这个模块的pid
fuser -mk /mnt/nfs 将会直接kill那个pid
```

## nfs 自动挂载技术

autofs 服务程序与 mount 命令不同之处在于它是一种**守护进程**，只有检测到用户试图访问一个尚未挂载的文件系统时才自动的检测并挂载该文件系统。

autofs 非常方便，主要有两点：
1、设置开机不一定要挂载的目录，当用的时候才实现自动挂载。
2、用户不使用自动挂载的目录一段的时间，会自动卸载。（默认时间为 5 分钟）,可以在 autofs.conf

### 安装 autofs 服务

```shell
# 在客户端执行以下命令：
yum install autofs -y
# 配置autofs
rpm -qc autofs
```

### 编辑/etc/auto.master

```shell
# 添加以下行：
/media /etc/nfs.misc
#/media 挂载点
#/etc/nfs.misc是对总访问目录的描述，用于子目录的编辑，用户权限分离
```

### 编辑 nfs.misc

```shell
/data/nfs -fstype=nfs,rw,sync 192.168.115.129:/data/nfs
/data/nfs -fstype=nfs,ro,sync 192.168.115.129:/data/nfs
```

### 启动 nfs

`systemctl start autofs`

### 权限验证

```shell
## 登录不同目录验证权限：
[root@node nfs]# ls
[root@node nfs]#
[root@node nfs]#
[root@node nfs]# cd xiaobei
[root@node xiaobei]# df
```

