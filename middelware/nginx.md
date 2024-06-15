# Nginx基础

> https://zhuanlan.zhihu.com/p/650074319
>
>
> 响应码：
>
> 400（Bad Request）：错误的请求头或请求体
>
> 408（Request timed out）：建立连接后，开始接收客户端的请求头，在超时时间内没有读取到返回408
>
> 413（Request entity too large）：Content-Length大于max_client_body_size

## 安装部署

**开源Nginx地址**：https://nginx.org

### **部署过程：**

1. 在/user/local/目录下下载nginx源码包并解压
2. 执行configure，进行编译前配置
3. 在/user/local/nginx-1.xx.x/目录下执行make和make install
4. 在/usr/local/目录下产生nginx目录，拥有下列目录
   - /conf
     - nginx.conf：
   - /html
     - 50x.html：
     - index.html:
   - /logs
     - access.log
     - error.log
     - nginx.pid
   - /sbin
     - **nginx**

### **conf目录**

- fastcgi：配置与Fastcgi进程通信的参数。nginx可以通过fastcgi通信，进行动态内容的生成和处理。

- scgi：与fastcgi类似

> fastcgi（fast common gateway interface）与scgi相比
>
> 1. 效率：前者效率高，会多路复用，在一个持久连接上处理多个请求。scgi每个请求都会建立和关闭
> 2. 移植性：后者简单、移植性更高

- mime.types

### **常用命令**

- 启动

  /usr/local/nginx/sbin/nginx

- 重载配置文件

  nginx -s reload

- 重新打开

  nginx -s reopen

- 立刻关闭

  nginx -s stop

- 等待会话结束关闭

  nginx -s quit

## 配置文件

### nginx.conf

- **全局配置块**

  设置master运行的参数，如 worker_processes

- **events块**

  用于配置连接处理的参数，如worker_connections，超时时间

- **http块**

  - **http全局**

    include：将其他文件包括进来

    default_type：默认http的body为二进制流

    sendfile：开启高效传输文件

    resolver：指定DNS服务器地址

    upstream：定义一组backend，在proxy_pass 中指定，根据lb选择合适的服务器

  - **server块**

    server的概念对应虚拟主机，

    - **server全局**

      server_name：

      error_page：

      - **location块**

        配置请求路由和页面处理

        - root：相对目录
  
    - upstream
  
      负载均衡器设置

### location路径映射

```text
location [ = | ~ | ~* | !~ | !~* | ^~ | @ ] uri {...}
```

`=`：精确匹配。如果匹配成功，立即停止搜索并处理此请求。

`~`：执行[正则匹配](https://www.zhihu.com/search?q=正则匹配&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A"650074319"})，区分大小写。

`~*`：执行正则匹配，不区分大小写。

`!~`：正则匹配，区分大小写不匹配。

`!~*`：正则匹配，不区分大小写不匹配。

`^~`：前缀匹配。如果匹配成功，不再匹配其他`location`，且不查询正则表达式。

`@`：指定命名的`location`，主要用于内部重定向请求，如 `error_page` 和 `try_files`。

`uri`：待匹配的请求字符串。可以是普通字符串或包含正则表达式

## HTTP模块

ngx_http_core_module

### 虚拟主机与请求的分发

http模块中，每个server就是一个虚拟主机。虚拟主机可以是一个域名，多个域名可以对应于同一个IP（IP有限）。nginx可以通过不同的域名做出不同的响应（根据请求头中的host字段）。

处理http请求时，nginx会取出header中的Host，与每个server_name匹配，由此决定由哪个server来处理

```
server{
	#支持通配符
	server_name www.test.com *.test.com;
	
	# 后面也可以加不同的参数
	listen 8080 default_server ssl;
	listen *:8080;
	listen 12.12.12.12;
	
	#重定向,on表示使用server_name里配置的第一个主机名代替原先请求中的host头部
	server_name_in_redirect on;
	
	#匹配url中的地址
	location uri {
	
	}

}
```

### 文件路径定义

```
location /download/ {
	
	# 相对于http请求的根目录，如果有一个请求的uri是/download/picture.jpg，
	# 返回的资源路径是/path/download/picture.jpg
	root path;

	# 相当于修改用户的请求，如果uri是/download/pwd，返回的是usr/download/pwd
	# 等价与root usr
	alias usr/download
	
	# 首页
	# 可以搭配root使用，http模块依次尝试返回
	index index.html index.php;
	
	# 错误码与重定向
	error_page 404	404.html
	error_page 502 503 50x.html
	error_page 403 http://forbidden.html
	# 重定向到另一个location处理
	error_page 404 @fallback
}

location @fallback {
	proxy_pass http://backend
	
}
```

### 内存及磁盘资源的分配

```

# 为请求头的分配的内存大小
clinet_header_buffer_size 1k

clinet_body_buffer_size 8k

# 预先创建的内存池
connection_pool_size
```

### 网络连接设置

```

# 读取请求头超时时间

clinet_header_timeout 60

clinet_body_timeout 60

# 服务端发送了，但是客户端一直没有接收
send_timeout

# 连接超时后向客户端发送RST，而不是正常的四次挥手。(一般不会开启，会带来一些问题)
# 可以避免服务端过多的FIN_WAIT_1、FIN_WAIT_2、TIME_WAIT
reset_timeout_connection

# 设置关闭用户连接的方式
lingering_close on|off|always

# 关闭闲置的连接，cs两端的策略可能不同
keepalived_timeout 75s

# 一条连接最多的请求量
keepalived_request
```

### 对客户端请求的限制

```

# 对请求方法的限制
limit_except GET {
	allow 12.12.12.0/24;
	deny all;
}

# 对客户端请求体大小的限制
# 服务端读取到Centent—Length时与该值比较，如果超过就返回413
client_max_body_size 1G
```

### 文件操作的优化

```

# 直接从内核态发送到网卡
sendfile on;

# 打开文件缓存
open_file_cache max=N[inactive=M] | off
```



## 负载均衡

反向代理简单过程：

1. 客户端向Nginx反向代理服务器发送请求
2. Nginx缓存请求到内存或磁盘
3. 根据负载均衡算法从后端上游服务器中选取一台建立连接
4. 后端服务器返回响应到Nginx
5. nginx边接收边返回到客户端，而不会完整缓存

由于nginx在接收客户端请求时需要将请求内容完全缓存，在完全接收之前并没有与后端服务器建立连接。当全部接收完之后，再与后端服务器建立连接，由于内网环境的速度会很快，所以建立连接（发送文件）的速度也快，会减少后端服务器的压力，将压力放在nginx上。

```

# upstream定义了一个真实的后端集群
# server 定义了一个上游服务器，可以是ip、域名、unix
# 

upstream backends {
	server backend1.com weight=5;
	server backend2.com max_fail=5 fail_timemout=30s;
}

server {
	location / {
		proxy_pass http://backends.com;
	}
}

# ip_hash表示按照客户端的ip取模，与weight参数不可同时配置

upstream backends {
	ip_hash;
	server backend1.com ;
	server backend1.com ;
}

proxy_pass_header
proxy_pass_request_headers
proxy_pass_body

# 当服务器向客户端发生下面的错误时，选择其他服务器返回请求
proxy_next_upstream [];

```

