# linux 介绍及安装

## 什么是云计算

云计算是分布式计算的一种，通过网络云将多个服务器组成一个超级计算系统，
以此来解决庞大的计算任务。

## 什么是云计算服务

云服务就是通过网络向用户提供一种按需付费的交易模式。
云服务一般可分为三个层面，分别是：
IaaS 基础设施即服务
PaaS 平台即服务
SaaS 软件即服务
![linux介绍及安装-1](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-1.png)

## 准备软件

- vmware 虚拟机
- centos7.x 镜像
- xshell 或者 crt
- 有道云笔记或者 typora

## Linux 操作系统简介

Linux 操作系统是基于 UNIX 以网络为核心的设计思想，是一个性能稳定的多用户操作系统，Linux 能运行各种工具软件、应用程序及网络协议，它支持安装在 32 位和 64 位 CPU 硬件上。

linux 即 linus's unix,在 1991 年的 10 月 5 日，还在读大学的 Linus Torvalds 写出了 linux 内核。

Linux 操作系统应用领域越来越广泛，尤其是近年来 Linux 在服务器领域飞速的发展，主要得益于 Linux 操作系统具备的如下优点：

1. 开源、免费；
2. 系统迭代更新；
3. 系统性能稳定；
4. 安全性高；
5. 内核小；
6. 应用领域广泛；
7. 使用及入门容易。

### Linux 操作系统发行版

Linux 操作系统主流发行版本包括：Red Hat Linux、CentOS、Ubuntu、SUSE Linux、Fedora Linux 等

#### Red Hat Linux

Red Hat Linux 1994 年创立，是最早的 Linux 发行版本之一，同时也是最著名的 Linux 版本，Red Hat Linux 已经创造了自己的品牌，也是读者经常听到的“红帽操作系统”。2018 年 10 月份 IBM 正式宣布以 340 亿美元收购红帽。

#### CentOS

社区企业版操作系统（Community Enterprise Operating System，CentOS）是 Linux 发行版之一，它是来自于 Red Hat Enterprise Linux 依照开放源代码所编译而成。由于出自同样的源代码，因此有些要求高度稳定性的服务器以 CentOS 替代商业版的 Red Hat Enterprise Linux 使用。

CentOS 于 Red Hat Linux 不同之处在于 CentOS 并不包含封闭的源代码软件，可以开源免费使用，得到运维人员、企业、程序员的青睐，CentOS 发行版操作系统是目前企业使用最多的系统之一

2014 年 7 月 7 日，正式发布 centos 7
2016 年 12 月 12 日，正式发布了 CentOS7.3。
2019 年 9 月 25 号，正式发布了 Centos8 的新版本。

#### Ubuntu

Ubuntu 是一个以桌面应用为主的 Linux 操作系统，其名称来自非洲南部祖鲁语或豪萨语的“ubuntu”一词（译为吾帮托或乌班图），意思是“人性”、“我的存在是因为大家的存在”，是非洲传统的一种价值观。Ubuntu 基于 Debian 发行版和 GNOME 桌面环境， Ubuntu 发行版操作系统的目标在于为一般用户提供一个最新的、同时稳定的以开放自由软件构建而成的操作系统，目前 Ubuntu 具有庞大的社区力量，用户可以方便地从社区获得帮助。

#### SUSE Linux

SUSE(发音 /ˈsuːsə/)，SUSE Linux 出自德国，SuSE Linux AG 公司发行维护的 Linux 发行版，是属于此公司的注册商标 2003 年 11 月 4 日，Novell 表示将会对 SUSE 提出收购。收购的工作于 2004 年 1 月完成。

#### Fedora Linux

Fedora 是一个知名的 Linux 发行版，是一款由全球社区爱好者构建的面向日常应用的快速、稳定、强大的操作系统。它允许任何人自由地使用、修改和重发布，无论现在还是将来。它由一个强大的社群开发，这个社群的成员以自己的不懈努力，提供并维护自由、开放源码的软件和开放的标准。

