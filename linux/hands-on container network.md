# hands-on network

> 目标：
>
> 1. 配置一个网桥，设置网段
> 2. 创建一个veth pair
> 3. 将其中一个veth放入一个网络命名空间
> 4. 切换到新的网络空间

> 主要命令：
>
> 1. brctl
> 2. ip
> 3. sysctl：在运行中修改linux内核的参数，常用网络

### 创建网桥，设置网段

```bash
brctl addbr mybr0
```

```bash
ip addr add 10.10.10.1/24 dev mybr0
```

```
ip link set mybr0 up
```



### 创建veth pair

```bash
ip link add veth0 type veth peer name veth1
```

向mybr0中添加一个veth

```
brctl addif mybr0 veth0
```

配置在容器内的veth ip地址：

```
ip addr add 10.10.10.2 dev veth1
```

激活veth pair：

```
ip link set veth0 up && ip link set veth1 up
```

### 创建命名空间并移动一个veth

```
ip netns add myns1
```

将一个veth移入`myns1`：

```
ip link set veth1 netns myns1
```

### 在新网络空间配置默认路由

```shell
ip netns exec myns1 bash
```

配置默认路由：（这里设置的是所有路由都经过veth1）

```shell
ip addr add default via 10.10.10.2 dev veth1
```

在 `myns1`中激活veth：

```
ip link set veth1 up
```

### 主机开启IP forward ，配置NAT

```
sysctl net.ipv4.ip_forward=1
# 设置为1表示：内核允许ip包在不同端口中转发
```

```
iptables -t nat -A POSTROUTING -o <host_interface> -j MASQUERADE
#  This command enables IP forwarding on the hostsystem and configures NAT to allow the network namespace to access the external network.
```

> 使用hostname -I 查看主机IP的时候，为什么会显示主机网卡ip和网桥IP？
>
> https://www.cnblogs.com/bakari/p/10529575.html
