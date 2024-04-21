# Netapp存储基本概念

> 以下是基于ONATP SystemManager产品和使用手册来说明

## 基础概念

### 集群结构

存储设备是一个整体。下图中包括两个节点：FAS2750和FAS8200.

横向的长条状是一个机架shelf，一个机架上又包含若干个磁盘disk。在每个节点上都配置了多个网络接口。

<img src="C:\Users\OK馒头\AppData\Roaming\Typora\typora-user-images\image-20231211140720848.png" alt="image-20231211140720848" style="zoom: 33%;" />

下图是整个存储设备的网络结构，外部client通过ether或者FC来访问存储，但是在该存储设备上只提供文件存储（NFS、CIFS）和对象存储（S3）。

<img src="C:\Users\OK馒头\AppData\Roaming\Typora\typora-user-images\image-20231211141322446.png" alt="image-20231211141322446" style="zoom:33%;" />

### 存储

- cluster

  默认一个存储设备就是一个集群，可以包含多个node

- shelf

  机架，一个node可以包括多个机架

- disk

  磁盘，一个shelf上可以插入多个磁盘

- storage VM

  SVM是一个逻辑概念，用于抽象存储和网络资源，向client暴露访问方式。向SVM中存储数据并不会绑定到具体的存储位置。一个SVM可以对应于一个tenant，并且设置该tenant的policy。

  一个SVM可以具有一个用于NAS流量的LIF和SAN流量的LIF。client只需要LIF地址即可访问。NAS和iSCSI协议需要IP地址，SAN需要FC的WWPN。管理员可以设置该SVM可以使用的存储协议。

  包括四种类型：

  - admin SVM：代表整个集群
  - node SVM：代表一个节点
  - system SVM：系统在IPSpace中为集群级别的通信创建系统SVM
  - 数据SVM：存储节点，需要想其中添加卷

  具体的绑定过程为：？？？

- volumes

  在netapp中，最常使用的是FlexVol类型的逻辑卷。SVM管理员可以向SVM中创建volume。

  volume可以被看做一个卷，在NAS存储场景下是一个被挂载到主机上的盘。

  client需要通过NFS、CIFS文件系统协议来访问volumes

  FlexVol类型提供了更高级的功能：snapshot，clone，mirror等。

  访问路径是   IP:/VolumeName

- LUNs

  LUN（logical unit number）用于表示一组存储设备。在netapp中，需要有支持FC和iSCI协议的SVM才可以创建LUN。

  LUN使用volumes作为存储资源，提供块级别的访问方式。

- Shares

  共享存储，允许多个客户端通过文件系统协议（NFS、CIFS）接入，被绑定到特定的SVM上。

  在SVM中映射为一个目录，访问路径是   \\\IP\DirName 

- Buckets

  对象存储，需要绑定到特定的SVM上，访问路径是  http ://domin/xxx

- Tiers

  在netapp中分为了多种层：高性能层（High-Performance Tiers，通常是SSD，用于存储数据库应用和关键数据），容量层（Capacity Tiers，通常是HDD，用于存储归档数据、备份数据）和中间层（Mid-Tiers）。

- LIF

  网络逻辑接口

### 网络

#### Ethernet Ports

- Link Aggregation Group
- vlan

#### FC Ports

### 管理

- 租户

  一个tenant对应一个SVM

- 集群管理员

  

- SVM管理员

## SAN

### 划分

### 扩容

## NAS

### 划分

### 扩容