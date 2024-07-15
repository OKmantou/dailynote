# iptable

## hook

netfilter是内核的处理数据包的模块，工作在内核态，由一些列的hook组成。数据经过内核协议栈时，根据数据包的状态（流向、数据头信息）会触发内核模块注册在hook上的数据包处理函数。

netfilter有五个hook点及触发时机

- prerouting（NF_IP_PRE_ROUTING）：数据包进入协议栈，但是没有经过路由
- input（NF_IP_LOCAL_IN）：经过路由判断为目的地址是本机
- forward（NF_IP_FORWARD）：经过路由判断目的地址为其他机器
- output（NF_IP_LOCAL_OUT）：本机产生的数据包，经过协议栈之前
- postrouting（NF_IP_POST_ROUTING）：本机产生或者转发的，经过路由之后触发

内核模块在hook上注册函数时，需要提供函数的优先级。当hook执行函数时，按照优先级执行。优先级就是表的优先级。

## table、chain、rule

iptable是运行在用户态的进程，方便用户对netfilter中的规则进行管理。netfilter提供了三个概念来对内核中的规则管理：

- table：表是一类处理规则的集合，包括raw、mangle、net、filter和secure表。每个表都有不同的作用，会有不同的链，最常使用的是net和filter表。
- chain：链对应hook，是一串规则
- rule：链上具体的规则。规则包含：匹配模式 + 处理动作

下面是不同的表包含的不同的链：

|          |            |       |         |        |             |
| -------- | ---------- | ----- | ------- | ------ | ----------- |
| Tables   | PREROUTING | INPUT | FORWARD | OUTPUT | POSTROUTING |
| raw      | ✅          |       |         | ✅      |             |
| mangle   | ✅          | ✅     | ✅       | ✅      | ✅           |
| nat      | ✅          |       |         | ✅      | ✅           |
| filter   |            | ✅     | ✅       | ✅      |             |
| security |            | ✅     | ✅       | ✅      |             |

### net table

用于做地址转换，常用于SNAT（外网到内网）和DNAT（外网到内网）

### filter table

判断包是否允许通过。如果允许主机转发数据包，还需要设置内核参数ipv4_ip_forward = 1

![](C:\Users\OK馒头\Desktop\dailynote\image\linux\liuhceng.png)

### 自定义链

用户可以向表中添加自己定义的链，如kubernetes中在filter表中有 KUBE-EXTERNAL-SERVICES和 KUBE-SERVICES链

### rule

规则动作包括如下：

- accept：运行通过
- drop：拒绝通过，不返回响应
- reject：拒绝通过，并返回拒绝响应
- redirect：端口映射
- snat：源地址转换
- dnat：目的地址转换

## 使用

```shell
1. 查看一个表中的某一条链
iptables -t filter -L KUBE-SERVICES -n --line-numbers

2. 向filter表中input链序号为2的位置，新增一条拒绝所有来自1.1.1.1的规则
iptables -t filter -I INPUT 2 -s 1.1.1.1/32 -j REJECT

3. 删除上面那条规则
iptables -t filter -D INPUT -s 1.1.1.1/32

4. 删除序号为2的规则
iptables -t filter -D INPUT 2

5. 向filter的input链追加一条规则
iptables -t filter -A INPUT -s 1.1.1.1/32 -j DROP

6. 禁止1.1.1.1访问本机22，21端口
iptables -t filter -I INPUT -s 1.1.1.1/32 -p tcp -dport 21,22 -j REJECT

7. 将访问本机80端口的请求转发到本机8080端口
iptables -t nat -I PREROUTING -s 1.1.1.1/32 -p tcp -dport 80 -j DNAT --to-destination 1.1.1.1:8080


```



```
向filter中新建一条链
iptables -t filter -N <new_chain>

修改链的默认策略(默认是accept)
iptables -t filter -P <new_chain> DROP
```

## k8s

- KUBE-NODEPORTS：当svc是nodeport类型时，先匹配该链，该链再跳转到KUBE-SERVICES
- KUBE-SERVICES：一个包含所有svc地址映射的链，用于将访问svc的流量转发到KUBE-SEP-xxx链
- KUBE-SVC-xxxx：一个svc对应一个链，该链用于将访问svc的流量nat到对应的endpoint链
- KUBE-SEP-xxx：一组endpoint链，其中包含了每个pod的地址
