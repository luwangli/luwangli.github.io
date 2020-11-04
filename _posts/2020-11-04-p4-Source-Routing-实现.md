---
layout:     post
title:      p4 Source Routing 实现
subtitle:   P4官方教程（六）
date:       2020-11-04
author:     BY beta
header-img: img/2020-11-04/head.jpg
catalog:    true
tags:
    - 工程
    - P4
---

## 1.实验source routing目的

本实验实现source routing，在发送端指定数据包的转发路径。具体而言，通过在数据包头部添加湍口列表字段，交换机在收到数据包时，提取相应的端口号，依据此端口号进行转发。

也就是说，在此实验中，控制平面并未给数据平面添加任何流表规则！

## 2.实验准备

## 3.P4代码修改步骤

### MyParser

解析器先进入状态parse_ethernet解析ethernet头部，并根据etherType的类型选择是否进入状态parse_srcRouting; 状态parse_srcRouting相对复杂一些，是要将输出端口栈都提取出来吗？

1. parse_ethernet

这个状态比较常规，根据ethernet.etherType 选择下一步

```
 68     state parse_ethernet {
 69         packet.extract(hdr.ethernet);
 70         /*
 71          * TODO: Modify the next line to select on hdr.ethernet.etherType
 72          * If the value is TYPE_SRCROUTING transition to parse_srcRouting
 73          * otherwise transition to accept.
 74          */
 75         transition select(hdr.ethernet.etherType) {
 76             TYPE_SRCROUTING : parse_srcRouting;
 77             default : accept;
 78         }
 79 /*        transition accept;*/
 80     } 
```

2. parse_srcRouting

解析源路由这个阶段是一个循环解析的过程，需要将srcRoutes数组的所有元素都提取。

```
 82     state parse_srcRouting {
 83         /*
 84          * TODO: extract the next entry of hdr.srcRoutes
 85          * while hdr.srcRoutes.last.bos is 0 transition to this state
 86          * otherwise parse ipv4
 87          */
 88         packet.extract(hdr.srcRoutes.next);
 89         transition select(hdr.srcRoutes.last.bos) {
 90             0 : parse_srcRouting;
 91             default: parse_ipv4;
 92         }
 93 /*        transition accept;*/
 94     }

```

### MyIngress

数据包头部被解析之后，进入Ingress阶段。根据实验要求，在这一阶段，数据包会根据srcRoute字段确定离开的端口。

1. action srcRoute_nhop

根据srcRoutes

```
125     action srcRoute_nhop() {
126         /*
127          * TODO: set standard_metadata.egress_spec
128          * to the port in hdr.srcRoutes[0] and
129          * pop an entry from hdr.srcRoutes
130          */
131         standard_metadata.egress_spec = hdr.srcRoutes[0].port;
132         hdr.srcRoutes.pop_front(1);
133     }

```



## 4.实验结果

1. 打开终端，进入source routing目录，输入make命令。

   ```
   make
   ```

   - 编译source_routing.p4
   - 启动Mininet实例，配置三角形网络拓扑，为主机分配IP地址
   - 为交换机配置转发逻辑

 实验拓扑如下：

2. 启动wireshark，监控s1-eth1,s1-eth2,s3-eth2端口

3. 在Mininet命令行中打开主机h1和h2

   ```
   xterm h1 h2
   ```

4. 在h2终端启动收包程序

   ```
   ./receive.py
   ```

5. 在h1终端启动发包程序

   ```
   ./send.py 10.0.2.2 "P4"
   ```

   此时会跳出提示，要求输入端口号列表，可以按照自己的想法为数据包设计转发路径。输入：

   ``` 
   2 3 2 2 1
   ```

   数据包的转发路径为 

   ```
   h1->s1->s2->s3->s1->s2-> h2
   ```

   

   - 查看h1发包信息，SourceRoute端口列表为【bos=0L  port=2L】【bos=0L port=3L】【bos=0L port=2L】【bos=0L port=2L】【bos=1L port=1L】

   ![image-20201104202936661](https://i.loli.net/2020/11/04/RiqynzIUhdFYrCk.png)

   - 打开s1-eth1的抓包，0x1234协议的数据包只有一个，这是因为我们只发了一个数据包。SourceRoute用红线标出，每个srcRoute是16bit，也就是两个字节。最后一个srcRoute为80 01，bos占1bit，h1的发包信息提示我们最后一个srcRoute的bos位置为1，所以二进制为 1【bos】 000000000000001【port】也就是16进制的0x8001.

   - 打开s2-eth2的抓包，0x1234协议的数据包有两个，这是因为按照我们的端口列表转发规则，会两次经过交换机。

   打开第一个数据包，如下可以看到srcRoute少了一个，这是因为经过一个路由器会提取两个字节，在交换机处理时对数组的pop操作。

   ![image-20201104205629597](https://i.loli.net/2020/11/04/9d3th8yVR2DrJMl.png)

   打开第二个数据包，如下可以看到srcRoute只有【bos=1L port=1L】。

   ![image-20201104210212909](https://i.loli.net/2020/11/04/VUjLRCPmMJtIqpc.png)

   - 查看h2收包信息，如下图，可以发现SourceRoute字段不存在，这是因为在交换机处理时对数组的pop操作，这也告诉我们，数据包的长度是可变的。

   ![image-20201104204609895](https://i.loli.net/2020/11/04/kvS2K6PjMqXxOd5.png)

## 总结

本次实验实现了源路由转发，这个实验向我们展示了可编程数据平面的灵活性、可定制性。在实验的过程中，对于包头的操作有了更深的理解，为了理解header stack的几个操作，查看了P4的白皮书.

## 参考

- P4-16 白皮书 https://p4.org/p4-spec/docs/P4-16-v1.0.0-spec.html

 