---
layout: post
title:  "Nginx 基础入门"
date:   2014-09-09 19:50:00
categories: tool
tags: server
author: "Victor"
---

## 什么是 Nginx

高性能的HTTP和反向代理服务器软件，还是 IMAP/POP3/SMTP 代理服务器。

## 为什么选择 Nginx

1. 都采用模块化结构设计，支持通用的语言接口，比如PHP，Perl，Python。
2. 支持正向和反向代理，虚拟主机，URL重写，压缩传输，SSL加密传输
3. Nginx对比Apache来说，节省内存资源，处理速度快，对 fcgi 的处理方式更好，安装包更小只有几百K
4. 作为负载均衡服务器，内部直接支持Rails和PHP。
5. 50000个并发连接数的响应且占用很低的内存
6. 保持10000个没有活动的连接只占用2.5MB内存


## Nginx 的模块和工作原理

Nginx由内核和模块组成，内核的设计非常微小和简洁，完成的工作也很简单，仅仅通过查找配置文件将客户端请求映射到一个 location block（用于URL匹配），而这个 location 中所配置的每个指令将会启动不同的模块去完成相应的工作。

1. 结构上分为 核心模块、基础模块和第三方模块
2. HTTP模块、EVENT模块和MAIL模块属于核心模块
3. HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块、HTTP Rewrite模块属于基础模块
4. HTTP Upstream Request Hash模块、Notice模块、HTTP Access Key模块属于第三方模块

模块是自动编译进 Nginx 的，启动 Nginx 就可以直接使用这些模块，不需要像 Apache 在配置文件中指定是否加载。

工作方式：

1. 单工作进程 主进程和一个工作进程，工作进程是单线程的
2. 多工作进程 每个工作进程包含多线程。默认为单工作进程模式。

## Nginx 的安装和配置

配置文件是一个纯文本文件，一般位于 Nginx 安装目录的 conf 目录下，整个配置文件是以 block 的形式组织的。每个 block 一般以一个大括号{}来表示，block可以分为几个层次，整个配置文件中 main 指定位于最高层，在 main 层下面可以有 Events， HTTP 等层级，而在 HTTP 层中游包含有 server 层，即 server block, server block 中又可分 location 层，可以有多个 location block

```
| main
|
| |  Events
|
| | HTTP
| | | Server
| | | | location
| | | |
| | | | location
| | |
| | | Server
|
```

配置文件主要分为4部分：main(全局设置)、server(主机设置)、upstream(负载均衡服务器设置)、location(URL匹配特定位置的设置)

* main 设置的指令影响其他所有设置
* server 部分的指令用于指定主机和端口
* upstream 用于负载均衡，设置后端服务器
* location 用于匹配网页位置

### Nginx 的全局配置

```nginx
user nobody nobody;
worker_processes 4;
error_log logs/error.log notice;
pid logs/nginx.pid
worker_rlimit_nofile 65535;
events {
  use epoll;
  worker_connections 65536;
}
```

* user 指定 Nginx Worker 进程运行用户和用户组，默认由 nobody 账号运行。
* worker_processes 指定进程数，每个进程 10 - 12M 内存。一般一个够用，多核 CPU 可以指定和 CPU数量一样多。
* error_log 定义全局错误日志文件，级别包含 debug，info，notice，warn，error，crit。其中debug最多，crit最少。
* worker_rlimit_nofile 指定 nginx进程可以打开的文件数目，需要调整的话用 ‘ulimit-n 65535’。参考文章末尾附加补充连接。
* events 设定工作模式以及连接数上限
* use 指定工作模式。这里选线比较多，分为标准，高效等等。linux 系统 epoll 是首选。
* worker_connections 定义 nginx 最大连接数。默认是 1024. 受 linux 系统进程打开文件数的限制。

普通模式下最大连接数是 worker_processes * worker_connections。
反向代理模式下最大连接数是 worker_processes * worker_connections/4。


### HTTP 服务器配置