Fedora 约每六个月会发布新版本，美国当地时间 2015 年 11 月 3 日，北京时间 2015 年 11 月 4 日，Fedora Project 宣布 Fedora 23 正式对外发布，2017 年 6 月发布 Fedora 26 版本。

### Linux 内核命名规则

Linux 内核是 Linux 操作系统的核心，一个完整的 Linux 发行版包括进程管理、内存管理、文件系统、系统管理、网络操作等部分。

Linux 内核版本命名在不同的时期有其不同的命名规范，其中在 2.X 版本中，X 如果为奇数表示开发版、X 如果为偶数表示稳定版，从 2.6.X 以及 3.X，内核版本命名就没有严格的约定规范。

从 Linux 内核 1994 年发布 1.0 发布到目前主流 3.X 版本，5.X 属于开发调试阶段。

查看 Linux 操作系统内核：

```shell
uname -a
Linux 1550684538 3.10.0-693.el7.x86_64 #1 SMP Tue Aug 22
21:09:27 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

也可以去官网下载内核：www.kernel.org
其中 Mainline 表示主线开发版本，Stable 表示稳定版本，稳定版本主要由 mainline 测试通过而发布。Longterm 表示长期支持版，会持续更新及 Bug 修复，如果长期版本被标记为 EOL（End of Life），则表示不再提供更新。

## linux 系统安装

**在安装 CentOS 操作系统时，如果没有多余的计算机裸机设备**，可以基于 Windows 主机上安装 Vmware workstation 工具，该工具的用途可以在真实机上模拟一个新的计算机完整的资源设备，包括：CPU、内存、硬盘、网卡、DVD 光驱、USB 接口、声卡，进而可以安装 CentOS7.4 系统。

**如果有多余的计算机裸机设备或者企业服务器**，可以将 CentOS 系统直接安装在多余的设备上，安装之前需要下载 CentOS7.2 操作系统镜像文件(International Organization for Standardization,ISO 9660 标准)，通过刻录工具，将 ISO 镜像文件刻录至 DVD 光盘或者 U 盘里，通过 DVD 或者 U 盘启动然后安装系统。

### 系统安装及环境准备

VMware workstation 14.0
CentOS 7.4 x86_64

#### CentOS 7.4 镜像下载

在各大镜像站下载：

```shell
http://mirrors.163.com
https://opsx.alibaba.com
直接选择centos/7/isos/x86_64/,会指向最新的版本，如果要想下载以前的版本，可以到其他目录下下载readme，根据其中的地址，进行下载。
```

其他发行版，可以在这里选择：

```shell
http://vault.centos.org/
```

#### vm 安装 centos 系统

- Vmware 安装好后，执行运行，单击“创建新的虚拟机”
- 新建虚拟机向导，选择自定义（高级）（C）选项 ：
  ![linux介绍及安装-2](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-2.png)
- 直接下一步
- 安装客户机操作系统，选择“稍后安装操作系统
- 选择客户机操作系统(选择 centos7 64 位)
  ![linux介绍及安装-3](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-3.png)
- 自定义虚拟机名字，以及选择虚拟机存放位置
- 选择给虚拟机分配几个 cpu，这个要根据物理机的 cpu 设备情况来配，选择默认就可以了
- 虚拟机内存设置，默认为 1024MB ，如果物理机内存不够，则可以设置为 512M
- 网络选择桥接模式：
  ![linux介绍及安装-4](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-4.png)
- 磁盘大小分配 20G，选择将虚拟磁盘划分为多个文件，便于拷贝，迁移
  > ps:如果硬盘容量小于 2TB，系统默认会使用 MBR 模式来安装，若需强制使用 GPT 分区，可以在安装时，先选择 install centos 7 ，然后按 tab 键，在 quiet 后面空格输入：inst.gpt
- 点击完成
- 点击 CD/DVD，选择使用 ISO 镜像文件
  ![linux介绍及安装-5](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-5.png)
- 点击启动此虚拟机，即可开始安装
- 用上下键选择第一个，直接回车，开始安装
  ![linux介绍及安装-6](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-6.png)
  ![linux介绍及安装-7](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-7.png)
  ![linux介绍及安装-8](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-8.png)
  ![linux介绍及安装-9](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-9.png)
  修改软件包，附加依赖库，如果对 linux 不属性，可以带 GUI 安装
  ![linux介绍及安装-10](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-10.png)
  ![linux介绍及安装-11](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-11.png)
  配置分区：
  ![linux介绍及安装-12](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-12.png)
  ![linux介绍及安装-13](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-13.png)
  ![linux介绍及安装-14](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-14.png)
  ![linux介绍及安装-15](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-15.png)
  ![linux介绍及安装-16](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-16.png)
  ![linux介绍及安装-17](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/linux介绍及安装-17.png)
- 安装完，重启服务器之后，就可以通过远程工具连接

## xshell 连接及配置

### 三种连接 xshell 的方式

- 第一种：终端直接 ssh 命令连接

```shell
Xshell 6 (Build 0197)
Copyright (c) 2002 NetSarang Computer, Inc. All rights reserved.
Type `help' to learn how to use Xshell prompt.
[D:\~]\$ ssh 192.168.115.129

Connecting to 192.168.115.129:22...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.

WARNING! The remote SSH server rejected X11 forwarding request.
Last login: Tue Jun 30 16:05:52 2020 from 192.168.115.1
[root@xiaobei ~]#

```

