# nginx 负载均衡

## 负载均衡目的

将前端超高并发访问转发至后端多台服务器进行处理，解决单个节点压力过大，造成 Web 服务响应过慢，严重的情况下导致服务瘫痪，无法正常提
供服务的问题。

## 工作原理

负载均衡分为四层负载均衡和七层负载均衡：

四层负载均衡是工作在七层协议的第四层-传输层，主要工作是转发。

它在接收到客户端的流量以后通过修改数据包的地址信息（目标地址和端口和源地址）将流量转发到应用服务器。

七层负载均衡是工作在七层协议的第七层-应用层，主要工作是代理。

它首先会与客户端建立一条完整的连接并将应用层的请求流量解析出来，再按照调度算法选择一个应用服务器，并与应用服务器建立另外一条连接
将请求发送过去。

![nginx负载均衡-七层协议](https://cdn.jsdelivr.net/gh/xiaobei930/picBed@master/learningNotes/nginx负载均衡-七层协议.jpg)

## 配置 7 层均衡

    前端服务器： 192.168.1.6
    后端服务器1：192.168.1.5
    后端服务器2：192.168.1.7
    这里后端服务器也可以通过配置虚拟主机实现

前端服务器主要配置 upstream 和 proxy_pass：

- upstream 主要是配置均衡池和调度方法。
- proxy_pass 主要是配置代理服务器 ip 或服务器组的名字。
- proxy_set_header 主要是配置转发给后端服务器的 Host 和前端客户端真实 ip。

### 配置前端 nginx

```shell
# 在http指令块下配置upstream指令块(/etc/nginx/conf.d/default.conf)
upstream web {
    server 192.168.1.5;
    server 192.168.1.7;
}
# 在location指令块配置proxy_pass
server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://web;
        proxy_next_upstream error http_404 http_502;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
# proxy_next_upstream error http_404 http_502;
# 通过这个指令，可以处理当后端服务返回404等报错时，直接将请求转发给其他服务器，而不是把报错信息返回客户端。

# proxy_set_header Host $host;
# 通过这个指令，把客户端请求的host，转发给后端。

# proxy_set_header X-Real-IP $remote_addr
# 通过这个指令，把客户端的IP转发给后端服务器，在后端服务器的日志格式中，添加$http_x_real_ip即可获取原始客户端的IP了。

```

### 配置后端 nginx

```shell
yum install nginx -y
echo "this is 1.5 page" > /usr/share/nginx/html/index.html
echo "this is 1.7 page" > /usr/share/nginx/html/index.htm
```

### 均衡方式

#### 轮询

```shell
upstream web {
    server 192.168.1.5;
    server 192.168.1.7;
}
```

访问前端 IP:

```shell
[root@192 ~]# while true;do curl 192.168.1.6;sleep 2;done
this is 1.5 page
this is 1.7 page
this is 1.5 page
this is 1.7 page
#可以看到后端服务器，非常平均的处理请求。
```

#### 轮询加权重

```shell
upstream web {
    server 192.168.1.5 weight=3;
    server 192.168.1.7 weight=1;
}
```

访问前端 IP:

```shell
[root@192 ~]# while true;do curl 192.168.1.6;sleep 1;done
this is 1.5 page
this is 1.5 page
this is 1.5 page
this is 1.7 page
this is 1.5 page
this is 1.5 page
this is 1.5 page
this is 1.7 page
#后端服务，根据权重比例处理请求，适用于服务器性能不均的环境。
```

#### 最大错误连接次数

错误的连接由 proxy_next_upstream， fastcgi_next_upstream 等指令决定，且默认情况下，后端某台服务器**出现故障**了，nginx 会自动将请求再次转发给其他正常的服务器（因为默认 proxy_next_upstream errortimeout）。所以即使我们没有配这个参数，nginx 也可以帮我们处理 error 和 timeout 的响应，但是没法处理 404 等报错。

为了看清楚本质，**可以先将 proxy_next_upstream 设置为 off**，也就是不将失败的请求转发给其他正常服务器，这样我们可以看到请求失败的结果。

```shell
upstream web {
    server 192.168.1.5 weight=1 max_fails=3
    fail_timeout=9s;
    #先将1.5这台nginx关了。
    server 192.168.1.7 weight=1;
}
server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://web;
        #proxy_next_upstream error http_404 http_502;
        proxy_next_upstream off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
# 在这里，我们将超时时间设置为9s，最多尝试3次，
这里要注意，尝试3次，依然遵循轮询的规则，并不是一个请求，连接3次，
而是轮询三次，有3次处理请求的机会。
```

访问前端 IP:

```shell
[root@192 ~]# while true;do curl -I 192.168.1.6
2>/dev/null|grep HTTP/1.1 ;sleep 3;done
HTTP/1.1 502 Bad Gateway
HTTP/1.1 200 OK
HTTP/1.1 502 Bad Gateway
HTTP/1.1 200 OK
HTTP/1.1 502 Bad Gateway
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 502 Bad Gateway
# 我们设置的超时时间为9s，我们是每3s请求一次。
我们可以看到后端一台服务器挂了后，请求没有直接转发给正常的服务器，而是直接返回了502。尝试三次后，等待9s，才开始再次尝试（最后一个
502）。
```

**把 proxy_next_upstream 开启**，再来访问看结果：

```shell
upstream web {
    server 192.168.1.5 weight=1 max_fails=3
    fail_timeout=9s;
    #先将1.5这台nginx关了。
    server 192.168.1.7 weight=1;
}
server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://web;
        proxy_next_upstream error http_404 http_502;
        #proxy_next_upstream off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

访问前端 IP：

```shell
[root@192 ~]# while true;do curl -I 192.168.1.6
2>/dev/null|grep HTTP/1.1 ;sleep 3;done
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
#可以看到现在没有502报错，请求都处理了。因为错误的响应码被
#proxy_next_upstream 获取，这次请求被转发给下一个正常的服务器了。
```

#### ip_hash

通过客户端 ip 进行 hash，再通过 hash 值选择后端 server 。

```shell
upstream web {
    ip_hash;
    server 192.168.1.5 weight=1 max_fails=3
    fail_timeout=9s;
    server 192.168.1.7 weight=1;
}
server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://web;
        proxy_next_upstream error http_404 http_502;
        #proxy_next_upstream off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

访问前端 IP：

```shell
[root@192 ~]# while true;do curl 192.168.1.6;sleep 2;done
this is 1.5 page
this is 1.5 page
this is 1.5 page
^C
[root@192 ~]# curl 192.168.1.7
this is 1.7 page
#可以看到1.7的服务是正常的，但是却不转发给1.7了，请求固定在了1.5的服务器上。
```

在使用负载均衡的时候会遇到会话保持的问题,常用的方法有:
a、ip_hash，根据客户端的 IP，将请求分配到不同的服务器上;
b、cookie，服务器给客户端下发一个 cookie，具有特定 cookie 的请求会分配给它的发布者。

#### url_hash

通过请求 url 进行 hash，再通过 hash 值选择后端 server

```shell
upstream nginx_web {
    hash $request_uri consistent;
    server 192.168.75.128;
    server 192.168.75.135;
}
```

#### 根据响应时间均衡

fair 算法会根据后端节点服务器的响应时间来分配请求，时间短的优先分配

```shell
# 下载模块：
wget https://github.com/gnosek/nginx-upstreamfair/archive/master.zip

# 解压：
unzip master.zip

# 修改源码bug：
sed -i 's/default_port/no_port/g'
ngx_http_upstream_fair_module.c

# 预编译：
./configure --prefix=/usr/local/nginx --addmodule=../echo-nginx-module --with-http_stub_status_module
--add-module=../nginx-upstream-fair-master

# 编译/安装：
make && make install

# 配置：
upstream web {
    fair;
    server 192.168.1.5 weight=1 max_fails=3
    fail_timeout=9s;
    server 192.168.1.7 weight=1;
}
```

#### 备用服务器

```shell
upstream web {
    server 192.168.1.5 weight=1 max_fails=3
    fail_timeout=9s;
    server 192.168.1.7 weight=1 backup;
}
# 1.7的服务器做备用服务器，只有在1.5得服务器不能提供服务时，才会自动顶上,否则，默认是不提供服务的。
```

## 配置四层均衡

    前端服务器：192.168.75.130
    后端服务器1：192.168.75.128
    后端服务器2：192.168.75.135

前端服务器主要配置 stream 和 upstream，注意该模块需要在预编译时指定，没有被默认编译进 nginx。

### 配置前端服务器

```shell
# 预编译：
./configure --prefix=/usr/local/nginx --addmodule=../echo-nginx-module --with-http_stub_status_module
--with-stream

# 编译/安装：
make && make install

# 在main全局配置stream：
events {
    worker_connections 1024;
}
stream {
    upstream web {
    # 必须要指定ip加port
        server 192.168.75.128:80;
        server 192.168.75.135:80;
    }
    server {
        listen 80;
        # 连接上游服务器超时间，超过则选择另外一个服务器
        proxy_connect_timeout 3s;
        # tcp连接闲置时间，超过则关闭
        proxy_timeout 10s;
        proxy_pass web;
    }
    log_format proxy '$remote_addr $remote_port $protocol $status [$time_iso8601] ' '"$upstream_addr" "$upstream_bytes_sent" "$upstream_connect_time"' ;
    access_log /usr/local/nginx/logs/proxy.log proxy;
}
```

### 配置后端测试页面

    echo "this is 128 page" > /usr/local/nginx/html/index.html
    echo "this is 135 page" > /usr/local/nginx/html/index.html

### 访问前端服务器

```shell
# 在后端135上访问130：
[root@node5 ~]# curl 192.168.75.130/index.html
this is 128 page

# 查看后端128日志：
[root@node2 ~]# tailf /usr/local/nginx/logs/access.log
192.168.75.130 - - [2020-07-31T11:07:56+08:00] "GET/index.html HTTP/1.1" 200 17 "-" "curl/7.29.0" "-"

# 查看前端130代理日志：
[root@node3 nginx-1.16.0]# tailf /usr/local/nginx/logs/proxy.log
192.168.75.135 57704 TCP 200 [2020-07-31T11:07:56+08:00]"192.168.75.128:80" "88" "0.001"
```

### 端口转发

```shell
# 前端130上配置如下：
stream {
    upstream web {
        server 192.168.75.128:22;
    }
    server {
        listen 2222;
        proxy_connect_timeout 3s;
        proxy_timeout 10s;
        proxy_pass web;
        #proxy_set_header X-Real-IP $remote_addr;
    }
    log_format proxy '$remote_addr $remote_port $protocol $status [$time_iso8601] ' '"$upstream_addr" "$upstream_bytes_sent" "$upstream_connect_time"' ;
    access_log /usr/local/nginx/logs/proxy.log proxy;
}
# 在另外一台服务器ssh连接：
[root@node5 ~]# ssh 192.168.75.130 -p 2222
root@192.168.75.130s password:
Last login: 01 08 2020 02:24:19 GMT from 192.168.75.130
[root@node2 ~]#
[root@node2 ~]#
# 可以看到，已经登录到了128服务器上。
[root@node2 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lovalid_lft forever preferred_lft forever
    inet6 ::1/128 scope host valid_lft forever preferred_lft forever
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdiscpfifo_fast state UP qlen 1000
    link/ether 00:50:56:2a:4e:7f brd ff:ff:ff:ff:ff:ff
    inet 192.168.75.128/24 brd 192.168.75.255 scope global ens32 valid_lft forever preferred_lft forever
    inet6 fe80::52eb:5720:d625:841/64 scope link valid_lft forever preferred_lft forever
```
