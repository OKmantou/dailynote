# Redis

https://zhuanlan.zhihu.com/p/534892012

## 常用配置

### 网络

- bind，port
- timeout：客户端连接idle状态超时时间，默认关闭，即不会断开idle连接
- tcp-keepalive：默认300s
- tls/ssl等配置：包括客户端、replicas、集群角色的证书等

### 内存

### 其他

- pidfile、logfile



## 持久化

### RDB

redis database是一种redis的持久化技术，通过手动执行save、bgsave或者配置save参数（按照策略执行bgsave）来对redis实例的数据执行快照。在执行快照时，bgsave会fork出一个子进程来将全量数据写入磁盘；而save命令在父进程中执行，会造成阻塞。

可以配置rdb的参数：rdbcompression、dbfilename、dir

通常会将rdb文件作为冷备。

### AOF

append of file是另一种数据持久化策略，redis会按照配置的策略，将客户端向redis写入的数据追加到一个aof文件中。可配置的策略appendfsync包括：

- always：每次客户端写入数据后，马上将内存刷新到磁盘。该策略会造成大量IO时间，但是数据不会丢失。
- everysec：每隔1s将内存的数据刷新到磁盘
- no：依靠os的刷盘策略

aof文件会不断增长，redis可以通过下面的参数来配置重写的触发条件，并且需要两个都满足。即使配置了两个参数，aof文件还会不断增长。重写可以优化aof文件的性能和大小。

- auto-aof-rewrite-percentage：当aof文件超过上次重写之后aof文件大小的百分之多少，才开始重写
- auto-aof-rewrite-min-size：aof文件超过该值才有可能触发重写

aof工作流程：

1. redis执行写命令，执行系统调用write，将命令写到aof文件内存中
2. 根据刷盘策略，执行系统调用fsync，将命令写到磁盘
3. 触发重写

当磁盘IO负载较高时，后台线程在执行fsync时会被阻塞，而redis进程又需要等待后台线程执行完才可以调用write写入aof文件内存，从而造成redis无法响应客户端请求，可能会造成超时。



## 高可用

### 主从模式

redis采用异步复制的方式，将master的数据同步到replicas。master只接收客户端的写操作，replicas接收读操作。主从满足最终一致性，当数据不是最新的时返回旧数据。

在主从模式下，master需要开启持久化，否则当master节点重启时数据丢失，而replicas接收空的master数据，会造成replicas的数据丢失。主从的复制流程：

1. replicas启动时，会向master节点发送psync
2. 如果replicas第一次启动，master会发送全量的rdb
3. 如果replicas有部分数据，会发送部分数据

可以通过如下配置来配置主从关系：

- replicaof
- masterauth

### 哨兵

在主从模式下，如果master节点宕机，需要人工切换主从。哨兵模式的作用是：

- 集群监控：监控各节点状态
- 消息通知：如果有故障会通知管理员
- 故障转移：主从切换
- 配置中心：故障转移后会通知客户端

### 集群





