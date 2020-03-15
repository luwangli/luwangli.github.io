---
layout:     post
title:      虚拟机3种网络配置区别
subtitle:   桥接、NAT、Host-only区别
date:       2020-03-15
author:     BY beta
header-img: img/2020-03-09/head.png
catalog:    true
tags:
    - 计算机网络
    - 工程项目
---

## 前言
在VMware创建虚拟机时，通常需要配置网络，常见的选项是桥接模式、NAT
模式和仅主机模式，那么三种网络设置的区别是什么呢？接下来我会给大家
做一个详细的解释。
## 1.虚拟网卡
在介绍网络设置之前，我们得知道虚拟网卡的概念。一个设备如果需要和其他
设备交互，那么必须使用网卡进行数据包的转发。那么在VMware虚拟机安装
完成之后，会自动添加两个虚拟网卡，VMnet1和VMnet8。

![network adapter](/img/2020-03-15/host-virtual-adapter.JPG)  

三种网络模式与虚拟网卡对应关系：
1. 桥接：VMnet0
2. NAT：VMnet8
3. Host-only：VMnet1

我们可以在宿主机的网络设置里面看到VMnet1和VMnet8两张网卡，那么VMnet0
在哪里呢？实际上，VMnet0应该理解成二层交换机，宿主机和虚拟机都连接在
这个交换机上。因为虚拟机和宿主机是平等的，并不是通过宿主机去访问
物理网络，因此在宿主机中是不存在VMnet0。[网友gsonliu](https://www.jianshu.com/p/305f7384cfe9)
的图解释的很好，如下:  

![VMnet0](/img/2020-03-15/vmnet0.JPG)

## 2.Bridge桥接模式
桥接模式默认使用VMnet0虚拟网卡。该模式可以认为是将宿主机、虚拟机都
连接到VMnet0上，因此虚拟机可以直接连接到物理网络，拓扑如下:  

![bridge topology](/img/2020-03-15/bridge-topo.JPG)

在虚拟机中我们查看ip地址（WLAN地址）可以发现
![bridge ip](/img/2020-03-15/bridge-ip-addr.JPG)

在宿主机中，我们是通过wifi连接网络，可以看到相应的IP地址

![host wlan ip](/img/2020-03-15/host-wlan-addr.JPG)

所以，我们可以理解成在桥接模式下的虚拟机，实际上就是一个独立的计算机
连接到了路由器，路由器给它分配了一个IP地址。
桥接模式是最为简单的网络配置，可以使虚拟机快速、方便的连接到物理网络。

## 3.NAT模式
NAT模式，全称为“Network address translation",即网络地址转换。该模式下
虚拟机连接到宿主机的虚拟网卡VMnet8，VMnet8充当路由器功能，负责将虚拟机上传
的数据包进行地址转化后发送到物理网络，物理网络返回的数据包也需转化地址后
通过VMnet8发送给虚拟机。拓扑如下：

![nat topo](/img/2020-03-15/nat-topo.JPG)

在虚拟机中查看ip地址可见：

![host ip](/img/2020-03-15/virtual-nat-addr.JPG)

在宿主机中查看VMnet8地址：

![host Vmnet8](/img/2020-03-15/host-nat-addr.JPG)

可以看到nat模式下，虚拟机的ip地址和VMnet8的ip在同一网段，
同时，在虚拟机ping百度，可以发现能够ping通。

![vm nat ping](/img/2020-03-15/vm-nat-pingbaidu.JPG)

## 4.host-only模式
host-only模式下，虚拟网络是一个封闭的网络，连接到该模式下的所有虚拟机
和宿主机之间可以互相通信，但是虚拟机不能访问物理网络。
也就是说，Host-only模式与NAT模式非常相同，有DHCP功能，但是
没有NAT功能，所以物理网络和虚拟网路是隔离的，拓扑如下：

![hosonly topo](/img/2020-03-15/hostonly-topo.JPG)

在虚拟机中查看ip地址：

![VM hostonly ip](/img/2020-03-15/virtual-hostonly-addr.JPG)

在宿主机中查看ip地址：

![host hostonly ip](/img/2020-03-15/host-only-addr.JPG)

可以看到host-only模式下，虚拟机的ip地址和VMnet1的ip是在同一网段。
在虚拟机下，ping百度(180.101.49.12)，可以发现无法ping通。这印证了
没有host-only模式下，物理网络与虚拟网络的隔离。

![vm ping baidu](/img/2020-03-15/vm-hostonly-pingbaidu.JPG)

## 参考
https://www.cnblogs.com/ggjucheng/archive/2012/08/19/2646007.html  

https://www.jianshu.com/p/305f7384cfe9  

https://blog.csdn.net/clevercode/article/details/45934233   
