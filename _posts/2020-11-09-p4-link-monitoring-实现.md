---
layout:     post
title:      p4 link monitoring 实现
subtitle:   P4官方教程（十一）
date:       2020-11-09
author:     BY beta
header-img: img/2020-11-09/head2.jpg
catalog:    true
tags:
    - 工程
    - P4
---

##　1.实验link monitoring目的

实现链路监控功能，允许主机获得其数据包传输路径上各链路的使用率。
有点像之前的显示拥塞通告实验，ECN是返回交换机内部队列长度，如果排队较长，则返回一个标识拥塞的值，但是对于终端而言，并不知道是那个交换机队列，也不知道链路利用率。

## 2. 实验思路

- probe_fwd字段,实现源路由
- probe_data存储每一个交换机添加的端口数据信息,如两次探测间隔和数据包数量
- 两个寄存器byte_cnt_reg last_time_reg存储数据包数量和上一次访问的时间

## 3.代码修改

## 4.实验结果

- 进入Mininet终端之后,输入

```
xterm h1 h1
```

打开两个h1的终端,因为我们实验是测试一个回环路径上各链路的利用率.结果如下

![image-20201109205945394](https://i.loli.net/2020/11/09/yBUxzrneQ9RASoh.png)

- 在h1和h4之间运行iperf流,理论上链路利用率会显著的提高吧

  但是结果显示链路利用率是有所下降的,应该是拥塞了吗?

```
iperf h1 h2
```

![image-20201109210414510](https://i.loli.net/2020/11/09/vABpuTfx4gVhwCK.png)

## 总结

实现了链路利用率探测功能

- 寄存器的使用
- 数据包头部的修改.不顾自己还是没有很明显压栈的顺序
- 