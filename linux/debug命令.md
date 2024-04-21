# 网络

## curl



## ip

### ip route

- 查询经过某个子网的路由

  ip route show 12.103.48.0/24

### ip -s show ens33

- -s：统计
- overrun-溢出，collsns-通信碰撞

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

- RES：resident memory size
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

  


## pstree

- p：显示pid
- a：显示命令行参数



# 内存

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

- 设备性能测试

  ```bash
  dd if=/dev/zero of=/path/to/testfile bs=1M count=1000
  #向testfile中写入1G
  ```

- 修复损坏的存储介质





# coredump和kdump

kdump更加底层，没有coredump好用。

coredump是systemd提供的工具，与journald的日志集成，提供了coredumpctl工具查看转储r日志

