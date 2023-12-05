## systemctl


- sysctl和systemctl和区别：
  - sysctl 是一个用于管理内核参数的命令。它允许查看、修改和持久化内核运行时的各种参数设置。这些参数控制着操作系统的各种行为和功能，如网络设置、文件系统缓存、内存管理等。
  - systemctl 是一个用于管理系统服务的命令。它用于启动、停止、重启、启用、禁用和管理系统服务。系统服务是在后台运行的进程，负责提供各种功能和服务，如网络服务、日志服务、定时任务等。

- umask：为用户创建文件或目录时设置一个默认的权限，该操作与chmod是相反的操作。

  umask使用的是补码。

  对于文件，最大值是6，因为os不允许创建文件时就拥有执行权限。

  对于目录，最大值是7，可以有执行权限。

  例如文件的umask=022，那么对应的chmod是644

- ulimit：设置用户进程资源限制。ulimit -a可以查看当前用户的资源限制，可以通过命令或者直接修改/etc/security/limits.d/下的文件

## LVM

logical volume manager，一种管理**磁盘分区**和**逻辑卷**的工具。

disk或partition可以创建为pv，多个pv可以创建为vg，一个vg可以创建出多个lv。同一个vg下的lv可以扩缩容，可以通过向vg中添加pv，来实现vg容量扩充。

pv和lv都会分配一个UUID，通过 `blkid`查看。

> 分区是针对磁盘的操作，相对LVM更底层，也是静态划分。划分的分区可以看作是独立、隔离的磁盘。

使用LVM划分卷的步骤：

1. 创建物理卷 physical volume

   使用 `pvcreate` 在一个或多个物理磁盘（硬盘或硬盘分区）划分为物理卷

2. 创建卷组 volume group

   使用 `vgcreate` 将一个或多个物理卷创建为一个卷组

3. 创建逻辑卷 logical volume

   使用 `lvcreate` 在卷组上创建逻辑卷。逻辑卷是一个可以分配空间的**块设备**。

4. 格式化逻辑卷

   使用文件系统工具对逻辑卷初始化

5. 挂载逻辑卷

## systemd-journald

systemd提供的系统日志服务，负责收集、存储和管理系统日志。其中包括 `systemd-coredump`部分，用于记录程序崩溃后的转储文件，并提供 `coredumpctl`工具，转储文件与journal日志集成，存储在 `/var/lib/systemd/coredump`下。