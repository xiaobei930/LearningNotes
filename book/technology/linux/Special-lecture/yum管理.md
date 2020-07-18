# yum 管理

yum 命令是在 Fedora 和 RedHat 以及 SUSE 中基于 rpm 的软件包管理器，它可以使系统管理人员交互和自动化地更细与管理 RPM 软件包，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装。

## yum 的工作原理

当我们执行 yum install nginx -y，yum 会先访问本地缓存，如果有直接安装，如果没有，则通过元数据找到该软件包，通过该软件内部数据库的提示，找到相应的依赖包，然后继续查找元数据中是否有这些依赖包，如果没有会提示依赖包没有镜像提供。如果 nginx 软件包和依赖包都找到了，就根据配置文件中的 url 去下载。

![yum、nfs文件共享服务器-yum图示](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/yum、nfs文件共享服务器-yum图示.jpg)

## 配置网络源

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

## 配置本地源

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

## 自动配置仓库

```shell
# 安装yum的扩展包：
yum install yum-utils -y
# 自动配置国内epel仓库：
yum-config-manager --addrepo=https://mirrors.tuna.tsinghua.edu.cn/epel/7/x86_64/
```

## 禁用/启用仓库

```shell
# epel 是仓库的id [epel]
yum-config-manager --disable epel
yum-config-manager --enable epel
# 查看仓库状态：
yum repolist all
```

## yum 常用命令

    yum repolist {all|enabled|disabled}             列出所有/已启用/已禁用的yum源
    yum list {all|installed|avaliable}              列出所有/已安装/可安装的软件包
    yum info package                                显示某一个软件包的信息
    yum install package                             安装软件包
    yum reinstall package                           重新安装软件包
    yum remove|erase package                        卸载软件包
    yum provides files                              查询某个文件是哪个软件包生成的
    yum search file                                 查询某个文件是哪个软件包生成的

## 同步外网源

在企业实际应用场景中，仅仅靠光盘里面的 RPM 软件包是不能满足需要，我们可以把外网的 YUM 源中的所有软件包同步至本地，可以完善本地 YUM 源的软件包数量及完整性

## 安装 reposync 工具

```shell
yum install yum-utils createrepo -y
```

## 同步源

```shell
# 创建本地目录:
mkdir /data/{centos,epel}

# 同步yum源：
reposync -r base -r updates -p /data/centos/

# 生成元数据：
createrepo /data/centos

# 结合前面所学制作本地源，如果想让其他服务器使用该源，后面可以结合nginx发布。
```
