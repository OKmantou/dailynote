# Nginx基础

> https://zhuanlan.zhihu.com/p/650074319

## 安装部署

**开源Nginx地址**：https://nginx.org

**源码内容：**

<img src="C:\Users\OK馒头\AppData\Roaming\Typora\typora-user-images\image-20231114170636076.png" alt="image-20231114170636076" style="zoom:67%;" />

**部署过程：**

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

**常用命令**

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

## 反向代理

## 负载均衡

