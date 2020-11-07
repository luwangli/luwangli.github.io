---
layout:     post
title:      p4 Caculator 实现
subtitle:   P4官方教程（七）
date:       2020-11-05
author:     BY beta
header-img: img/2020-11-05/head.jpg
catalog:    true
tags:
    - 工程
    - P4
---

## 1.实验Caculator目的

通过用户编写的自定义协议头实现基本计算器。
看到这个题目觉得有些奇怪，实验的目标难道是让我们知道，交换机可以实现计算器的功能？

## 2. 实验环境

## 3. P4代码修改步骤

### 头文件header定义

在本实验中，需要自定义协议头存储相关的计算信息。headers分为两部分，ethernet_t是标准的头部，p4calc_t是我们需要自己定义的头部。头部的格式要求如下：

![image-20201105154939608](https://i.loli.net/2020/11/05/RYfQyXxdmTqjNPr.png)

每个数字代表一个字节，所以P占1个字节、Operand A占4个字节.代码修改：

```
 70 header p4calc_t {
 71     bit<8>  op;
 72 /* TODO
 73  * fill p4calc_t header with P, four, ver, op, operand_a, operand_b, and res
 74    entries based on above protocol header definition.
 75  */
 76     bit<8> P;
 77     bit<8> four;
 78     bit<8> ver;
 79     bit<8> op;
 80     bit<32> operand_a;
 81     bit<32> operand_b;
 82     bit<32> res;
 83 }

```

### parser

之前一直以为在P4中，只有packet.extract一种提取方式。在做此实验时，发现还有lookahead的方式

### MyIngress



#### 疑惑

table calculate中存在多个action，比如加、减等，而显然根据本实验，程序应当选择其中任一的action。而根据我之前对table的理解，内部的action会按照顺序执行？所以这里是有一个冲突，我的理解应当存在某些偏差。

答：阅读P4-16白皮书之后，了解到P4可以预编译一些静态条目，静态条目指向固定算法的实现。

## 4.实验结果

按照实验要求，输入相应的数值，会得到相关的结果

![image-20201107100456563](https://i.loli.net/2020/11/07/czSmdwXseUohBWM.png)

## 总结

 Caculator实验整体不难，当时它使用了P4里面更复杂的语法，比如const entry静态条目，当我们需要一些固定算法时，可以将这些静态条目插入满足预编译，同时可以优化存储。再者，发现了新的数据包提取方式，不过我还没能区分extract和lookahead的差异性。