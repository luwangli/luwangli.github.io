---
layout:     post
title:      p4 load_balance 实现
subtitle:   P4官方教程（八）
date:       2020-11-07
author:     BY beta
header-img: img/2020-11-07/head.jpg
catalog:    true
tags:
    - 工程
    - P4
---

## 1.实验load_balance目的

基于等价多路径转发实现负载均衡，看到这个实验以及相关的材料，负载均衡在此实验中的表现是假定有两个目标主机，对于发送端而言，它可以将数据包发送给任一的接收端。大概模拟的是服务器为客户端提供服务，对于管理者而言，为了避免一个服务器的负载过大，会采用多个服务器分散压力。

## 2.实验思路

因为教程中写道，控制平面的流表规则已经写好，所以我想到的是查看交换机s1的流表规则，它的table分为哪些类，有哪些动作。

打开s1的json配置文件，看到table有这么几个

- MyIngress.ecmp_group: 猜测是内置了hash函数，如果数据包来源于10.0.0.1,则将五元组hash（应该端口不同，所以同一源地址的数据包会有不同的hash结果）并选择目的地址。中间通过meta存储、传输这个信息。
- MyIngress.ecmp_nhop：根据meta值选择不同的路径
- MyEgress.send_frame: 



## 3.代码修改步骤

### MyIngress

我想理解的是，如何在Ingress阶段调用hash函数，我试图在P4的白皮书搜索hash，当时很遗憾，没有找到函数调用原型；单独查看实现的代码，我也无法确定hash的内部过程。

那么我该去哪里找寻相关的资料呢：

- github的讨论区
- bmv2的实现





## 4.实验结果

按照实验要求，在h1输入发包指令，可以看到有些包发送给h2，有些包发送给h3

![image-20201108105653389](https://i.loli.net/2020/11/08/tDdbwNO5gRUpl9k.png)

![image-20201108105708938](https://i.loli.net/2020/11/08/quKh9vGtIs78gne.png)

## 总结

实现了简单的负载均衡。交换机会根据数据包携带的头部计算出目标主机、并遵守相关的路径转发。

- 了解如何在交换机内部使用hash函数
- 学习了使用meta在内部传输信息