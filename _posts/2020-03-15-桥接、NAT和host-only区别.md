---
layout:     post
title:      �����3��������������
subtitle:   �Žӡ�NAT��Host-only����
date:       2020-03-15
author:     BY beta
header-img: img/2020-03-09/head.png
catalog:    true
tags:
    - ���������
    - ������Ŀ
---

## ǰ��
��VMware���������ʱ��ͨ����Ҫ�������磬������ѡ�����Ž�ģʽ��NAT
ģʽ�ͽ�����ģʽ����ô�����������õ�������ʲô�أ��������һ�����
��һ����ϸ�Ľ��͡�
## 1.��������
�ڽ�����������֮ǰ�����ǵ�֪�����������ĸ��һ���豸�����Ҫ������
�豸��������ô����ʹ�������������ݰ���ת������ô��VMware�������װ
���֮�󣬻��Զ������������������VMnet1��VMnet8��

![network adapter](/img/2020-03-15/host-virtual-adapter.JPG)  

��������ģʽ������������Ӧ��ϵ��
1. �Žӣ�VMnet0
2. NAT��VMnet8
3. Host-only��VMnet1

���ǿ������������������������濴��VMnet1��VMnet8������������ôVMnet0
�������أ�ʵ���ϣ�VMnet0Ӧ�����ɶ��㽻���������������������������
����������ϡ���Ϊ���������������ƽ�ȵģ�������ͨ��������ȥ����
�������磬��������������ǲ�����VMnet0��[����gsonliu](https://www.jianshu.com/p/305f7384cfe9)
��ͼ���͵ĺܺã�����:  

![VMnet0](/img/2020-03-15/vmnet0.JPG)

## 2.Bridge�Ž�ģʽ
�Ž�ģʽĬ��ʹ��VMnet0������������ģʽ������Ϊ�ǽ����������������
���ӵ�VMnet0�ϣ�������������ֱ�����ӵ��������磬��������:
![bridge topology](/img/2020-03-15/bridge-topo.JPG)

������������ǲ鿴ip��ַ��WLAN��ַ�����Է���
![bridge ip](/img/2020-03-15/bridge-ip-addr.JPG)

���������У�������ͨ��wifi�������磬���Կ�����Ӧ��IP��ַ

![host wlan ip](/img/2020-03-15/host-wlan-addr.JPG)

���ԣ����ǿ����������Ž�ģʽ�µ��������ʵ���Ͼ���һ�������ļ����
���ӵ���·������·��������������һ��IP��ַ��
�Ž�ģʽ����Ϊ�򵥵��������ã�����ʹ��������١���������ӵ��������硣

## 3.NATģʽ
NATģʽ��ȫ��Ϊ��Network address translation",�������ַת������ģʽ��
��������ӵ�����������������VMnet8��VMnet8�䵱·�������ܣ�����������ϴ�
�����ݰ����е�ַת�����͵��������磬�������緵�ص����ݰ�Ҳ��ת����ַ��
ͨ��VMnet8���͸���������������£�

![nat topo](/img/2020-03-15/nat-topo.JPG)

��������в鿴ip��ַ�ɼ���

![host ip](/img/2020-03-15/virtual-nat-addr.JPG)

���������в鿴VMnet8��ַ��

![host Vmnet8](/img/2020-03-15/host-nat-addr.JPG)

���Կ���natģʽ�£��������ip��ַ��VMnet8��ip��ͬһ���Σ�
ͬʱ���������ping�ٶȣ����Է����ܹ�pingͨ��

![vm nat ping](/img/2020-03-15/vm-nat-pingbaidu.JPG)

## 4.host-onlyģʽ
host-onlyģʽ�£�����������һ����յ����磬���ӵ���ģʽ�µ����������
��������֮����Ի���ͨ�ţ�������������ܷ����������硣
Ҳ����˵��Host-onlyģʽ��NATģʽ�ǳ���ͬ����DHCP���ܣ�����
û��NAT���ܣ��������������������·�Ǹ���ģ��������£�

![hosonly topo](/img/2020-03-15/hostonly-topo.JPG)

��������в鿴ip��ַ��

![VM hostonly ip](/img/2020-03-15/virtual-hostonly-addr.JPG)

���������в鿴ip��ַ��

![host hostonly ip](/img/2020-03-15/host-only-addr.JPG)

���Կ���host-onlyģʽ�£��������ip��ַ��VMnet1��ip����ͬһ���Ρ�
��������£�ping�ٶ�(180.101.49.12)�����Է����޷�pingͨ����ӡ֤��
û��host-onlyģʽ�£�������������������ĸ��롣

![vm ping baidu](/img/2020-03-15/vm-hostonly-pingbaidu.JPG)

## �ο�
https://www.cnblogs.com/ggjucheng/archive/2012/08/19/2646007.html
https://www.jianshu.com/p/305f7384cfe9
https://blog.csdn.net/clevercode/article/details/45934233   