```nginx
http {
  include conf/mime.types;
  default_type application/octet-stream;
  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
            '$status $body_bytes_sent "$http_referer" '
            '"$http_user_agent" "$http_x_forwarded_for"';
  client_max_body_size 20m;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 60;
  client_header_timeout 10;
  client_body_timeout 10;
  send_timeout 10;
}
```

* include 实现对配置文件所包含文件的设定，减少主配置文件的复杂度。
* default_type 设定默认类型为二进制流，也就是说当 nginx 不能解析请求的时候默认就下载。
* log_format 是nginx的httolog模块指令，设定日志输出格式。
* client_max_body_size 设置允许客户端请求的最大单个文件字节数
* client_header_buffer_size 来自客户端请求头的 header_buffer 大小，一般来说 1kb 足够，如果有自定义的信息和cookies需要更大的空间，可以改这里。
* large_client_header_buffers 指定客户端请求中较大的消息头的缓存最大数量和大小，4是个数，32kb是大小。
* sendfile 是开启高效传输模式，配合下面两个指令防止网络阻塞。
* keepalive_timeout 保持客户端连接活动的超时时间。超过这个时间，服务器关闭连接。
* client_header_timeout 客户端请求头读取超时时间。超过这个时间返回408错误。
* client_body_timeout 客户端请求主体内容的读取时间，超时返回408错误。
* send_timeout 响应客户端的超时时间，限于两个连接活动之间的时间，超过这个时间客户端没有任何活动，nginx关闭连接。

### HTTPGzip 模块配置

通过```/opt/nginx/sbin/nginx -v``` 查看 nginx 的编译参数来看是否安装此模块。

```nginx
gzip on;
gzim_min_length 1k;
gzip_buffers 4 16k;
gzip_http_version 1.1;
gzip_comp_level 2;
gzip_types text/plain application/x-javascript text/css application/xml;
gzip_vary on;
```

* gzip 开启gzip压缩
* gzim_min_length 设置允许压缩的页面最小字节数，从 header 头的 Content-Length 中获取。默认是0，意思是不管页面多大都压缩。建议设置成大于 1K 的字节数，小于1K可能越压越大。
* gzip_buffers 申请4个单位 16K 的内存作为压缩结果流缓存，默认是申请于原始数据大小相同的内存空间进行压缩。
* gzip_http_version HTTP协议版本
* gzip_comp_level 压缩比，1最小速度最快，9压缩比最大，传输速度最快但处理最慢，也最消耗CPU。
* gzip_types 压缩类型
* gzip_vary 让前端的缓存服务器经过 gzip 的压缩页面，例如 Squid 缓存经过 Nginx 的压缩数据。

### 负载均衡配置

```nginx
upstream indba.net {
  ip_hash;
  server 192.168.12.133:1010;
  server 192.168.12.134:1010 down;
  server 192.168.12.134:1010 max_fails=3 fail_timeout=20s;
}
```

upstream 通过一个调度算法来实现客户端IP到后端服务器的负载均衡。indba.net 这个名称任意指定，后面用的地方直接调用即可。

Nginx 支持4中调度算法

1. 轮询，默认。每个请求按时间顺序逐一分配到不同的后端服务器，某台司机就自动分配给下一个。
2. Weight，指定轮询值，Weight越大，分配到的几率越高，用户后端服务器的性能不均的情况下。
3. ip_hash 同一个IP分配给固定的后端服务器，方便session共享
4. fair 根据页面大小和加载时间长短智能进行负载均衡。需要安装第3方模块。
5. url_hash 根据 url 定位到不同的后端服务器，方便进行缓存，需要安装第三方模块。

* down 表示不参与负载均衡
* backup 预留的备份机器，其他都忙的时候采用它
* max_fails 允许请求失败的次数，默认是1.超过最大次数时，返回 proxy_next_upstream 模块定义的错误
* fail_timeout 经历了 max_fails 次失败后，暂停服务器的时间。

当调度算法是 ip_hash 时，后端服务器的状态不能使 weight 和 backup

### server 虚拟主机配置

