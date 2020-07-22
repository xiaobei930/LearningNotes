# Nginx 虚拟主机

## 虚拟主机作用

虚拟主机提供了在同一台服务器上运行多个网站的功能。

## 虚拟主机的三种模式

- 基于多域名方式配置虚拟主机
- 基于多端口配置虚拟主机
- 基于多 ip 配置虚拟主机

**基于多域名的虚拟主机**是最常见的一种虚拟主机。

只需配置你的 DNS 服务器，将每个主机名映射到正确的 IP 地址，然后配置 Nginx 服务器，令其识别不同的主机名就可以了。

    网域名称系统（DNS，Domain Name System，将域名和IP地址相互映射的一个分布式数据库）是因特网的一项核心服务，它作为可以将域名和IP地址相互映射的一个分布式数据库，能够使人更方便地访问互联网，而不用去记住能够被机器直接读取的IP地址数串。
    参考：
    https://baike.baidu.com/item/%E5%9F%9F%E5%90%8D/86062?fr=aladdin

**基于多端口的虚拟主机**，可以使用同一个 ip，通过访问不同的端口来访问。

**基于多 IP 的虚拟主机**可以通过添加多个网卡或者在一块物理网卡上绑定多个 IP 地址来实现

ps:如果没有特殊要求，**最好还是使用基于多域名的虚拟主机**。

### 多域名配置虚拟主机

#### 配置 hosts 文件

hosts 文件：在本机计算机上面，配置本地的 ip 地址和域名映射关系，通常用于测试。

- Windows 本地 hosts 文件

```shell
C:\Windows\System32\drivers\etc\hosts
192.168.115.129 www.xiaobei.com
```

- Linux 本地 hosts 文件

```shell
vim /etc/hosts
192.168.115.129 www.xiaobei.com
```

#### 创建虚拟主机配置文件

```shell
#在http指定块中添加：
include /usr/local/nginx/conf/vhost/*.conf;

#创建配置主机配置文件
mkdir -p /usr/local/nginx/conf/vhost

vim /usr/local/nginx/conf/vhost/www.xiaobei.com.conf

server {
    listen 80;
    server_name www.xiaobei.com;
    location / {
        root /usr/local/nginx/html/xiaobei;
        index index.html index.htm;
    }
}

vim /usr/local/nginx/conf/vhost/www.xb.com.conf

server {
    listen 80;
    server_name www.xb.com;
    location / {
        root /usr/local/nginx/html/xb;
        index index.html index.htm;
    }
}
```

### 多端口配置虚拟主机

```shell
server {
    listen 8080;
    server_name www.xiaobei.com;
    location / {
        root /usr/share/nginx/html/xiaobei;
        index index.html;
    }
}
```

### 多 IP 配置虚拟主机

```shell
cp /etc/sysconfig/network-scripts/ifcfg-ens32{,:1}

vim /etc/sysconfig/network-scripts/ifcfg-ens32:1

#修改以下信息:
NAME="ens32:1"
DEVICE="ens32:1"
IPADDR=192.168.115.129

# 重启服务：
systemctl restart network
server {
    listen 192.168.115.129:80;
    server_name www.xiaobei.com;
    location / {
        root /usr/share/nginx/html/xiaobei;
        index index.html;
    }
}

#重启nginx服务：
systemctl restart nginx
```

# Nginx 反向代理

## 反向代理概念

反向代理是 nginx 的一个重要功能，在编译安装时会默认编译该模块。在配置文件中主要配置 proxy_pass 指令。

代理服务器接受客户端的请求，然后把请求代理给后端真实服务器进行处理，然后再将服务器的响应结果返给客户端。

## 反向代理作用

与正向代理（**正向代理主要是代理客户端的请求**）相反，**反向代理主要是代理服务器返回的数据**，所以它的作用主要有以下两点：

1. 可以防止内部服务器被恶意攻击（内部服务器对客户端不可见）。
2. 为负载均衡和动静分离提供技术支持。

## 反向代理语法

```shell
Syntax: proxy_pass URL;
Default: —
Context: location, if in location, limit_except
```

代理服务器的**协议**，可支持 http 与 https。
**地址**可以指定为域名或 IP 地址，以及可选端口。

例如：

```shell
proxy_pass http://192.168.115.129;
proxy_pass http://192.168.115.129:8080;
proxy_pass http://localhost:9000/uri/;
```

## 实例一

**location 和 proxy_pass 都不带 uri 路径。**

代理服务器：192.168.115.129
后端服务器：192.168.115.132

```shell
#代理服务器的简单配置：
location / {
    proxy_pass http://192.168.115.129;
}
# proxy_pass 转发请求给后端服务器
```

```shell
#后端服务器的配置：
location / {
    echo $host;
    root html;
    index index.html index.htm;
}
# echo $host 这个主要是来看下后端接收到的Host是什么。
```

### 验证

