---
title: 网络系列：常用网络设备
date: 2019/2/10 20:41:20
categories:
  - 网络
tags:
  - 网络设备
---

# 网络设备


| 设备           | 工作层级 | 功能                                                         |
| -------------- | -------- | ------------------------------------------------------------ |
| **集线器**     | L1       | 广播数据包到所有端口，无智能过滤                             |
| **交换机**     | L2       | 基于 MAC 地址对数据帧进行转发，支持 VLAN 划分                |
| **路由器**     | L3       | 基于 IP 地址路由数据包（Packet），隔离广播域，支持 NAT 和防火墙 |
| **三层交换机** | L3       | 结合交换机和路由器的功能，高速转发并支持路由策略             |

## **交换机**

### 工作原理

交换机根据每个**端口**接收到的数据帧源地址进行**MAC地址学习**，内部维护一张**MAC地址表**。在后续的通讯中，发往特定MAC地址的数据包仅被转发至该MAC地址对应的端口，而非所有端口。

### **MAC 地址表建立过程**

初始状态下，交换机的MAC地址表为空。

1. **接收数据包**：交换机从某个端口接收数据包，交换机会检查数据包的头部信息（包括源MAC和IP地址、目的MAC和IP地址）。同时将源MAC地址和其端口号记录到MAC地址表中
2. **查看MAC地址表**
   - **找到目的MAC** -> 交换机对数据包进行端口**转发**
   - **未找到目的MAC** -> 交换机会**泛洪**数据包，所有主机均会收到该数据包（除了接收端口）
3. **更新MAC地址表**
   - **收到响应数据包** -> 交换机记录响应数据包的源MAC地址和端口号，并更新MAC地址表
   - **未收到响应数据包** -> 说明目的主机可能不在网络中或者没有响应，数据包**丢弃**（交换机默认行为）

### **三层交换机**

三层交换机（L3 Switch）可以处理网络层协议，通过对缺省网关的查询学习来建立局域网（如企业内网、校园网）不同网段的直接连接。

**首次路由与后续转发**

1. 当主机A与不同子网的主机B通信时，三层交换机需要通过缺省网关进行路由决策：
   - 主机A发送ARP请求到缺省网关（三层交换机的IP）。
   - 交换机查询路由表找到目标网段的下一跳地址，并通过ARP广播获取主机B的MAC地址。
2. 交换机将IP地址与MAC地址的映射记录到转发表（如FIB表和邻接表），并更新二层MAC地址表。
3. 当数据包再次到达交换机后，直接通过二层转发（MAC地址表），无需重复路由，再次拆包分析IP地址（即“直接连接”）。

## 网卡Bond

所谓bond，就是把多个物理网卡绑定成一个逻辑上的网卡，使用同一个IP工作，有时服务器带宽不够了也可以用作增加带宽。

借助于网卡bond技术，不仅可以提高网络传输速度，更重要的是，还可以确保在其中一块网卡出现故障时，依然可以正常提供网络服务。

网卡绑定mode共有七种(0~6) bond0、bond1、bond2、bond3、bond4、bond5、bond6。常用的有三种：

- mode=0（平衡负载模式）：平时两块网卡均工作，且自动备援，但需要在与服务器本地网卡相连的交换机设备上进行端口聚合来支持绑定技术。
- mode=1（自动备援模式）：平时只有一块主网卡工作，在它故障后自动替换为另外的网卡。
- mode=6（平衡负载模式）：平时两块网卡均工作，且自动备援，无须交换机设备提供辅助支持。

**1.Bond准备工作**

首先要确定服务器上的网卡规划用途，以及哪些网卡已插网线，一般是有两块网卡对应两根网线，分别连接不同的交换机。

```sh
[root@master01 network-scripts]# ethtool p4p2
Settings for p4p2:
        Supported ports: [ FIBRE ]
        Supported link modes:   1000baseKX/Full
                                10000baseKR/Full
                                25000baseCR/Full
                                25000baseKR/Full
                                25000baseSR/Full
        Supported pause frame use: Symmetric
        Supports auto-negotiation: Yes
        Supported FEC modes: None BaseR
        Advertised link modes:  1000baseKX/Full
                                10000baseKR/Full
                                25000baseCR/Full
                                25000baseKR/Full
                                25000baseSR/Full
        Advertised pause frame use: Symmetric
        Advertised auto-negotiation: Yes
        Advertised FEC modes: None
        Speed: 10000Mb/s
        Duplex: Full
        Port: FIBRE
        PHYAD: 0
        Transceiver: internal
        Auto-negotiation: on
        Supports Wake-on: d
        Wake-on: d
        Current message level: 0x00000004 (4)
                               link
        Link detected: yes
```

