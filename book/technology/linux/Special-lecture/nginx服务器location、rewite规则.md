# Location 基础知识

## 概念

我们可以通过配置 Location 指令块，来决定客户端发过来的请求 URI 如何处理。

## 语法

```shell
Syntax: location [ = | ~ | ~* | ^~ ] uri { ... } location @name { ... }
Default: —
Context: server, location
```

location 配置可以有两种配置方法，可以在 server 指令块和 location 指令块配置。
1、修饰符 + uri（资源路径）
2、@ + name

### 修饰符

```shell
= ：精确匹配（必须全部相等）
~ ：大小写敏感（正则表达式）
~*：忽略大小写（正则表达式），这里要注意忽略大小写的意思是请求的字符大小写都可以，但是不会进行大小转换，请求的大小写对应的文件必须存在。
^~ ：只需匹配uri部分
@ ：内部服务跳转
```

# Location 配置实例

## 精准匹配

=，精准匹配，一般是匹配某个具体文件。

```shell
location = /index.html {
    [ configuration ]
}
# 则匹配到`http://www.xiaobei.com/index.html`这种请求。
```

## 大小写敏感匹配

~，大小写敏感（正则表达式）。

```shell
location ~ /XIAOBEI/ {
    [ configuration ]
}
#请求示例
#http://www.xiaobei.com/XIAOBEI/ [成功]
#http://www.xiaobei.com/xiaobei/ [失败]
```

## 大小写不敏感匹配

~\*，大小写忽略（正则表达式）。

```shell
location ~* /xiaobei.html {
    [ configuration ]
}
# 则会忽略 uri 部分的大小写
#http://www.xiaobei.com/xiaobei.html [成功] 可以成功匹配，但是目录中要xiaobei.html文件
#http://www.xiaobei.com/XIAOBEI.html [成功] 可以成功匹配，但是目录中要XIAOBEI.html文件
```

## 指定后缀匹配

匹配以 gif、jpg、jpeg 结尾的文件

```shell
location ~* \.(gif|jpg|jpeg)$ {
    [ configuration ]
}
#http://www.xiaobei.com/img/xiaobei.jpg [成功]
```

## 忽略正则匹配

^~，只匹配以 uri 开头，匹配成功以后，会停止搜索后面的正则表达式匹配。

```shell
location ^~ /img/ {
    [ configuration ]
}
#以 /img/ 开头的请求，都会匹配上
#http://www.xiaobei.com/img/xiaobei.jpg [成功]
#http://www.xiaobei.com/img/xiaobei.png [成功]
```

如果配置了忽略正则匹配，那么所有请求 /img/ 下的图片会被上面的处理，因为 ^~指令匹配到了，则不检查正则表达式。对比这两个 location，可以设置不同目录，相同文件进行实验。

# Location 优先级

完整范例：

```shell
这里有一简短的localtion配置：
location / {
    echo "this is $request_uri";
}
location ~* \.(jpg|png) {
    echo "this is ~* \.(jpg|png)";
}
location ~ \.(jpg|png) {
    echo "this is ~ \.(jpg|png)";
}
location ^~ /img/xiaobei.jpg {
    echo "this is ^~ /img/xiaobei.jpg";
}
location = /img/xiaobei.jpg {
    echo "this is = /img/xiaobei.jpg";
}

```

如果客户端的请求是：`curl 192.168.115.129/img/xiaobei.jpg`
那么按照匹配规则顺序应该是这样的：

1. 取出 uri：/img/xiaobei.jpg
2. 去匹配 localtion 规则，查找有没有 = /img/xiaobei.jpg 的规则，有则停止匹配。

```shell
[root@localhost ~]# curl 192.168.115.129/img/xiaobei.jpg
this is = /img/xiaobei.jpg
```

3. 将 location = /img/xiaobei.jpg 规则注释，继续查找有没有 ^~ /img/的规则。

```shell
[root@www ~]# curl 192.168.115.129/img/xiaobei.jpg
this is ^~ /img/xiaobei.jpg
```

4. 将 location ^~ /img/注释，这是它会去查找有没有正则匹配规则。

```shell
location / {
echo "this is $request_uri";
}
location ~* \.(jpg|png)$ {
echo "this is ~* \.(jpg|png)";
}
location ~ \.(jpg|png)$ {
echo "this is ~ \.(jpg|png)";
}
#location ^~ /img/xiaobei.jpg {
# echo "this is ^~ /img/xiaobei.jpg";
#}
#location = /img/xiaobei.jpg {
# echo "this is = /img/xiaobei.jpg";
#}
其中，第二个和的第三个规则都是正则，这时会按照至上而下的顺序匹配。
[root@localhost ~]# curl 192.168.129.115/img/xiaobei.jpg
this is ~* \.(jpg|png)
```

5. 其他的都注释后，因为优先匹配规则都没有找到，最后匹配到/img/规则

```shell
[root@localhost ~]# curl 192.168.115.129/img/xiaobei.jpg
this is /img/xiaobei.jpg
```

# rewrite 规则

Nginx 的 rewrite 功能需要 pcre 软件的支持，即通过 perl 兼容正则表达式语句进行规则匹配的。

默认参数编译 nginx 就会支持 rewrite 的模块，但是也必须要 pcre 的支持。

rewrite 是实现 URL 重写的关键指令，根据 regex（正则表达式）部分内容，重定向到 replacement，结尾是 flag 标记。

## rewrite 语法

```shell
rewrite <regex> <replacement> [flag];
            正则   替代内容    flag标记
