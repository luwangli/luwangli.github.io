---
layout:     post
title:      p4 Multi-Hop Route Inspection 实现
subtitle:   P4官方教程（五）
date:       2020-11-2
author:     BY beta
header-img: img/2020-11-02/head.jpg
catalog:    true
tags:
    - 工程
    - P4
---

## 1.实验MRI目的

本实验拓展基本L3转发，实现简易版本的带内遥测-多跳路由检测Multi-Hop Route Inspection。

MRI允许用户记录数据包经过的路径和队列长度。本实验也写好了控制平面。只需修改数据平面，编写P4程序，将交换机ID和队列长度添加到数据包包头

## 2.实验环境

## 3.P4代码修改步骤

### MyParser

解析部分比以往的要跟复杂,因为涉及到自己定义的元数据metadata,头部变多..

1. 按照实验的要求 ,在解析ipv4_option时,如果value等于IPV4_OPTION_MRI值,那么进入parse_mri阶段,默认直接接受.修改代码如下:

```
   state parse_ipv4_option {
        /*
        * TODO: Add logic to:
        * - Extract the ipv4_option header.
        *   - If value is equal to IPV4_OPTION_MRI, transition to parse_mri.
        *   - Otherwise, accept.
        */
        packet.extract(hdr.ipv4_option);
        transition select(hdr.ipv4_option.option) {
            IPV4_OPTION_MRI : parse_mri;
                default :accept;
        }
       /* transition accept;*/
    }
```

2. 本实验对于添加交换机的ID和队列长度,有长度的限制,MAX_HOPS = 9. 所以在解析parse_mri时,需要对剩余容量做一个判断.修改代码如下:

```
state parse_mri {
        /*
        * TODO: Add logic to:
        * - Extract hdr.mri.
        * - Set meta.parser_metadata.remaining to hdr.mri.count
        * - Select on the value of meta.parser_metadata.remaining
        *   - If the value is equal to 0, accept.
        *   - Otherwise, transition to parse_swtrace.
        */
        packet.extract(hdr.mri);
        meta.parse_metadata.remaining = hdr.mri.count;
        transicion select(meta.parser_metadata.remaining) {
            0 : accept;
            default : parser_swtrace;
        }
       /* transition accept;*/
    }

```

3. 解析parse_swtrace.在这里有一些疑惑,按照提示提取hdr.swtraces.next, 由于swtraces是数组,next可以直接指向第一个元素?

   ```
   144		state parse_swtrace {
   145         /*
   146         * TODO: Add logic to:
   147         * - Extract hdr.swtraces.next.
   148         * - Decrement meta.parser_metadata.remaining by 1
   149         * - Select on the value of meta.parser_metadata.remaining
   150         *   - If the value is equal to 0, accept.
   151         *   - Otherwise, transition to parse_swtrace.
   152         */
   153         packet.extract(hdr.swtraces.next);
   154         meta.parser_metadata.remaining = meta.parser_metadata.remaining - 1;
   155         transition select(meta.parser_metadata.remaining) {
   156             0 : accept;
   157             default : parse_swtrace;
   158         }
   159 /*        transition accept;*/
   160     }
   
   ```

   

   

### MyEgress

在MyEgress过程中，需要添加交换机的ID和队列长度到数据包的头部。先使用push_front(1)在数组的前端添加一个空元素，根据实验的提示,在前端添加的元素默认是非法，所以需要先将其设为合法，之后才能赋值。

```
According to the P4_16 spec, pushed elements are invalid, so we need to call setValid(). Older bmv2 versions would mark the new header(s) valid automatically (P4_14 behavior), but starting with version 1.11, bmv2 conforms with the P4_16 spec.
```

代码段如下：

