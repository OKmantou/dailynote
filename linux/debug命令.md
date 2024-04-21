# 网络

## curl



## ip

### ip route

- 查询经过某个子网的路由

  ip route show 12.103.48.0/24
  
- 查询到一个ip的路由

  ip route get ip



### ip addr

- 向dev添加ip地址

  ip addr add 12.103.48.95/24 dev eth0

  

### ip neigh

- 查询arp缓存

  ip neigh show

## ss



## tcpdump

**option**

- -c：抓包数量
- -i：侦听端口，默认是所有已经打开的设备，如：eth0
- -e：显示mac地址
- -A：使用ASCIⅡ打印，常用于web页面
- -w：写入文件
- -n，-nn：不进行反查DNS
- -v，-vv：详细显示

**过滤器**

- 对象类型

  - net：网络，如：net 192.168.10.0
  - host：主机：如：host 192.168.10.1
  - port：端口：如：port 80

- 传输方向

  - src
  - dst
  - src and dst = host
  - src or dst

- 协议

  ip，tcp，udp，icmp

**use case**

1. 监听目的地址为本机的、端口为22的数据包

   ```shell
   tcpdump -i eth0 dst host localhost and port 22
   ```

   





## nc

**option**

- l：监听
- p：指定端口
- v：verbose

**传文件**

- 服务端：nc -lvp 8080 < filename
- 客户端：nc -v `server-ip` 8080 > filename



## iptables



# 进程

## top

**metrics**

- RSS：resident memory size驻留在物理内存的大小
- R：process status
  - D uninterrupible sleep
  - R running
  - S sleeping
  - Z zombie
  - T stopped by job control signal
- VIRT：virtual memory size

**option**

- H：显示进程的层次结构
- d 秒数 c 次数
- p：显示指定进程的信息
- u：指定用户的进程
- 按照某一指标排序
  - shift+p：CPU
  - shift+m：内存
  - shift+n：网络带宽




## pidstat

-r -p pid

VSZ：进程虚拟内存大小

RSS：驻留的物理内存大小

## pmap 

-x：pid

total kB：显示WSS


## pstree

- p：显示pid
- a：显示命令行参数



# 内存

## 概念辨析

/sys/fs/cgroup/memory/memory.stat

/proc/meminfo

https://itnext.io/from-rss-to-wss-navigating-the-depths-of-kubernetes-memory-metrics-4d7d77d8fdcb

> 匿名内存（anonymous memory）：一块在进程的虚拟内存空间中分配的，但是没有任何文件或设备与之对应的内存。匿名内存不会从文件中加载数据，也不会将内存写入文件或设备，主要用于进程的临时读写，通常用于进程堆栈的分配。常见的匿名内存有：动态内存分配、函数调用的栈空间分配、线程的栈空间分配、进程间通信。
> 
>
> 文件映射（mmap）：将文件或设备映射到进程的虚拟地址空间，从而达到通过读写内存来读写文件。通常用于大文件高效读写、进程间通信。
> 
>
>
> 共享内存（shared memory）：操作系统为多个进程分配同一块内存区，多个进程可以通过互斥的读写共享内存来实现进程通信。通常用于多进程的数据交换、进程间同步。
>
>
> 交换空间（swap memory）：操作系统级别的磁盘（单独的swap磁盘），由所有进程共享同一块交换空间。
>
> 交换缓存内存（swap cache memory）：进程在物理内存中被交换出后，存储在内核中的缓存区域。当进程需要访问被交换出的页面时，可以首先从Swap Cache Memory中读取，而无需访问较慢的磁盘交换空间。

- RSS：resident memory size，驻留在物理内存的数量，由os提供。The amount of anonymous and swap cache memory

- WSS：work set size，进程当前活动（指经常访问，不包括不常访问的内存，即使它驻留在物理内存中；也包括驻留在磁盘的内存）的内存，通过页面访问的频率动态测算。可以用来评估进程的活动内存需求和性能特征。

  当RSS > WSS，表示给程序过量分配内存

  当RSS < WSS，表示程序需要进行页面置换才能运行（所需内存没有全部常驻物理内存）。如果inactive file小，则可能发生oom

- VIRT/VSZ：虚拟内存总量
- cache：为了快速访问而将数据拷贝到的一块区域
- buffer：在写入持久存储区前，用于将数据首先拷贝到buffer

![img](https://miro.medium.com/v2/resize:fit:1118/1*qkh8lDc8TgQlyl_ojF-94A.png)

### 节点内存

node-exporter计算内存的方式不包括os使用的cache和buffer

```
node_memory_MemTotal_bytes - node_memory_MemFree_bytes - 
(node_memory_Buffers_bytes + node_memory_Cached_bytes)
```

### 容器内存

包括三个部分：container_memory_wss ，container_memory_rss，container_memory_usage_bytes

## vmstat：

查看虚拟内存和系统活动

**option：**



**字段：**

- r：运行队列中的进程数，即等待中的进程数
- b：处于不可中断睡眠状态的进程数

可以查看系统中的负载情况、内存和交换区使用情况、磁盘活动情况



# 设备

## iostat

记录CPU、磁盘、分区的IO参数。通过读取/proc目录实现

**option**

- -c，-d：仅显示CPU、设备使用率
- 2 5 ：每2秒显示5条
- -p sda：显示sda的所有分区
- -t：打印时间
- -x：打印扩展信息
  - `rrqm/s`：表示每秒从磁盘请求的读取请求合并数（合并的读取请求）
  - `await`：表示平均每个I/O操作的等待时间（毫秒）
  - `%util`：表示磁盘的使用率，即磁盘处于活动状态的时间百分比




## dd

- 创建磁盘镜像

  ```bash
  dd if=/dev/sda of=/dev/sdb bs=4M
  # 从sda复制到sdb
  ```

- 备份

  ```bash
  dd if=/dev/sda of=/path/to/backup.img bs=4M
  ```

- 安全擦除磁盘

  ```bash
  dd if=/dev/urandom of=/dev/sda bs=4M
  ```

- 创建固定大小文件

  ```bash
  dd if=/dev/zero of=/path/to/testfile bs=1M count=1000
  #向testfile中写入1000M大小的文件，count是写入次数，而不是写入的文件数量
  
  dd if=/dev/urandom of=/path/to/file bs=1M count=1000
  ```

- 修复损坏的存储介质





# coredump和kdump

kdump更加底层，没有coredump好用。

coredump是systemd提供的工具，与journald的日志集成，提供了coredumpctl工具查看转储r日志