> ethtool查看网卡信息，`Link detected：yes`表示有网线插入；
>
> 如果`Link detected:no` 的话，尝试用`ifup ethxxx`，如果依然为no的话，才能说明此网卡确实没有网线插入。

**2.网卡Bond配置**

方法一：命令行配置

```sh
# 创建一个名为 storagepub 的网卡绑定接口，类型为 bond，模式为 802.3ad
ip link add storagepub type bond mode 802.3ad xmit_hash_policy layer3+4

# 关闭网卡
ip link set ens4np1 down

# 将网卡 ens4np1 加入到名为 storagepub 的绑定接口，成为其从属（slave）设备
ip link set ens4np1 master storagepub

# 启用网卡
ip link set ens4np1 up

# 启用绑定接口 storagepub，使其可用
ip link set storagepub up

# 查看绑定接口 storagepub 的详细信息，包括模式、成员网卡、负载均衡策略等。
cat /proc/net/bonding/storagepub
```

方法二：修改bond网卡的配置文件

```sh
[root@master01 network-scripts]# cat ifcfg-bond-storagepub
DEVICE=storagepub
BONDING_OPTS="mode=4 miimon=100 xmit_hash_policy=1"
TYPE=Bond
BONDING_MASTER=yes	# 表示该设备是绑定主设备（master）。
BOOTPROTO=static	# 使用静态 IP 配置（不依赖 DHCP）
PEERDNS=no
IPV4_FAILURE_FATAL=no
IPV6INIT=no	# 不启用 IPv6 配置
NAME=bond-storagepub
ONBOOT=yes
IPADDR=172.22.88.177
NETMASK=255.255.255.0

[root@master01 network-scripts]# cat ifcfg-p4p2
TYPE=Ethernet
BOOTPROTO=static
DEVICE=p4p2
ONBOOT=yes
MASTER=storagepub	# 指定绑定接口的主设备为 storagepu。
SLAVE=yes	# 指定该网卡是绑定接口的从设备（slave）
```

3.**重启网络验证**

```
[root@master01 ~]# systemctl restart network

[root@master01 ~]# ip a |grep storagepub
11: p4p2: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc mq master storagepub state UP group default qlen 1000
19: storagepub: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 172.22.88.177/24 brd 172.22.88.255 scope global noprefixroute storagepub
```

# 常见问题

## 交换机的VLAN怎么划分的

为了解决广播域带来的问题，引入了VLAN (Virtual Local Area Network)，即虚拟局域网技术：

通过在交换机上部署VLAN，可以将一个规模较大的广播域在逻辑上划分成若干个规模较小的广播域，由此可以有效地提升网络的安全性，同时减少垃圾流量，节约网络资源。

每个VLAN网络（子网）是一个独立的广播域，VLAN内的设备可以相互通信，而不同VLAN之间的设备默认不能直接通信，必须通过三层设备（如路由器或三层交换机）进行路由才能互通。

划分VLAN的方法有多种，常见的包括：

- **基于端口划分（常用）**：将交换机的端口分配到不同的VLAN中。例如，将端口1和2分配到VLAN10，端口3和4分配到VLAN20。
- **基于MAC地址划分**：根据设备的MAC地址将设备分配到不同的VLAN中。
- **基于IP子网划分**：根据设备的IP地址和子网掩码来划分VLAN。
- **基于协议划分**：根据设备使用的协议类型来划分VLAN。

## 不同网段如何实现通信

不同网段的设备需要通过 **路由设备** 实现通信，具体流程如下：

1. 主机A（192.168.1.10）向主机B（192.168.2.20）发送数据包：
   - 主机A发现目标IP不在同一网段，因此将数据包的目标MAC地址设为 **默认网关**（如L3交换机的IP 192.168.1.1）。
2. 三层交换机接收数据包：
   - 根据目标IP 192.168.2.20 查询路由表，找到下一跳接口（如连接192.168.2.0网段的端口）。
   - 发送 **ARP请求** 到目标网段，获取主机B的MAC地址。
3. 转发数据包：
   - L3交换机将数据包的目标MAC改为主机B的MAC，源MAC改为自己的接口MAC，然后通过二层转发到目标网段。
4. 后续通信优化：
   - 三层交换机会缓存 IP-MAC映射 和 路由路径，后续数据包直接通过二层转发，无需重复路由。

## 路由器和交换机区别

1. 交换机一般工作在数据链路层，而路由器则工作在网络层。
2. 路由器可连接超过两个以上不同的网络，而交換机只能连接两个。
3. 交换机是根据MAC地址转发数据帧，而路由器则是根据IP地址来转发IP数据包/分组。
4. 交换机主要是用于组建局域网，而路由器则更多用于广域网（WAN）或异构网络互联。
