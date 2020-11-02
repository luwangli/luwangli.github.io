---
layout:     post
title:      p4 Explicit Congestion Notification 实现
subtitle:   P4官方教程（四）
date:       2020-10-31
author:     BY beta
header-img: img/2020-10-31/head.jpg
catalog:    true
tags:
    - 工程
    - P4
---

## 1.实验Explicit Congestion Notification目的

本实验实现了显示的拥塞通知。交换机会根据内部的排队情况发出拥塞通知，进而调整发送端的发包速率使得数据包不会丢包。

具体而言，如果终端支持Explici		t Congestion Nodification（ECN），那么包头会有ipv4.ecn这个字段，且初始值为1或2。当数据包在交换机中的排队队列大于阈值时，会将ipv4.ecn修改3。接收端将这个值发给接收端，接收端则降低发包速率。

读完官方的教程，大致理解ECN实验需要完成数据平面的编写工作，而无需考虑控制平面。

## 2.实验环境

## 3.原始代码运行

1. 在终端输入make。脚本会编译p4文件，启动mininet，分配给主机相应的ip地址。实验拓扑如下：

![image-20201031164205562](https://i.loli.net/2020/10/31/EtfRNPGjBCTz8xk.png)

我们希望h1和h2之间是低速流量， h11和h22之间是高速流量。从图中可以看到，控制平面的规则使得h1和h2之间、h11和h22之间的流量都是经过s1和s2，而且本实验将s1和s2之间的带宽设置为512kbps，那么当流量过大时，就会在交换机内部出现数据包排队的情况。

2. 在Mininet终端中启动四个主机终端

```
xterm h1 h11 h2 h22
```

3. 新建终端启动wireshark，捕捉端口。在Mininet命令行中输入net，可以获取网络的拓扑，我们捕捉交换机S1的s1-eth1端口（连接h11）、交换机S1的s1-eth2端口（h22）、交换机S2的端口s2-eth1端口和交换机S2的端口s2-eth2

4. 在h2的终端，输入以下命令，捕捉数据包

   ```
   ./receive.py
   ```

   

5. 在h22的终端，输入以下命令，启动iperf UDP服务器

   ```
   iperf -s -u
   ```

   

6. 在h1的终端，输入以下命令，发送数据包，每秒发送一个数据包、持续30s

   ```
   ./send.py 10.0.2.2 "P4" 30
   ```

在h2的终端可以收到数据包，如下：

![image-20201031171548828](https://i.loli.net/2020/10/31/GOZp4sVuFoBXDif.png)

7. 在h11的终端，输入以下命令，发包15秒

   ```
   iperf -c 10.0.2.22 -t 15 -u
   ```

   ![image-20201031202654111](https://i.loli.net/2020/10/31/hr7ztbVS6eMTcaB.png)

   在实验中可以看到，一开始h2的终端是可以

## 4. 实验步骤

较为简单

## 5.实验结果

和第3节相同，但可以看到当h11开始发包之后，h2收到的数据包中tos字段从0x1变成了0x3

![image-20201031204458660](https://i.loli.net/2020/10/31/a2dIQf9TFiK6Dhg.png)

当我们打开s2-eth2的wireshark的抓包，发现第一阶段的收包都是每秒一个，中间的第二阶段收包变慢，2-3秒一个，结果如下图

![image-20201031204816488](https://i.loli.net/2020/10/31/lfmJVKwpEt9RkUx.png)

选择第一阶段的数据包，可以看到其ECN字段为1

![image-20201031212838985](https://i.loli.net/2020/10/31/LbBx2f3qUHQTZC5.png)

再选择第二阶段的数据包，查看ECN字段为3

![image-20201031212956533](C:/Users/Administrator/AppData/Roaming/Typora/typora-user-images/image-20201031212956533.png)

## 总结

本实验让我们看到了PISA架构下，数据包在交换机处理时会有metdata字段。实际过程中也是依靠standard_metadata.enq_qdepth 做出阈值判断，这也告诉我们，如果要自己开发相应的处理逻辑，是可以依靠standard_metadata字段。