```shell
[root@localhost ~]# curl 192.168.115.129
192.168.115.132
# 获取的请求Host是后端服务器ip，去掉该指令，验证请求结果。
[root@localhost ~]# curl 192.168.115.129
this is 132 page
# 可以看到我们访问的是129，但是得到的结果是132的发布目录文件。
```

## 实例二

**proxy_pass 没有设置 uri 路径**，但是**代理服务器的 location 有 uri**，那么代理服务器将把客户端请求的地址传递给后端服务器。

```shell
# 代理服务器的配置：
location /document/data/ {
    proxy_pass http://192.168.115.132;
}

# 后端服务器的配置：
location / {
    # echo $host;
    root html/uri;
    index index.html index.htm;
}
```

### 验证

```shell
[root@localhost ~]# mkdir -p /usr/local/nginx/html/uri/document/data/
[root@localhost ~]# echo "this is /usr/local/nginx/html/uri/document/data/ test" > /usr/local/nginx/html/uri/document/data/index.html
[root@localhost ~]# curl 192.168.115.129/document/data/
this is /usr/local/nginx/html/uri/document/data/ test
# 完整请求路径 是在后端服务器的/usr/local/nginx/html/uri 后追加客户端请求的路径 /document/data/
```

## 实例三

如果**proxy_pass 设置了 uri 路径**，则需要注意，此时，proxy_pass 指令所指定的 uri 会覆盖 location 的 uri。

```shell
# 代理服务器的配置：
location / {
    proxy_pass http://192.168.0.114/data/;
}

# 后端服务器的配置：
location / {
    root html;
    index index.html index.htm;
}
```

### 验证

```shell
[root@localhost ~]# mkdir -p /usr/local/nginx/html/data/
[root@localhost ~]# echo "this is /usr/local/nginx/html/data test。" > /usr/local/nginx/html/data/index.html
[root@localhost ~]# curl 192.168.0.109
this is /usr/local/nginx/html/data test。

#这样看好像很正常，但是稍作修改。
```

这次加上 location 的 uri，后端服务器加个子目录：

```shell
# 代理服务器的配置：
location /document/ {
    proxy_pass http://192.168.0.114/data/;
}

# 后端服务器的配置：
location / {
    #echo $host;
    root html;
    index index.html index.htm;
}
```

### 再次验证

```shell
[root@localhost ~]# curl 192.168.115.129/document/
this is /usr/local/nginx/html/data test。

# 该路径还是 proxy_pass 指定的uri路径，与location 没有关系了！
```

## 获取远程客户端真实 ip 地址

```shell
# 代理服务器配置：
location / {
    proxy_pass http://192.168.0.114;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For
    $proxy_add_x_forwarded_for;
}

# 后端服务器配置：
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent
    "$http_referer" '
    '"$http_user_agent"
    "$http_x_real_ip" "$http_x_forwarded_for"';
access_log logs/access.log main;
```

# 缓存代理服务器实战

在代理服务器的磁盘中保存请求目标的内容，加快响应速度，减少应用服务器（后端服务器）上的资源开销，比如多客户端请求相同的资源，代理缓存命中后，对于应用服务器来说，只发生了一次资源调度。

而浏览器上的缓存配置，一般来说是用来减少本地 IO 的，请求目标的内容会存放在浏览器本地。

```shell
# 代理服务器配置：
proxy_cache_path /data/nginx/cache max_size=10g
levels=1:2 keys_zone=nginx_cache:10m inactive=10m
use_temp_path=off;
upstream nginx {
    server 192.168.0.114;
}

server {
    listen 80;
    server_name localhost;
    location / {
        root html;
        index index.html index.htm;
        proxy_pass http://nginx;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_cache nginx_cache;
        proxy_cache_key $host$uri$is_args$args;
        proxy_cache_valid 200 304 302 1d;
    }
}

/data/nginx/cache #缓存资源存放路径
levels #设置缓存资源的递归级别，默认为levels=1:2，表示Nginx为将要缓存的资源生成的key从后依次设置两级保存。
key_zone #在共享内存中设置一块存储区域来存放缓存的key和metadata，这样nginx可以快速判断一个request是否命中或者未命中缓存，1m可以存储8000个key，10m可以存储80000个key
max_size #最大cache空间，如果不指定，会使用掉所有diskspace，当达到配额后，会删除不活跃的cache文件
inactive #未被访问文件在缓存中保留时间，本配置中如果60分钟未被访问则不论状态是否为expired，缓存控制程序会删掉文件。inactive默认是10分钟。需要注意的是，inactive和expired配置项的含义是不同的，expired只是缓存过期，但不会被删除，inactive是删除指定时间内未被访问的缓存文件
use_temp_path #如果为off，则nginx会将缓存文件直接写入指定的cache文件中，而不是使用temp_path存储，official建议为off，避免文件在不同文件系统中不必要的拷贝
proxy_cache #启用proxy cache，并指定key_zone。如果proxy_cache off表示关闭掉缓存。
```