```nginx
server {
  listen 80;
  server_name 192.168.12.188 www.ixdba.net;
  index index.html index.htm index.jsp;
  root /web/wwwroot/www.ixdba.net;
  charset gb2312;
}
```

* server 标志虚拟主机的定义开始
* listen 用户指定虚拟主机的服务器端口
* server_name 用来指定 IP 地址或域名
* index 默认首页
* root 指定网页根目录，相对目录和绝对目录都行
* charset 用户设置网页的默认编码格式
* access_log 用来指定虚拟主机的访问日志路径

```access_log logs/www.ixdba.net.access.log main;```

### URL匹配配置

URL地址匹配是Nginx配置中最灵活的部分。location 支持正则表达式，也支持条件判断，可以通过 location 指令实现动、静态网页的过滤处理。

下面代码意思是扩展名为这些的静态文件都交给 Nginx 处理，而expires用来指定静态文件的过期时间是30天。

```nginx
location ~ .*\.(gif|jpg|jpeg|png|swf)$ {
  root /web/wwwroot/www.ixdba.net;
  expires 30d;

}
```

下面的意思是将这两个目录的所有文件都交给 Nginx 处理，这两个目录包含在 /web/wwwroot/www.ixdba.net

```nginx
location ~ ^/(upload|html) {
  root /web/wwwroot/www.ixdba.net;
  expires 30d;

}
```

把所有 jsp 文件交给本机的 8080 端口处理

```nginx
location ~ .*.jsp$ {
  index index.jsp;
  proxy_pass http://localhost:8080;

}
```

### StubStatus模块配置

用来获取 Nginx 自上次启动以来的工作状态，非核心模块需要编译安装时手工指定才能使用。

### 启动，关闭和重启

检查配置文件的正确性, -t 用于检测文件是否正确，-c 指定配置文件路径

```bash
/opt/nginx/sbin/nginx -t
/opt/nginx/sbin/nginx -t -c /opt/nginx/conf/nginx.conf
```

-v 显示版本信息 -V 显示编译信息

```bash
/opt/nginx/sbin/nginx -v
/opt/nginx/sbin/nginx -V
```

常用的信号

* QUIT 处理完请求后，关闭进程
* HUP 不中断用户请求，关闭原进程，开启新进程。平滑的重启Nginx
* USR1 日志切换，每天生成一个新的日志文件时，用这个命令。
* USR2 平滑的升级可执行程序
* WINCH 从容关闭工作进程


```bash
/opt/nginx/sbin/nginx
kill -XXX pid
kill -HUP `cat /opt/nginx/logs/nginx.pid`
```
## Nginx 常用配置实例


### 标准配置

```nginx
http {
  server {
    listen 80;
    server_name www.domain1.com;
    access_log logs/domain1.access.log main;

    location / {
      index index.html;
      root /web/www/domain1.com/htdocs;
    }
  }

  server {
    listen 80;
    server_name www.domain2.com;
    access_log logs/domain2.access.log main;

    location / {
      index index.html;
      root /web/www/domain2.com/htdocs;
    }
  }
  include /opt/nginx/conf/vhosts/www.domain3.com.conf;
}
```

```nginx
  server {
    listen 80;
    server_name www.domain3.com;
    access_log logs/domain3.access.log main;

    location / {
      index index.html;
      root /web/www/domain3.com/htdocs;
    }
  }
```

### 负载均衡的配置

下面是负载均衡的配置样例，先定义了一个负载均衡组 myserver，然后在 location 中通过 proxy_pass http://myserver 实现负载均衡的调度功能，proxy_pass 的意思是指定后端服务器的地址和端口，可以是主机名或者IP地址，也可以是 upstream 设定的均衡组的名称。

proxy_next_upstream 用来定义故障转移策略。当发生这些错误的时候自动使用另外一台服务器。