```
218     action add_swtrace(switchID_t swid) {
219         /*
220         * TODO: add logic to:
221         - Increment hdr.mri.count by 1
222         - Add a new swtrace header by calling push_front(1) on hdr.swtraces.
223         - Set hdr.swtraces[0].swid to the id parameter
224         - Set hdr.swtraces[0].qdepth to (qdepth_t)standard_metadata.deq_qdepth
225         - Increment hdr.ipv4.ihl by 2
226         - Increment hdr.ipv4.totalLen by 8
227         - Increment hdr.ipv4_option.optionLength by 8
228         */
229         hdr.mri.count = hdr.mri.count + 1;
230         hdr.swtraces.push_front(1);
231         hdr.swtraces[0].setValid();
232         hdr.swtraces[0].swid = id;
233         hdr.swtraces[0].qdepth = standard_metadata.deq_qdepth;
234         hdr.ipv4.ihl = hdr.ipv4.ihl + 2;
235         hdr.ipv4.totalLen = hdr.ipv4.totalLen + 8 ;
236         hdr.ipv4_option.optionLength = hdr.ipv4_option.optionLength + 8;
237     }

```





## 4.实验结果

本实验需要通过在终端捕获数据包并查看其交换机ID和队列长度。在实验中存在两个发送端，分别是低速发送端h1和高速发送端h2，如果我们在实验中发现，当只有h1发包时，队列较短；而当h1和h2同时发包时，队列应该较长。

1. 打开终端，进入mri目录，输入make命令

   ```
   make
   ```

   编译mri.p4程序，启动Mininet，生成实验拓扑，分配主机和交换机相应的IP。控制平面将对应的流表下发给交换机。

   实验网络拓扑如下：

   ![image-20201102195241024](https://i.loli.net/2020/11/02/fw2pQ4ka1AVeNxr.png)

2. 新建一个终端，以CLI登录S1，查看其安装的流表，如下

   ![image-20201102200914687](https://i.loli.net/2020/11/02/XMxUNcVhJLyu26i.png)

   可以看到s1交换机的ID为1.

3. 在mininet终端中输入以下命令。打开终端h1、h2、h11和h22.

   ```
   xterm h1 h2 h11 h22
   ```

4. 在h2的终端，开启收包服务

   ```
   ./receive.py
   ```

5. 在h22的终端，启动iperf的UDP服务器

   ```
   iperf -s -u
   ```

6. 新建一个终端，启动wireshark监控s2-eth2 和s2-eth1端口。

7. 在h1的终端，使用send.py程序发包，每秒发送一个数据包给h2，持续10秒

   ```
   ./send.py 10.0.2.2 "P4 mri" 10
   ```

   在h2的终端显示经过交换机s1 s2，队列长度都为0

   ![image-20201102202228795](https://i.loli.net/2020/11/02/yFuKMbDfnX15cZO.png)

   在wireshark中查看s2-eth2，打开其中一个数据包。可以看到32bit的交换机ID和32bit的队列长度

   ![image-20201102203217824](https://i.loli.net/2020/11/02/WNnKzjRlIHbUaAT.png)

   此外，我们可以对比s1-eth2和s2-eth2端口抓取数据包的长度，发现在s1-eth2数据包的长度为，s2-eth2数据包长度为，如下两图。这是因为数据包每经过一个交换机，都会插入交换机ID(32bit)和队列长度(32bit)

   ![image-20201105085519491](https://i.loli.net/2020/11/05/mSJ1rv5Ds7VAgkK.png)

   ![image-20201105085554338](https://i.loli.net/2020/11/05/qshibg9moRu3cCQ.png)

8. 在h1终端使用send.py程序发包

   ```
   ./send.py 10.0.2.2 "P4 mri" 30
   ```

   同时在h11终端使用iperf发包

   ```
   iperf -c 10.0.2.22 -t 15 -u
   ```

   此时可以在h2的终端看到s1的队列长度不再是0，而是56.

   ![image-20201102204328391](https://i.loli.net/2020/11/02/m6y97Lgilr8hFjY.png)

同时我们可以看到h22的wireshark抓包并没有显示中间路径的交换机ID和队列，这是因为iperf发包的格式并不完全符合我们在交换机定义的解析流程，所以没有修改头部。

## 总结

本实验帮助我们理解如何使用metadata将所需交换机内部信息添加到数据包头部，实现了一个简单的带内遥测实例。

