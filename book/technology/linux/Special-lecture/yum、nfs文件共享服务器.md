# yum、nfs 文件共享服务器

## yum 管理

yum 命令是在 Fedora 和 RedHat 以及 SUSE 中基于 rpm 的软件包管理器，它可以使系统管理人员交互和自动化地更细与管理 RPM 软件包，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装。

### yum 的工作原理

当我们执行 yum install nginx -y，yum 会先访问本地缓存，如果有直接安装，如果没有，则通过元数据找到该软件包，通过该软件内部数据库的提示，找到相应的依赖包，然后继续查找元数据中是否有这些依赖包，如果没有会提示依赖包没有镜像提供。如果 nginx 软件包和依赖包都找到了，就根据配置文件中的 url 去下载。

![yum、nfs文件共享服务器-yum图示](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/yum、nfs文件共享服务器-yum图示.jpg)

### 配置网络源

    # 安装163的yum源：
    wget -O /etc/yum.repos.d/CentOS7-Base-163.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo

    # 安装阿里云的yum源：
    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

    重新执行: yum makecache

搜狐没有现成的 yum 源文件下载，需要自己配置：

```shell

[sohu]
name=Centos-$releasever-sohu
baseurl=http://mirrors.sohu.com/centos/$releasever/os/$basearch
gpgcheck=1
gpgkey=http://mirrors.sohu.com/centos/$releasever/os/$basearch/RPM-GPG-KEY-CentOS-$releasever

```

### 配置本地源

```shell
mkdir /mnt/cdrom
mount /dev/cdrom /mnt/cdrom
vim /etc/yum.repos.d/centos-7-local.repo
[local]
name=centos-$releasever-local
baseurl=file:///mnt/cdrom
gpgcheck=1
gpgkey=file:///mnt/cdrom/RPM-GPG-KEY-CentOS-$releasever
```

### 自动配置仓库

```shell
# 安装yum的扩展包：
yum install yum-utils -y
# 自动配置国内epel仓库：
yum-config-manager --addrepo=https://mirrors.tuna.tsinghua.edu.cn/epel/7/x86_64/
```

### 禁用/启用仓库

```shell
# epel 是仓库的id [epel]
yum-config-manager --disable epel
yum-config-manager --enable epel
# 查看仓库状态：
yum repolist all
```

### yum 常用命令

    yum repolist {all|enabled|disabled}             列出所有/已启用/已禁用的yum源
    yum list {all|installed|avaliable}              列出所有/已安装/可安装的软件包
    yum info package                                显示某一个软件包的信息
    yum install package                             安装软件包
    yum reinstall package                           重新安装软件包
    yum remove|erase package                        卸载软件包
    yum provides files                              查询某个文件是哪个软件包生成的
    yum search file                                 查询某个文件是哪个软件包生成的

### 同步外网源

在企业实际应用场景中，仅仅靠光盘里面的 RPM 软件包是不能满足需要，我们可以把外网的 YUM 源中的所有软件包同步至本地，可以完善本地 YUM 源的软件包数量及完整性

### 安装 reposync 工具

```shell
yum install yum-utils createrepo -y
```

### 同步源

```shell
# 创建本地目录:
mkdir /data/{centos,epel}

# 同步yum源：
reposync -r base -r updates -p /data/centos/

# 生成元数据：
createrepo /data/centos

# 结合前面所学制作本地源，如果想让其他服务器使用该源，后面可以结合nginx发布。
```

## NFS 文件共享服务器

NFS 是 network file sytem 的缩写，他最大的特点就是可以通过网络，让不同的机器，不同的系统实现文件共享。NFS 客户端可以将 NFS 服务器共享的目录挂载在本地的文件系统中，访问目录就如同访问自己本地目录一样。

### NFS 工作原理

![yum、nfs文件共享服务器-NFS原理](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/yum、nfs文件共享服务器-NFS原理.jpg)

1. 首先 NFS 服务器端开启 rpcbind；
2. 然后服务端开启 NFS 服务，这时 NFS 的各项功能都需要向 RPC 服务注册，这时 rpc 会通知 portmap 模块将可用的端口分配给 statd，rquotad 等;
3. 然后 NFS 客户端 RPC 服务就会通过网络向 NFS 服务端的 RPC 服务的 111 端口发出 NFS 文件存取功能的询问请求。
4. NFS 服务端的 RPC 服务找到对应的已注册的 NFSdaemon 端口后，通知 NFS 客户端的 RPC 服务。
5. 此时 NFS 客户端就可获取到 nfs 服务端各个进程的正确端口，然后通过 客户端 rpc 就直接与 NFS 服务器的 rpc 进行存取数据了（rpc 知道了 nfs 的 具体端口，就可以实现远程调用，即传输）。

### NFS 安装和部署

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

### 配置服务端

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

### 配置客户端

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

### 解读 exports 文件

```shell
[root@localhost ~]# exportfs -v
/data/nfs 192.168.115.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

    其中：
    rw:可读可写
    ro:仅可读
    sync:是指数据同步写入内存和磁盘
    root_squash:如果客户端用root身份访问，则被压缩成nobody,权限也将受到限制。
    no_root_squash:也就是不压缩，客户端使用root身份登录，全有所有权限，很危险。
    all_squash:不管访问者是什么身份，包括root，全部压缩至匿名用户。
    no_all_squash:保留访问用户的身份uid以及gid,一般只能查看，不能修改，权限问题，但是可以强制保存。

### 报错处理

#### 卸载时报错

```shell
umount.nfs4: /mnt/nfs: device is busy
umount -l /mnt/nfs 强行解除挂载
或者使用
fuser -m /mnt/nfs 将会显示使用这个模块的pid
fuser -mk /mnt/nfs 将会直接kill那个pid
```