```nginx
http {
  upstream myserver {
    server 192.168.12.181:80 weight=3 max_fails=3 fail_timeout=20s;
    server backend1.example.com weight=5;
    server unix:/tmp/backend3;
    server backup1.example.com:8080   backup;
  }

  server {
    listen 80;
    server_name www.domain.com 192.168.12.189;
    index index.html index.htm;
    root /indba/web/wwwroot;

    location / {
      proxy_pass http://myserver;
      proxy_next_upstream http_500 http_502 http_503 error timeout invalid_header;
      include /opt/nginx/conf/proxy.conf;
    }
  }
}
```

nginx 的代理功能是通过 http proxy 模块实现的，这个模块默认已经安装。

```nginx
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_forwarded_for;
client_body_buffer_size 128k;
proxy_connect_timeout 90;
proxy_send_timeout 90;
proxy_read_timeout 90;
proxy_buffer_size 4k;
proxy_buffers 4 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
```

* proxy_set_header 设置后端服务器获取用户的主机名或真实IP，以及代理服务器的真实IP
* client_body_buffer_size 指定客户端请求主题缓冲区大小，可以理解为先保存到本地再传给用户
* proxy_connect_timeout 与后端服务器连接超时的时间，发起HTTP握手等候响应的超时时间
* proxy_send_timeout 表示后端服务器的数据回传时间，在规定时间内必须传完所有数据，否则 nginx 断开连接
* proxy_read_timeout nginx 从后端服务器获取信息的时间，表示连接建立成功后，nginx等待后端服务器的响应时间，也就是nginx进入后端的排队之中等候处理的时间
* proxy_buffer_size 缓冲区大小，默认大小等于 proxy_buffers
* proxy_buffers 缓存区大小和数量，nginx 从后端服务器获取的响应信息会放置到缓冲区
* proxy_busy_buffers_size 系统很忙时可以使用的 proxy_buffers 大小，官方推荐是 proxy_buffers * 2
* proxy_temp_file_write_size 缓存临时文件的大小


### 防盗链的配置

```nginx
location ~* \.(jpb|gif|png|swf|flv|wma|wmv|asf|mp3|zip|rar)$ {
  valid_referers none blocked *.ixdba1.net ixdba1.net;
  if ($valid_referers) {
    rewirte ^/http://www.ixdba.net/img/error.gif;
    #return 403;
  }
  location /images {
    root /opt/nginx/html;
    valid_referers none blocked *ixdba1.net ixdba1.net;
    if ($valid_referers) {
      return 403;
    }
  }
}
```

上面的意思是来自 ixdba1.net 的请求可以访问这些文件资源。 if{} 中的内容的意思是，如果地址不是上面的指定地址就跳转到 rewirte 指定的地址，也可以返回 403 错误。更复杂的处理可以使用 HttpAccessKeyModule 模块进行配置。


### 日志分割配置

把下面的脚本加入到 linux 的 crontab 守护进程，每天0点执行，可以实现日志的每天分割了。其中 savepath_log 指定分割后的日志存放路径， nglogs 指定日志存放路径，最后一行实现日志的自动切换功能。

```bash
#/bin/bash
savepath_log = '/home/nginx/logs'
nglogs = '/opt/nginx/logs'

mkdir -p $savepath_log/$(data + %Y)/$(data + %m)
mv $nglogs/access.log $savepath_log/$(data + %Y)/$(data + %m)/access.$(date+%Y%m%d).log
mv $nglogs/error.log $savepath_log/$(data + %Y)/$(data + %m)/error.$(date+%Y%m%d).log
kill -USR1 `cat /opt/nginx/logs/nginx.pid`
```

### CGI

公共网接口 Common Gateway Interface。在服务器上运行

### FastCGI

像是一个 long-live 型的CGI，可以运行在网站服务器以外的主机上并且接受来自其他网站服务器的请求。


## Nginx 性能优化技巧

涉及到重新编译，暂不介绍

## 相关知识点

* [LINUX ulimit 命令](http://www.cnblogs.com/wangkangluo1/archive/2012/06/06/2537677.html)
* [Module nginx http upstream module](http://nginx.org/en/docs/http/ngx_http_upstream_module.html)