- 第二种：在标签上方直接输入用户名及主机 ip 连接
  `ssh://root@ip:端口`
- 第三种：写入常用列表（推荐）

### 关闭 selinux/firewalld

```shell
# 临时关闭selinux
setenforce 0
# 永久关闭selinux
vi /etc/selinux/config
SELINUX=disabled
# 临时关闭防火墙：
systemctl stop firewalld
# 永久关闭防火墙：
systemctl disable firewalld
```

## 虚拟机网络连接方式

### 桥接模式

通过桥接模式网络连接，虚拟机中的虚拟网络适配器可连接到主机系统中的物理网络适配器，在桥接模式下，虚拟机 ip 地址需要与主机在同一个网段，如果需要联网，则网关与 DNS 需要与主机网卡一致。
**修改静态 IP 地址：**

```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens32
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
#把默认得dhcp修改为none或者static,表示设置静态IP地址
BOOTPROTO="none"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
NAME="ens32"
UUID="eca2f15c-eacc-48cf-9eea-9251567a4b18"
DEVICE="ens32"
#设置为yes
ONBOOT="yes"
# 静态ip地址得配置，参考物理主机得IP配置：
IPADDR=192.168.1.33
GATEWAY=192.168.1.254
NETMASK=255.255.255.0
DNS1=8.8.8.8
DNS2=114.114.114.114
```

### nat 模式

使用 NAT 模式网络时，主机系统上会建立单独的专用网络，虚拟机在外部网络中不必具有自己的 IP 地址。在默认配置中，虚拟机会在此专用网络中通过 DHCP 服务器获取地址。虚拟机和主机系统共享一个网络标识，此标识在外部网络中不可见。在 NAT 模式中，主机网卡直接与虚拟 NAT 设备相连，然后虚拟 NAT 设备与虚拟 DHCP 服务器一起连接在虚拟交换机 VMnet8
上。

### 仅主机模式

Host-Only 模式其实就是 NAT 模式去除了虚拟 NAT 设备，然后使用 VMwareNetwork Adapter VMnet1 虚拟网卡连接 VMnet1 虚拟交换机来与虚拟机通信的，Host-Only 模式将虚拟机与外网隔开，使得虚拟机成为一个独立的系统，只与主机相互通讯。
