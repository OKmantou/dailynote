# Redis

## 部署

### 单机

1. 建立redis用户和组

2. 修改内核参数vm.overcommit_memory

   在/etc/sysctl.conf中添加，或者 echo 1 > /porc/sys/vm/overcommit_memory

3. 修改限制 /etc/security/limits.conf

   redis soft nofile 10240

   redis hard nofile 10240

   redis soft nproc 10240

   redis hard nproc 10240

4. 安装依赖 zlib，tcl，gcc，openssl

5. 编译并安装redis

   make；make test；make install PREFIX=/usr/local/redis

6. 在/opt目录下安装redis配置文件

   /opt/redis/6379/{data,conf,log}

### 集群

## 配置文件

1. 后台启动与pid

   daemonize yes

   pidfile /opt/redis/6379/data/redis.pid

2. 身份认证

   requirepass xxx；masterauth xxx；

3. 禁用特定命令：rename-command KEYS ""

## 监控

### 监控指标