正则：perl兼容正则表达式语句进行规则匹配
替代内容：将正则匹配的内容替换成replacement
flag标记：rewrite支持的flag标记

# flag标记说明：
last #本条规则匹配完成终止当前location的规则，继续向下匹配新的location URI规则
break #本条规则匹配完成即终止，不再匹配后面的任何规则
redirect #返回302临时重定向，浏览器地址会显示跳转后的URL地址，关闭服务，无法重定向。
permanent #返回301永久重定向，浏览器地址栏会显示跳转后的URL地址，关闭服务，依然可以重定向，清除缓存失效。
```

## rewrite 实例

### 实现域名跳转

```shell
#方案一：
server {
    listen 80;
    server_name xiaobei.com;
    rewrite ^/(.*)$ http://www.xiaobei.com/$1 permanent;
}
server {
    listen 80;
    server_name www.xiaobei.com;
    location / {
        root /data/www/;
        index index.html index.htm;
    }
}

#方案二：
server {
    listen 80;
    server_name www.xiaobei.com xiaobei.com;
    if ( $host != 'www.xiaobei.com' ) {
        rewrite ^/(.*)$ http://www.xiaobei.com/$1 permanent;
    }
    location / {
        root /data/www/;
        index index.html index.htm;
    }
}
#ps：本地需要做hosts解析
#192.168.115.129 www.xiaobei.com xiaobei.com
```

### 实现不同终端跳转

```shell
server {
    listen 80;
    server_name xiaobei.com www.xiaobei.com;
    root /usr/share/nginx/html/test;
    if ( $http_user_agent ~* "iphone|android") {
        rewrite ^/(.*)$
        http://m.xiaobei.com/$1;
    }
    index index.html;
}

server {
    listen 80;
    server_name m.xiaobei.com;
    root /data/www/m;
    index index.html;
    location / {
        default_type text/html;
        return 200 "this is iphone|android html";
    }
}
```

### 实现浏览器的语言跳转

```shell
# 根据浏览器的语言跳转到指定url:
server {
    listen 80;
    server_name xiaobei.com www.xiaobei.com;
    root /usr/share/nginx/html/test;
    index index.html;
    location / {
        if ( $http_accept_language ~ "^zh-CN" ) {
            rewrite ^/(.*) /zh/$1 ;
        }
        if ( $http_accept_language ~ "^en" ) {
            rewrite ^/(.*) /en/$1 ;
        }
        root html;
        index index.html index.htm;
    }

    location ^~ /zh/ {
        root html/;
        index index.html;
    }

    location ^~ /en/ {
        root html/;
        index index.html;
    }
}

mkdir -p /usr/share/nginx/html/test/zh
mkdir -p /usr/share/nginx/html/test/en
echo "this is 中文 " >
/usr/share/nginx/html/test/zh/index.html
echo "this is English " >
/usr/share/nginx/html/test/en/index.html

```

### 实现错误页面返回首页

```shell
error_page 404 =200 /index.html;
```

### 实现错误页面返回腾讯公益页面

```shell
error_page 404 =200 /404.html;
vim /usr/local/nginx/html/404.html
```

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>公益404</title>
  </head>
  <body>
    <script
      type="text/javascript"
      src="//qzonestyle.gtimg.cn/qzone/hybrid/app/404/search_children.js" charset="utf-8"></script>
  </body>
</html>
```
