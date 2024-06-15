## Network/NetworkManager模块

在RHEL/Centos6之前，网络功能时通过一些列网络相关的脚本文件实现（如/etc/init.d/network文件）。

从RHEL/Centos7以后，网络功能默认由NetworkManager以服务的形式提供，NetworkManager是一个能动态控制和配置网络的守护进程，对应NetworkManager.service服务，配置文件为/etc/networkmanager/networkmanager.conf。Network模块也会以network.service的形式存在。

两个服务只能使用一个。

### network.service

可以被systemd进程托管。在服务启动时，会执行 /etc/init.d/network 脚本文件，该文件会读取ifcfg文件，并检测NetworkManager是否已经启动该设备。如果已经启动，则/etc/init.d/network脚本部操作，否则启动该设备。

### networkmanager.service

默认不执行任何脚本，当在 /etc/NetworkManager/dispatcher.d 目录下存在root权限的文件时，按照字母顺序顺序执行。

### 网络配置文件

network和networkmanager都使用下面两个配置文件

- 全局配置文件 /etc/sysconfig/network，修改完后使用systemctl restart network
- 网卡相关的配置文件：/etc/sysconfig/network-scripts：nmcli connection reload

## route

- 默认路由

  当目的地址匹配不到任何一条路由时，则会选择默认路由。默认路由是目的地址为0.0.0.0的路由，掩码也为0.0.0.0，Flags标志为UG。

- Gateway是0.0.0.0

  表示数据包直接发往目的网络，而无需经过任何网关。这种情况通常出现在目的网络与本机网络直连的情况下，无需网关转发（代表该网络已知？）。	