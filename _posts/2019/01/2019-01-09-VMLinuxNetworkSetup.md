---
layout: post
title: VMware Centos7 Network Setup
subtitle: 在VMware中配置Centos7单体及克隆后的网络配置
date: 2019-01-09
author: BF
header-img: img/bf/snow_02.jpg
catalog: true
tags:
  - linux
  - hadoop
  - network
  - vmware
---

# 背景

没想到搞虚拟机搞了这么久，终于配置成功了，不过踩坑也是难免的，记录一下。

目前要做的是搭建 Hadoop 集群环境，但是在配置多台 linux 虚拟机时网络一直没有弄好，远程无法访问，出现各种问题。主要原因是自己这块基础不牢，网上的资料也太杂。

更重要的是，我使用的是带 GUI 的 CentOS，导致了有两套 Network 管理，所以冲突了，尴尬。

# 环境

- VMWare 12
- CentOS7
- 主机 Win10
- NAT 模式

# NAT 模式

因为采用 NAT 模式对虚拟机进行网络配置，所以需要将主机网络共享给虚拟机 NAT 网卡，如下：

![2019-01-09-VMLinuxNetworkSetup.01.jpg](/img/post/2019/01/2019-01-09-VMLinuxNetworkSetup.01.jpg)

启用之后, 则 VMnet8 网卡的配置要如下所示，固定 ip 网段为 192.168.137.\*

![2019-01-09-VMLinuxNetworkSetup.02.jpg](/img/post/2019/01/2019-01-09-VMLinuxNetworkSetup.02.jpg)

# CentOS7 网络配置

下面以克隆的一台机子为例（被克隆的机子为 hadoop00/192.168.137.100），配置新的网络。

## Mac Address

当刚克隆的时候，机子的 Mac 地址也是一样的，所以要在启动前先重新生成（00:50:56:24:A9:10 就是新生成的 mac 地址）：

![2019-01-09-VMLinuxNetworkSetup.03.jpg](/img/post/2019/01/2019-01-09-VMLinuxNetworkSetup.03.jpg)

## hostname

原来机器的 hostname 是 hadoop00，所以我们需要使用`hostnamectl set-hostname hadoop01`将名字改成`hadoop01`

## /etc/hosts

下面是增加 hosts 配置，在之后建立集群时，需要将多台机子的 ip，hostname 信息都要添加进来。

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 hadoop01
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.137.101 hadoop01
```

## 修改网卡配置

首先我们可以使用`ifconfig`先查看一下网卡信息（这是我已经配置好了），
我们可以看到网卡的名字为`eno16777736`, Mac 地址和我们生成的是一致的。

```bash
[root@hadoop01 ~]# ifconfig
eno16777736: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.137.101  netmask 255.255.255.0  broadcast 192.168.137.255
        inet6 fe80::250:56ff:fe24:a910  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:24:a9:10  txqueuelen 1000  (Ethernet)
        RX packets 405  bytes 45790 (44.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 245  bytes 42636 (41.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:d8:7c:eb  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
所以我们需要配置`/etc/sysconfig/network-scripts/ifcfg-eno16777736`文件，这里便是核心的配置。

内容如下所示：
```
DVICEE=eno16777736
NAME=eno16777736
TYPE=Ethernet
IPADDR=192.168.137.101 # ip地址
PREFIX=24
NETMASK=255.255.255.0 # 子网掩码
NETWORK=192.168.137.0 # ip段
GATEWAY=192.168.137.2 # 网关地址
BROADCAST=192.168.137.255 # 广播地址, 网关地址最后一位换成255
DEFROUTE=yes
ONBOOT=yes
USERCTL=yes
BOOTPROTO=static
IPV4_FAILURE_FATAL=yes
HWADDR=00:50:56:24:A9:10 # 这里填执行ifconfig命令后, ens33(这里名称可能不同)的mac地址
IPV6INIT=no
DNS1=8.8.8.8
DNS2=114.114.114.114
```

之后有一些资料中说要删除`/etc/udev/rules.d/70-persistent-ipoib.rules`,但我查看过，里面是空的，可能是我使用的系统关系，想来不删除也没有关系的。

# 重启网卡服务

在这里，一般使用:
```
systemctl restart network.service
```
也可以用:
```bash
service network stop
service network start
```
而我像开头说的，因为带了GUI，所以我这边还需要停掉一个`NetworkManager`服务，使用:
```bash
service NetworkManager stop
chkconfig NetworkManager off
```

此外，为了方便，防火墙可以关闭掉：
```bash
//临时关闭
systemctl stop firewalld
//禁止开机启动
systemctl disable firewalld
```
这样子的话，下次开机，就能直接通过ssh访问了。

# 最后

主要是自己不熟悉一些概念，所以遇到问题没有办法找到根源，只能有时间再好好啃啃鸟哥的私房菜了。


参考：

- [VMware 虚拟机中 Centos7 网络配置及 ping 不通思路](http://blog.51cto.com/bestlope/1977074)
- [RTNETLINK answers: File exists 错误解决方法](https://blog.csdn.net/u010719917/article/details/79423180)
- [VMware 下克隆 centos7 后的网络配置及主机名问题](https://blog.csdn.net/mo_ing/article/details/81036339)
- [CentOS 7 开机 network service 不启动的问题](https://blog.csdn.net/lcr_happy/article/details/69849834)
- [Centos 怎么关闭 NetworkManager 服务](https://jingyan.baidu.com/article/b24f6c82c38bd486bfe5dab4.html)
- [VMware克隆CentOS7，解决网络配置问题](https://blog.csdn.net/airufengye/article/details/81566454)